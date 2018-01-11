ANSW K8S CLUSTER
================


The ANSW kubernetes cluster in aws is deployed using KOPS prefered method of aws deployments.

its currently configured for an HA production setup using weave networking and 2 nodes.

the kops state is currently being saved to an S3 bucket called `answ-k8s-cluster-state`

INITIAL SETUP
-------------

in order to start managing the cluster you need to install the kops cli tool, if your on mac you can do it by using brew

`brew install kops`

if you dont have brew simply download the binary from github

```
wget -O kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-darwin-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/
```

same method can be used for `linux`

```
wget -O kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/
```

you will also need the kubectl cli tool in order to use kubernetes you can get similary to kops

`brew install kubectl`

OR Linux:

```
curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

MANAGING THE CLUSTER
--------------------

the follow configuration is being used

```
export AWS_ACCESS_KEY_ID=<changeme>
export AWS_SECRET_ACCESS_KEY=<changeme>
export NAME=answ.k8s.local
export KOPS_STATE_STORE=s3://answ-k8s-cluster-state
export ZONES=ap-southeast-1a,ap-southeast-1b
```

once the env vars are set using a correct AWS secret and key, you can start managing your clusters, if you want to list the number of clusters on your state you can simple use.

To create a cluster, please Follow [CREATE OR DESTROY](#create-or-destroy) to create the cluster.

`kops get cluster`

```
$ kops get cluster
NAME		CLOUD	ZONES
answ.k8s.local	aws	ap-southeast-1a,ap-southeast-1b
```

you need to get the kubectl config in order to use some of the kops commands (kubectl is used to manage kubernetes but used to validate some kops commands against the cluster)

to get your config from the state run

`kops export kubecfg answ.k8s.local`

to edit your global cluster config run

`kops edit cluster answ.k8s.local`

that command will open your default editor with a kubernetes like resource defining your cluster config (basic kubernetes api settings, etcd instances etc.) documentation on cluster config can be found [here](https://github.com/kubernetes/kops/blob/master/docs/cluster_spec.md)

EDITING CLUSTER RESOURCES
-------------------------

in order to edit or create new node pools you can use kops edit capabilities

`kops edit ig --name=answ.k8s.local nodes`

there you can control the labels of the nodes being created, the zones they will be created on and instance type

after you edited any resource you need to apply the changes to the cluster.

`kops update cluster answ.k8s.local`

that will be a dry run (no changes apply) so you can verify what will actually happen, after your confident in your changes you can apply them with

`kops update cluster answ.k8s.local --yes`

some changes will require instances to be restarted or recreated and that can be done via

`kops rolling-update cluster --yes`

as with update you can remove the `--yes` to do a dry run first.



KOPS deploy every server using instance groups, this means you can have diferent pools of servers with diferent server types depending on your needs.

you can get the list of current deployed IG's by doing

`kops get ig --name=answ.k8s.local`

```
$ kops get ig --name=answ.k8s.local
NAME				ROLE	MACHINETYPE	MIN	MAX	ZONES
master-ap-southeast-1a-1	Master	m3.medium	1	1	ap-southeast-1a
master-ap-southeast-1a-2	Master	m3.medium	1	1	ap-southeast-1a
master-ap-southeast-1b-1	Master	m3.medium	1	1	ap-southeast-1b
nodes				Node	t2.micro	2	2	ap-southeast-1a,ap-southeast-1b
```

and edit them by

`kops edit ig --name=answ.k8s.local master-ap-southeast-1a-1`


CREATE OR DESTROY
-----------------

kops requires programatic access to aws, and programatic iam account with this permissions will be enought

AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess

you can create this user by using the aws cli (**Optional:** You can login to amazon console to create IAM and s3 bucket manually)

First install [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-linux.html), then run the configuration:

```
$ aws configure
AWS Access Key ID [None]: <input your key>
AWS Secret Access Key [None]: <input your key>
Default region name [None]:
Default output format [None]:
```

Create group and user:
```
aws iam create-group --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops

aws iam create-user --user-name kops
aws iam add-user-to-group --user-name kops --group-name kops
```

#### `Create Access Key`
```
aws iam create-access-key --user-name kops
```
#### `Output`
**`Please save the SecretAccessKey and AccessKeyId`**
```
{
    "AccessKey": {
        "UserName": "kops",
        "Status": "Active",
        "CreateDate": "2018-01-11T11:11:11.111Z",
        "SecretAccessKey": "<SecretAccessKey String>",
        "AccessKeyId": "<AccessKeyId String>"
    }
}
```


Create an s3 bucket is also necesary to store cluster state

```
aws s3api create-bucket --bucket answ-k8s-cluster-state --region us-east-1
aws s3api put-bucket-versioning --bucket answ-k8s-cluster-state  --versioning-configuration Status=Enabled
```

`note that both of the above operation will error out if the resources already exist`

#### Create cluster

Configuration: [use the key created for the created user](#output)
```
export AWS_ACCESS_KEY_ID=<AccessKeyId String>
export AWS_SECRET_ACCESS_KEY=<SecretAccessKey String>
export NAME=answ.k8s.local
export KOPS_STATE_STORE=s3://answ-k8s-cluster-state
export ZONES=ap-southeast-1a,ap-southeast-1b
```
Create cluster:
```
kops create cluster \
    --zones $ZONES \
    --master-zones $ZONES \
    --master-count 3 \
    --master-volume-size 100 \
    --networking weave \
    --node-count 2 \
    --node-size t2.micro \
    --node-volume-size 250 \
    ${NAME}
