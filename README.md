# K8s_Portainer
Installing Kubernetes and using Portainer to interact.

## Windows host based setup
- Install Docker for Windows with the Linux subsystem (WSL)  
- Install a local docker registry  
  - `docker pull registry:2.7.1`  
  - `docker volume create registry_data`  
  - `docker run -d -p 5000:5000 --restart always -v registry_data:/var/lib/registry --name registry registry:2.7.1`  
- Install Portainer-CE  
  - `docker pull portainer/portainer-ce:2.1.1`  
  - `docker tag portainer/portainer-ce:2.1.1 localhost:5000/portainer-ce:2.1.1`  
  - `docker push localhost:5000/portainer-ce:2.1.1`  

## More to come!
