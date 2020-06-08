# conda创建和删除虚拟环境

创建虚拟环境

```
conda create -n $ENV_Name python=3.6
```

删除虚拟环境

```
# 全部删除
conda remove -n $ENV_Name --all
# 删除指定包
conda remove -n $ENV_Name package_name
```

