# Set Up Time Check Pod in Kubernetes

[Ref:::](https://shubhamksawant.medium.com/kubernetes-time-check-pod-bc28d15c543d)

* Create a pod called `time-check` in the `xfusion` namespace. The pod should contain a container named `time-check`, utilizing the `busybox` image with the `latest` tag (specify as `busybox:latest`)
    - Create a config map named `time-config` with the data `TIME_FREQ=4` in the same namespace
    - Configure the `time-check` container to execute the command: `while true; do date; sleep $TIME_FREQ;done`. Ensure the result is written `/opt/sysops/time/time-check.log`. Also, add an environmental variable `TIME_FREQ` in the container, fetching its value from the config map `TIME_FREQ` key.
    - Create a volume `log-volume` and mount it at `/opt/sysops/time` within the container.

    <details>
    <summary>time-config.yaml</summary>

      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: time-config
        namespace: xfusion
      data:
        TIME_FREQ: "4"
      
    </details>

    <details>
    <summary>time-check-pod.yaml</summary>

      apiVersion: v1
      kind: Pod
      metadata:
        name: time-check
        namespace: xfusion
      spec:
        containers:
        - name: time-check
          image: busybox:latest
          command: ["/bin/sh", "-c"]
          args:
            - while true; do date >> /opt/sysops/time/time-check.log; sleep $TIME_FREQ; done
          env:
          - name: TIME_FREQ
            valueFrom:
              configMapKeyRef:
                name: time-config
                key: TIME_FREQ
          volumeMounts:
          - name: log-volume
            mountPath: /opt/sysops/time
        volumes:
        - name: log-volume
          emptyDir: {}
      
    </details>

---

* create & verify

  ```bash
  kubectl create ns xfusion

  kubectl apply -f time-config.yaml

  kubectl apply -f time-check-pod.yaml

  kubectl  -n xfusion  get all

  kubectl -n xfusion exec time-check -- cat /opt/sysops/time/time-check.log

  ```
    <details>
    <summary>outputs</summary>

      # kubectl create ns xfusion

      namespace/xfusion created

      # kubectl apply -f time-config.yaml

      configmap/time-config created

      # kubectl apply -f time-check-pod.yaml

      pod/time-check created

      # kubectl  -n xfusion  get all

      NAME             READY   STATUS    RESTARTS   AGE
      pod/time-check   1/1     Running   0          39s

      # kubectl -n xfusion exec time-check -- cat /opt/sysops/time/time-check.log

      Tue Sep 23 03:00:51 UTC 2025
      Tue Sep 23 03:00:55 UTC 2025
      Tue Sep 23 03:00:59 UTC 2025
      Tue Sep 23 03:01:03 UTC 2025
      Tue Sep 23 03:01:07 UTC 2025

    </details>