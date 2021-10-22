# install-freezer

Setup instructions for using the `container-freezer` on knative

## Set up a cluster

### minikube

``` bash
minikube start --cpus 3 --container-runtime containerd \
    --extra-config=apiserver.service-account-signing-key-file=/var/lib/minikube/certs/sa.key \
    --extra-config=apiserver.service-account-key-file=/var/lib/minikube/certs/sa.pub \
    --extra-config=apiserver.service-account-issuer=api \
    --extra-config=apiserver.service-account-api-audiences=api,concurrency-state-hook \
    --extra-config=apiserver.authorization-mode=Node,RBAC \
    --extra-config=kubelet.authentication-token-webhook=true

kubectl label nodes minikube knative.dev/container-runtime=containerd
```

In a separate terminal, runtime

``` bash
minikube tunnel
```

## Install Knative serving and related components

``` bash
kubectl apply -f serving-crds.yaml
kubectl apply -f serving-core.yaml

kubectl apply -f kourier.yaml
kubectl patch configmap/config-network --namespace knative-serving --type merge --patch '{"data":{"ingress.class":"kourier.ingress.networking.knative.dev"}}'
  
kubectl apply -f serving-default-domain.yaml
```

## Install container-freezer

``` bash
kubectl apply -f freezer.yaml

kubectl patch configmap/config-deployment -n knative-serving --type merge -p '{"data":{"concurrencyStateEndpoint":"http://$HOST_IP:9696"}}'
```

 
## Run sample app

``` bash
kubectl apply -f example.yaml
```
