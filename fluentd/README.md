🚿 Deploying Fluentd on Kubernetes as a DaemonSet
Welcome to the third lab of our hands-on course! 🎉 In this lab, we'll explore how to deploy Fluentd on Kubernetes as a DaemonSet. This ensures that a Fluentd instance runs on every node, enabling efficient log collection from both the nodes and the pods. We'll also cover how to configure Fluentd to forward these logs to Elasticsearch, providing a centralized logging solution. 📚

🤔 Introduction to Fluentd and Kubernetes
Before we dive into the deployment process, let's understand the key components involved.

What is Fluentd?
Fluentd is an open-source data collector designed for processing logs and events. It's highly flexible, allowing you to collect data from multiple sources, transform it as needed, and send it to various destinations.

Why Kubernetes?
Kubernetes is a container orchestration platform that automates the deployment, scaling, and management of containerized applications. By deploying Fluentd on Kubernetes, we can harness its scalability and management features for efficient log collection.

🚀 Preparing Your Kubernetes Cluster
Ensure your Kubernetes cluster is up and running by executing the following command in the attached terminal:

kubectl get nodes
You should see controlplane in READY state in the output.

🌿 Deploying Fluentd as a DaemonSet
A DaemonSet ensures that a copy of the pod runs on each node in the cluster. This is perfect for log collection, as we want Fluentd to collect logs from every node.

Creating the Fluentd DaemonSet
Create a Fluentd Configuration File
First, create a Fluentd configuration file (say fluentd-config.conf) that specifies how logs should be collected and forwarded.

<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>
<match **>
  @type elasticsearch
  host elasticsearch.default.svc.cluster.local
  port 9200
  logstash_format true
</match>
Here, source and match are directives. Following is a list of all directives and the respective aspects of logging they relate to:

source: Input sources.
match: Output destinations.
filter: Event processing pipelines.
system: System-wide configuration.
label: Group the output and filter for internal routing.
worker: Limit to the specific workers.
@include: Include other files.
Apart from directives, fluentd also uses various input and output plugins.

Define the DaemonSet
Next, create a Kubernetes DaemonSet definition (say fluentd-daemonset.yaml). This file describes the Fluentd pod that will be deployed to each node.

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.11-debian-elasticsearch
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch.default.svc.cluster.local"
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: dockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: dockercontainers
        hostPath:
          path: /var/lib/docker/containers
Deploy the DaemonSet
Apply the DaemonSet definition to your cluster.

kubectl apply -f fluentd-daemonset.yaml
⚙️ Configuring Fluentd for Kubernetes Logs
Fluentd needs to be configured to collect logs from both Kubernetes nodes and pods. The configuration we defined earlier in fluentd-config.conf sets Fluentd to listen for logs and forward them to Elasticsearch.

Collecting Node Logs
The DaemonSet configuration mounts the /var/log directory from the host into the Fluentd container, allowing Fluentd to access and collect system and application logs from the nodes.

Collecting Pod Logs
Similarly, mounting /var/lib/docker/containers enables Fluentd to collect logs from Docker containers, including those managed by Kubernetes pods.

Forwarding Logs to Elasticsearch
The Fluentd configuration specifies Elasticsearch as the destination for logs. Ensure that Elasticsearch is running and accessible from within your Kubernetes cluster.

<match **>
  @type elasticsearch
  host elasticsearch.default.svc.cluster.local
  port 9200
  logstash_format true
</match>
The above configuration forwards all collected logs to Elasticsearch, where they can be indexed and made searchable.

This has been made possible because of the use of output plugin fluent-plugin-elasticsearch which is used to forward data to Elasticsearch. The match section displays the parameters it uses.
