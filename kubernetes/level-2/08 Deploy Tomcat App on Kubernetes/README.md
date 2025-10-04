# Deploy Tomcat App on Kubernetes

1. Create a namespace named **tomcat-namespace-xfusion**.

2. Create a **deployment** for tomcat app which should be named as **tomcat-deployment-xfusion** under the same namespace you created. Replica count should be **1**, the container should be named as **tomcat-container-xfusion**, its image should be **gcr.io/kodekloud/centos-ssh-enabled:tomcat** and its container port should be **8080**

3. Create a **service** for tomcat app which should be named as **tomcat-service-xfusion** under the same namespace you created. Service type should be **NodePort** and nodePort should be **32227**


```bash
kubectl create ns tomcat-namespace-xfusion

kubectl config set-context --current --namespace tomcat-namespace-xfusion

kubectl config get-contexts

kubectl create deployment tomcat-deployment-xfusion --image gcr.io/kodekloud/centos-ssh-enabled:tomcat --replicas=1 --port=8080 --dry-run=client -oyaml > tomcat.yaml

# change the container name
vi tomcat.yaml

kubectl apply -f tomcat.yaml

kubectl expose deployment tomcat-deployment-xfusion --name tomcat-service-xfusion --type NodePort --port 8080 --target-port 8080

# change the NodePort port
kubectl edit svc tomcat-service-xfusion

kubectl get all

kubectl port-forward service/tomcat-service-xfusion 8080:8080 &

curl localhost:8080
```

<details>
<summary>tomcat.yaml</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: tomcat-deployment-xfusion
  name: tomcat-deployment-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat-deployment-xfusion
  template:
    metadata:
      labels:
        app: tomcat-deployment-xfusion
    spec:
      containers:
      - image: gcr.io/kodekloud/centos-ssh-enabled:tomcat
        name: tomcat-container-xfusion
        ports:
        - containerPort: 8080
```
</details>

<details>
<summary>outputs</summary>

  #### kubectl create ns tomcat-namespace-xfusion
    namespace/tomcat-namespace-xfusion created

  #### kubectl config set-context --current --namespace tomcat-namespace-xfusion
    Context "kind-kodekloud" modified.

  #### kubectl config get-contexts
    CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
    *         kind-kodekloud   kind-kodekloud   kind-kodekloud   tomcat-namespace-xfusion

  #### kubectl apply -f tomcat.yaml 
    deployment.apps/tomcat-deployment-xfusion created

  #### kubectl expose deployment tomcat-deployment-xfusion --name tomcat-service-xfusion --type NodePort --port 8080 --target-port 8080
    service/tomcat-service-xfusion exposed

  #### kubectl edit svc tomcat-service-xfusion
    ports:
    - nodePort: 31956
    ---
    ports:
    - nodePort: 32227
    ======
    service/tomcat-service-xfusion edited

  #### kubectl get all
    NAME                                             READY   STATUS    RESTARTS   AGE
    pod/tomcat-deployment-xfusion-5f9979b957-kvfl5   1/1     Running   0          2m49s

    NAME                             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    service/tomcat-service-xfusion   NodePort   10.96.13.199   <none>        8080:32227/TCP   109s

    NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/tomcat-deployment-xfusion   1/1     1            1           2m49s

    NAME                                                   DESIRED   CURRENT   READY   AGE
    replicaset.apps/tomcat-deployment-xfusion-5f9979b957   1         1         1       2m49s

  #### kubectl port-forward service/tomcat-service-xfusion 8080:8080 &
    [1] 4525
    Forwarding from [::1]:8080 -> 8080

  #### curl localhost:8080
    Handling connection for 8080
    <!DOCTYPE html>
    <!--
    To change this license header, choose License Headers in Project Properties.
    To change this template file, choose Tools | Templates
    and open the template in the editor.
    -->
    <html>
        <head>
            <title>SampleWebApp</title>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
        </head>
        <body>
            <h2>Welcome to xFusionCorp Industries!</h2>
            <br>
        
        </body>
    </html>
</details>