# Manage Secrets in Kubernetes

1. We already have a secret key file **official.txt** under **/opt** location on **jump host**. Create a **generic secret** named **official**, it should contain the password/license-number present in **official.txt** file.

2. Also create a **pod** named **secret-xfusion**.

3. Configure pod's **spec** as container name should be **secret-container-xfusion**, image should be **fedora** with **latest** tag (remember to mention the tag with image). Use **sleep** command for container so that it remains in running state. Consume the created secret and mount it under **/opt/cluster** within the container.

4. To verify you can exec into the container **secret-container-xfusion**, to check the secret key under the mounted path **/opt/cluster**. Before hitting the **Check** button please make sure pod/pods are in running state, also validation can take some time to complete so keep patience.
---

<details>
<summary>steps</summary>

  #### cat /opt/official.txt 
    5ecur3

  #### kubectl create secret generic official --from-file=/opt/official.txt
    secret/official created

  #### create a pod template
  ```bash
  kubectl run secret-xfusion --image fedora:latest --dry-run=client -oyaml  > pod.yaml
  ```
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: null
    labels:
      run: secret-xfusion
    name: secret-xfusion
  spec:
    volumes:
    - name: sec-vol
      secret:
        secretName: official
    containers:
    - image: fedora:latest
      name: secret-container-xfusion
      command: ["/bin/bash", "-c", "sleep infinity"]
      resources: {}
      volumeMounts:
      - name: sec-vol
        mountPath: "/opt/cluster"
    dnsPolicy: ClusterFirst
    restartPolicy: Always
  ```

  #### kubectl apply -f pod.yaml 
    pod/secret-xfusion created

  #### kubectl get po
    NAME             READY   STATUS    RESTARTS   AGE
    secret-xfusion   1/1     Running   0          19s

  #### verify 
  ```bash
  kubectl exec secret-xfusion -- ls /opt/cluster
  # official.txt

  kubectl exec secret-xfusion -- cat /opt/cluster/official.txt
  # 5ecur3
  ```
</details>