# Setup bookinfo with Minikube
Steps required to run the bookinfo demo application for ISTIO with minikube for dummies.

## Start minikube
```bash
minikube start --memory=8192 --cpus=8  \
  --extra-config=controller-manager.cluster-signing-cert-file="/var/lib/minikube/certs/ca.crt" \
  --extra-config=controller-manager.cluster-signing-key-file="/var/lib/minikube/certs/ca.key" \
  --vm-driver=virtualbox
```
```
😄  minikube v1.5.0 on Ubuntu 19.10
🔥  Creating virtualbox VM (CPUs=8, Memory=8192MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.16.2 on Docker 18.09.9 ...
    ▪ controller-manager.cluster-signing-cert-file=/var/lib/minikube/certs/ca.crt
    ▪ controller-manager.cluster-signing-key-file=/var/lib/minikube/certs/ca.key
🚜  Pulling images ...
🚀  Launching Kubernetes ... 
⌛  Waiting for: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"
```
wait for a long time. To verify
```bash
kubectl get pods -n kube-system
```
```
NAME                               READY   STATUS    RESTARTS   AGE
coredns-5644d7b6d9-9dqz8           1/1     Running   0          111s
coredns-5644d7b6d9-dxmn4           1/1     Running   0          111s
etcd-minikube                      1/1     Running   0          45s
kube-addon-manager-minikube        1/1     Running   0          2m2s
kube-apiserver-minikube            1/1     Running   0          64s
kube-controller-manager-minikube   1/1     Running   0          39s
kube-proxy-sbxtq                   1/1     Running   0          111s
kube-scheduler-minikube            1/1     Running   0          41s
storage-provisioner                1/1     Running   0          110s
```

