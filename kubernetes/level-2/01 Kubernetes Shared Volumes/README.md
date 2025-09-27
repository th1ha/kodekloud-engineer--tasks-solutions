# Kubernetes Shared Volumes

1. Create a pod named `volume-share-xfusion`
2. For the first container, use image `fedora` with `latest` tag only and remember to mention the tag i.e `fedora:latest`, container should be named as `volume-container-xfusion-1`, and run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/beta`
3. For the second container, use image `fedora` with the `latest` tag only and remember to mention the tag i.e `fedora:latest`, container should be named as `volume-container-xfusion-2`, and again run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/games`
4. Volume name should be `volume-share` of type `emptyDir`
5. After creating the pod, exec into the first container i.e `volume-container-xfusion-1`, and just for testing create a file `beta.txt` with any content under the mounted path of first container i.e `/tmp/beta`
6. The file `beta.txt` should be present under the mounted path `/tmp/games` on the second container `volume-container-xfusion-2` as well, since they are using a shared volume.

[Creating a Pod that runs two Containers](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/#creating-a-pod-that-runs-two-containers)

```bash
curl -LO /tmp/pod.yaml https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/two-container-pod.yaml

vim two-container-pod.yaml

kubectl  apply -f two-container-pod.yaml

kubectl  get po

kubectl exec volume-share-xfusion -c volume-container-xfusion-1 -- touch /tmp/beta/beta.txt

kubectl exec volume-share-xfusion -c volume-container-xfusion-1 -- ls /tmp/beta/

kubectl exec volume-share-xfusion -c volume-container-xfusion-2 -- ls /tmp/games
```
  <details>
  <summary>two-container-pod.yaml</summary>
        
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: volume-share-xfusion
  spec:
    restartPolicy: Never
    volumes:
    - name: volume-share
      emptyDir: {}

    containers:
    - name: volume-container-xfusion-1
      image: fedora:latest
      command: ["/bin/sh"]
      args: ["-c", "sleep 3600"]
      volumeMounts:
      - name: volume-share
        mountPath: /tmp/beta

    - name: volume-container-xfusion-2
      image: fedora:latest
      command: ["/bin/sh"]
      args: ["-c", "sleep 3600"]
      volumeMounts:
      - name: volume-share
        mountPath: /tmp/games
  ```

  </details>

  <details>
  <summary>outputs</summary>

    # kubectl  apply -f two-container-pod.yaml

    pod/volume-share-xfusion created

    # kubectl  get po

    NAME                   READY   STATUS    RESTARTS   AGE
    volume-share-xfusion   2/2     Running   0          39s

    # kubectl exec volume-share-xfusion -c volume-container-xfusion-1 -- touch /tmp/beta/beta.txt

    # kubectl exec volume-share-xfusion -c volume-container-xfusion-1 -- ls /tmp/beta/

    beta.txt

    # kubectl exec volume-share-xfusion -c volume-container-xfusion-2 -- ls /tmp/games

    beta.txt
  </details>