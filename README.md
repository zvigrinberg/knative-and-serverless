# knative-and-serverless

- Show how to set up a serverless service.
- Show how to get start with Knative Eventing 

## Pre-requisites.
- Kubernetes/Openshift cluster or local cluster/container cluster with [minikube](https://minikube.sigs.k8s.io/docs/start/) or [kind](https://kind.sigs.k8s.io/)

## Knative Serving

### Installing

1. Connect to cluster, in these instruction we'll use a minikube single node cluster.
```shell
minikube start
```

2. Install Custom Resources Definitions resources(CRDS) of Knative Serving:
```shell
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.8.0/serving-crds.yaml
```

3. Install All Knative Serving resources:
```shell
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.8.0/serving-core.yaml
```

4. Installing an Istio networking layer
```shell
kubectl apply -l knative.dev/crd-install=true -f https://github.com/knative/net-istio/releases/download/knative-v1.8.0/istio.yaml
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.8.0/istio.yaml
```

5. Install the Knative istio controller:
```shell
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.8.0/net-istio.yaml
```

6. Retrieve the external ip or dns name of loadbalancer by running either one of the following command: 
```shell
export INGRESS_GATEWAY=$(kubectl --namespace istio-system get service istio-ingressgateway -o=jsonpath="{..hostname}")
export INGRESS_GATEWAY=$(kubectl --namespace istio-system get service istio-ingressgateway -o=jsonpath="{..ip}")
```

7. Ensure that all knative serving pods are up and running, and ready:
```shell
kubectl get pods -n knative-serving
```

8. Define DNS by installing Magic DNS , that enables Knative Serving to use sslip.io as the DNS suffix
```shell
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.8.0/serving-default-domain.yaml
```

9. Installing a demo serverless service
```shell
[zgrinber@zgrinber knative-and-serverless]$ kn service create helloworld-go --image gcr.io/knative-samples/helloworld-go
Creating service 'helloworld-go' in namespace 'default':

  0.092s The Route is still working to reflect the latest desired specification.
  0.105s ...
  0.164s Configuration "helloworld-go" is waiting for a Revision to become ready.
 38.129s ...
 38.180s Ingress has not yet been reconciled.
 38.697s Waiting for load balancer to be ready
 39.141s Ready to serve.

Service 'helloworld-go' created to latest revision 'helloworld-go-00001' is available at URL:
http://helloworld-go.default.10.97.107.16.sslip.io
```

10. Wait one Minute
11. See that the pods of service are terminating:
```shell
[zgrinber@zgrinber knative-and-serverless]$ kubectl get pod -l app=helloworld-go-00001 -w
NAME                                             READY   STATUS        RESTARTS   AGE
helloworld-go-00001-deployment-9df76b95d-td2pf   3/3     Terminating   0          71s
helloworld-go-00001-deployment-9df76b95d-td2pf   2/3     Terminating   0          91s
helloworld-go-00001-deployment-9df76b95d-td2pf   0/3     Terminating   0          93s
helloworld-go-00001-deployment-9df76b95d-td2pf   0/3     Terminating   0          93s
helloworld-go-00001-deployment-9df76b95d-td2pf   0/3     Terminating   0          93s

```
12. Invoke The Serivce:
```shell
xdg-open http://helloworld-go.default.10.97.107.16.sslip.io/
```

13. Check the status of pods(Keep window open)
```shell
[zgrinber@zgrinber knative-and-serverless]$ kubectl get pod -l app=helloworld-go-00001 -w
NAME                                             READY   STATUS        RESTARTS   AGE
helloworld-go-00001-deployment-9df76b95d-td2pf   3/3     Terminating   0          71s
helloworld-go-00001-deployment-9df76b95d-td2pf   2/3     Terminating   0          91s
helloworld-go-00001-deployment-9df76b95d-td2pf   0/3     Terminating   0          93s
helloworld-go-00001-deployment-9df76b95d-td2pf   0/3     Terminating   0          93s
helloworld-go-00001-deployment-9df76b95d-td2pf   0/3     Terminating   0          93s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     Pending       0          0s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     Pending       0          0s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     Init:0/1      0          0s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     PodInitializing   0          3s
helloworld-go-00001-deployment-9df76b95d-59bbn   1/3     Running           0          4s
helloworld-go-00001-deployment-9df76b95d-59bbn   2/3     Running           0          4s
helloworld-go-00001-deployment-9df76b95d-59bbn   3/3     Running           0          5s
```
14. Wait another minute, and then checks again(the service scaled down to zero):
```shell
NAME                                             READY   STATUS        RESTARTS   AGE
helloworld-go-00001-deployment-9df76b95d-td2pf   3/3     Terminating   0          71s
helloworld-go-00001-deployment-9df76b95d-td2pf   2/3     Terminating   0          91s
helloworld-go-00001-deployment-9df76b95d-td2pf   0/3     Terminating   0          93s
helloworld-go-00001-deployment-9df76b95d-td2pf   0/3     Terminating   0          93s
helloworld-go-00001-deployment-9df76b95d-td2pf   0/3     Terminating   0          93s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     Pending       0          0s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     Pending       0          0s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     Init:0/1      0          0s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     PodInitializing   0          3s
helloworld-go-00001-deployment-9df76b95d-59bbn   1/3     Running           0          4s
helloworld-go-00001-deployment-9df76b95d-59bbn   2/3     Running           0          4s
helloworld-go-00001-deployment-9df76b95d-59bbn   3/3     Running           0          5s
helloworld-go-00001-deployment-9df76b95d-59bbn   3/3     Terminating       0          66s
helloworld-go-00001-deployment-9df76b95d-59bbn   2/3     Terminating       0          91s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     Terminating       0          97s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     Terminating       0          97s
helloworld-go-00001-deployment-9df76b95d-59bbn   0/3     Terminating       0          97s

```
