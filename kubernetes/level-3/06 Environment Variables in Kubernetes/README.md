# Environment Variables in Kubernetes

1. Create a **pod** named **envars**.

2. Container name should be **fieldref-container**, use image **redis** preferable **latest** tag, use command **`'sh', '-c'`** and args should be
```bash
'while true; do
      echo -en '/n';
                                  printenv NODE_NAME POD_NAME;
                                  printenv POD_IP POD_SERVICE_ACCOUNT;
              sleep 10;
         done;'
```
>(Note: please take care of indentations)

3. Define **Four** environment variables as mentioned below:
  - The first **env** should be named as **NODE_NAME**, set valueFrom fieldref and fieldPath should be **spec.nodeName**.
  - The second **env** should be named as **POD_NAME**, set valueFrom fieldref and fieldPath should be **metadata.name**.
  - The third **env** should be named as **POD_IP**, set valueFrom fieldref and fieldPath should be **status.podIP**.
  - The fourth **env** should be named as **POD_SERVICE_ACCOUNT**, set valueFrom fieldref and fieldPath shoulbe be **spec.serviceAccountName**.

4. Set restart policy to **Never**.

5. To check the output, exec into the pod and use **printenv** command.
---
[Doc - Use Pod fields as values for environment variables 
](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/#use-pod-fields-as-values-for-environment-variables)

<details>
<summary>steps</summary>

  #### create a pod template
  ```bash
  kubectl run envars --image redis:latest --dry-run=client -oyaml > envars.yaml
  ```
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: envars
  spec:
    containers:
    - name: fieldref-container
      image: redis:latest
      command: ["sh", "-c"]
      args:
        - |
          while true; do
            echo -en '\n';
            printenv NODE_NAME POD_NAME;
            printenv POD_IP POD_SERVICE_ACCOUNT;
            sleep 10;
          done;
      env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
    restartPolicy: Never
  ```

  #### kubectl apply -f envars.yaml 
    pod/envars created

  #### kubectl get po
    NAME     READY   STATUS    RESTARTS   AGE
    envars   1/1     Running   0          14s

  #### kubectl exec envars -- printenv | grep -E "NODE_NAME|POD_NAME|POD_IP|POD_SERVICE_ACCOUNT"
    NODE_NAME=kodekloud-control-plane
    POD_NAME=envars
    POD_IP=10.244.0.5
    POD_SERVICE_ACCOUNT=default
</details>