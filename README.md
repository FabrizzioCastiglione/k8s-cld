# Cloud with K8S + Docker in Production

El primer paso es tener una Master con requisitos adecuados para poder hacer soportes a los nodos. 


Kubernetes (“k8s”) es un sistema de código libre para la automatización del despliegue, ajuste de escala y manejo de aplicaciones en contenedores que fue originalmente diseñado por Google y donado a la Cloud Native Computing Foundation (parte de la Linux Foundation). Soporta diferentes entornos para la ejecución de contenedores, incluido Docker.

<p align="center">
     <img src='https://user-images.githubusercontent.com/68827543/166177213-30ff57fa-41b8-4410-956a-396c79d22ceb.jpg'>
 </p>

Docker es un proyecto de código abierto que automatiza el despliegue de aplicaciones dentro de contenedores de software, proporcionando una capa adicional de abstracción y automatización de virtualización de aplicaciones en múltiples sistemas operativos. ​Docker utiliza características de aislamiento de recursos del kernel Linux, tales como cgroups y espacios de nombres (namespaces) para permitir que "contenedores" independientes se ejecuten dentro de una sola instancia de Linux, evitando la sobrecarga de iniciar y mantener máquinas virtuales.

<p align="center">
     <img src='https://user-images.githubusercontent.com/68827543/166177804-77d70d43-f375-4019-bc30-d978a29c36d6.jpg'>
 </p>

sudo apt-get update -y  && sudo apt-get install apt-transport-https -y

Change to root user
sudo su -
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

Enable IP tables
#We need to enable IT tables for pod to pod communication.
modprobe br_netfilter
sysctl -p
sudo sysctl net.bridge.bridge-nf-call-iptables=1

Install Docker on both Master and Worker nodes
apt-get install docker.io -y

Add ubuntu user to Docker group
usermod -aG docker ubuntu
systemctl restart docker
systemctl enable docker.service

Type exit to come out of root user.
Install Kubernetes Modules
sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni

sudo systemctl daemon-reload
sudo systemctl start kubelet
sudo systemctl enable kubelet.service
sudo systemctl status docker

Initialize Kubeadm on Master Node(only on Master Node)

#Execute the below command as root user to initialize Kubernetes Master node.
sudo su - 
kubeadm init

#Now type exit to exit from root user and execute below commands as normal user

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config



Installing the Weave Net Add-On
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
It make take a few mins to execute the above command and show show the below message.


Now execute the below command to see the pods.

kubectl get pods  --all-namespaces



Now login to Worker Node

Join worker node to Master Node
The below command will join worker node to master node, execute this a normal user by putting sudo before:


 
sudo kubeadm join <master_node_ip>:6443 --token xrvked.s0n9771cd9x8a9oc \
    --discovery-token-ca-cert-hash sha256:288084720b5aad132787665cb73b9c530763cd1cba10e12574b4e97452137b4a



Go to Master and type the below command
kubectl get nodes
the above command should display both Master and worker nodes.



It means Kubernetes Cluster - both Master and worker nodes are setup successfully and up and running!!!

Deploy Nginx on a Kubernetes Cluster
Let us run some apps to make sure they are deployed to Kuberneter cluster. We will do this in master node. The below command will create deployment:

kubectl create deployment nginx --image=nginx


View Deployments
kubectl get deployments 

Create as a service 

 
kubectl create service nodeport nginx --tcp=80:80

kubectl get svc
run the above command to see a summary of the service and the ports exposed.


Now go Master or worker node, enter public dns and access it with port exposed

You should see the welcome page!

