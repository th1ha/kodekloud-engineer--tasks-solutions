# Create Countdown Job in Kubernetes

[Doc::: jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)

* Create a job named `countdown-nautilus`
    - The spec template should be named `countdown-nautilus` (under metadata), and the container should be named `container-countdown-nautilus`
    - Utilize image `ubuntu` with `latest` tag (ensure to specify as `ubuntu:latest`), and set the restart policy to `Never`
    - Execute the command `sleep 5`

    <details>
    <summary>jobs.yaml</summary>

      apiVersion: batch/v1
      kind: Job
      metadata:
        name: countdown-nautilus
      spec:
        template:
          metadata:
            name: countdown-nautilus
          spec:
            containers:
            - name: container-countdown-nautilus
              image: ubuntu:latest
              command: ["sleep", "5"]
            restartPolicy: Never
      
    </details>

---

* create & verify

  ```bash
  kubectl  apply -f jobs.yaml

  kubectl  get all

  ```
    <details>
    <summary>outputs</summary>

      # kubectl apply -f cron.yaml

      job.batch/countdown-nautilus created

      # kubectl  get all

      NAME                           READY   STATUS      RESTARTS   AGE
      pod/countdown-nautilus-4wjx5   0/1     Completed   0          105s

      NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
      service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   12m

      NAME                           COMPLETIONS   DURATION   AGE
      job.batch/countdown-nautilus   1/1           13s        105s

    </details>