---

# Three-Tier Architecture Deployment on **Azure AKS**

Stan's Robot Shop is a sample microservice application you can use as a sandbox to test and learn containerized application orchestration, monitoring, and observability techniques.
It is **not** intended to be a fully production-ready microservices reference implementation, but by deploying and experimenting with it, you will gain hands-on experience with AKS, Helm, CI/CD, and monitoring stacks.

To be clear, this sample is for **learning and experimentation** — the error handling is minimal and security hardening is not implemented by default.

You can get more detailed background from the original [blog post](https://www.instana.com/blog/stans-robot-shop-sample-microservice-application/) about the application.

---

## Application Stack

This microservices demo includes:

* NodeJS ([Express](http://expressjs.com/))
* Java ([Spring Boot](https://spring.io/))
* Python ([Flask](http://flask.pocoo.org))
* Golang
* PHP (Apache)
* MongoDB
* Redis
* MySQL ([Maxmind](http://www.maxmind.com) data)
* RabbitMQ
* Nginx
* AngularJS (1.x)

Instana agents are pre-configured for distributed tracing, metrics, and observability.
You’ll need an [Instana trial account](https://instana.com/trial?utm_source=github&utm_medium=robot_shop) to view performance dashboards.

---

## 1. Building from Source

If you want to build locally before deploying to AKS:

```bash
export INSTANA_AGENT_KEY="<your_instana_agent_key>"
docker-compose build
docker-compose push   # if pushing to your own Azure Container Registry
```

If you skip this step, pre-built images from Docker Hub will be used.

---

## 2. Running Locally

```bash
docker-compose pull
docker-compose up
```

Load generator (optional):

```bash
docker-compose -f docker-compose.yaml -f docker-compose-load.yaml up
```

---

## 3. Deploying to Azure AKS

1. **Create an AKS Cluster**

```bash
az group create --name robotshop-rg --location eastus
az aks create --resource-group robotshop-rg --name robotshop-cluster --node-count 3 --enable-addons monitoring --generate-ssh-keys
az aks get-credentials --resource-group robotshop-rg --name robotshop-cluster
```

2. **(Optional) Create Azure Container Registry (ACR)**

```bash
az acr create --resource-group robotshop-rg --name robotshopacr --sku Basic
az aks update -n robotshop-cluster -g robotshop-rg --attach-acr robotshopacr
```

3. **Deploy via Helm**

```bash
helm repo add robotshop https://<your-helm-chart-repo-or-local-path>
helm install robotshop ./K8s/helm
```

4. **Get the External IP**

```bash
kubectl get svc web
```

Access the app via:

```
http://<EXTERNAL-IP>
```

---

## 4. Monitoring & Observability

* **Prometheus & Grafana**: Integrated for metrics from `cart` and `payment` services.
* **Instana Agent on AKS**: Deploy via [Instana Helm chart](https://github.com/instana/helm-charts).
* **Jaeger**: Optional for distributed tracing.

---

## 5. Load Generation on AKS

```bash
kubectl apply -f K8s/load-gen.yaml
```

This uses [Locust](https://locust.io) inside AKS to simulate user traffic.

---

## 6. Website Monitoring / EUM

Set the following in your Helm values file:

```yaml
web:
  env:
    INSTANA_EUM_KEY: "<your_eum_key>"
    INSTANA_EUM_REPORTING_URL: "<your_eum_reporting_url>"
```

---

## 7. Prometheus Endpoints

* **Cart Service**: `http://<host>:8080/api/cart/metrics`
* **Payment Service**: `http://<host>:8080/api/payment/metrics`

---

This setup gives you a **full three-tier microservices architecture** on Azure AKS with CI/CD, monitoring, and load generation — a great foundation for real-world Kubernetes skills.


