
## **Project: Workflow - Monitoring**

### **Overview**

In the **Monitoring** project, we focus on **Observability** which is broken down into three key aspects:

1. **Metrics (What)** - Monitoring
   - **Prometheus** is used to gather and store metrics.
   - Visualized through **Grafana** or **Kibana**.

2. **Logs (Why)** - Logging
   - **Loki** and **Promtail** are utilized for collecting logs.

3. **Traces (How)** - Tracing
   - **Jaeger** & **OpenTelemetry** are employed to trace the flow of requests.

#### **Node Exporter & Kube-State-Metrics**
- **Node Exporter** runs on port `9100` to monitor the health of nodes.
- **Kube-State-Metrics** helps track the state of Kubernetes objects (e.g., deployments, pods).

For complete **360-degree monitoring**, all components transport data to **Prometheus**. Prometheus handles time-series data, which can be queried and visualized using **Grafana**.

---

## **Steps to Set Up Monitoring Using Prometheus & Grafana**

1. **Clone the Project:**
   ```bash
   git clone https://github.com/LondheShubham153/k8s-kind-voting-app
   ```

2. **Verify Helm Installation:**
   ```bash
   helm version
   ```

3. **Install Prometheus & Grafana Using Helm:**

   - Create a directory for monitoring:
     ```bash
     mkdir Monitoring && cd Monitoring/
     ```

   - Add the Prometheus Helm repository:
     ```bash
     helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
     helm repo update
     ```

   - Install the Prometheus stack using Helm:
     ```bash
     helm install prometheus-stack prometheus-community/kube-prometheus-stack \
       --namespace monitoring \
       --set prometheus.service.nodePort=30000 \
       --set grafana.service.nodePort=31000 \
       --set grafana.service.type=NodePort \
       --set prometheus.service.type=NodePort
     ```

4. **Monitor Kubernetes Components:**

   - **Prometheus Node Exporter** and **Kube-State-Metrics** will be automatically deployed on your nodes.
   
   - Verify Pods and Services:
     ```bash
     kubectl get pods -n monitoring
     kubectl get svc -n monitoring
     ```

   - **Important Services**: 
     - `prometheus-operated`
     - `prometheus-stack-grafana`

5. **Port Forwarding:**
   - To access **Prometheus** and **Grafana** dashboards locally, use:
     ```bash
     kubectl port-forward svc/prometheus-stack-kube-prom-prometheus 9090:9090 -n monitoring --address=0.0.0.0 &
     kubectl port-forward svc/prometheus-stack-grafana 3000:80 -n monitoring --address=0.0.0.0
     ```

6. **Retrieve Grafana Admin Password:**
   ```bash
   kubectl get secret prometheus-stack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
   ```

7. **Grafana Dashboards:**
   - Login to Grafana with username `admin` and the password retrieved above.
   - **Prometheus** should already be connected as the data source.
   - You can add custom dashboards by searching for Grafana dashboard IDs and importing them into your Grafana instance.

8. **Sample Application Monitoring:**
   - Port-forward the services of your application (e.g., voting app):
     ```bash
     kubectl port-forward svc/result 5001:5001 --address=0.0.0.0 &
     kubectl port-forward svc/vote 5000:5000 --address=0.0.0.0 &
     ```

9. **Verify Monitoring Data:**
   - After interacting with the app, you should see activity in Grafana's graphs.
   - You can track which node is handling which pod by running:
     ```bash
     kubectl get pods -o wide
     ```

### **Conclusion:**
This setup allows you to monitor your Kubernetes cluster, application, and infrastructure effectively. Prometheus collects data, while Grafana visualizes it, enabling full observability. Large-scale organizations rely on such monitoring systems to track the performance and health of their infrastructure.

