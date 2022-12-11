# k3s-cluster creation and deployment of wordpress





## **WHAT IS K3S?** :
 Many of us may wonder that what is k3s because we have heard k8s but something like k3s sounds error with term of k8s. So lets get basic understanding of k3s as:   
- K3s is a lightweight, easy to install, deploy and manage version of stock Kubernetes(k8s).
- It is a certified kubernetes distribution.
- Further more features can be read [HERE](https://docs.k3s.io/)

### Prerequisites 

Now for this deployment I was confused with where to deploy, how to deploy because everywhere it was showing that at least two machines are needed to make a master node and one worker node.

> Two machines that are required atleast : Create two VM in virtual box or anywhere required.

> **Note: To make cluster always set the VM network to Bridge instead of NAT or any other**

> Use **sudo** where you get result of permission denied. For e.g. `kubectl get nodes` if gives result permission denied, use `sudo kubectl get nodes` <br>
> If getting result Username is not in the sudoers file. Then execute `su -` .This will take you in superuser mode, here execute command `adduser your-username-of-machine sudo` . Now logout from shutdown menu and login again.

Now there are many methods to deploy which I found were:
- deploying and configuring manually each machine [Here is the link for your reference](https://itnext.io/setup-your-own-kubernetes-cluster-with-k3s-b527bf48e36a)
- Second I found a way by using Ansible.
- Last was that mentioned in this documentation beginning.

-------------------------------------------------------------

## **So Let's start the cluster deployment.**

_Till this part the virtual box is running with atleast 2 linux VM_


### Setting up Master Node/Control Plane
 1.  Open terminal of one VM which you want to make as master node or control plane and execute following command

> `sudo apt-get install curl`<br>

> `curl -sfL https://get.k3s.io | sh -`

Let everything get installed. It will take few minutes. 

 2. You can check if your master node is ready by executing `kubectl get nodes`

 3. A token is generated which helps in connecting worker node to master node. Access this token by running the following code :
> `cat /var/lib/rancher/k3s/server/node-token`

You will see a token something like this `K173832c8dd5a175bf2123d840ad491850199cbd19309ab3b37827047cdd6319b04::server:faddf0a734d338cd66d4ab19fb4bed73` copy it somewhere. It will be required when we will connect worker node to master node.

### Setting up Worker Node
Move to another VM which you want to set up as worker node and open terminal and follow instructions below:

1. Check for _curl_ in terminal. If curl is not installed execute `sudo apt-get install curl`.<br>
Execute following command to get this machine joined as **worker node**:

> `curl -sfL https://get.k3s.io | K3S_URL=https://ip-address:6443 K3S_TOKEN="token-that-you-have-copied-in-worker-node-in-step3" sh -`.<br>
Ip-address can be found by going to master VM and type command `kubectl get nodes -o wide`. The internal ip of master plane will be used in the above command.

2. You have to execute this same command just above in all VM's which you want to get joined as worker node in Master plane. 

--------------------------------------------------------------------------------------------------------------------

### Deployment of Mysql and Wordpress in the cluster.

You have to perform this in master node terminal.<br>
**1.** Download the MySQL deployment configuration file. 

> ```curl -LO https://k8s.io/examples/application/wordpress/mysql-deployment.yaml```.

- Download the WordPress configuration file by 
> `curl -LO https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml`. 

_Here some of the changes are required to make the deployment._
_Change the Selector type to **NodePort** instead of loadbalancer_ exposed to with url and you can access the wordpress_

- Create a `kustomization.yaml` which has the configuration and password of the mysql <br>[For Reference](https://github.com/sattract/k3s-cluster/blob/main/kustomization.yaml)

> Remember to change password in kustomization.yaml if using reference file.

**2.** After `kustomization.yaml` is created use the code below to deploy these yaml files and you will have the working wordpress on k3s cluster. 
> `kubectl apply -k ./`

You can get Verify that a PersistentVolume got dynamically provisioned by 

> `kubectl get pvc` 

and check mysql by 

>`kubectl get secrets`

**3.** Use the following command to view port no.of wordpress: <br>
> `kubectl get svc`

The port no. should be between 30000-32767.

**4.** To access the Wordpress, go to browser and enter 
> `https://public-ip:port-no`<br>
- The public ip of control plane vm or worker node vm (If you are using cloud for VM then in azure it can be accessed from overview of that VM, for AWS its visible right below the Instance console) 
- Port no is can be found in previous step.
![image](https://user-images.githubusercontent.com/114693937/206908217-67ed4618-b4ad-4dfd-ba9e-18b9924e9857.png)
