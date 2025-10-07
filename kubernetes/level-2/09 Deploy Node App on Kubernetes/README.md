# Deploy Node App on Kubernetes

1. Create a deployment using **gcr.io/kodekloud/centos-ssh-enabled:node image**, replica count must be **2**.

2. Create a service to expose this app, the service type must be **NodePort**, targetPort must be **8080** and nodePort should be **30012**.

3. Make sure all the pods are in **Running** state after the deployment.

4. You can check the application by clicking on **NodeApp** button on top bar.

> You can use any labels as per your choice.


```bash
kubectl create deployment node-app --image="gcr.io/kodekloud/centos-ssh-enabled:node" --replicas=2 --port=8080

kubectl expose deployment node-app --type NodePort --port 8080 --target-port 8080

# change the NodePort port
kubectl edit svc node-app

kubectl port-forward svc/node-app 8080:8080 &

curl localhost:8080
```

<details>
<summary>outputs</summary>

  #### kubectl create deployment node-app --image="gcr.io/kodekloud/centos-ssh-enabled:node" --replicas=2 --port=8080
    deployment.apps/node-app created

  #### kubectl expose deployment node-app --type NodePort --port 8080 --target-port 8080
    service/node-app exposed

  #### kubectl edit svc node-app
    ports:
    - nodePort: 32102
    ---
    ports:
    - nodePort: 30012
    ======
    service/node-app edited

  #### kubectl port-forward svc/node-app 8080:8080 &
    [1] 3530
    Forwarding from [::1]:8080 -> 8080

  #### curl localhost:8080
    Handling connection for 8080
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <title>About Sharks</title>
</details>