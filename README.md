# install-kubernetes
Personal repo for kubernetes deployment on bare metal or VM cloud-init after I have forgotten how to do it again :-)

### ALL Credit go to Olorunfemi Kawonise @ https://bit.ly/3PAgkHp which gave me the first success in deploying kubernetes from kubeadm - Thanks for the help Olorunfemi, this script is a slight tweak on that one to migrate to the new repo. I copied a few steps verbatim too for my own reminder in case the medium page vanishes in future.

#### First become root on a fresh ubuntu master node with git installed
```
sudo -i
```
git clone https://github.com/PsyMan2000/install-kubernetes
```
#### run the script to install Kubernetes (change for version first - eg future 1.30 etc)
```
sh install-kubernetes/install_kubernetes.sh
```

#### Initialize the control plane in the master node as the root user
```
kubeadm init
```

#### Set up kubeconfig as normal ubuntu user

The `kubeconfig` file is a central configuration file that allows you to authenticate, interact with, and manage Kubernetes clusters effectively. It encapsulates authentication credentials, context information, cluster details, and more, making it a critical component for anyone working with Kubernetes clusters. (I actually did this on a non node member linux machine too, my workhorse with ansible, terraform, etc)


#### exit root to your normal user  
 
```
mkdir -p $HOME/.kube
```
``` 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
```
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config  
```
#### to verify, if kubectl is working or not, run the following command.  
```
kubectl get pod -A
```

#### Deploy the network plugin (weave network)
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml  
```
#### verify if weave is deployed successfully  
```
kubectl get pods -A
```
#### Create the master join token on the master node - RUN AS ROOT
```
kubeadm token create — print-join-command
```
#### Join worker node to Kubernetes cluster

If you wish to add worker nodes to your cluster, perform Step 1 to Step 4 on your worker node. Then still on the worker node execute the join command generated on the master node to join the cluster.

``` 
#example. do not use this , use the output from the last section

kubeadm join 172.30.20.20:6443 — token cdm6fo.dhbrxyleqe5suy6e \  
 — discovery-token-ca-cert-hash sha256:1fc51686afd16c46102c018acb71ef9537c1226e331840e7d401630b96298e7d
```

_Note: Do not execute_ `_kubeadm init_` _on worker node as the machine is not a master node that needs a control plane._


