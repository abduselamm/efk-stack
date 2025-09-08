Here’s your content neatly formatted in Markdown:

````markdown
# 📜 Deploying Kibana on Kubernetes for Log Visualization

In the vast sea of data that modern applications generate, logs hold a treasure trove of insights. Visualizing and analyzing these logs can be like navigating through a dense fog without a compass. This is where **Kibana**, a powerful open-source visualization tool, comes into play.  

When paired with **Elasticsearch** for storage and search, and **Kubernetes** for orchestration, it forms a robust setup for managing application logs. This guide will walk you through deploying Kibana on Kubernetes to tap into the insights your logs offer.

---

## ⛪ Understanding Kibana

Imagine Kibana as the lens that brings the details of your data into focus, allowing you to navigate through the complexities with ease.  

It connects to Elasticsearch, where your logs are stored, and lets you create visualizations such as charts and graphs. These visualizations are then organized into **dashboards**, providing you with insights at a glance.

---

## ✍ Architecture Diagram


![Kibana Architecture](./kibana-architecture.png)


---

## 🚀 Deploying Kibana

Deploying Kibana involves creating a **Deployment** and a **Service** in Kubernetes.  

- The **Deployment** ensures Kibana is running and manages replicas.  
- The **Service** makes Kibana accessible.

### Creating the Kibana Deployment

**Configuration File:** Create a configuration file for the Kibana Deployment and name it `kibana-deployment.yaml`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: docker.elastic.co/kibana/kibana:7.10.0
        env:
          - name: ELASTICSEARCH_HOSTS
            value: "http://elasticsearch:9200"
        ports:
        - containerPort: 5601
````

This YAML file defines a Deployment named `kibana` with a single replica.
It uses the official Kibana image and connects to Elasticsearch via the `ELASTICSEARCH_HOSTS` environment variable.

* Kibana uses **port 5601** by default.
* It communicates with Elasticsearch using an HTTP REST API.
* To secure Kibana, enable the security features of Elasticsearch.

**Deploying Kibana:**

```bash
kubectl apply -f kibana-deployment.yaml
```

---

### Exposing Kibana Through a Service

After deploying Kibana, you need to make it accessible. This is done by creating a **Kubernetes Service**.

**Service Configuration File:** Create a file named `kibana-service.yaml`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kibana
spec:
  type: NodePort
  ports:
  - port: 5601
    targetPort: 5601
    nodePort: 30001
  selector:
    app: kibana
```

This Service exposes Kibana on a NodePort, making it accessible from outside the cluster.

**Creating the Service:**

```bash
kubectl apply -f kibana-service.yaml
```

---

## ✨ Accessing Kibana

With Kibana deployed and exposed, you can now access its interface using the IP address of any node in your cluster and the node port specified in the Service configuration:

```
http://<node-ip>:30001
```

* To explore raw logs shipped by **Fluentd**, open the **Discover** tab.
* Use **Kibana Query Language (KQL)** to search your data.
* You must first specify an **index pattern** to select which data to explore.

Once explored, you can:

* Create **dashboards** to aggregate your data.
* **Import or export dashboards** (exported in `.ndjson` format).
