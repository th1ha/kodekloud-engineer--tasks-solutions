# Schedule Cronjobs in Kubernetes

[Doc::: Cron-jobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#example)

* Create a cronjob named `nautilus`
    - Set Its schedule to something like `*/3 * * * *.` You can set any schedule for now
    - Name the container `cron-nautilus`
    - Utilize the `nginx` image with `latest` tag (specify as `nginx:latest`)
    - Execute the dummy command `echo Welcome to xfusioncorp!`
    - Ensure the restart policy is `OnFailure`

    <details>
    <summary>cron.yaml</summary>

      apiVersion: batch/v1
      kind: CronJob
      metadata:
      name: nautilus
      spec:
      schedule: "*/3 * * * *"
      jobTemplate:
          spec:
          template:
              spec:
              containers:
              - name: cron-nautilus
                  image: nginx:latest
                  imagePullPolicy: IfNotPresent
                  command:
                  - /bin/sh
                  - -c
                  - echo Welcome to xfusioncorp!
              restartPolicy: OnFailure
      
    </details>

---

* create & verify

  ```bash
  kubectl apply -f cron.yaml

  kubectl  get all

  kubectl logs nautilus-29307195-6psvm

  ```
    <details>
    <summary>outputs</summary>

      # kubectl apply -f cron.yaml

      cronjob.batch/nautilus created

      # kubectl  get all

      NAME                          READY   STATUS      RESTARTS   AGE
      pod/nautilus-29307195-6psvm   0/1     Completed   0          45s

      NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
      service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   27m

      NAME                     SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
      cronjob.batch/nautilus   */3 * * * *   False     0        45s             100s

      NAME                          COMPLETIONS   DURATION   AGE
      job.batch/nautilus-29307195   1/1           10s        45s

      # kubectl logs nautilus-29307195-6psvm

      Welcome to xfusioncorp!

    </details>