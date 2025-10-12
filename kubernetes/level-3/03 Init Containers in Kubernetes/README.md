# Init Containers in Kubernetes

1. Create a **Deployment** named as **ic-deploy-xfusion**.

2. Configure **spec** as replicas should be **1**, labels **app** should be **ic-xfusion**, template's metadata lables **app** should be the same **ic-xfusion**.

3. The **initContainers** should be named as **ic-msg-xfusion**, use image **debian** with **latest** tag and use command **`'/bin/bash'`**, **`'-c'`** and **`'echo Init Done - Welcome to xFusionCorp Industries > /ic/media'`**. The volume mount should be named as **ic-volume-xfusion** and mount path should be **/ic**.

4. Main container should be named as **ic-main-xfusion**, use image debian with latest tag and use command **`'/bin/bash'`**, **`'-c'`** and **`'while true; do cat /ic/media; sleep 5; done'`**. The volume mount should be named as **ic-volume-xfusion** and mount path should be **/ic**.

5. Volume to be named as **ic-volume-xfusion** and it should be an emptyDir type.

[Doc - Init container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use)

<details>
<summary>steps</summary>

  #### kubectl create deployment ic-deploy-xfusion --image debian:latest --dry-run=client -oyaml > ic.yaml
  - vi ic.yaml
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      app: ic-xfusion
    name: ic-deploy-xfusion
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: ic-xfusion
    template:
      metadata:
        labels:
          app: ic-xfusion
      spec:
        volumes:
          - name: ic-volume-xfusion
            emptyDir: {}
        initContainers:
          - name: ic-msg-xfusion
            image: debian:latest
            command: ['/bin/bash', '-c', 'echo Init Done - Welcome to xFusionCorp Industries > /ic/media']
            volumeMounts:
              - name: ic-volume-xfusion
                mountPath: /ic
        containers:
          - image: debian:latest
            name: ic-main-xfusion
            command: ['/bin/bash', '-c', 'while true; do cat /ic/media; sleep 5; done']
            volumeMounts:
              - name: ic-volume-xfusion
                mountPath: /ic
  ```
  #### kubectl apply -f ic.yaml 
    deployment.apps/ic-deploy-xfusion created

  #### kubectl get po --label-columns app=ic-xfusion
    NAME                                 READY   STATUS    RESTARTS   AGE   APP=IC-XFUSION
    ic-deploy-xfusion-59c486f6cf-5vcsb   1/1     Running   0          69s

  #### kubectl get po --show-labels 
    NAME                                 READY   STATUS    RESTARTS   AGE   LABELS
    ic-deploy-xfusion-59c486f6cf-5vcsb   1/1     Running   0          76s   app=ic-xfusion,pod-template-hash=59c486f6cf

  #### kubectl exec ic-deploy-xfusion-87c4cd85f-97h5b -c ic-main-xfusion -- cat /ic/media
    Init Done - Welcome to xFusionCorp Industries

  #### kubectl logs ic-deploy-xfusion-87c4cd85f-97h5b -c ic-main-xfusion
    Init Done - Welcome to xFusionCorp Industries
    Init Done - Welcome to xFusionCorp Industries
    Init Done - Welcome to xFusionCorp Industries
</details>