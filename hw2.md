# Homework 2

Student: Junlin Zeng, Tiancan Yu

## API Gateway exposing the HTTP

Install the edgectl for ambassador

```bash
wget https://metriton.datawire.io/downloads/darwin/edgectl
sudo mv edgectl /usr/local/bin/
sudo chmod a+x /usr/local/bin/edgectl
edgectl install
```

Use the `skaffold` to turn on the example service. Create a `Mapping` config to expose the frontend service:

```bash
git clone git@github.com:sysustrange/microservices-demo.git
cd microservices-demo/microservices-demo/
skaffold run
kubectl apply -f mapping.yaml
```

Check the pods and services

Note that we haven't deploy the frontend-external service. (which is an "evil" service in the `microservice-demo` exposing the frontend service automatically. We deleted this service to prove we did expose the frontend service using Ambassador.)

And we expose the frontend service by the API Gateway.

![Screen Shot 2020-03-03 at 10.27.10 PM](img/Screen%20Shot%202020-03-03%20at%2010.27.10%20PM-3292591.png)

![Screen Shot 2020-03-03 at 10.27.22 PM](img/Screen%20Shot%202020-03-03%20at%2010.27.22%20PM.png)


# Authentication


Build [our auth service](https://github.com/yutiancan/ambassador-auth-service) first. We used the example service provided in Ambassador and we made changes to this source file to use the `username:password` for all the urls. Nothing should be seen if such username and password are not provided. We built a docker image and upload it to the docker hub.

```bash
cd ../ambassador-auth-service
bash travis-build.sh
```

Our docker image is in here.

```http
https://hub.docker.com/r/tiancanyu/ambassador-auth-service
```

![Screen Shot 2020-03-03 at 9.54.50 PM](img/Screen%20Shot%202020-03-03%20at%209.54.50%20PM.png)



Deploy the service first in the same namespace. And also apply our filter file as well, in which contains the definition of external-filter and also our filerPolicy, which we just mark all the urls.

```
cd ../microservices-demo/
kubectl apply -f user-auth.yaml
kubectl apply -f extenal-filter.yaml
```


Check for deployment

```bash
kubectl get svc -n ambassador
```

![Screen Shot 2020-03-03 at 10.10.01 PM](img/Screen%20Shot%202020-03-03%20at%2010.10.01%20PM.png)



```bash
kubectl get pods -n ambassador
```

![Screen Shot 2020-03-03 at 10.10.44 PM](img/Screen%20Shot%202020-03-03%20at%2010.10.44%20PM.png)


Verify the auth, luckily, you won't be able to see our page without auth.

```bash
curl -Lkv http://localhost
```

![Screen Shot 2020-03-03 at 9.52.01 PM](img/Screen%20Shot%202020-03-03%20at%209.52.01%20PM.png)



```bash
curl -Lkv -u username:password http://localhost
```

![Screen Shot 2020-03-03 at 9.52.54 PM](img/Screen%20Shot%202020-03-03%20at%209.52.54%20PM.png)

![Screen Shot 2020-03-03 at 9.53.09 PM](img/Screen%20Shot%202020-03-03%20at%209.53.09%20PM.png)



To see it more clearly.

```bash
curl -Lkv -u username:password http://localhost > ~/Desktop/test.html
```

![Screen Shot 2020-03-03 at 9.53.59 PM](img/Screen%20Shot%202020-03-03%20at%209.53.59%20PM.png)

![Screen Shot 2020-03-03 at 10.13.21 PM](img/Screen%20Shot%202020-03-03%20at%2010.13.21%20PM.png)



Test via web browser

open Chrome and enter

```http
http://localhost
```

![Screen Shot 2020-03-03 at 9.57.50 PM](img/Screen%20Shot%202020-03-03%20at%209.57.50%20PM.png)



Enter username and password

![Screen Shot 2020-03-03 at 9.58.04 PM](img/Screen%20Shot%202020-03-03%20at%209.58.04%20PM-3291776.png)



See the home page of the web service

![Screen Shot 2020-03-03 at 9.58.12 PM](img/Screen%20Shot%202020-03-03%20at%209.58.12%20PM.png)



Bench mark with LoadGenerator

```bash
cd ../loadgenerator-benchmark
```



Before  add the Authetication+API Gateway

delete auth

```bash
kubectl delete -f extenal-filter.yaml
kubectl delete -f user-auth.yaml 
```

![Screen Shot 2020-03-03 at 10.32.53 PM](img/Screen%20Shot%202020-03-03%20at%2010.32.53%20PM.png)

We decomment the following lines in microservices-demo/microservices-demo/kubernetes-manifests/frontend.yaml as following

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-external
  namespace: ambassador
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - name: http
    port: 80
    targetPort: 8080
```



Apply the change

```bash
kubectl delete svc ambassador -n ambassador
kubectl delete svc ambassador-admin -n ambassador
kubectl delete svc ambassador-redis -n ambassador

skaffold delete
skaffold run
```



Then wait for 1 - 3 minutes. Now, can see the frontend-extenal service start and the ambassador services get down.

```bash
kubectl get svc -n ambassador
```

![Screen Shot 2020-03-03 at 11.23.39 PM](img/Screen%20Shot%202020-03-03%20at%2011.23.39%20PM.png)



Run benchmark

```bash
export FRONTEND_ADDR=localhost
bash loadgen.sh
```

![Screen Shot 2020-03-03 at 11.11.22 PM](img/Screen%20Shot%202020-03-03%20at%2011.11.22%20PM.png)

![Screen Shot 2020-03-03 at 11.13.14 PM](img/Screen%20Shot%202020-03-03%20at%2011.13.14%20PM.png)

![Screen Shot 2020-03-03 at 11.16.17 PM](img/Screen%20Shot%202020-03-03%20at%2011.16.17%20PM.png)

We can see that it always success.



After adding the Authetication+API Gateway



Comment the following lines in microservices-demo/microservices-demo/kubernetes-manifests/frontend.yaml as following

```yaml
# ---
# apiVersion: v1
# kind: Service
# metadata:
#   name: frontend-external
#   namespace: ambassador
# spec:
#   type: LoadBalancer
#   selector:
#     app: frontend
#   ports:
#   - name: http
#     port: 80
#     targetPort: 8080
```

modify loadgen.sh as following

```bash
set -e
trap "exit" TERM

#export FRONTEND_ADDR="localhost"

if [[ -z "${FRONTEND_ADDR}" ]]; then
    echo >&2 "FRONTEND_ADDR not specified"
    exit 1
fi

set -x

# if one request to the frontend fails, then exit
STATUSCODE=$(curl -Lk -u username:password --silent --output /dev/stderr --write-out "%{http_code}" http://${FRONTEND_ADDR})
if test $STATUSCODE -ne 200; then
    echo "Error: Could not reach frontend - Status code: ${STATUSCODE}"
    exit 1
fi

# else, run loadgen
locust --host="http://${FRONTEND_ADDR}" --no-web -c "${USERS:-10}" 2>&1
```



Change the deployment and run benchmark

```
edgectl install
skaffold delete
skaffold run
kubectl apply -f mapping.yaml
kubectl apply -f extenal-filter.yaml
kubectl apply -f user-auth.yaml 
bash loadgen.sh
```

we can see that the auth block the request without username and password

![Screen Shot 2020-03-03 at 10.19.06 PM](img/Screen%20Shot%202020-03-03%20at%2010.19.06%20PM.png)



To see the result of  request with username and password

we modity the loadgen.sh​ as following.

```

```

