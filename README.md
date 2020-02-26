# Run demo

[toc]



```bash
git clone https://github.com/GoogleCloudPlatform/microservices-demo.git
```



```bash
brew install kubectl
```

delete all the previous files

```bash
brew install skaffold
kubectl delete pods --all
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
docker rmi -f $(docker images -q)
```



Add support for Kubernetes in Docker desktop

![Screen Shot 2020-02-07 at 12.16.22 AM](img/Screen Shot 2020-02-07 at 12.16.22 AM.png)



```bash
kubectl get nodes
```

![Screen Shot 2020-02-07 at 12.17.40 AM](img/Screen Shot 2020-02-07 at 12.17.40 AM.png)

```bash
curl -L https://istio.io/downloadIstio | sh -
```



![Screen Shot 2020-02-07 at 12.25.28 AM](img/Screen Shot 2020-02-07 at 12.25.28 AM.png)



```bash
cd istio-1.4.3
```

![Screen Shot 2020-02-07 at 12.25.53 AM](img/Screen Shot 2020-02-07 at 12.25.53 AM.png)

```bash
export PATH=$PWD/bin:$PATH
```

![Screen Shot 2020-02-07 at 12.26.33 AM](img/Screen Shot 2020-02-07 at 12.26.33 AM.png)

```bash
istioctl version
istioctl manifest generate --set profile=demo | kubectl delete -f -
istioctl manifest apply --set profile=demo
```

![Screen Shot 2020-02-07 at 12.27.21 AM](img/Screen Shot 2020-02-07 at 12.27.21 AM.png)

```bash
kubectl get svc -n istio-system
```

![Screen Shot 2020-02-07 at 12.36.06 AM](img/Screen Shot 2020-02-07 at 12.36.06 AM.png)



```bash
kubectl label namespace default istio-injection=enabled
```



```bash
kubectl apply -f ./istio-manifests
```



```bash
skaffold run
```

![Screen Shot 2020-02-07 at 12.55.58 AM](img/Screen Shot 2020-02-07 at 12.55.58 AM.png)

```
kubectl get pods
```

![Screen Shot 2020-02-07 at 12.55.27 AM](img/Screen Shot 2020-02-07 at 12.55.27 AM.png)

![Screen Shot 2020-02-07 at 12.56.16 AM](img/Screen Shot 2020-02-07 at 12.56.16 AM.png)

```bash
kubectl -n istio-system get service istio-ingressgateway
```

![Screen Shot 2020-02-07 at 12.56.42 AM](img/Screen Shot 2020-02-07 at 12.56.42 AM.png)

```bash
INGRESS_HOST="$(kubectl -n istio-system get service istio-ingressgateway \
   -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')"
```

```bash
echo "$INGRESS_HOST"
```

![Screen Shot 2020-02-07 at 12.33.32 AM](img/Screen Shot 2020-02-07 at 12.33.32 AM.png)



![Screen Shot 2020-02-07 at 12.34.11 AM](img/Screen Shot 2020-02-07 at 12.34.11 AM.png)

```bash
skaffold delete
```



![Screen Shot 2020-02-07 at 12.58.44 AM](img/Screen Shot 2020-02-07 at 12.58.44 AM-1055140.png)

![Screen Shot 2020-02-07 at 12.34.57 AM](img/Screen Shot 2020-02-07 at 12.44.51 AM.png)