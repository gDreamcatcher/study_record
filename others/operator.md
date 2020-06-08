


```shell
operator-sdk init --domain sine.com --repo github.com/gDreamcatcher/sine
go get github.com/onsi/gomega@v1.10.2
go get github.com/onsi/ginkgo@v1.14.1
go get github.com/go-logr/logr@v0.3.0
rm -rf api/v1alpha1/sine_types.go config/samples/test_v1alpha1_sine.yaml controllers/sine_controller.go

docker rmi gdream/memcached-operator:v0.0.1
make manifests
make docker-build docker-push IMG=gdream/memcached-operator:v0.0.1
make deploy IMG=gdream/memcached-operator:v0.0.1