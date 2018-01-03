# kubernetes

### Install minikube in macOS/Ubuntu (don’t install ubuntu in Virtualbox):
1. Install [Virtualbox](https://www.virtualbox.org/)

2. Install minikube in MacOS:
```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

Install minikube in Ubuntu:
```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

3. Install kubectl in MacOS:
```shell
curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

Install kubectl in Ubuntu:
```shell
curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

4. Start up minikube
```shell
minikube start --vm-driver=virtualbox
```

5. After a few minutesthe minikube instance will be ready. then use kubectl to verify that everything is working:
```shell
>kubectl cluster-info
```

result should be:
Kubernetes master is running at https://192.168.99.100:8443
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

6. To access the dashboard. Run the next command to get the dashboard.
```shell
minikube dashboard
```

7. Stop Minikube
```shell
minikube stop
```

### Ref:
* http://www.admintome.com/blog/minikube-introduction/
* https://github.com/kubernetes/minikube/blob/v0.24.1/README.md
* https://kubernetes-v1-4.github.io/docs/getting-started-guides/minikube/#install-minikube
* https://www.youtube.com/watch?v=GqyJgvTgqq4

### Example To create a Node.js application

1. Save this code in a folder named hellonode with the filename 'server.js':
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

2. Create a Docker container image
Create a file, also in the hellonode folder, named 'Dockerfile':
```shell
FROM node:6.9.2
EXPOSE 8080
COPY server.js .
CMD node server.js
```
(NOTE: You can run 'node -v' to check your node version in terminal)

3. Make sure you are using the Minikube Docker daemon, run:
```shell
eval $(minikube docker-env)
```

Later, when you no longer wish to use the Minikube host, you can undo this change by running eval
```shell
$(minikube docker-env -u).
```

4. Build your Docker image, using the Minikube Docker daemon:
```shell
docker build -t hello-node:v1 .
```
5. Use the kubectl run command to create a Deployment that manages a Pod:
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
