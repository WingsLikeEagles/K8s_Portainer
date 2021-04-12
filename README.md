# K8s_Portainer
Installing Kubernetes and using Portainer to interact.

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