```

you can validate your cluster with

`kops validate cluster --name answ.k8s.local`


#### Destroy Cluster
To destroy the current cluster (delete all resources managed by kops) you can simply do

`kops delete cluster ${NAME} --yes`

remember that remove `--yes` will do dry runs.


KUBERNETES
----------


you now can use your kubernetes cluster.

`kubectl get nodes`

```
$ kubectl get nodes
NAME                                               STATUS    ROLES     AGE       VERSION
ip-172-20-39-138.ap-southeast-1.compute.internal   Ready     master    1h        v1.8.4
ip-172-20-46-237.ap-southeast-1.compute.internal   Ready     node      57m       v1.8.4
ip-172-20-46-58.ap-southeast-1.compute.internal    Ready     master    1h        v1.8.4
ip-172-20-69-110.ap-southeast-1.compute.internal   Ready     master    1h        v1.8.4
ip-172-20-83-25.ap-southeast-1.compute.internal    Ready     node      51m       v1.8.4
```


and see the system pods running

`kubectl -n kube-system get pods`

```
$ kubectl -n kube-system get pods
NAME                                                                       READY     STATUS    RESTARTS   AGE
dns-controller-5cbcd846f9-8lns2                                            1/1       Running   0          1h
etcd-server-events-ip-172-20-39-138.ap-southeast-1.compute.internal        1/1       Running   0          1h
etcd-server-events-ip-172-20-46-58.ap-southeast-1.compute.internal         1/1       Running   0          1h
etcd-server-events-ip-172-20-69-110.ap-southeast-1.compute.internal        1/1       Running   0          1h
etcd-server-ip-172-20-39-138.ap-southeast-1.compute.internal               1/1       Running   0          1h
etcd-server-ip-172-20-46-58.ap-southeast-1.compute.internal                1/1       Running   0          1h
etcd-server-ip-172-20-69-110.ap-southeast-1.compute.internal               1/1       Running   0          1h
kube-apiserver-ip-172-20-39-138.ap-southeast-1.compute.internal            1/1       Running   2          1h
kube-apiserver-ip-172-20-46-58.ap-southeast-1.compute.internal             1/1       Running   1          1h
kube-apiserver-ip-172-20-69-110.ap-southeast-1.compute.internal            1/1       Running   0          1h
kube-controller-manager-ip-172-20-39-138.ap-southeast-1.compute.internal   1/1       Running   0          1h
kube-controller-manager-ip-172-20-46-58.ap-southeast-1.compute.internal    1/1       Running   0          1h
kube-controller-manager-ip-172-20-69-110.ap-southeast-1.compute.internal   1/1       Running   0          1h
kube-dns-7f56f9f8c7-8f27t                                                  3/3       Running   0          57m
kube-dns-7f56f9f8c7-j5h7b                                                  3/3       Running   0          57m
kube-dns-autoscaler-f4c47db64-vkjtj                                        1/1       Running   0          57m
kube-proxy-ip-172-20-39-138.ap-southeast-1.compute.internal                1/1       Running   0          1h
kube-proxy-ip-172-20-46-237.ap-southeast-1.compute.internal                1/1       Running   0          57m
kube-proxy-ip-172-20-46-58.ap-southeast-1.compute.internal                 1/1       Running   0          1h
kube-proxy-ip-172-20-69-110.ap-southeast-1.compute.internal                1/1       Running   0          1h
kube-proxy-ip-172-20-83-25.ap-southeast-1.compute.internal                 1/1       Running   0          51m
kube-scheduler-ip-172-20-39-138.ap-southeast-1.compute.internal            1/1       Running   0          1h
kube-scheduler-ip-172-20-46-58.ap-southeast-1.compute.internal             1/1       Running   0          1h
kube-scheduler-ip-172-20-69-110.ap-southeast-1.compute.internal            1/1       Running   0          1h
weave-net-6bmfq                                                            2/2       Running   0          57m
weave-net-6h6wb                                                            2/2       Running   0          1h
weave-net-7bxst                                                            2/2       Running   0          1h
weave-net-rdkrq                                                            2/2       Running   0          1h
weave-net-x6cfx                                                            2/2       Running   0          51m
```
