### Installing Nginx based Ingress Controller on our 3 Node K8s Cluster

#### Install ingress controller (Master Node only)
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.1/deploy/static/provider/baremetal/deploy.yaml
```

#### Verify the installation
```
kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --watch
```

#### Finding the version of Ingress Controller installed
```
POD_NAMESPACE=ingress-nginx
POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app.kubernetes.io/name=ingress-nginx --field-selector=status.phase=Running -o jsonpath='{.items[0].metadata.name}')

kubectl exec -it $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version
```
The expected output is
<pre>
[root@master openshift-sep-2021]# <b>kubectl exec -it $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version</b>
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v1.0.1
  Build:         abab0396757dcd6f72018ee66611db18df838b17
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.9
-------------------------------------------------------------------------------
</pre>