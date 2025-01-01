

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

#### Conclusion

- In Kubernetes, you work with **pods**, **deployments**, **services**, **volumes**, and **ingress** to manage and expose applications.
- Use **jobs** for one-time tasks, and **DaemonSets** for ensuring a pod runs on every node.
- Ensure proper namespace management to avoid errors.
- **Ingress** helps manage routing traffic to multiple services based on URL paths.


================================================================================================


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
Here’s an even more detailed version with extended explanations, best practices, examples, and command breakdowns for each section:

---

### **Probes in Kubernetes**

**Overview**:
- Kubernetes uses probes to monitor the health and status of containers.
- There are three main types of probes:
  1. **Liveness Probe**: Checks if the application is running and healthy.
  2. **Readiness Probe**: Determines if the application is ready to handle requests.
  3. **Startup Probe**: Specifically used to check if an application has successfully started.

**Configuration Options**:
Probes support multiple types of checks:
1. **HTTP GET**: Sends an HTTP request to a specific endpoint.
2. **TCP Socket**: Checks if a specific port is listening for connections.
3. **Command Execution**: Executes a command inside the container. A zero exit code indicates success.

**Example YAML**:
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10

readinessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 5
  periodSeconds: 10

startupProbe:
  exec:
    command:
    - cat
    - /tmp/ready
  failureThreshold: 30
  periodSeconds: 5
```

**Key Parameters**:
- `initialDelaySeconds`: Delay before the probe starts.
- `periodSeconds`: Frequency of probe execution.
- `timeoutSeconds`: Timeout for each probe.
- `failureThreshold`: Number of failures before marking the container as unhealthy.
- `successThreshold`: Number of successes before marking the container as healthy (for readiness/startup probes).

---

### **Taints and Tolerations**

**Concepts**:
- **Taints**: Used to repel pods from nodes.
- **Tolerations**: Allow pods to bypass taints and be scheduled on tainted nodes.
- Taints are key-value pairs with an optional effect. Effects include:
  - `NoSchedule`: Pods without tolerations won’t be scheduled on the node.
  - `PreferNoSchedule`: Kubernetes avoids scheduling pods unless absolutely necessary.
  - `NoExecute`: Existing pods are evicted if they don’t tolerate the taint.

**Command Examples**:
1. **Add a Taint**:
   ```bash
   kubectl taint node kind-worker env=prod:NoSchedule
   ```
2. **Remove a Taint**:
   ```bash
   kubectl taint node kind-worker env=prod:NoSchedule-
   ```
3. **View Taints on a Node**:
   ```bash
   kubectl describe node kind-worker
   ```

**Toleration YAML**:
```yaml
tolerations:
- key: "env"
  operator: "Equal"
  value: "prod"
  effect: "NoSchedule"
```

**Use Cases**:
- Isolating workloads by environment (e.g., production vs. staging).
- Ensuring critical applications run on dedicated nodes.

---

### **Autoscaling in Kubernetes**

**1. Horizontal Pod Autoscaler (HPA)**:
- Scales pods based on CPU/memory usage or custom metrics.
- **Prerequisites**:
  - Metrics server must be installed.
  - Ensure resource requests/limits are set for the deployment.

**HPA Example**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Commands**:
- **Create HPA**:
  ```bash
  kubectl autoscale deployment web-app --cpu-percent=70 --min=2 --max=10
  ```
- **View HPA Status**:
  ```bash
  kubectl get hpa
  ```

---

**2. Vertical Pod Autoscaler (VPA)**:
- Adjusts CPU/memory resource requests for pods.
- Useful for stateful apps where horizontal scaling isn’t practical.

**Installation**:
1. Clone the autoscaler repo:
   ```bash
   git clone https://github.com/kubernetes/autoscaler.git
   cd autoscaler/vertical-pod-autoscaler
   ./hack/vpa-up.sh
   ```
2. Define VPA for an app:
   ```yaml
   apiVersion: autoscaling.k8s.io/v1
   kind: VerticalPodAutoscaler
   metadata:
     name: web-app-vpa
   spec:
     targetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: web-app
     updatePolicy:
       updateMode: "Auto"
   ```

---

### **Node Affinity**

**Types of Node Affinity**:
1. **requiredDuringSchedulingIgnoredDuringExecution**:
   - Enforces strict node selection.
2. **preferredDuringSchedulingIgnoredDuringExecution**:
   - Kubernetes tries to schedule on preferred nodes but falls back to others if unavailable.

**Example**:
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: "node-role.kubernetes.io/worker"
          operator: In
          values:
          - true
```

