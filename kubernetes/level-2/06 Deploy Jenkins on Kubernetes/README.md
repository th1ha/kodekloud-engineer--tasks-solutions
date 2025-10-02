# Deploy Jenkins on Kubernetes

1. Create a namespace **jenkins**

2. Create a Service for jenkins deployment. Service name should be **jenkins-service** under **jenkins** namespace, type should be **NodePort**, nodePort should be **30008**

3. Create a Jenkins Deployment under **jenkins** namespace, It should be name as **jenkins-deployment** , labels **app** should be **jenkins** , container name should be **jenkins-container** , use **jenkins/jenkins** image , containerPort should be **8080** and replicas count should be **1**.

> Make sure to wait for the pods to be in running state and make sure you are able to access the Jenkins login screen in the browser before hitting the Check button.

[Doc - Jenkins-Kubernetes](https://www.jenkins.io/doc/book/installing/kubernetes/)

```bash
kubectl create ns jenkins

kubectl config set-context --current --namespace jenkins

kubectl config get-contexts

# vi jenkins-sa.yaml

kubectl  apply -f jenkins-sa.yaml

# vi jenkins-svc.yaml

# vi jenkins-deployment.yaml

kubectl apply -f jenkins-svc.yaml -f jenkins-deployment.yaml

kubectl get all
```
  <details>
  <summary>jenkins-sa.yaml</summary>
        
  ```yaml
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: jenkins-admin
  rules:
    - apiGroups: [""]
      resources: ["*"]
      verbs: ["*"]
  ---
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: jenkins-admin
    namespace: jenkins
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: jenkins-admin
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: jenkins-admin
  subjects:
  - kind: ServiceAccount
    name: jenkins-admin
    namespace: jenkins
  ```
  </details>

  <details>
  <summary>jenkins-svc.yaml</summary>
        
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: jenkins-service
    namespace: jenkins
  spec:
    selector:
      app: jenkins
    type: NodePort
    ports:
      - port: 8080
        targetPort: 8080
        nodePort: 30008
  ```
  </details>

  <details>
  <summary>jenkins-deployment.yaml</summary>
        
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: jenkins-deployment
    namespace: jenkins
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: jenkins
    strategy:
      type: Recreate
    template:
      metadata:
        labels:
          app: jenkins
      spec:
        serviceAccountName: jenkins-admin
        containers:
          - name: jenkins-container
            image: jenkins/jenkins
            imagePullPolicy: Always
            ports:
              - name: httpport
                containerPort: 8080
              - name: jnlpport
                containerPort: 50000
            livenessProbe:
              httpGet:
                path: "/login"
                port: 8080
              initialDelaySeconds: 90
              periodSeconds: 10
              timeoutSeconds: 5
              failureThreshold: 5
            readinessProbe:
              httpGet:
                path: "/login"
                port: 8080
              initialDelaySeconds: 60
              periodSeconds: 10
              timeoutSeconds: 5
              failureThreshold: 3
            volumeMounts:
              - name: jenkins-data
                mountPath: /var/jenkins_home
        volumes:
          - name: jenkins-data
            emptyDir: {}
  ```
  </details>

  <details>
  <summary>outputs</summary>

  #### kubectl create ns jenkins
    namespace/jenkins created

  #### kubectl config set-context --current --namespace jenkins
    Context "kind-kodekloud" modified.
  
  #### kubectl config get-contexts
    CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
    *         kind-kodekloud   kind-kodekloud   kind-kodekloud   jenkins

  #### kubectl  apply -f jenkins-sa.yaml
    clusterrole.rbac.authorization.k8s.io/jenkins-admin created
    serviceaccount/jenkins-admin created
    clusterrolebinding.rbac.authorization.k8s.io/jenkins-admin created

  #### kubectl apply -f jenkins-svc.yaml -f jenkins-deployment.yaml
    service/jenkins-service created
    deployment.apps/jenkins-deployment created

  #### kubectl get all
    NAME                                      READY   STATUS    RESTARTS   AGE
    pod/jenkins-deployment-6fdb466b7b-8t7xk   1/1     Running   0          95s

    NAME                      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    service/jenkins-service   NodePort   10.96.34.254   <none>        8080:30008/TCP   95s

    NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/jenkins-deployment   1/1     1            1           95s

    NAME                                            DESIRED   CURRENT   READY   AGE
    replicaset.apps/jenkins-deployment-6fdb466b7b   1         1         1       95s
  </details>