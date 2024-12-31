

## Kubernetes Setup and Configuration

1. **Create a Kubernetes Cluster with Kind (Kubernetes in Docker):**
   ```bash
   $ kind create cluster --config config1.yml
   ```

2. **Get Cluster Information:**
   ```bash
   $ kubectl cluster-info --context kind-kind
   ```

3. **Get Kubernetes Contexts:**
   ```bash
   $ kubectl config get-contexts
   ```

4. **Set Current Context:**
   ```bash
   $ kubectl config use-context kind-kind
   ```

---

## Pods and Deployments

1. **Apply Pod YAML:**
   ```bash
   $ kubectl apply -f pod.yml
   pod/nginx-pod created
   ```

2. **Get Pods in a Specific Namespace (e.g., nginx):**
   ```bash
   $ kubectl get pods -n nginx
   ```

3. **Execute Command Inside a Pod:**
   ```bash
   $ kubectl exec -it nginx-pod -n nginx -- bash
   root@nginx-pod:/# exit
   exit
   ```

---

## Deployment and Scaling

1. **Deployment API Version:**
   - For deployments, use `apiVersion: apps/v1` in the YAML file.

2. **Replication Controller:**
   - A replication controller ensures that a specified number of replicas of a pod are running at any time. It cannot do rolling updates.
   - Deployments, on the other hand, can perform rolling updates, updating pods one by one.

3. **Label and Selector in Deployments:**
   - In the deployment YAML, labels are used to select the pods that the deployment manages.

4. **Delete Pod Using YAML:**
   ```bash
   $ kubectl delete -f pod.yml
   pod "nginx-pod" deleted
   ```

5. **Get Deployment Information:**
   ```bash
   $ kubectl get deployment -n nginx
   NAME               READY   UP-TO-DATE   AVAILABLE   AGE
   nginx-deployment   2/2     2            2           15s
   ```

6. **Get Pods in Default Namespace:**
   ```bash
   $ kubectl get pods
   No resources found in default namespace.
   ```

7. **Get Pods in Specific Namespace (e.g., nginx):**
   ```bash
   $ kubectl get pods -n nginx
   NAME                                READY   STATUS    RESTARTS   AGE
   nginx-deployment-55fb85df89-cxh77   1/1     Running   0          30s
   nginx-deployment-55fb85df89-mfkrt   1/1     Running   0          30s
   ```

8. **Scale Deployment (Increase Replica Count):**
   ```bash
   $ kubectl scale deployment/nginx-deployment -n nginx --replicas=5
   deployment.apps/nginx-deployment scaled
   ```

9. **Check Pods After Scaling:**
   ```bash
   $ kubectl get pods -n nginx
   NAME                                READY   STATUS              RESTARTS   AGE
   nginx-deployment-55fb85df89-8v7tj   1/1     Running             0          5s
   nginx-deployment-55fb85df89-cxh77   1/1     Running             0          86s
   nginx-deployment-55fb85df89-mdgrk   0/1     ContainerCreating   0          5s
   nginx-deployment-55fb85df89-mfkrt   1/1     Running             0          86s
   nginx-deployment-55fb85df89-tznx8   1/1     Running             0          5s
   ```

10. **Rolling Updates (Pod Replacement):**
    - Rolling updates ensure that updates are applied gradually (one pod at a time).
    - Check the pods during an update:
    ```bash
    $ kubectl get pods -n nginx
    ```

---

## Other Resources

### DaemonSets:
- A **DaemonSet** ensures that all nodes in the cluster run at least one replica of a specific pod. DaemonSets do not support replicas.

### Jobs:
- **Jobs** run tasks that are executed to completion and do not stay running (e.g., batch tasks). 
- Example Job YAML:
   ```yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: demo-job
   spec:
     template:
       spec:
         containers:
         - name: batch-container
           image: busybox:latest
           command: ["sh", "-c", "echo Hello bhai~ && sleep 10"]
         restartPolicy: Never
   ```

- **Check Job Logs:**
   ```bash
   $ kubectl logs pod/demo-job-2xnz8 -n nginx
   Hello bhai~
   ```

### Persistent Volumes:
- **Persistent Volume (PV):** Provides storage resources in Kubernetes.
- **Persistent Volume Claim (PVC):** Claims a specific volume for use within a pod.
- **Example PVC:**
   ```bash
   $ kubectl get pvc
   NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
   local-pvc   Bound    local-pv   1Gi        RWO            local-storage   10s
   ```

### Using Persistent Volume in a Deployment:
   ```yaml
   volumes:
   - name: my-volume
     persistentVolumeClaim:
       claimName: local-pvc
   containers:
   - name: my-container
     image: nginx:latest
     volumeMounts:
     - mountPath: /var/www/html
       name: my-volume
   ```

### Deleting Resources:
- Deleting volumes and PVCs requires careful steps:
  - Delete the PVC first.
  - Delete the associated PV after the claim is removed.

---

## Networking: Services and Ingress

1. **Create a Service (ClusterIP):**
   - A **Service** exposes a set of pods as a network service.
   ```yaml
   kind: Service
   apiVersion: v1
   metadata:
     name: nginx-service
     namespace: nginx
   spec:
     selector:
       app: nginx
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
     type: ClusterIP
   ```

2. **Port Forwarding to Expose Service:**
   - Exposes the service on a specific port for external access.
   ```bash
   $ kubectl port-forward service/nginx-service -n nginx 80:80 --address=0.0.0.0
   ```