---

### **RBAC (Role-Based Access Control)**

**Key Concepts**:
- Roles and RoleBindings are namespace-specific.
- ClusterRoles and ClusterRoleBindings apply cluster-wide.

**Commands**:
1. **List all Roles**:
   ```bash
   kubectl get roles -n <namespace>
   ```
2. **View Role Details**:
   ```bash
   kubectl describe role <role-name> -n <namespace>
   ```

**ClusterRole Example**:
```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cluster-reader
rules:
- apiGroups: [""]
  resources: ["nodes", "pods"]
  verbs: ["get", "list", "watch"]
```

**ClusterRoleBinding Example**:
```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cluster-reader-binding
subjects:
- kind: User
  name: jane.doe@example.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-reader
```

---

### **Helm**

**Basic Commands**:
1. **List Helm Releases**:
   ```bash
   helm list -n <namespace>
   ```
2. **Delete a Helm Release**:
   ```bash
   helm uninstall <release-name> -n <namespace>
   ```
3. **View Chart Values**:
   ```bash
   helm get values <release-name> -n <namespace>
   ```

**Best Practices**:
- Use `values.yaml` for configuration.
- Use Helm hooks (`pre-install`, `post-install`, etc.) to perform actions during lifecycle events.

---

### **Custom Resource Definitions (CRDs)**

**Creating a Custom Resource**:
1. Define the CRD (e.g., `devopsbatches` as shown earlier).
2. Apply the CRD:
   ```bash
   kubectl apply -f crd.yaml
   ```
3. Create instances of the custom resource:
   ```yaml
   apiVersion: google.com/v1
   kind: DevOpsBatch
   metadata:
     name: devops-batch1
   spec:
     name: "Batch 1"
     duration: "6 months"
     mode: "online"
     platform: "Kubernetes"
   ```

4. List resources:
   ```bash
   kubectl get devopsbatches
   ```

---

### **Service Mesh**

**Features**:
- Traffic splitting, retry policies, and mutual TLS (mTLS).
- Observability with tools like Grafana, Kiali, and Jaeger.

**Istio Installation**:
```bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-<version>
istioctl install --set profile=demo -y
```

**Traffic Management Example**:
- Split traffic between two versions of an app:
  ```yaml
  apiVersion: networking.istio.io/v1beta1
  kind: VirtualService
  metadata:
    name: web-app
  spec:
    hosts:
    - "web-app.example.com"
    http:
    - route:
      - destination:
          host: web-app-v1
          subset: v1
        weight: 80
      - destination:
          host: web-app-v2
          subset: v2
        weight: 20
  ```



### **Service Mesh with Istio: Detailed Guide**

Service Mesh simplifies communication between microservices by providing advanced traffic management, security, observability, and resilience features. In this guide, we explore setting up a sample application in Kubernetes using Istio as the service mesh.

---

### **Basic Concepts**

1. **Components of Istio**:
   - **Envoy**: Sidecar proxy that handles service-to-service communication, load balancing, and security (TLS termination, etc.).
   - **Istiod**: The control plane that manages service discovery, configuration, and certificates.
   - **Ingress/Egress Gateway**: Controls inbound and outbound traffic for the mesh.

2. **Use Case Example**:
   - Services:
     - **Voting Service** (`5000`)
     - **Results Service** (`5001`)
     - **Worker Service** (`5002`)
   - These microservices communicate internally through Istio-managed service-to-service communication.

