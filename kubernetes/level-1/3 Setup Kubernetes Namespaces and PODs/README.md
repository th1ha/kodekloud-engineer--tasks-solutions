# Setup Kubernetes Namespaces and PODs

* Create a `namespace` named `dev` and deploy a POD within it. Name the `pod dev-nginx-pod` and use the `nginx` image with the `latest` tag. Ensure to specify the tag as `nginx:latest`

  ```bash
  kubectl get ns

  kubectl create ns dev

  kubectl -n dev run dev-nginx-pod --image nginx:latest

  kubectl  -n dev  get all
  ```
    <details>
      <summary>outputs</summary>

        # kubectl get ns
        NAME                 STATUS   AGE
        default              Active   3m10s
        kube-node-lease      Active   3m10s
        kube-public          Active   3m10s
        kube-system          Active   3m10s
        local-path-storage   Active   3m1s

        # kubectl -n dev get all
        NAME                READY   STATUS    RESTARTS   AGE
        pod/dev-nginx-pod   1/1     Running   0          44s
    </details>