### **3-Tier Chat Application on Kubernetes**  
This project involves deploying a **3-tier Chat Application** on Kubernetes using Minikube for local development. The application consists of the following components:

---

### **Components**  

#### 1. **Frontend (React)**  
- Requires Docker for containerization.  
- **Steps**:  
  1. Build a Docker image for the React application.  
  2. Create a Kubernetes Pod.  
  3. Deploy the Pod using a Deployment resource.  
  4. Expose the application using a Service.  
  5. Connect to the database for persistent data.  

#### 2. **Backend (Node.js)**  
- Requires Docker for containerization.  
- **Steps**:  
  1. Build a Docker image for the Node.js backend.  
  2. Create a Kubernetes Pod.  
  3. Deploy the Pod using a Deployment resource.  
  4. Expose the application using a Service.  
  5. Connect to the database for storing application data.  

#### 3. **Database (MongoDB)**  
- Utilizes the official MongoDB Docker image.  
- **Steps**:  
  1. Configure MongoDB with default ports (`27017`) and environment variables like `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD`.  
  2. Create a Kubernetes Pod.  
  3. Deploy the Pod using a Deployment resource.  
  4. Expose MongoDB using a Service.  

---

### **Setting Up Minikube**  
Start Minikube with Docker as the driver:  
```bash
$ minikube start --driver=docker
```

---

### **Repository for Reference**  
The source code is available at:  
[Full-Stack Chat Application](https://github.com/iemafzalhassan/full-stack_chatApp)  

---

### **Docker Image Creation and Pushing to Docker Hub**  
1. **Backend**:  
   ```bash
   $ docker build -t pranjalkumar09/chatapp-backend:latest .
   $ docker push pranjalkumar09/chatapp-backend:latest
   ```  
2. **Frontend**:  
   ```bash
   $ docker build -t pranjalkumar09/chatapp-frontend:latest .
   $ docker push pranjalkumar09/chatapp-frontend:latest
   ```  
3. **Database**:  
   Use the official MongoDB image directly without modifications.  

---

### **Setting Up Kubernetes Resources**  

#### **1. Creating the Namespace**  
```bash
$ kubectl create -f namespace.yml
```

#### **2. Configuring Deployments**  
Ensure the `spec` section in Deployment files specifies:  
- Correct container ports  
- Required environment variables (from Dockerfiles and `.env`):  

For **MongoDB**:  
```yaml
env:
- name: MONGO_INITDB_ROOT_USERNAME
  value: "root"
- name: MONGO_INITDB_ROOT_PASSWORD
  value: "password"
```

For **Backend**:  
```yaml
env:
- name: MONGODB_URI
  value: "mongodb://root:password@mongodb"
- name: JWT_SECRET
  value: "your_jwt_secret"
- name: PORT
  value: "5001"
```

#### **3. Exposing Services**  
- Use `ClusterIP` or `NodePort` based on requirements.  
- MongoDB will default to port `27017`.  

---

### **Running Frontend and Backend Services Locally**  
After setting up the services:  
```bash
$ sudo -E kubectl port-forward service/frontend -n chat-app 80:80 &
```  
This will run the **Frontend** on `localhost`.  

For the **Backend**, ensure MongoDB is properly connected using the `MONGODB_URI` environment variable:  
```yaml
env:
- name: MONGODB_URI
  value: "mongodb://root:password@mongodb"
```

---

### **Ingress for Accessing Multiple Services**  
An ingress configuration can be added to manage routing to both the frontend and backend services.  

---

### **Command Workflow for Resource Management**  
1. Clean up existing resources:  
   ```bash
   kubectl delete -f namespace.yml
   ```  
2. Apply Kubernetes configurations in the following order:  
   ```bash
   kubectl apply -f namespace.yml
   kubectl apply -f secret.yml
   kubectl apply -f mongodb-pv.yml
   kubectl apply -f mongodb-pvc.yml
   kubectl apply -f mongodb-deployment.yml
   kubectl apply -f mongodb-service.yml
   kubectl apply -f backend-deployment.yml
   kubectl apply -f backend-service.yml
   kubectl apply -f frontend-deployment.yml
   kubectl apply -f frontend-service.yml
   ```  
3. Run the frontend service locally:  
   ```bash
   sudo -E kubectl port-forward service/frontend -n chat-app 80:80 &
   ```

---

### **Common Issues and Solutions**  
1. **Namespace Mismatch**:  
   Ensure all resource configurations (e.g., Deployments, Services) use the correct namespace (`chat-app`).  

2. **Missing Environment Variables**:  
   Check for correct spelling and inclusion of essential environment variables like `JWT_SECRET`, `MONGODB_URI`, and `PORT`.  

3. **Backend `/api` Not Found**:  
   If `/api` returns "Not Found," it indicates the backend is accessible but no routes are configured for the requested path.  

---

### **Recommendations for Scaling and Decoupling**  
1. Use an **API Gateway** or reverse proxy for dynamic routing.  
2. Leverage **Helm Charts** for managing Kubernetes configuration templates.  
3. Adopt a **microservices architecture** for scalability and fault tolerance.  

