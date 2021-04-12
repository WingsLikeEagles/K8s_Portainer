# K8s_Portainer
Installing Kubernetes and using Portainer to interact.

Hardware requirements:
BLUF: 16 GB RAM and 8 cores  
This is designed to be run on a single computer or laptop.  In order for this to work with several nodes the computer needs to have better than average resources.  Keep in mind, using VM's you only want to utilize about half of the system resources to leave resources for the host OS and hyervisor to work smoothly.  I recommend the following:  
- RAM
  - In this design each node will consume 2 GB.  If you have a Master, and 2 worker nodes that is 6 GB.
  - 2 GB will be needed for the Portainer and local registry.
  - Total: 16 GB (8 GB for nodes and resources, double this to get host resource requirement of 16 GB)
- CPU
  - Not sure here, but I would assume at least one (1) core for each node and one (1) for Portainer and registry containers.
  - Total: 8 cores (4 cores for the nodes and containers, doubled for matching host resources)

## Install VirtualBox
The nodes will each run as a separate VM on the host.  VirtualBox (or VMWare Workstaion) is used to provide this capability.  

## Download and install the base Linux OS for the nodes
I am experimenting with CentOS Core.  Or CentOS Atomic.  
https://www.projectatomic.io/docs/kubernetes/  
https://wiki.centos.org/SpecialInterestGroup/Atomic/ContainerizedMaster  

## Install Docker, local Registry, and Portainer-CE
- Install Docker
  -  For Windows be sure to install the Linux subsystem (WSL)  
  -  For Linux, you can follow my steps for CentOS 7 here https://github.com/WingsLikeEagles/Docker_Portainer_setup
- Install a local docker registry  
  - `docker pull registry:2.7.1`  
  - `docker volume create registry_data`  
  - `docker run -d -p 5000:5000 --restart always -v registry_data:/var/lib/registry --name registry registry:2.7.1`  
- Install Portainer-CE  
  - `docker pull portainer/portainer-ce:2.1.1`  
  - `docker tag portainer/portainer-ce:2.1.1 localhost:5000/portainer-ce:2.1.1`  
  - `docker push localhost:5000/portainer-ce:2.1.1`  
  - `docker volume create portainer_data`  
  - Portainer needs to be able to access the Docker engine socket to control and monitor the containers:  
    - `docker run -d -p 9000:9000 --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data localhost:5000/portainer-ce:2.1.1`  
    - May need to expose port 8000... more on that later...
  - Login to the new local Portainer-CE web site:  
  - http://localhost:9000/  
  - Create the admin account and password.
  - Click on the left button to confirm you have linked the /var/run/docker.sock

## More to come!
