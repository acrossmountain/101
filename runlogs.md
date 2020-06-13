
# setup local env
https://minikube.sigs.k8s.io/docs/
## install virtualbox
https://www.virtualbox.org/
## download minikube
https://github.com/kubernetes/minikube/releases
## start minikube

```
minikube start \
    --cpus 4 \
    --memory 4096 \
    --network-plugin=cni \
    --enable-default-cni \
    --container-runtime=containerd \
    --bootstrapper=kubeadm \
    --kubernetes-version v1.15.4 \
    --image-mirror-country=cn \
    --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```

# docker
## overlay fs
```
mkdir upper lower merged work
echo "from lower" > lower/in_lower.txt 
echo "from upper" > upper/in_upper.txt
echo "from lower" > lower/in_both.txt 
echo "from upper" > upper/in_both.txt 
sudo mount -t overlay overlay -o lowerdir=`pwd`/lower,upperdir=`pwd`/upper,workdir=`pwd`/work `pwd`/merged
cat merged/in_both.txt

echo 'new file' > merged/new_file
ls -l */new_file 

rm merged/in_both.txt
ls -l upper/in_both.txt  lower/in_both.txt  merged/in_both.txt

mount -t overlay overlay
      -o lowerdir:/dir1:/dir2:/dir3:...:/dir25,upperdir=...
```
## namespace
```
lsns -t net
cd /proc/25452/ns/
```
## cgroup
```
cat /proc/25452/cgroup
11:pids:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
10:freezer:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
9:hugetlb:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
8:perf_event:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
7:blkio:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
6:cpuset:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
5:memory:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
4:devices:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
3:cpu,cpuacct:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
2:net_cls,net_prio:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
1:name=systemd:/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
# ls -l /sys/fs/cgroup/memory//kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
```
```
ls -l /sys/fs/cgroup/memory/kubepods/besteffort/pod8d80a5f8-cb1e-4b28-ba54-393e6b363e20/a99d384f32fc7aeb8a06934e387ed9ea30992676257a61af37d705805f1dffb7
```
# microsoft demo, run nginx as webserver

```
kubectl run --image=nginx nginx
```

#show running pod
```
kubectl get po --show-labels -owide -w
```
# expose svc
```
kubectl expose deploy nginx --selector run=nginx --port=80 --type=NodePort
```
# check svc detail
k get svc
#check nodeip
```
minikube ssh
ifconfig eth1
```
#access service
```
curl <nodeip>:<nodeport>
```
#run envoy
```
kubectl create configmap envoy-config --from-file=envoy.yaml
kubectl create -f envoy-deploy.yaml
kubectl expose deploy envoy --selector run=envoy --port=10000 --type=NodePort
```
#access service
```
curl <nodeip>:<nodeport>
```
# scale up/down/failover

## configmap
```
cat game.properties

#configmap from file
kubectl create configmap game-config --from-file=game.properties
kubectl create configmap game-env-config --from-env-file=game.properties
kubectl get configmap -oyaml game-config
```
#configmap from literal
```
kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm
#downward api pod
kubectl create -f downward-api-pod.yaml
kubectl get po downward-api-pod
kubectl logs -f downward-api-pod
```

# volume

```
kubectl create -f configmap-volume-pod.yaml
kubectl get po
kubectl logs -f configmap-volume-pod
```
#readiness probe
```
kubectl create -f centos-readiness.yaml
```
#multiple container pods

columns
```
kubectl get svc  -o=custom-columns=NAME:.metadata.name,CREATED:'.metadata.annotations'
```


# istio
```
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

enable access log

```
istioctl manifest apply --set values.global.proxy.accessLogFile="/dev/stdout"
```

enable mts
istioctl manifest apply --set values.global.mtls.enabled=true --set values.global.controlPlaneSecurityEnabled=true

enable tracing
istioctl manifest apply --set values.tracing.enabled=disable --set values.tracing.provider=zipkin

#create dr
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
#vs all v1
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
#header based vs
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
#delete vs
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
# add fix delay to rating
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
#proxy-config
istioctl pc

# proxy-status
istioctl ps

# check authn status
istioctl authn tls-check details-v1-74f858558f-qqg99

kubectl apply -f samples/bookinfo/policy/mixer-rule-productpage-ratelimit.yaml

# kubebuilder
kubebuilder init --domain example.com
kubebuilder create api --group infra --version v1 --kind WebService
make install

# operator sdk
# create new project
```
operator-sdk new memorycache-operator
```
# add types

```
cd memorycache-operator
operator-sdk add api --api-version=cache.example.com/v1alpha1 --kind=Memcached
```
# modify types and generate new code

```
operator-sdk generate k8s
operator-sdk generate openapi
```
# operator helm
```
operator-sdk new nginx-operator --type=helm --kind=Nginx --api-version=web.example.com/v1alpha1
operator-sdk add crd --api-version=web.example.com/v1alpha1 --kind=Envoy --update-watches=true
```

