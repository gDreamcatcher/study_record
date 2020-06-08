#! https://zhuanlan.zhihu.com/p/374234707
SCANN代码阅读笔记

[代码地址](https://github.com/google-research/google-research/tree/master/scann)

[论文地址](https://arxiv.org/pdf/1908.10396.pdf)

[python代码示例](https://github.com/google-research/google-research/blob/master/scann/docs/example.ipynb)

python安装 

```shell
wget https://storage.googleapis.com/scann/releases/1.0.0/scann-1.0.0-cp36-cp36m-linux_x86_64.whl
pip install scann-1.0.0-cp36-cp36m-linux_x86_64.whl
```

python源码安装

```shell
yum install -y zip bzip2 flex  gcc make gcc++ git clang
#安装gcc
#下载gcc 9.3.0 代码 
git clone -b releases/gcc-9.3.0 https://github.com/gcc-mirror/gcc.git
cd gcc
./contrib/download_prerequisites
./configure --disable-multilib --prefix=/home/gcc
make -j8
make install
ln -s /home/gcc/bin/gcc /bin/gcc
ln -s /home/gcc/bin/g++ /bin/g++

#安装bazel
wget https://github.com/bazelbuild/bazel/releases/download/3.4.1/bazel-3.4.1-installer-linux-x86_64.sh
./bazel-3.4.1-installer-linux-x86_64.sh --user
export PATH=$PATH:/root/bin
bazel --version

# 编译scann
yum install async
python configure.py
CC=clang bazel build -c opt --copt=-mavx2 --copt=-mfma --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" --cxxopt="-std=c++17" --copt=-fsized-deallocation --copt=-w :build_pip_pkg
./bazel-bin/build_pip_pkg
```

c++代码解析

这里只看python示例代码中c++后台是怎么运行的

```python
searcher = scann.ScannBuilder(normalized_dataset, 10, "dot_product").tree(
    num_leaves=2000, num_leaves_to_search=100, training_sample_size=250000).score_ah(
    2, anisotropic_quantization_threshold=0.2).reorder(100).create_pybind()

def create_pybind(self):
  config = self.create_config()
  return scann_ops_pybind.create_searcher(self.db, config,
                                          self.training_threads)

def create_searcher(db, scann_config, training_threads=0):
  return ScannSearcher(
      scann_pybind.ScannNumpy(db, scann_config, training_threads)) 
# 可以看到这里python调用了ScannNumpy来获取search，下面看一下后面是怎么实现的
```



```c++
// scann/scann_ops/cc/scann_npy.cc
ScannNumpy::ScannNumpy(const np_row_major_arr<float>& np_dataset,
                       const std::string& config, int training_threads) {
  if (np_dataset.ndim() != 2)
    throw std::invalid_argument("Dataset input must be two-dimensional");
  // 这里把输入的dataset转换成了ConstSpan， using ConstSpan = absl::Span<const T>;
  ConstSpan<float> dataset(np_dataset.data(), np_dataset.size());
  // 调用scann.cc里的初始化函数
  RuntimeErrorIfNotOk("Error initializing searcher: ",
                      scann_.Initialize(dataset, np_dataset.shape()[1], config,
                                        training_threads));
}
```



```c++
// scann/scann_ops/cc/scann.cc
Status ScannInterface::Initialize(ConstSpan<float> dataset,
                                  DimensionIndex dimensionality,
                                  const std::string& config,
                                  int training_threads) {
  // 从字符串中解析出config，这里面用的config是protobuf格式的
  ::google::protobuf::TextFormat::ParseFromString(config, &config_);
  // 设置线程数，如果为0那么可用的线程数等于总线程等于cpu数量减一，设置线程池
  if (training_threads < 0)
    return InvalidArgumentError("training_threads must be non-negative");
  if (training_threads == 0) training_threads = absl::base_internal::NumCPUs();
  SingleMachineFactoryOptions opts;

  opts.parallelization_pool =
      StartThreadPool("scann_threadpool", training_threads - 1);
  return Initialize(dataset, dimensionality, opts);
}
// 这个函数主要是做了一些准备工作，解析config，检查数据格式，设置线程池，再继续看下一个初始化函数
```



```c++
// scann/scann_ops/cc/scann.cc
Status ScannInterface::Initialize(ConstSpan<float> ds_span,
                                  DimensionIndex dimensionality,
                                  SingleMachineFactoryOptions opts) {
  if (ds_span.empty()) return InvalidArgumentError("Dataset must be non-empty");

  dimensionality_ = dimensionality;
  n_points_ = ds_span.size() / dimensionality_;

  // 这里将数据准换成DenseDataSet 定义在scann/data_format/dataset.h
  vector<float> dataset_vec(ds_span.data(), ds_span.data() + ds_span.size());
  auto dataset = absl::make_unique<DenseDataset<float>>(dataset_vec, n_points_);
	
  /* 这里判断配置文件中config是否要分片，分片的类型是否是SPHERICAL(类型总共有两种GENERIC, SPHERICAL)，
 		 如果是SPHERICAL就把数据集的normalization_设置成tensorflow::scann_ops::UNITL2NORM ()
    enum Normalization : uint8_t {
      NONE = 0,  //不做归一化
      UNITL2NORM = 1, // norm归一化
      STDGAUSSNORM = 2, // 不支持
      UNITL1NORM = 3 // 不支持
    };
  */
  
  if (config_.has_partitioning() &&
      config_.partitioning().partitioning_type() ==
          PartitioningConfig::SPHERICAL)
    dataset->set_normalization_tag(tensorflow::scann_ops::UNITL2NORM);
  // 初始化scann对象
  TF_ASSIGN_OR_RETURN(
      scann_, SingleMachineFactoryNoSparse<float>(config_, std::move(dataset),
                                                  std::move(opts)));

  // 这里config用的是DotProductDistance 但是partition和hash的时候用的都是默认的SquaredL2Distance
  const std::string& distance = config_.distance_measure().distance_measure();
  const absl::node_hash_set<std::string> negated_distances{
      "DotProductDistance", "BinaryDotProductDistance", "AbsDotProductDistance",
      "LimitedInnerProductDistance"};
  result_multiplier_ =
      negated_distances.find(distance) == negated_distances.end() ? 1 : -1;
  return OkStatus();
}
```



```c++
// scann\base\single_machine_factory_no_sparse.cc
template <typename T>
StatusOr<unique_ptr<SingleMachineSearcherBase<T>>> SingleMachineFactoryNoSparse(
    const ScannConfig& config, shared_ptr<TypedDataset<T>> dataset,
    SingleMachineFactoryOptions opts) {
  opts.type_tag = TagForType<T>();
  TF_ASSIGN_OR_RETURN(auto searcher, SingleMachineFactoryUntypedNoSparse(
                                         config, dataset, std::move(opts)));
  return {
      unique_cast_unsafe<SingleMachineSearcherBase<T>>(std::move(searcher))};
}

StatusOrSearcherUntyped SingleMachineFactoryUntypedNoSparse(
    const ScannConfig& config, shared_ptr<Dataset> dataset,
    SingleMachineFactoryOptions opts) {
  return internal::SingleMachineFactoryUntypedImpl<NoSparseLeafSearcher>(
      config, dataset, opts);
}
```



```c++
// scann\base\internal\single_machine_factory_impl.h
template <typename LeafSearcherT>
StatusOrSearcherUntyped SingleMachineFactoryUntypedImpl(
    const ScannConfig& config, shared_ptr<Dataset> dataset,
    SingleMachineFactoryOptions opts) {
  GenericSearchParameters params;
  // 从config读取搜索的参数
  SCANN_RETURN_IF_ERROR(params.PopulateValuesFromScannConfig(config));
  // 校验参数 还不清楚是做什么用的
  if (params.reordering_dist->NormalizationRequired() != NONE && dataset &&
      dataset->normalization() !=
          params.reordering_dist->NormalizationRequired()) {
    return InvalidArgumentError(
        "Dataset not correctly normalized for the exact distance measure.");
  }

  if (params.pre_reordering_dist->NormalizationRequired() != NONE && dataset &&
      dataset->normalization() !=
          params.pre_reordering_dist->NormalizationRequired()) {
    return InvalidArgumentError(
        "Dataset not correctly normalized for the pre-reordering distance "
        "measure.");
  }

  if (opts.type_tag == kInvalidTypeTag) {
    CHECK(dataset) << "Code fails to wire-through the type tag";
    opts.type_tag = dataset->TypeTag();
  }
	
  // 重点 在这里实例化scann对象，往下看SingleMachineFactoryImplClass的实现
  TF_ASSIGN_OR_RETURN(auto searcher,
                      SCANN_CALL_FUNCTION_BY_TAG(
                          opts.type_tag,
                          SingleMachineFactoryImplClass<
                              LeafSearcherT>::template SingleMachineFactoryImpl,
                          config, dataset, params, &opts));
  CHECK(searcher) << "Returning nullptr instead of Status is a bug";

  if (config.crowding().enabled() && opts.crowding_attributes) {
    SCANN_RETURN_IF_ERROR(
        searcher->EnableCrowding(std::move(opts.crowding_attributes)));
  }

  searcher->set_creation_timestamp(opts.creation_timestamp);
  return {std::move(searcher)};
}

```



```c++
// 往下看SingleMachineFactoryImplClass的实现
// scann\base\internal\single_machine_factory_impl.h line54
template <typename T>
  static StatusOrSearcherUntyped SingleMachineFactoryImpl(
      ScannConfig config, const shared_ptr<Dataset>& dataset,
      const GenericSearchParameters& params,
      SingleMachineFactoryOptions* opts) {
    config.mutable_input_output()->set_in_memory_data_type(TagForType<T>());
    SCANN_RETURN_IF_ERROR(CanonicalizeScannConfigForRetrieval(&config));
    auto typed_dataset = std::dynamic_pointer_cast<TypedDataset<T>>(dataset);
    if (dataset && !typed_dataset) {
      return InvalidArgumentError("Dataset is the wrong type");
    }
		// 到这里跳转// scann\base\single_machine_factory_no_sparse.cc line618 SingleMachineFactoryLeafSearcher 
    // 到这里数据分桶和量化模型计算完毕
    TF_ASSIGN_OR_RETURN(auto searcher,
                        LeafSearcherT::SingleMachineFactoryLeafSearcher(
                            config, typed_dataset, params, opts));
    auto* typed_searcher =
        down_cast<SingleMachineSearcherBase<T>*>(searcher.get());

  	// 设置query时候的reorder参数，
    TF_ASSIGN_OR_RETURN(
        auto reordering_helper,
        ReorderingHelperFactory<T>::Build(config, params.reordering_dist,
                                          typed_dataset, opts));
    typed_searcher->EnableReordering(std::move(reordering_helper),
                                     params.post_reordering_num_neighbors,
                                     params.post_reordering_epsilon);
    if (config.has_compressed_reordering()) {
      DCHECK(!typed_searcher->needs_dataset());
      typed_searcher->ReleaseDatasetAndDocids();
      typed_searcher->set_compressed_dataset(opts->compressed_dataset);
    }

    return {std::move(searcher)};
  }
};
```

```c++
// scann\base\single_machine_factory_no_sparse.cc line618 SingleMachineFactoryLeafSearcher 
template <typename T>
  static StatusOrSearcherUntyped SingleMachineFactoryLeafSearcher(
      const ScannConfig& config, const shared_ptr<TypedDataset<T>>& dataset,
      const GenericSearchParameters& params,
      SingleMachineFactoryOptions* opts) {
    if (internal::NumQueryDatabaseSearchTypesConfigured(config) != 1) {
      return InvalidArgumentError(
          "Exactly one single-machine search type must be configured in "
          "ScannConfig if using SingleMachineFactory.");
    }

    if (config.has_partitioning()) {
      return TreeXHybridFactory<T>(config, dataset, params, opts); // 进入这里
    } else if (config.has_brute_force()) {
      if (std::is_same<T, float>::value &&
          config.brute_force().fixed_point().enabled() &&
          opts->pre_quantized_fixed_point) {
        return BruteForceFactory(config.brute_force(), params,
                                 opts->pre_quantized_fixed_point.get());
      } else {
        return BruteForceFactory(config.brute_force(), dataset, params);
      }
    } else if (config.has_hash()) {
      return HashFactory<T>(dataset, config, opts, params);
    } else {
      return UnknownError("Unhandled case");
    }
  }

// scann\base\single_machine_factory_no_sparse.cc line524
template <typename T>
StatusOrSearcherUntyped TreeXHybridFactory(
    const ScannConfig& config, const shared_ptr<TypedDataset<T>>& dataset,
    const GenericSearchParameters& params, SingleMachineFactoryOptions* opts) {
  //如果使用了hash并且config.distance_measure == "dot_product" 则返回TreeAhHybridResidualFactory
  // 参考scann_builder.py line196
  if (config.hash().asymmetric_hash().use_residual_quantization()) {
    return TreeAhHybridResidualFactory<T>(config, dataset, params, opts); // 进入这里
  } else if (std::is_same<T, float>::value &&
             config.brute_force().fixed_point().enabled() &&
             opts->pre_quantized_fixed_point) {
    return PretrainedSQTreeXHybridFactory(config, nullptr, params, opts);
  } else {
    return NonResidualTreeXHybridFactory<T>(config, dataset, params, opts);
  }
}

// scann\base\single_machine_factory_no_sparse.cc line165
template <>
StatusOrSearcherUntyped TreeAhHybridResidualFactory<float>(
    const ScannConfig& config, const shared_ptr<TypedDataset<float>>& dataset,
    const GenericSearchParameters& params, SingleMachineFactoryOptions* opts) {
  // 创建partition
  unique_ptr<Partitioner<float>> partitioner;
  if (config.partitioning().has_partitioner_prefix()) {
    return InvalidArgumentError("Loading a partitioner is not supported.");
  } else {
    TF_ASSIGN_OR_RETURN(partitioner,
                        CreateTreeXPartitioner<float>(dataset, config, opts));
  }
  unique_ptr<KMeansTreeLikePartitioner<float>> kmeans_tree_partitioner(
      dynamic_cast<KMeansTreeLikePartitioner<float>*>(partitioner.release()));
  if (!kmeans_tree_partitioner) {
    return InvalidArgumentError(
        "Tree AH with residual quantization only works with KMeans tree as a "
        "partitioner.");
  }
	
  // 转换dataset类型
  auto dense = dynamic_pointer_cast<const DenseDataset<float>>(dataset);
  if (dataset && !dense) {
    return InvalidArgumentError(
        "Tree-AH with residual quantization only works with dense data.");
  }
  if (params.pre_reordering_dist->specially_optimized_distance_tag() !=
      DistanceMeasure::DOT_PRODUCT) {
    return InvalidArgumentError(
        "Tree-AH with residual quantization only works with dot product "
        "distance for now.");
  }
  auto result = make_unique<TreeAHHybridResidual>(
      dense, params.pre_reordering_num_neighbors,
      params.pre_reordering_epsilon);
  
	// 将数据分桶
  vector<std::vector<DatapointIndex>> datapoints_by_token = {};
  if (dataset && dataset->empty()) {
    datapoints_by_token.resize(kmeans_tree_partitioner->n_tokens());
  } else {
    if (opts->datapoints_by_token) {
      datapoints_by_token = std::move(*opts->datapoints_by_token);
    } else if (dense) {
      TF_ASSIGN_OR_RETURN(datapoints_by_token,
                          kmeans_tree_partitioner->TokenizeDatabase(
                              *dense, opts->parallelization_pool.get()));
    } else {
      return InvalidArgumentError(
          "For Tree-AH hybrid with residual quantization, either "
          "database_wildcard or tokenized_database_wildcard must be provided.");
    }
    if (datapoints_by_token.size() > kmeans_tree_partitioner->n_tokens()) {
      return InvalidArgumentError(
          "The pre-tokenization (ie, datapoints_by_token) specifies %d "
          "partitions, versus the kmeans partitioner, which only has %d "
          "partitions",
          datapoints_by_token.size(), kmeans_tree_partitioner->n_tokens());
    }
    if (datapoints_by_token.size() < kmeans_tree_partitioner->n_tokens()) {
      datapoints_by_token.resize(kmeans_tree_partitioner->n_tokens());
    }
  }
	
  // 计算每个桶的量化模型
  shared_ptr<const asymmetric_hashing2::Model<float>> ah_model;
  if (opts->ah_codebook) {
    TF_ASSIGN_OR_RETURN(ah_model, asymmetric_hashing2::Model<float>::FromProto(
                                      *opts->ah_codebook));
  } else if (config.hash().asymmetric_hash().has_centers_filename()) {
    return InvalidArgumentError("Centers files are not supported.");
  } else if (dense) {
    if (opts->hashed_dataset) {
      return InvalidArgumentError(
          "If a pre-computed hashed database is specified for tree-AH hybrid "
          "then pre-computed AH centers must be specified too.");
    }
    TF_ASSIGN_OR_RETURN(
        auto quantization_distance,
        GetDistanceMeasure(
            config.hash().asymmetric_hash().quantization_distance()));
    TF_ASSIGN_OR_RETURN(
        auto residuals,
        TreeAHHybridResidual::ComputeResiduals(
            *dense, kmeans_tree_partitioner.get(), datapoints_by_token,
            config.hash()
                .asymmetric_hash()
                .use_normalized_residual_quantization()));
    asymmetric_hashing2::TrainingOptions<float> training_opts(
        config.hash().asymmetric_hash(), quantization_distance, residuals);
    TF_ASSIGN_OR_RETURN(
        ah_model, asymmetric_hashing2::TrainSingleMachine(
                      residuals, training_opts, opts->parallelization_pool));
  } else {
    return InvalidArgumentError(
        "For Tree-AH hybrid with residual quantization, either "
        "centers_filename or database_wildcard must be provided.");
  }

  if (!dense) {
    DCHECK(opts->hashed_dataset);
    SCANN_RETURN_IF_ERROR(result->set_docids(opts->hashed_dataset->docids()));
  }

  result->set_database_tokenizer(
      absl::WrapUnique(down_cast<KMeansTreeLikePartitioner<float>*>(
          kmeans_tree_partitioner->Clone().release())));
  SCANN_RETURN_IF_ERROR(result->BuildLeafSearchers(
      config.hash().asymmetric_hash(), std::move(kmeans_tree_partitioner),
      std::move(ah_model), std::move(datapoints_by_token),
      opts->hashed_dataset.get(), opts->parallelization_pool.get()));
  opts->datapoints_by_token = nullptr;
  return {std::move(result)};
}
```





## search方法

```c++
// scann\scann_ops\cc\scann_npy.cc line74
std::pair<pybind11::array_t<DatapointIndex>, pybind11::array_t<float>>
ScannNumpy::Search(const np_row_major_arr<float>& query, int final_nn,
                   int pre_reorder_nn, int leaves) {
  if (query.ndim() != 1)
    throw std::invalid_argument("Query must be one-dimensional");

  DatapointPtr<float> ptr(nullptr, query.data(), query.size(), query.size());
  NNResultsVector res;
  auto status = scann_.Search(ptr, &res, final_nn, pre_reorder_nn, leaves);// 进入这里
  RuntimeErrorIfNotOk("Error during search: ", status);

  // 转换成python需要的输出格式
  pybind11::array_t<DatapointIndex> indices(res.size());
  pybind11::array_t<float> distances(res.size());
  auto idx_ptr = reinterpret_cast<DatapointIndex*>(indices.request().ptr);
  auto dis_ptr = reinterpret_cast<float*>(distances.request().ptr);
  scann_.ReshapeNNResult(res, idx_ptr, dis_ptr);
  return {indices, distances};
}

// scann\scann_ops\cc\scann.cc line125
Status ScannInterface::Search(const DatapointPtr<float> query,
                              NNResultsVector* res, int final_nn,
                              int pre_reorder_nn, int leaves) const {
  // 参数的意思 final_nn 最终返回的数据量; leaves需要搜索的叶子数; pre_reorder_nn事先召回所需的数据量
  if (query.dimensionality() != dimensionality_)
    return InvalidArgumentError("Query doesn't match dataset dimsensionality");
  
  // 从这里可以看出是有两种reordering格式，但是后面好像不支持compressed方式
  bool has_reordering =
      config_.has_exact_reordering() || config_.has_compressed_reordering();
  
  // 下面就是设置reordered的参数，如果有reorder那么pre_reorder_nn一定被提前设定过
  int post_reorder_nn = -1;
  if (has_reordering)
    post_reorder_nn = final_nn;
  else
    pre_reorder_nn = final_nn;

  SearchParameters params;
  params.set_pre_reordering_num_neighbors(pre_reorder_nn);
  params.set_post_reordering_num_neighbors(post_reorder_nn);
  // 这里因为要搜索的叶子数量在初始化的时候就设置好了，搜索的时候允许用户修改
  if (leaves > 0) {
    auto tree_params = std::make_shared<TreeXOptionalParameters>();
    tree_params->set_num_partitions_to_search_override(leaves);
    params.set_searcher_specific_optional_parameters(tree_params);
  }
  scann_->SetUnspecifiedParametersToDefaults(&params);  
  return scann_->FindNeighbors(query, params, res); // 进入这里
}

// scann\base\single_machine_base.cc line275
template <typename T>
Status SingleMachineSearcherBase<T>::FindNeighbors(
    const DatapointPtr<T>& query, const SearchParameters& params,
    NNResultsVector* result) const {
  DCHECK(result);
  // 两种reorder方式只能有一个，目前使用的是exact方式
  DCHECK_LE((compressed_reordering_enabled() + exact_reordering_enabled()), 1);
  
  
  SCANN_RETURN_IF_ERROR(
      FindNeighborsNoSortNoExactReorder(query, params, result));

  if (reordering_helper_) {
    SCANN_RETURN_IF_ERROR(ReorderResults(query, params, result));
  }

  return SortAndDropResults(result, params);
}


// scann\base\single_machine_base.cc line291
template <typename T>
Status SingleMachineSearcherBase<T>::FindNeighborsNoSortNoExactReorder(
    const DatapointPtr<T>& query, const SearchParameters& params,
    NNResultsVector* result) const {
  DCHECK(result);
  bool reordering_enabled =
      compressed_reordering_enabled() || exact_reordering_enabled();
  SCANN_RETURN_IF_ERROR(params.Validate(reordering_enabled));
  if (!this->supports_crowding() && params.pre_reordering_crowding_enabled()) {
    return InvalidArgumentError(
        std::string(
            "Crowding is enabled but not supported for searchers of type ") +
        typeid(*this).name() + ".");
  }

  // 又一次校验参数
  if (dataset() && !dataset()->empty() &&
      query.dimensionality() != dataset()->dimensionality()) {
    return FailedPreconditionError(
        StrFormat("Query dimensionality (%u) does not match database "
                  "dimensionality (%u)",
                  static_cast<uint64_t>(query.dimensionality()),
                  static_cast<uint64_t>(dataset()->dimensionality())));
  }

  // 这个实现分为好多个brute_force.cc scalar_quantized_brute_force.cc searcher.cc tree_ah_hybrid_residual.cc 
  // tree_x_hybrid_smmd.cc, 这里使用的是tree_ah_hybrid_residual.cc
  return FindNeighborsImpl(query, params, result); // 进入这里，这个实现分为好多个
}

// scann\tree_x_hybrid\tree_ah_hybrid_residual.cc
Status TreeAHHybridResidual::FindNeighborsImpl(const DatapointPtr<float>& query,
                                               const SearchParameters& params,
                                               NNResultsVector* result) const {
  // 这里好像是走了缓存，记录了需要搜索的类中心
  auto query_preprocessing_results =
      params.unlocked_query_preprocessing_results<
          UnlockedTreeAHHybridResidualPreprocessingResults>();
  if (query_preprocessing_results) {
    return FindNeighborsInternal1(
        query, params, query_preprocessing_results->centers_to_search(),
        result);
  }

  // 这里就是用了搜索时传递的leaves参数修改了初始化时候的值
  int num_centers = 0;
  auto tree_x_params =
      params.searcher_specific_optional_parameters<TreeXOptionalParameters>();
  if (tree_x_params) {
    int center_override = tree_x_params->num_partitions_to_search_override();
    if (center_override > 0) num_centers = center_override;
  }
  vector<KMeansTreeSearchResult> centers_to_search;
  SCANN_RETURN_IF_ERROR(query_tokenizer_->TokensForDatapointWithSpilling(
      query, num_centers, &centers_to_search));
  return FindNeighborsInternal1(query, params, centers_to_search, result);
}
```



```c++
// scann\partitioning\kmeans_tree_partitioner.cc line200
template <typename T>
Status KMeansTreePartitioner<T>::TokensForDatapointWithSpilling(
    const DatapointPtr<T>& dptr, int32_t max_centers_override,
    vector<KMeansTreeSearchResult>* result) const {
  DCHECK(result);
  // 参数解析 dptr是query，max_centers_override是num_centers， result是result
  
  // 这里分两种情况query和训练，这里选择query
  if (this->tokenization_mode() == UntypedPartitioner::QUERY) {
    // 如果不设置max_centers_override那么就搜所有的桶 或者初始化中设置的桶
    const auto max_centers = max_centers_override > 0
                                 ? max_centers_override
                                 : query_spilling_max_centers_;

    if (query_tokenization_type_ == ASYMMETRIC_HASHING) {
      // 如果设置了reorder那么搜索的桶是原本的10倍
      int pre_reordering_num_neighbors =
          TokenizationSearcher()->reordering_enabled()
              ? max_centers * kAhMultiplierSpilling
              : max_centers;
      return TokensForDatapointWithSpillingUseSearcher(
          dptr, result, max_centers, pre_reordering_num_neighbors); // 进入这里
    }

    return kmeans_tree_->Tokenize(
        dptr, *query_tokenization_dist_,
        KMeansTree::TokenizationOptions::UserSpecifiedSpilling(
            query_spilling_type_, query_spilling_threshold_, max_centers,
            static_cast<KMeansTree::TokenizationType>(query_tokenization_type_),
            populate_residual_stdev_),
        result);
  } else if (this->tokenization_mode() == UntypedPartitioner::DATABASE) {
    // 省略
  } else {
    return InternalError(absl::StrCat("Unknown tokenization mode:  ",
                                      this->tokenization_mode()));
  }
}

// scann\partitioning\kmeans_tree_partitioner.cc line338
template <typename T>
Status KMeansTreePartitioner<T>::TokensForDatapointWithSpillingUseSearcher(
    const DatapointPtr<T>& dptr, vector<KMeansTreeSearchResult>* result,
    int32_t num_neighbors, int32_t pre_reordering_num_neighbors) const {
  if (!TokenizationSearcher()) {
    return FailedPreconditionError(
        "CreateAsymmetricHashingSearcherForTokenization must "
        "be called first.");
  }

  Datapoint<float> dp;
  DatapointPtr<float> query = ToFloat(dptr, &dp);
  float threshold = numeric_limits<float>::infinity();
  if (query_spilling_type_ == QuerySpillingConfig::ABSOLUTE_DISTANCE) {
    threshold = query_spilling_threshold_;
  }
  SearchParameters params(pre_reordering_num_neighbors,
                          numeric_limits<float>::infinity(), num_neighbors,
                          threshold);
  NNResultsVector search_result;
  Status status =
      TokenizationSearcher()->FindNeighbors(query, params, &search_result);

  if (!status.ok()) return status;

  DCHECK(is_one_level_tree_);
  result->clear();
  result->reserve(search_result.size());
  const auto* root = kmeans_tree_->root();
  for (const auto& elem : search_result) {
    DCHECK_LE(elem.first, kint32max);
    result->emplace_back(KMeansTreeSearchResult{
        &root->Children()[elem.first], elem.second,
        populate_residual_stdev_ && elem.first < root->residual_stdevs().size()
            ? root->residual_stdevs()[elem.first]
            : 1.0});
    DCHECK_EQ(elem.first, result->back().node->LeafId());
  }
  return OkStatus();
}
```

