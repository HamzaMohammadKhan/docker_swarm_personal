# docker_swarm_personal

Deployed docker swarm on our compaines QA env for EKYC 

Docker R1 QA EKYC

1) First, check if there is a data directory of high storage inside the env, if not contact data center team to create one

2) Stop the docker service using systemctl
systemctl stop docker.service

3) Add the files in the data storage either by using cp -a or rsync -aP, the difference would if you do cp -a, which is safe but sometimes if there symlinks they get broken so it is better to use rsync -aP, the -a preserves and copies all files links etc and -P shows the progress 

rsync -aP /var/lib/docker/ /data/docker

4) Now add the path of data-root inside the daemon.json file in /etc/docker, if not added then create one

mkdir /etc/docker 
cd /etc/docker/
touch daemon.json
vim daemon.json

After performing these actions save the file and start docker.service 
systemctl start docker.service

5) DISCLIMAR 
if you face issue of when creating registry and this error shows up 
cqbt4u6i40dx4nxie9ueb7dk0    \_ registry.1   registry:2   cpekyc360int.bankalhabibuat.com   Shutdown        Failed 2 seconds ago      "starting container failed: error evaluating symlinks from mount source "/data/docker/volumes/registry/_data": lstat /data/docker/volumes/registry/_data: no such file or directory"
( by using command docker service ps registry --no-trunc ) 
this  appears when you move the var/lib data to /data/docker folder, and if volume is already created, there will be traces of the old volume, so when you create a new one delete the old one and then create. 

Following command is for docker registry on port 5000 with /data/docker being the main volume folder
docker service create \
  --name registry \
  --publish published=5000,target=5000 \
  --restart-condition any \
  --mount type=volume,source=registry,target=/var/lib/registry \
  --constraint 'node.role == manager' \
  --env REGISTRY_HTTP_ADDR=0.0.0.0:5000 \
  registry:2
