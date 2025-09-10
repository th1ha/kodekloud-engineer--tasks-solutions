# Deploy Pods in Kubernetes Cluster

### Create a pod named `pod-nginx` using the nginx image with the `latest` tag. Ensure to specify the tag as `nginx:latest`

```bash
kubectl run pod-nginx --image nginx:latest --dry-run=client -oyaml > pod-nginx.yaml
```

### Set the app label to `nginx_app`, and name the `container` as `nginx-container`

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod-nginx
    app: nginx_app
  name: pod-nginx
spec:
  containers:
  - image: nginx:latest
    name: nginx-container
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

### create a pod

```bash
kubectl  apply -f pod-nginx.yaml

kubectl get po
```