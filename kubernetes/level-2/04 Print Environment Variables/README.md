# Print Environment Variables

* Create a `pod` named `print-envars-greeting`
* Configure spec as, the container name should be `print-env-container` and use `bash` image.
* Create three environment variables:
  - `GREETING` and its value should be `Welcome to`
  - `COMPANY` and its value should be `DevOps`
  - `GROUP` and its value should be `Group`
* Use command `["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']` (please use this exact command), also set its `restartPolicy` policy to `Never` to avoid crash loop back.
* You can check the output using `kubectl logs -f print-envars-greeting` command.

[Environment variable for a container](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/#define-an-environment-variable-for-a-container)

```bash
kubectl run print-envars-greeting --image bash:latest --dry-run=client -oyaml > pod.yaml

# change the container name and add env
vi pod.yaml

kubectl  apply -f pod.yaml

kubectl logs -f print-envars-greeting
```
  <details>
  <summary>pod.yaml</summary>
        
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: null
    labels:
      run: print-envars-greeting
    name: print-envars-greeting
  spec:
    containers:
    - image: bash:latest
      name: print-env-container
      resources: {}
      command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
      env:
        - name: GREETING
          value: "Welcome to"
        - name: COMPANY
          value: "DevOps"
        - name: GROUP
          value: "Group"
    dnsPolicy: ClusterFirst
    restartPolicy: Never
  ```
  </details>

  <details>
  <summary>outputs</summary>

    # kubectl  apply -f pod.yaml
    pod/print-envars-greeting created

    # kubectl logs -f print-envars-greeting
    Welcome to DevOps Group

  </details>