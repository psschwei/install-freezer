# install-freezer

Setup instructions for using the Knative `container-freezer`

![freezer in action](demo/freezer-in-action.gif)

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

In a separate terminal, run

``` bash
minikube tunnel
```

### IBM Kubernetes Service

Follow [standard process](https://cloud.ibm.com/docs/containers?topic=containers-clusters) to create a new cluster. No special configuration needed.

Tested on a 3-node, single zone (wdc04) v1.22.2_1522 cluster.

## Install Knative serving and related components

``` bash
kubectl apply -f serving-crds.yaml
kubectl apply -f serving-core.yaml

kubectl apply -f kourier.yaml
kubectl patch configmap/config-network \
    --namespace knative-serving \
    --type merge \
    --patch '{"data":{"ingress.class":"kourier.ingress.networking.knative.dev"}}'
  
kubectl apply -f serving-default-domain.yaml
```

## Install container-freezer

``` bash
kubectl apply -f freezer.yaml

kubectl patch configmap/config-deployment \
    -n knative-serving \
    --type merge \
    -p '{"data":{"concurrencyStateEndpoint":"http://$HOST_IP:9696"}}'
```

 
## Run sample app

``` bash
kubectl apply -f example.yaml
```

### Using the sample app

The [sample app](example.yaml) runs a background goroutine that logs a message every half-second or so while running. If you deploy it normally and watch the `user-container` logs, you'll see it printing constantly. If you enable the container-freezer and then deploy it, you'll see it only prints while the server is servicing http requests.

## Source Code

https://www.github.com/knative/serving

https://www.github.com/knative-sandbox/container-freezer

