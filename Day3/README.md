### How Kubernetes allots a single IP address to a group of containers in a Pod
The pause container provides the Network stack and all other containers in the same Pod shares the network stack
of the pause container.
```
docker run -d --name pause --hostname pause k8s.gcr.io/pause:3.5
docker run -dit --name c1 --network=container:pause ubuntu:20.04 /bin/bash
```

If you find the IP Address of the pause container, it would have acquired an IP Address
```
docker inspect pause | grep IPA
```

Now let's check the IP address of c1 container
```
docker exec -it c1 bash
hostname -i
```
Interestingly, 'pause' container and 'c1' containers both shares the same IP.  This is the technique that is used
in K8s Pods.

### Storing Docker Login Credentials as Secrets in Kubernetes

First we need to perform a docker login 
```
su -
docker login
```
When it prompts for your docker user and password, type the same.  Your docker credential will be stored
in /root/.docker/config.json

We need to create Kubernetes secrets to store the docker credentials with K8s cluster securely. This secret will then
be used to pull the Docker images.

```
kubectl create secret generic regcred \
   --from-file=.dockerconfigjson=/root/.docker/config.json \
   --type=kubernetes.io/dockerconfigjson
```

You may verify if the secret is created
```
kubectl get secrets
```
The expected output is
<pre>
[root@master ~]# <b>kubectl create secret generic regcred \
> --from-file=.dockerconfigjson=/root/.docker/config.json \
> --type=kubernetes.io/dockerconfigjson</b>
secret/regcred created
[root@master ~]# kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-sft5f   kubernetes.io/service-account-token   3      135m
<b>regcred               kubernetes.io/dockerconfigjson        1      8s</b>
</pre>

### Using the regcred K8s secret to pull docker images
Let navigate to /home/rps/Training in master node i.e wherever you generally run the kubectl commands.
```
cd /home/rps
mkdir Training
cd Training
git clone https://github.com/tektutor/openshift-sep-2021.git
cd openshift-sep-2021
cd Day3/
touch mypod.yml
```
Make sure the below content is pasted in the mypod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: docker-hub
spec:
  containers:
  - name: docker-hub
    image: nginx:1.20
  imagePullSecrets:
  - name: regcred
```
You may create the pod using the mypod.yml manifest file
```
kubectl apply -f mypod.yml
```

Now you may verify if k8s is able to use the docker credentials to pull the docker hub images
```
kubectl get po
```
The expected output is
<pre>
[root@master ~]# kubectl get po 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-56f64654d5-9hnsz   1/1     Running   0          5m6s
nginx-56f64654d5-ls49b   1/1     Running   0          5m6s
nginx-56f64654d5-sf9ss   1/1     Running   0          4m12s
nginx-56f64654d5-tv5jr   1/1     Running   0          4m15s
</pre>
The number of pods you see will depend on how many pods to scaled up.


### Creating a nginx deployment
```
kubectl create deploy nginx --image=nginx:1.20
```
### List nginx deploy,replicaset and pods
```
kubectl get deploy,rs,po
```

### Scale up nginx deployment
```
kubectl scale deploy/nginx --replicas=4
```

### List po
```
kubectl get po
```


### Creating a NodePort external service for deployment nginx
```
kubectl expose deploy nginx --type=NodePort --port=80
```

### Finding more details about the nginx service
```
kubectl describe svc nginx
```
The expected output is
<pre>
[root@master ~]# <b>kubectl describe svc/nginx</b>
Name:                     nginx
Namespace:                default
Labels:                   app=nginx
Annotations:              <none>
Selector:                 app=nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.105.85.242
IPs:                      10.105.85.242
Port:                   <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                <unset> <b>31720/TCP</b>
Endpoints:                192.168.189.74:80,192.168.235.141:80,192.168.235.142:80 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
</pre>

### Accessing the NodePort service
#### Approach 1
curl http://NodeIP:NodePort
curl http://NodeHostname:NodePort
```
curl http://master:31720
curl http://worker1:31720
curl http://worker2:31720
```

#### Approach 2
curl http://service-name:service-port
```
curl http://nginx:80
```