---

### **Istio Installation**

1. **Download and Install Istio**:
   ```bash
   curl -L https://istio.io/downloadIstio | sh -
   cd istio-1.24.2
   ```

2. **Add Istio CLI (`istioctl`) to PATH**:
   ```bash
   export PATH=$PWD/bin:$PATH
   ```
   Optionally, move the binary to `/usr/local/bin` for global access:
   ```bash
   sudo mv bin/istioctl /usr/local/bin/
   ```

3. **Install Istio**:
   - Install with the demo profile:
     ```bash
     istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
     ```

---

### **Labeling Namespace for Sidecar Injection**

Istio works by injecting **Envoy sidecars** into your pods. To enable automatic injection for a namespace:
```bash
kubectl label namespace default istio-injection=enabled --overwrite
```

You can verify the labels:
```bash
kubectl get ns default --show-labels
```

---

### **Deploy a Sample Application**

1. **Bookinfo Application**:
   Istio provides a sample application, **Bookinfo**, which demonstrates service-to-service communication. Deploy it:
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.24/samples/bookinfo/platform/kube/bookinfo.yaml
   ```

2. **Verify Services and Pods**:
   - Check the services:
     ```bash
     kubectl get services
     ```
   - Test inter-service communication:
     ```bash
     kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings \
     -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
     ```

---

### **Exposing the Application to External Traffic**

1. **Create a Gateway**:
   - To allow external access to the application, create a Gateway:
     ```bash
     kubectl apply -f samples/bookinfo/gateway-api/bookinfo-gateway.yaml
     ```

2. **Annotate Gateway for ClusterIP**:
   ```bash
   kubectl annotate gateway bookinfo-gateway networking.istio.io/service-type=ClusterIP --namespace=default
   ```

3. **Check Gateway Status**:
   ```bash
   kubectl get gateway
   ```

---

### **Visualizing Service Mesh with Kiali**

1. **Install Kiali**:
   - Deploy Istio addons:
     ```bash
     kubectl apply -f samples/addons
     ```
   - Wait for Kiali to be ready:
     ```bash
     kubectl rollout status deployment/kiali -n istio-system
     ```

2. **Access the Kiali Dashboard**:
   - Open the dashboard:
     ```bash
     istioctl dashboard kiali
     ```
   - This provides a graphical representation of service-to-service communication, latency, request rates, and error rates.

---

### **Advanced Traffic Management**

1. **Traffic Splitting**:
   - Route 80% of traffic to version `v1` and 20% to version `v2`:
     ```yaml
     apiVersion: networking.istio.io/v1beta1
     kind: VirtualService
     metadata:
       name: bookinfo
     spec:
       hosts:
       - "bookinfo.example.com"
       http:
       - route:
         - destination:
             host: productpage
             subset: v1
           weight: 80
         - destination:
             host: productpage
             subset: v2
           weight: 20
     ```

2. **Retry and Timeout Policies**:
   - Define retries and timeouts for services:
     ```yaml
     apiVersion: networking.istio.io/v1beta1
     kind: VirtualService
     metadata:
       name: reviews
     spec:
       hosts:
       - reviews
       http:
       - route:
         - destination:
             host: reviews
             subset: v1
         retries:
           attempts: 3
           perTryTimeout: 2s
     ```

---

### **Key Commands Recap**

- **Download Istio**:
  ```bash
  curl -L https://istio.io/downloadIstio | sh -
  ```
- **Install Istio**:
  ```bash
  istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
  ```
- **Enable Sidecar Injection**:
  ```bash
  kubectl label namespace default istio-injection=enabled --overwrite
  ```
- **Deploy Bookinfo App**:
  ```bash
  kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.24/samples/bookinfo/platform/kube/bookinfo.yaml
  ```
- **Check Gateway**:
  ```bash
  kubectl get gateway
  ```
- **Launch Kiali**:
  ```bash
  istioctl dashboard kiali
  ```

---

