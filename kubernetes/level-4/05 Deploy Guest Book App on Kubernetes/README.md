# Deploy Guest Book App on Kubernetes

## BACK-END TIER

1. Create a deployment named **redis-master** for Redis master.
  - Replicas count should be **1**.
  - Container name should be **master-redis-nautilus** and it should use image **redis**.
  - Request resources as **CPU** should be **100m** and **Memory** should be **100Mi**.
  - Container port should be redis default port i.e **6379**.

2. Create a service named **redis-master** for Redis master. Port and targetPort should be Redis default port i.e **6379**.

3. Create another deployment named **redis-slave** for Redis slave.
  - Replicas count should be **2**.
  - Container name should be **slave-redis-nautilus** and it should use **gcr.io/google_samples/gb-redisslave:v3** image.
  - Requests resources as **CPU** should be **100m** and **Memory** should be **100Mi**.
  - Define an environment variable named **GET_HOSTS_FROM** and its value should be dns.
  - Container port should be Redis default port i.e **6379**.

4. Create another service named **redis-slave**. It should use Redis default port i.e 6379.

## FRONT END TIER

1. Create a deployment named **frontend**.
  - Replicas count should be **3**.
  - Container name should be **php-redis-nautilus** and it should use **gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff** image.
  - Request resources as **CPU** should be **100m** and **Memory** should be **100Mi**.
  - Define an environment variable named as **GET_HOSTS_FROM** and its value should be **dns**.
  - Container port should be **80**.

2. Create a service named **frontend**. Its **type** should be **NodePort**, port should be **80** and its **nodePort** should be **30009**.

Finally, you can check the **guestbook app** by clicking on **App** button.

> You can use any labels as per your choice.

---

<details>
<summary>steps</summary>

  #### create a deployment and service
  ```bash
  kubectl create deployment redis-master --image redis --replicas 1 --port 6379 --dry-run=client -oyaml > redis-master.yaml

  kubectl create service clusterip redis-master --tcp 6379:6379 --dry-run=client -oyaml >> redis-master.yaml

  kubectl create deployment redis-slave --image gcr.io/google_samples/gb-redisslave:v3 --replicas 2 --port 6379 --dry-run=client -oyaml > redis-slave.yaml

  kubectl create service clusterip redis-slave --tcp 6379:6379 --dry-run=client -oyaml >> redis-slave.yaml

  kubectl create deployment frontend --replicas 3 --image gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff --port 80 --dry-run=client -oyaml > frontend.yaml

  kubectl create service nodeport frontend --tcp 80:80 --node-port 30009 --dry-run=client -oyaml >> frontend.yaml
  ```

  #### redis-master deployment
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: redis-master
    name: redis-master
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: redis-master
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: redis-master
      spec:
        containers:
        - image: redis
          name: master-redis-nautilus
          ports:
          - containerPort: 6379
          resources:
            requests:
              memory: "100Mi"
              cpu: "100m"
  ---
  apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: null
    labels:
      app: redis-master
    name: redis-master
  spec:
    ports:
    - name: 6379-6379
      port: 6379
      protocol: TCP
      targetPort: 6379
    selector:
      app: redis-master
    type: ClusterIP
  ```

  #### redis-slave deployment
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: redis-slave
    name: redis-slave
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: redis-slave
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: redis-slave
      spec:
        containers:
        - image: gcr.io/google_samples/gb-redisslave:v3
          name: slave-redis-nautilus
          ports:
          - containerPort: 6379
          resources:
            requests:
              memory: "100Mi"
              cpu: "100m"
          env:
            - name: GET_HOSTS_FROM
              value: dns
  ---
  apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: null
    labels:
      app: redis-slave
    name: redis-slave
  spec:
    ports:
    - name: 6379-6379
      port: 6379
      protocol: TCP
      targetPort: 6379
    selector:
      app: redis-slave
    type: ClusterIP
  ```

  #### frontend deployment
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: frontend
    name: frontend
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: frontend
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: frontend
      spec:
        containers:
        - image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
          name: php-redis-nautilus
          ports:
          - containerPort: 80
          resources:
            requests:
              memory: "100Mi"
              cpu: "100m"
          env:
            - name: GET_HOSTS_FROM
              value: dns
  ---
  apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: null
    labels:
      app: frontend
    name: frontend
  spec:
    ports:
    - name: 80-80
      nodePort: 30009
      port: 80
      protocol: TCP
      targetPort: 80
    selector:
      app: frontend
    type: NodePort
  ```
</details>