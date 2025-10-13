# Persistent Volumes in Kubernetes

1. Create a **PersistentVolume** named as **pv-xfusion**. Configure the **spec** as storage class should be **manual**, set capacity to **4Gi**, set access mode to **ReadWriteOnce**, volume type should be **hostPath** and set path to **/mnt/devops** (this directory is already created, you might not be able to access it directly, so you need not to worry about it).

2. Create a **PersistentVolumeClaim** named as **pvc-xfusion**. Configure the **spec** as storage class should be **manual**, request **3Gi** of the storage, set access mode to **ReadWriteOnce**.

3. Create a **pod** named as **pod-xfusion**, mount the persistent volume you created with claim name **pvc-xfusion** at document root of the web server, the container within the pod should be named as **container-xfusion** using image **nginx** with **latest** tag only (remember to mention the tag i.e **nginx:latest**).

4. Create a node port type service named **web-xfusion** using node port **30008** to expose the web server running within the pod.

[Doc - Pod to use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

<details>
<summary>nginx.yaml</summary>

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-xfusion
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/devops"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-xfusion
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-xfusion
  labels:
    app: web
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: pvc-xfusion
  containers:
    - name: container-xfusion
      image: nginx:latest
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```
</details>

<details>
<summary>steps</summary>

  #### kubectl apply -f nginx.yaml 
    persistentvolume/pv-xfusion created
    persistentvolumeclaim/pvc-xfusion created
    pod/pod-xfusion created

  #### kubectl get pv,pvc
    NAME                          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
    persistentvolume/pv-xfusion   4Gi        RWO            Retain           Bound    default/pvc-xfusion   manual                  14s

    NAME                                STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    persistentvolumeclaim/pvc-xfusion   Bound    pv-xfusion   4Gi        RWO            manual         14s

  #### kubectl get po
    NAME          READY   STATUS    RESTARTS   AGE
    pod-xfusion   1/1     Running   0          29s

  #### kubectl expose pod pod-xfusion --name web-xfusion --type NodePort --port 80 --target-port 80
    service/web-xfusion exposed

  #### kubectl edit svc web-xfusion 
    ports:
    - nodePort: 31110
    ---
    ports:
    - nodePort: 30008
    =====
    service/web-xfusion edited

  #### kubectl port-forward svc/web-xfusion 8080:80 &
    [1] 4738
    Forwarding from [::1]:8080 -> 80

  #### curl localhost:8080
    Handling connection for 8080
    <html>
    <head><title>403 Forbidden</title></head>
    <body>
    <center><h1>403 Forbidden</h1></center>
    <hr><center>nginx/1.29.2</center>
    </body>
    </html>

  #### kubectl exec pod-xfusion -- sh -c "echo 'Hello World' > /usr/share/nginx/html/index.html"

  #### curl localhost:8080
    Handling connection for 8080
    Hello World
</details>