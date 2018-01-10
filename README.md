# kubernetes
### HOWTO:
#### 1. [Install minikube in macOS/Ubuntu](#install-minikube-in-macosubuntu-dont-install-ubuntu-in-virtualbox)
#### 2. [Create a Node.js example app deployment and service](#example-to-create-a-nodejs-application)
#### 3. [Update the example application](#update-the-example-app)
#### 4. [Scaling example application](#scaling-your-deployments-eg-scaling-hello-node-in-this-example)
#### 5. [Delete the deployment and service](#delete-the-deployment-and-service)
#### 6. [Using YAML file to manage application](#using-yaml-file-to-manage-application)

## Install minikube in macOS/Ubuntu (don’t install ubuntu in Virtualbox):
#### 1. Install [Virtualbox](https://www.virtualbox.org/)

#### 2. Install minikube in MacOS:
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

Install minikube in Ubuntu:
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

#### 3. Install kubectl in MacOS:
```
curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

Install kubectl in Ubuntu:
```
curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

#### 4. Start up minikube

```
minikube start --vm-driver=virtualbox
```

#### 5. After a few minutesthe minikube instance will be ready. then use kubectl to verify that everything is working:
```
kubectl cluster-info
```
result should be:
Kubernetes master is running at https://192.168.99.100:8443
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

#### 6. To access the dashboard. Run the next command to get the dashboard.
```
minikube dashboard
```

#### 7. Stop Minikube
```
minikube stop
```

### Ref:
* http://www.admintome.com/blog/minikube-introduction/
* https://github.com/kubernetes/minikube/blob/v0.24.1/README.md
* https://kubernetes-v1-4.github.io/docs/getting-started-guides/minikube/#install-minikube
* https://www.youtube.com/watch?v=GqyJgvTgqq4

## Example To create a Node.js application

#### 1. Save this code in a folder named hellonode with the filename 'server.js':
```
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
```
FROM node:6.9.2
EXPOSE 8080
COPY server.js .
CMD node server.js
```
(**NOTE**: You can run 'node -v' to check your node version in terminal)

#### 3. Make sure you are using the Minikube Docker daemon, run:
```
eval $(minikube docker-env)
```

Later, when you no longer wish to use the Minikube host, you can undo this change by running eval
```
$(minikube docker-env -u).
```

#### 4. Build your Docker image, using the Minikube Docker daemon:
```
docker build -t hello-node:v1 .
```
#### 5. Use the kubectl run command to create a Deployment that manages a Pod:
```
kubectl run hello-node --image=hello-node:v1 --port=8080
```

View the Deployment:
```
kubectl get deployments
```

View the Pod:
```
kubectl get pods
```

#### 6. Expose the Pod using the kubectl expose command:
```
kubectl expose deployment hello-node --type=NodePort
```
Change '--type=NodePort' to '--type=LoadBalancer' if LoadBalancer is setting up

View the Service you just created:
```
kubectl get services
```

Output should look like:
```
NAME         CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
hello-node   10.0.0.71    <pending>     8080:3xxxx/TCP   6m
kubernetes   10.0.0.1     <none>        443/TCP    14d
```
(**NOTE**: The --type=LoadBalancer flag indicates that you want to expose your Service outside of the cluster. On cloud providers that support load balancers, an external IP address would be provisioned to access the Service. If load balancer is supported,the above output result will show the EXTERNAL-IP. But **minikube dones't support** load balancers, it's always pending)

#### 7. Access the service internally:
```
minikube service hello-node
```

### Update the example app
Edit the server.js file to return a new message:
```
response.end('Hello World Again!');
```

Build a new version of your image:
```
docker build -t hello-node:v2 .
```

Update the image of your Deployment:
```
kubectl set image deployment/hello-node hello-node=hello-node:v2
```

Run your app again to view the new message:
```
minikube service hello-node
```

### Ref
* https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/#create-your-nodejs-application

### Scaling Your deployments (e.g. Scaling hello-node in this example)

#### 1. list current deployments
```
kubectl get deployments
```

##### example result:
```
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           1h
```

#### 2. Scaling up/down to 4 pods
```
kubectl scale deployments/hello-node --replicas=4
```

```
kubectl get deployments
```
##### example result:
```
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   4         4         4            1           1h
```

```
kubectl get pods -o wide
```
##### example result:
```
NAME                          READY     STATUS    RESTARTS   AGE       IP           NODE
hello-node-75ddf8bc9d-2xnlh   1/1       Running   0          1m        172.17.0.7   minikube
hello-node-75ddf8bc9d-9xhgj   1/1       Running   0          1h        172.17.0.5   minikube
hello-node-75ddf8bc9d-gh7xx   1/1       Running   0          1m        172.17.0.6   minikube
hello-node-75ddf8bc9d-vxs4k   1/1       Running   0          1m        172.17.0.4   minikube
```
(Now their are 4 running pods)

### Ref
* https://kubernetes.io/docs/tutorials/kubernetes-basics/scale-intro/

### Delete the deployment and service
```
kubectl delete deployment hello-node
```
```
kubectl delete service hello-node
```
### Using YAML file to manage application
#### 1. Create and go into working directory
```
mkdir /demo/application1
cd /demo/application1
```

#### 2. Create example.app1.deployment.yaml file
```
vim example.app1.deployment.yaml file
```
insert the code:
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-node
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hello-node
        tier: hello-app
        trcak: stable
    spec:
      containers:
      - name: hello-node
        image: "hello-node:v1"
        ports:
          - containerPort: 8085
```
#### 3. Create example.app1.service.yaml file
```
vim example.app1.service.yaml file
```
insert the code:
```
apiVersion: v1
kind: Service
metadata:
  name: hello-node
spec:
  selector:
    app: hello-node
    tier: hello-app
  ports:
  - protocol: "TCP"
    port: 80
    targetPort: 8085
  type: NodePort
```
#### 4. Create deployment and service
deployment:
```
kubectl create -f example.app1.deployment.yaml
```
service:
```
kubectl create -f example.app1.service.yaml
```

Or all yaml file in current directory
```
kubectl create -f .
```
To view the running deployment and service:
```
kubectl get deployments
kubectl get service
```