## Install istio
Install some isio version accoring to [documentation](https://istio.io/docs/setup/#downloading-the-release).

```bash
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.3.3 sh -
cd istio-1.3.3
export PATH=$PWD/bin:$PATH
```
Install it in [Kubernetes](https://istio.io/docs/setup/install/kubernetes/)

```bash
for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
# TLS version
kubectl apply -f install/kubernetes/istio-demo-auth.yaml
```
Wait for a long time.
To verify
```bash
kubectl get services -n istio-system
```
```
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                                                                                      AGE
grafana                  ClusterIP      10.99.28.21      <none>        3000/TCP                                                                                                                                     74s
istio-citadel            ClusterIP      10.109.168.10    <none>        8060/TCP,15014/TCP                                                                                                                           74s
istio-egressgateway      ClusterIP      10.108.196.151   <none>        80/TCP,443/TCP,15443/TCP                                                                                                                     75s
istio-galley             ClusterIP      10.103.9.251     <none>        443/TCP,15014/TCP,9901/TCP                                                                                                                   75s
istio-ingressgateway     LoadBalancer   10.110.53.227    <pending>     15020:30843/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:31290/TCP,15030:31027/TCP,15031:31132/TCP,15032:31154/TCP,15443:30942/TCP   74s
istio-pilot              ClusterIP      10.103.116.49    <none>        15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       74s
istio-policy             ClusterIP      10.105.32.74     <none>        9091/TCP,15004/TCP,15014/TCP                                                                                                                 74s
istio-sidecar-injector   ClusterIP      10.111.132.56    <none>        443/TCP,15014/TCP                                                                                                                            74s
istio-telemetry          ClusterIP      10.110.199.148   <none>        9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       74s
jaeger-agent             ClusterIP      None             <none>        5775/UDP,6831/UDP,6832/UDP                                                                                                                   74s
jaeger-collector         ClusterIP      10.109.30.248    <none>        14267/TCP,14268/TCP                                                                                                                          74s
jaeger-query             ClusterIP      10.109.71.177    <none>        16686/TCP                                                                                                                                    74s
kiali                    ClusterIP      10.111.68.253    <none>        20001/TCP                                                                                                                                    74s
prometheus               ClusterIP      10.99.193.238    <none>        9090/TCP                                                                                                                                     74s
tracing                  ClusterIP      10.111.196.208   <none>        80/TCP                                                                                                                                       74s
zipkin                   ClusterIP      10.111.87.57     <none>        9411/TCP                                                                                                                                     74s
```
  
```bash
kubectl get pods -n istio-system
```
```
NAME                                      READY   STATUS      RESTARTS   AGE
grafana-d8465c484-rmw57                   1/1     Running     0          3m50s
istio-citadel-67f6594c46-46x95            1/1     Running     0          3m50s
istio-egressgateway-599844d77-55z4h       1/1     Running     0          3m50s
istio-galley-69945d8845-pbwqn             1/1     Running     0          3m50s
istio-grafana-post-install-1.3.3-dl7cj    0/1     Completed   0          3m51s
istio-ingressgateway-866ff9856d-xvrgd     1/1     Running     0          3m50s
istio-pilot-dc4dd86b7-f9wc5               2/2     Running     0          3m50s
istio-policy-55646b8dbc-ncdsq             2/2     Running     2          3m50s
istio-security-post-install-1.3.3-p6d2q   0/1     Completed   0          3m51s
istio-sidecar-injector-6d967869b5-npwmh   1/1     Running     0          3m49s
istio-telemetry-76bc49b4c5-xk862          2/2     Running     3          3m50s
istio-tracing-7f8b785b8c-qshn6            1/1     Running     0          3m49s
kiali-5cd7c8b6d-gj2hw                     1/1     Running     0          3m50s
prometheus-6f74d6f76d-hln7m               1/1     Running     0          3m50s
```

## Install sample application
Follow the instruction [here](https://istio.io/docs/examples/bookinfo/).

```bash
# Enable sidecar injection
kubectl label namespace default istio-injection=enabled
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml
```

to verify
```bash
kubectl get services
```
```
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.108.242.73    <none>        9080/TCP   21s
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP    8m28s
productpage   ClusterIP   10.110.113.220   <none>        9080/TCP   21s
ratings       ClusterIP   10.104.89.8      <none>        9080/TCP   21s
reviews       ClusterIP   10.106.254.78    <none>        9080/TCP   21s
```

```bash
kubectl get pods
```
```
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-78d78fbddf-xw7gp       2/2     Running   0          2m20s
productpage-v1-596598f447-hqjnx   2/2     Running   0          2m19s
ratings-v1-6c9dbf6b45-pjzb6       2/2     Running   0          2m20s
reviews-v1-7bb8ffd9b6-vfg2d       2/2     Running   0          2m20s
reviews-v2-d7d75fff8-pv4rc        2/2     Running   0          2m20s
reviews-v3-68964bc4c8-bpfkt       2/2     Running   0          2m20s
```

to access it using minikube
```bash
minikube service -n istio-system istio-ingressgateway
```
```
|--------------|----------------------|--------------------------------|--------------------------------|
|  NAMESPACE   |         NAME         |          TARGET PORT           |              URL               |
|--------------|----------------------|--------------------------------|--------------------------------|
| istio-system | istio-ingressgateway | status-port http2 https tcp    | http://192.168.99.102:30843    |
|              |                      | https-kiali https-prometheus   | http://192.168.99.102:31380    |
|              |                      | https-grafana https-tracing    | http://192.168.99.102:31390    |
|              |                      | tls                            | http://192.168.99.102:31400    |
|              |                      |                                | http://192.168.99.102:31290    |
|              |                      |                                | http://192.168.99.102:31027    |
|              |                      |                                | http://192.168.99.102:31132    |
|              |                      |                                | http://192.168.99.102:31154    |
|              |                      |                                | http://192.168.99.102:30942    |
|--------------|----------------------|--------------------------------|--------------------------------|
```

this will list a lots of ports. Use http2 URL.
e.g. open http://192.168.99.102:31380/productpage



