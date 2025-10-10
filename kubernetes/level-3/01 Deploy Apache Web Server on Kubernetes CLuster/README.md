# Deploy Apache Web Server on Kubernetes CLuster

1. Create a **namespace** named as **httpd-namespace-xfusion**.

2. Create a **deployment** named as **httpd-deployment-xfusion** under newly created namespace. For the deployment use **httpd** image with **latest** tag only and remember to mention the tag i.e **httpd:latest**, and make sure replica counts are **2**.

3. Create a **service** named as **httpd-service-xfusion** under same namespace to expose the deployment, nodePort should be **30004**.

<details>
<summary>details</summary>

  #### kubectl create ns httpd-namespace-xfusion
    namespace/httpd-namespace-xfusion created

  #### kubectl create deployment httpd-deployment-xfusion --image httpd:latest --replicas=2 -n  httpd-namespace-xfusion
    deployment.apps/httpd-deployment-xfusion created

  #### kubectl -n httpd-namespace-xfusion expose deployment httpd-deployment-xfusion --name httpd-service-xfusion --type NodePort --port 80 --target-port 80
    service/httpd-service-xfusion exposed

  #### kubectl -n httpd-namespace-xfusion edit svc httpd-service-xfusion
    ports:
    - nodePort: 30285
    ---
    ports:
    - nodePort: 30004
    =====
    service/httpd-service-xfusion edited

  #### kubectl -n httpd-namespace-xfusion port-forward services/httpd-service-xfusion 8080:80 &
    [1] 4336
    Forwarding from [::1]:8080 -> 80

  #### curl localhost:8080
    Handling connection for 8080
    <html><body><h1>It works!</h1></body></html>
</details>