3. **Ingress:**
   - Ingress allows routing traffic to different services based on the URL path (e.g., `/nginx` -> nginx service, `/notes` -> notes service).
   - Example Ingress controller deployment:
   ```bash
   kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
   ```

4. **Ingress Annotations:**
   - Annotations in the Ingress resource control the behavior of ingress controllers:
   ```yaml
   nginx.ingress.kubernetes.io/rewrite-target: /
   ```

---

## Docker Integration

1. **Check Running Docker Containers:**
   ```bash
   $ docker ps
   CONTAINER ID   IMAGE                              COMMAND                  CREATED        STATUS                            PORTS                                       NAMES
   7fa822c84e6e   kindest/node:v1.31.2               "/usr/local/bin/entr…"   16 hours ago   Up 2 hours                        kind-worker
   5f573a9b15e5   kindest/node:v1.31.2               "/usr/local/bin/entr…"   16 hours ago   Up 2 hours                        kind-worker2
   ```

2. **Access a Docker Container:**
   ```bash
   $ docker exec -it 7fa822c84e6e bash
   root@kind-worker:/# cd /mnt/data
   ```

---

## Conclusion

- In Kubernetes, you work with **pods**, **deployments**, **services**, **volumes**, and **ingress** to manage and expose applications.
- Use **jobs** for one-time tasks, and **DaemonSets** for ensuring a pod runs on every node.
- Ensure proper namespace management to avoid errors.
- **Ingress** helps manage routing traffic to multiple services based on URL paths.


================================================================================================
### **Concise and Enhanced Notes on Kubernetes, StatefulSets, and Related Concepts**

---

#### **Stateless vs. Stateful Applications**

- **Stateless Applications**: 
  - Examples: Django, Spring, Nginx.
  - These applications do not retain any information between requests.
  - Kubernetes objects used:
    - **Deployment**
    - **ReplicaSets**
    - **DaemonSets**

- **Stateful Applications**:
  - Example: MySQL.
  - These applications retain data and state across requests.
  - Kubernetes object used:
    - **StatefulSet** (with replicas)
    - Ensures the preservation of unique identities for each pod.

---

#### **StatefulSets in Kubernetes**

- **StatefulSet**: Used for stateful applications. It manages the deployment of pods with stable, unique identities and persistent storage.
- **Headless Service**: A service without a ClusterIP (`None` in the spec), allowing each pod to have a unique DNS name while remaining inaccessible externally.
- **Replicas**: StatefulSets can have multiple replicas, each with stable identity and persistent volume.

**Example StatefulSet Configuration**:
```yaml
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: mysql-statefulset
  namespace: mysql
spec:
  serviceName: mysql-service
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: root
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
```

---

#### **Pod Management in StatefulSets**

- **Pod Behavior**: If a pod in a StatefulSet is deleted, Kubernetes ensures a new pod with the same name and storage is created.
- **Example**: 
  ```bash
  kubectl get pods -n mysql
  NAME                  READY   STATUS              RESTARTS   AGE
  mysql-statefulset-0   1/1     Running             0          6m56s
  mysql-statefulset-1   1/1     Running             0          3m35s
  mysql-statefulset-2   0/1     ContainerCreating   0          37s
  ```
- After deleting a pod, Kubernetes re-creates it with the same name and attaches it to the appropriate Persistent Volume.

---

#### **Interacting with MySQL Pod in StatefulSet**

- To connect to MySQL inside a StatefulSet pod:
  ```bash
  kubectl exec -it mysql-statefulset-1 -n mysql -- mysql -u root -p --socket=/var/lib/mysql/mysql.sock
  ```
- **Example**: Viewing MySQL databases:
  ```sql
  show databases;
  ```

---

#### **Configuration with ConfigMaps and Secrets**

- **ConfigMap**: Store non-sensitive data such as configuration variables (e.g., database name).
  - **Example**:
    ```yaml
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: mysql-config-map
      namespace: mysql
    data:
      MYSQL_DATABASE: mydb
    ```
  - Access in StatefulSet:
    ```yaml
    - name: MYSQL_DATABASE
      valueFrom:
        configMapKeyRef:
          name: mysql-config-map
          key: MYSQL_DATABASE
    ```

- **Secrets**: Used for storing sensitive data (e.g., passwords) in Base64-encoded format.
  - **Example**:
    ```bash
    echo "root" | base64
    cm9vdAo=
    ```
  - Secret in StatefulSet:
    ```yaml
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql-secret
          key: MYSQL_ROOT_PASSWORD
    ```

---

#### **Resource Quotas and Limits**

- **Resource Requests and Limits**: Control CPU and memory usage for pods to prevent any single pod from consuming all resources.
  - **Example**:
    ```yaml
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 200m
        memory: 256Mi
    ```
  - This is defined under `spec.containers` in the `StatefulSet`.

---

### **Summary**

- **Stateless**: Deployment, ReplicaSets, DaemonSets (no persistent state).
- **Stateful**: StatefulSets (with Persistent Volumes, Headless Services, stable identities).
- **ConfigMaps** store non-sensitive configuration, while **Secrets** store sensitive data.
- **Resource limits** ensure pods don't over-consume resources.



================================================



  probes ->  (request)
    liveness probe
    readliness probe
    start probe 


