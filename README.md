### **Step 1: Setup Monitoring Namespace and Resources**

1. **Create the Monitoring Namespace:**
   ```bash
   kubectl create namespace monitoring
   ```

2. **Apply Necessary Resources for Built-In Monitoring:**
   ```bash
   kubectl apply -n monitoring -f grafana.yaml -f kube-state-metrics.yaml
   kubectl apply -n monitoring -f prom-configmap.yaml -f prom-rbac.yaml -f prom-svc.yaml -f prom-deployment.yaml
   ```

3. **Configure MySQL for Monitoring:**
   In your MySQL object YAML, enable monitoring by adding the following specification under `spec`:

   ```yaml
   spec:
     monitor:
       agent: prometheus.io/builtin
   ```

### **Step 2: Port Forwarding and Grafana Configuration**

1. **Port Forward Grafana and Prometheus:**

   - Forward Grafana:
     ```bash
     kubectl port-forward -n monitoring service/grafana 3000:80
     ```

   - Forward Prometheus:
     ```bash
     kubectl port-forward -n monitoring service/prometheus 9090
     ```

2. **Access Grafana:**
   - Open your browser and go to [http://localhost:3000](http://localhost:3000).
   - Log in using the credentials:
     - **Username:** `admin`
     - **Password:** `prom-operator`

3. **Create an API Key in Grafana:**
   - Navigate to **Configuration** > **API Keys**.
   - Create a new API key, copy and save it.

4. **Add Prometheus as a Data Source in Grafana:**
   - Go to **Configuration** > **Data Sources**.
   - Add a new data source with the following details:
     - **Name:** Prometheus
     - **URL:** `http://prometheus.monitoring.svc:9090`
   - Test and save the data source.

5. **Update Grafana YAML with API Key and MySQL Details:**
   - In `grafana-mysql.yaml`, set the API key and specify the MySQL database name and namespace under the `app` field.

   ```yaml
   grafana:
     apikey: <your-api-key>
   app:
     name: <your-mysql-database-name>
     namespace: <your-mysql-namespace>
   ```

### **Step 3: Install Panopticon, KubeDB Metrics, and Grafana Dashboards**

1. **Install Panopticon using Helm:**
   ```bash
   helm upgrade -i panopticon appscode/panopticon \
     -n kubeops --create-namespace --version=v2024.2.5 \
     --set monitoring.enabled=true \
     --set monitoring.agent=prometheus.io/builtin \
     --set-file license=kubedb-license-file.txt
   ```

2. **Install KubeDB Metrics using Helm:**
   - Add the AppsCode Helm repository and update:
     ```bash
     helm repo add appscode https://charts.appscode.com/stable/
     helm repo update
     ```
   - Install the KubeDB metrics:
     ```bash
     helm upgrade -i kubedb-metrics appscode/kubedb-metrics \
       -n kubedb --create-namespace --version=v2024.6.4
     ```

3. **Install KubeDB Grafana Dashboards:**
   ```bash
   helm upgrade -i kubedb-grafana-dashboards appscode/kubedb-grafana-dashboards \
     -n kubeops --create-namespace --version=v2024.6.4 \
     --values ./grafana-mysql.yaml
   ```