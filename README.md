# K8s_Portainer
Installing Kubernetes and using Portainer to interact.

Hardware requirements:
BLUF: 16 GB RAM and 8 cores  
This is designed to be run on a single computer or laptop.  In order for this to work with several nodes the computer needs to have better than average resources.  Keep in mind, using VM's you only want to utilize about half of the system resources to leave resources for the host OS and hypervisor to work smoothly.  I recommend the following:  
- RAM
  - In this design each node will consume 2 GB.  If you have a Master, and 2 worker nodes that is 6 GB.
  - 2 GB will be needed for the Portainer and local registry.
  - Total: 16 GB (8 GB for nodes and resources, double this to get host resource requirement of 16 GB)
- CPU
  - Not sure here, but I would assume at least one (1) core for each node and one (1) for Portainer and registry containers.
  - Total: 8 cores (4 cores for the nodes and containers, doubled for matching host resources)
- Disk Space
  - Undefined at this point... count on at least 100 GB (maybe less? More?)
  - Depends on what services you intend to run and how big their containers are.
- For an Air Gapped environment:
  - You will want a VM to run as a local registry for the docker containers for the Kubernetes nodes.
  - Requires at least 512 MB of RAM and one (1) CPU core (probably small enough to not impact system)
  - Only required during installation or updates (can be shut down after)???

## Install VirtualBox
The nodes will each run as a separate VM on the host.  VirtualBox (or VMWare Workstation) is used to provide this capability.  

## Download and install the base Linux OS for the nodes
I am experimenting with CentOS Core.  Or CentOS Atomic.  
https://www.projectatomic.io/docs/kubernetes/  
https://wiki.centos.org/SpecialInterestGroup/Atomic/ContainerizedMaster  

## Install Docker, local Registry, and Portainer-CE
- Install Docker
  -  For Windows be sure to install the Linux subsystem (WSL)  
  -  For Linux, you can follow my steps for CentOS 7 here https://github.com/WingsLikeEagles/Docker_Portainer_setup
- Spin up the Registry VM and Install a local docker registry (2 GB RAM, create an Internal Network)
  - Using the CentOS Core ISO file, create a new VM
    - This VM will have an outward facing Network interface and an internal one
      - This is not required if you manually copy the containers to the VM
      - Add an additional NIC for Internet access to download containers
  - `docker pull registry:2.7.1`  
  - `docker volume create registry_data`  
  - `docker run -d -p 5000:5000 --restart always -v registry_data:/var/lib/registry --name registry registry:2.7.1`
  - Edit the /etc/hosts file  
    - add `192.168.123.101 reg reg.local` use your appropriate IP for your Local registry VM (this one we just created)
  - Add Insecure Registries setting to Docker Engine config (this allows use without HTTPS since is is only local)
    - Edit `/etc/docker/daemon.json` to add the following:
    - `{`
    - `  "insecure-registries" : ["reg.local:5000"]`
    - `}`
    - Restart the Docker Engine service `systemctl restart docker`
- Install Portainer-CE  
  - `docker pull portainer/portainer-ce:2.1.1`  
  - `docker tag portainer/portainer-ce:2.1.1 reg.local:5000/portainer-ce:2.1.1`  
  - `docker push reg.local:5000/portainer-ce:2.1.1`  
  - `docker volume create portainer_data`  
  - Portainer needs to be able to access the Docker socket to control and monitor the containers (maybe some day they will implement proper security, for now, selinux on CentOS and RHEL require the use of --privileged):  
    - `docker run -d -p 9000:9000 --privileged --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data localhost:5000/portainer-ce:2.1.1`  
    - May need to expose port 8000... more on that later...
  - Login to the new local Portainer-CE web site:  
  - http://localhost:9000/  
  - Create the admin account and password.
  - Click on the left button to confirm you have linked the /var/run/docker.sock

## Pull the required containers to your local registry
- CentOS Atomic runs the Kubernetes Master node services as containers.  We need those images in our local registry!
  - Based on https://wiki.centos.org/SpecialInterestGroup/Atomic/ContainerizedMaster  
- Kubernetes API Server
  - `docker pull registry.centos.org/centos/kubernetes-apiserver`
  - `docker tag registry.centos.org/centos/kubernetes-apiserver reg.local:5000/centos/kubernetes-apiserver:latest`
  - `docker push reg.local:5000/centos/kubernetes-apiserver:latest`
- Kubernetes Controller Master
  - `docker pull registry.centos.org/centos/kubernetes-controller-manager`
  - `docker tag registry.centos.org/centos/kubernetes-controller-manager reg.local:5000/centos/kubernetes-controller-manager:latest`
  - `docker push reg.local:5000/centos/kubernetes-controller-manager:latest`
- Kubernetes Scheduler
  - `docker pull registry.centos.org/centos/kubernetes-scheduler`
  - `docker tag registry.centos.org/centos/kubernetes-scheduler reg.local:5000/centos/kubernetes-scheduler:latest`
  - `docker push reg.local:5000/centos/kubernetes-scheduler:latest`

## Master Node
- Create a New VM in VirtualBox using the CentOS Core ISO downloaded earlier (2 GB RAM, Internal Network same as Reg.local)
- Edit /etc/hosts file adding `192.168.123.101 reg reg.local` (use IP address from Internal NIC on Local Registry VM created earlier)
- Add Insecure Registries setting to Docker Engine config (this allows use without HTTPS since is is only local)
  - Edit `/etc/docker/daemon.json` to add the following:
  - <code>{<br/>
  "insecure-registries" : ["reg.local:5000"]<br/>
}</code>
  - Restart the Docker Engine service `systemctl restart docker`
- Pull the required containers to your Master Node from your local Registry VM
  - `docker pull reg.local:5000/centos/kubernetes-apiserver:latest`
  - `docker pull reg.local:5000/centos/kubernetes-controller-manager:latest`
  - `docker pull reg.local:5000/centos/kubernetes-scheduler:latest`
- Add Kubernetes Containers as Services to Master Node
  - https://wiki.centos.org/SpecialInterestGroup/Atomic/ContainerizedMaster
- Add Services to /etc/systemd/system/ and enable them (still needs major work, these services assume K8s is installed locally on the host/VM which is not the case here)
  - Clone this repo to the Master VM (or copy these files another way)
    - `git clone https://github.com/WingsLikeEagles/K8s_Portainer/`
    - `cd K8s_Portainer`
  - Copy services files from this repo to /etc/systemd/system/
    - `cp etc/systemd/system/kube-* /etc/systemd/system/`
  - chown and chmod the files
    - `chown root:root /etc/systemd/system/kube-*`
    - `chmod 755 /etc/systemd/system/kube-*`
  - Enable the services to start at boot
    - `systemctl enable kube-apiserver.service`
    - `systemctl enable kube-controller-manager.service`
    - `systemctl enable kube-scheduler.service`
  - Start the services (this may need to happen later)
    - `systemctl start kube-apiserver.service`
    - `systemctl start kube-controller-manager.service`
    - `systemctl start kube-scheduler.service`

- To Be Continued...
