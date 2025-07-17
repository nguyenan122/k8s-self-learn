

### Export by Docker
```bash
# Save image from docker
docker image save docker.io/library/java-less:v1 > java-less-docker.tar
```
### Export by CTR
```bash
# Save image from ctr
ctr -n=k8s.io image export java-less-ctr.tar docker.io/library/java-less:v1

# tranfer tar file to worker node
scp java-less.tar ${IP_OF_WORKER_NODE}
```


### Import to Containerd/k8s
```bash
# Import local images vào k8s
ssh ${IP_OF_WORKER_NODE}
ctr -n=k8s.io images ls
ctr -n=k8s.io images import java-less.tar
ctr -n=k8s.io images ls | grep java
#(Kết quả):  docker.io/library/java-less:v1
```
### Import to Docker
```bash
docker load --input java-less-ctr.tar
#5f70bf18a086: Loading layer [==================================================>]  1.024kB/1.024kB
#Loaded image: java-less:v1


docker image ls
#REPOSITORY   TAG       IMAGE ID       CREATED             SIZE
#java-less    v1        417e708c3197   About an hour ago   226MB


```