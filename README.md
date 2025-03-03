# Monitoring a custom application

## Overview
In this project, I implemented a comprehensive monitoring solution for my Node.js application using Prometheus and Grafana. The monitoring system runs within a Kubernetes cluster and provides real-time metrics visualization.

![diagram](https://github.com/Princeton45/monitor-custom-app/blob/main/images/diagram.png)

## Technologies Used
- Node.js
- Prometheus
- Kubernetes
- Docker
- Docker Hub
- Grafana

## Implementation Steps

### 1. Configuring Node.js Application for Metrics Collection
I integrated the Prometheus client library into my Node.js application to collect and expose metrics. The application now exposes a `/metrics` endpoint that Prometheus can scrape.

*[Suggested image: Screenshot of the Node.js code showing Prometheus integration]*

### 2. Containerizing the Application
I created a Dockerfile for my Node.js application and built a Docker image. This image was then pushed to Docker Hub for use in my Kubernetes deployment.

*[Suggested image: Docker build and push commands output]*

### 3. Kubernetes Deployment
I deployed the Node.js application to my Kubernetes cluster using deployment manifests. The configuration included appropriate service definitions to expose the metrics endpoint.

*[Suggested image: kubectl get pods/services output showing running application]*

### 4. Prometheus Configuration
I configured Prometheus to discover and scrape metrics from my application by setting up the appropriate scrape configuration and deploying Prometheus to the Kubernetes cluster.

*[Suggested image: Prometheus targets page showing the application endpoint being scraped successfully]*

### 5. Grafana Dashboard Creation
I set up Grafana with Prometheus as a data source and created custom dashboards to visualize the application metrics.

*[Suggested image: Screenshot of the Grafana dashboard showing application metrics]*

## Results
The monitoring system now provides real-time visibility into my application's performance. Key metrics are visualized through custom Grafana dashboards, enabling quick identification of potential issues and performance bottlenecks.

*[Suggested image: Full monitoring pipeline overview showing data flow from application to Prometheus to Grafana]*

## Future Improvements
- Set up alerting based on metric thresholds
- Add more detailed application-specific metrics
- Implement logging integration with the monitoring system