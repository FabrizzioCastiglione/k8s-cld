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

# Cloud Montage

Para la contruccion de la Cloud se necesita el despliege de 'KUBERNETES' tanto como el Maestro/Esclavo. Iniciamos instancias en lso nodos y en el master.
```bash
sudo apt install curl
sudo apt-get update -y  && sudo apt-get install apt-transport-https -y
```
Se inicia usuario root y se descargan las llaves de de 'Kubenetes'.
```bash
sudo su -
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
```

Se desabilidta la memoria de swap (intercambio) para mejor rendimiento.
```bash
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Se habilitan la tablas IP para los puertos, asi activamos las tablas IT para la cuminicacion de pod a pod.
We need to enable IT tables for pod to pod communication.
```bash
modprobe br_netfilter
sysctl -p
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

Install Docker on both Master and Worker nodes
```bash
apt-get install docker.io -y
```

Agregamos docker como super usuario y lo reiniciamos.
```bash
usermod -aG docker user
systemctl restart docker
systemctl enable docker.service
```


Instalamos kubeadm para la instalcion de kubernetes. Salimos de super usuario.
```bash
apt-get update && apt-get install -y kubeadm
exit
sudo systemctl daemon-reload
sudo systemctl start kubelet
sudo systemctl enable kubelet.service
sudo systemctl status docker
```

Enfrenté un problema similar recientemente. El problema era el controlador de cgroup. El controlador cgroup de Kubernetes se configuró en sistemas, pero la ventana acoplable se configuró en systemd. Así que creé /etc/docker/daemon.json y agregué a continuación:

```bash
sudo nano /etc/docker/daemon.json

{
    "exec-opts": ["native.cgroupdriver=systemd"]
}

```

Reiniciamos servicios.

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
```

Iniciamos kubeadm en el nodo master.

```bash
sudo su - 
kubeadm init --pod-network-cidr 10.244.0.0/16
```
Despues Kubernetes 

![init](https://user-images.githubusercontent.com/68827543/170141818-1d25beab-bdd4-41ac-b74f-7abb062ca2d6.png)


* Guardar token del nodo master

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=$HOME/admin.conf
```

Instalamos Weave Net Add-On
```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Ingresar el token en los los equipos esclavos para hacer el nodo.

```bash
sudo kubeadm join <master_node_ip>:6443 --token xrvked.s0n9771cd9x8a9oc \
    --discovery-token-ca-cert-hash sha256:288084720b5aad132787665cb73b9c530763cd1cba10e12574b4e97452137b4a
```
Se revisa los nodos conectados en el 'Master'.
```bash
sudo su -
kubectl get nodes
```
Significa Kubernetes Cluster: ¡tanto los nodos maestros como los trabajadores están configurados correctamente y en funcionamiento.

Implementar Nginx en un clúster de Kubernetes Ejecutemos algunas aplicaciones para asegurarnos de que se implementen en el clúster de Kuberneter. Haremos esto en el nodo maestro. El siguiente comando creará la implementación:

```bash
kubectl create deployment nginx --image=nginx
```
Ver implementaciones kubectl obtener implementaciones
```bash
kubectl get deployments
```

![deploy 1](https://user-images.githubusercontent.com/68827543/170144797-035d2cf9-2630-4c9c-9b94-51641fb5044e.png)

Se creara el el servicio de nginx
 ```bash
kubectl create service nodeport nginx --tcp=80:80
```
![expose svc](https://user-images.githubusercontent.com/68827543/170145175-95e11c86-dcd7-4f6e-9949-c2f741e0bbaa.png)

Vemos el resumen de los puertos abiertos.

```bash
kubectl get svc
 ```
 
![get svc](https://user-images.githubusercontent.com/68827543/170145302-c89b140f-4f51-4e5f-8c87-d01ad3210faf.png)

Ahora vaya al nodo maestro o trabajador, ingrese dns públicos y acceda con el puerto expuesto

![access app](https://user-images.githubusercontent.com/68827543/170145512-81f56789-afa6-4b44-9240-05d373778a1c.png)


