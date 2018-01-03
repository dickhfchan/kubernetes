# kubernetes

## Install minikube in macOS/Ubuntu (don’t install ubuntu in Virtualbox):
#### 1. Install [Virtualbox](https://www.virtualbox.org/)

#### 2. Install minikube in MacOS:
```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

Install minikube in Ubuntu:
```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

#### 3. Install kubectl in MacOS:
```shell
curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

Install kubectl in Ubuntu:
```shell
curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

#### 4. Start up minikube

```shell
minikube start --vm-driver=virtualbox
```

#### 5. After a few minutesthe minikube instance will be ready. then use kubectl to verify that everything is working:
```shell
kubectl cluster-info
```
result should be:
Kubernetes master is running at https://192.168.99.100:8443
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

#### 6. To access the dashboard. Run the next command to get the dashboard.
```shell
minikube dashboard
```

#### 7. Stop Minikube
```shell
minikube stop
```

### Ref:
* http://www.admintome.com/blog/minikube-introduction/
* https://github.com/kubernetes/minikube/blob/v0.24.1/README.md
* https://kubernetes-v1-4.github.io/docs/getting-started-guides/minikube/#install-minikube
* https://www.youtube.com/watch?v=GqyJgvTgqq4

## Example To create a Node.js application

#### 1. Save this code in a folder named hellonode with the filename 'server.js':
```shell
var http = require('http');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!');
};
var www = http.createServer(handleRequest);
www.listen(8080);
```

#### 2. Create a Docker container image
Create a file, also in the hellonode folder, named 'Dockerfile':
```shell
FROM node:6.9.2
EXPOSE 8080
COPY server.js .
CMD node server.js
```
(**NOTE**: You can run 'node -v' to check your node version in terminal)

#### 3. Make sure you are using the Minikube Docker daemon, run:
```shell
eval $(minikube docker-env)
```

Later, when you no longer wish to use the Minikube host, you can undo this change by running eval
```shell
$(minikube docker-env -u).
```

#### 4. Build your Docker image, using the Minikube Docker daemon:
```shell
docker build -t hello-node:v1 .
```
#### 5. Use the kubectl run command to create a Deployment that manages a Pod:
```shell
kubectl run hello-node --image=hello-node:v1 --port=8080
```

View the Deployment:
```shell
kubectl get deployments
```

View the Pod:
```shell
kubectl get pods
```

#### 6. Expose the Pod using the kubectl expose command:
```shell
kubectl expose deployment hello-node --type=LoadBalancer
```

View the Service you just created:
```shell
kubectl get services
```

Output should look like:
```shell
NAME         CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
hello-node   10.0.0.71    <pending>     8080:3xxxx/TCP   6m
kubernetes   10.0.0.1     <none>        443/TCP    14d
```
(**NOTE**: The --type=LoadBalancer flag indicates that you want to expose your Service outside of the cluster. On cloud providers that support load balancers, an external IP address would be provisioned to access the Service. If load balancer is supported,the above output result will show the EXTERNAL-IP. But **minikube dones't support** load balancers, it's always pending)

#### 7. Access the service internally:
```shell
minikube service hello-node
```

### Update the example app
Edit the server.js file to return a new message:
```shell
response.end('Hello World Again!');
```

Build a new version of your image:
```shell
docker build -t hello-node:v2 .
```

Update the image of your Deployment:
```shell
kubectl set image deployment/hello-node hello-node=hello-node:v2
```

Run your app again to view the new message:
```shell
minikube service hello-node
```

### Scaling Your deployments (e.g. Scaling hello-node in this example)

#### 1. list current deployments
```shell
kubectl get deployments
```

##### example result:
```shell
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           1h
```

#### 2. Scaling up/down to 4 pods
```shell
kubectl scale deployments/hello-node --replicas=4
```

```shell
kubectl get deployments
```
##### example result:
```shell
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   4         4         4            1           1h
```

```shell
kubectl get pods -o wide
```
##### example result:
```shell
NAME                          READY     STATUS    RESTARTS   AGE       IP           NODE
hello-node-75ddf8bc9d-2xnlh   1/1       Running   0          1m        172.17.0.7   minikube
hello-node-75ddf8bc9d-9xhgj   1/1       Running   0          1h        172.17.0.5   minikube
hello-node-75ddf8bc9d-gh7xx   1/1       Running   0          1m        172.17.0.6   minikube
hello-node-75ddf8bc9d-vxs4k   1/1       Running   0          1m        172.17.0.4   minikube
```
(Now their are 4 running pods)


### Delete the deployment and service
```shell
kubectl delete deployment hello-node
```
```shell
kubectl delete service hello-node
```
