# kubernetes

Install minikube in macOS/Ubuntu (don’t install ubuntu in Virtualbox):
1. Install Virtualbox
https://www.virtualbox.org/

2. Install minikube in MacOS:
>curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

Install minikube in Ubuntu:
>curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

3. Install kubectl in MacOS:
>curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/

Install kubectl in Ubuntu:
>curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/

4. Start up minikube
>minikube start --vm-driver=virtualbox

5. After a few minutesthe minikube instance will be ready. then use kubectl to verify that everything is working:
>kubectl cluster-info

result should be:
Kubernetes master is running at https://192.168.99.100:8443
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

6. To access the dashboard. Run the next command to get the dashboard.
>minikube dashboard

Ref:
http://www.admintome.com/blog/minikube-introduction/
https://github.com/kubernetes/minikube/blob/v0.24.1/README.md
https://kubernetes-v1-4.github.io/docs/getting-started-guides/minikube/#install-minikube
https://www.youtube.com/watch?v=GqyJgvTgqq4

Note:
*It is not working if you run minikube in ubuntu that installed in virtualbox.
