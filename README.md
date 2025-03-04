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

### The Application that Prometheus will be monitoring

![app](https://github.com/Princeton45/monitor-custom-app/blob/main/images/app.png)


## Implementation Steps

### 1. Configuring Node.js Application for Metrics Collection
I integrated the Prometheus client library into my Node.js application to collect and expose metrics. The application now exposes a `/metrics` endpoint that Prometheus can scrape.

The metrics the app will be exposing to Prometheus:

1) The number of requests the app. is getting.
2) The duration of the requests (basically, how long the app. takes to handle a request).

First I added the `prom-client` API library in the `package.json` file. Prom-client is the official Prometheus client library used for Node.js applications to expose metrics to Prometheus.

```json
{
  "name": "node-project",
  "version": "1.0.0",
  "description": "",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "author": "NJ",
  "license": "ISC",
  "dependencies": {
    "express": "4.17.1",
    "prom-client": "13.1.0"
  }
}
```

I then added the below code to `server.js`

```js
const client = require('prom-client');
const collectDefaultMetrics = client.collectDefaultMetrics;
// Probe every 5th second.
collectDefaultMetrics({ timeout: 5000 });

const httpRequestsTotal = new client.Counter({
  name: 'http_request_operations_total',
  help: 'Total number of Http requests'
})

const httpRequestDurationSeconds = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of Http requests in seconds',
  buckets: [0.1, 0.5, 2, 5, 10]
})

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType)
  res.end(await client.register.metrics())
})

app.get('/', function (req, res) {
    // Simulate sleep for a random number of milliseconds
    var start = new Date()
    var simulateTime = Math.floor(Math.random() * (10000 - 500 + 1) + 500)

    setTimeout(function(argument) {
      // Simulate execution time
      var end = new Date() - start
      httpRequestDurationSeconds.observe(end / 1000); //convert to seconds
    }, simulateTime)

    httpRequestsTotal.inc();
    res.sendFile(path.join(__dirname, "index.html"));
});
```

Below is a screenshot of the `/metrics` endpoint being exposed on the application so Prometheus can pull from it

![metrics](https://github.com/Princeton45/monitor-custom-app/blob/main/images/metrics.png)


### 2. Containerizing the Application
I created a Dockerfile for my Node.js application and built a Docker image. This image was then pushed to Docker Hub for use in my Kubernetes deployment.

```Dockerfile
FROM node:13-alpine

RUN mkdir -p /usr/app

COPY package*.json /usr/app/
COPY app /usr/app/

WORKDIR /usr/app

EXPOSE 3000

RUN npm install
CMD ["node", "server.js"]
```

![docker-build](https://github.com/Princeton45/monitor-custom-app/blob/main/images/docker-build.png)

![docker-hub](https://github.com/Princeton45/monitor-custom-app/blob/main/images/docker-hub.png)

### 3. Kubernetes Deployment
I deployed the Node.js application to my Kubernetes cluster using deployment manifests. The configuration included appropriate service definitions to expose the metrics endpoint.

But first, I needed to create a secret in the EKS environment so that my cluster can be authenticated to pull from my private Dockerhub registry

`kubectl create secret docker-registry my-registry-key --docker-server=https://index.docker.io/v1/ --docker-username=prince450 --docker-password=<password>`

Then I added the secret to the deployment section in `k8s-config.yaml`

```yaml
spec:
      imagePullSecrets:
      - name: my-registry-key
```

![app-running](https://github.com/Princeton45/monitor-custom-app/blob/main/images/app-running.png)

*[Suggested image: kubectl get pods/services output showing running application]*

### 4. Prometheus Configuration
I configured Prometheus to discover and scrape metrics from my application by setting up the appropriate scrape configuration and deploying Prometheus to the Kubernetes cluster.

I first created the ServiceMonitor component that will point to the /metrics endpoint in the Node app.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: monitoring-node-app
  labels:
    release: monitoring
    app: nodeapp
spec:
  endpoints:
  - path: /metrics
    port: service
    targetPort: 3000
  namespaceSelector:
    matchNames:
    - default
  selector:
    matchLabels:
      app: nodeapp
```

![service-monitor](https://github.com/Princeton45/monitor-custom-app/blob/main/images/service-monitor.png)

Prometheus automatically discovered the app from the service monitor and now the app target is in Prometheus:

![target](https://github.com/Princeton45/monitor-custom-app/blob/main/images/target.png)


The metrics are now able to be viewed from the `/metrics` endpoint of the application on Prometheus


![query](https://github.com/Princeton45/monitor-custom-app/blob/main/images/query.png)


### 5. Grafana Dashboard Creation
I set up Grafana with Prometheus as a data source and created custom dashboards to visualize the application metrics.

I created a `Requests Per Second` panel and a `Requests Duration` panel using PromQl query

![grafana](https://github.com/Princeton45/monitor-custom-app/blob/main/images/grafana.png)
