# Deploy Applications with Kubernetes Deployments

* Create a `deployment` named `nginx` to deploy the application nginx using the image `nginx:latest` (ensure to specify the tag)
  ```bash
  kubectl create deployment nginx --image nginx:latest

  # deployment.apps/nginx created
  ```

* verify
  ```bash
  kubectl  get all
  ```