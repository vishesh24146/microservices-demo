# Online Boutique on Google Cloud Platform (GKE)

![Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo/raw/main/docs/img/online-boutique-frontend-1.png)

## Overview

This guide provides step-by-step instructions for deploying Google's Online Boutique demo application on Google Kubernetes Engine (GKE). Online Boutique is a cloud-native microservices demo application that simulates an e-commerce website.

## Prerequisites

- Google Cloud Platform account with billing enabled
- `gcloud` CLI installed and configured
- `kubectl` CLI installed
- `git` installed
- `istioctl` installed

## Deployment Instructions

### Setting up the Environment

1. Login to your Google Cloud Console account.

2. Create a new project in Google Cloud:
   ```
   # Example project name: grspro
   # Set region to Asia-Southeast1
   ```

3. Configure your gcloud CLI to use the new project:
   ```bash
   gcloud config set project grspro
   ```

4. Export environment variables for future commands:
   ```bash
   export PROJECT_ID=grspro
   export REGION=asia-southeast1-a
   ```

5. Enable the Google Container API:
   ```bash
   gcloud services enable container.googleapis.com --project=${PROJECT_ID}
   ```

6. Create a GKE cluster:
   ```bash
   gcloud container clusters create grsonlineb01 \
       --zone=asia-southeast1-a \
       --num-nodes=3 \
       --machine-type=e2-standard-4 \
       --enable-autoscaling \
       --min-nodes=1 \
       --max-nodes=5
   ```

### Deploying Online Boutique

1. Clone the Online Boutique repository:
   ```bash
   git clone --depth 1 --branch v0 https://github.com/GoogleCloudPlatform/microservices-demo.git
   ```

2. Install Istio service mesh:
   ```bash
   istioctl install
   ```

3. Enable Istio sidecar injection for the default namespace:
   ```bash
   kubectl label namespace default istio-injection=enabled
   ```

4. Deploy the application:
   ```bash
   cd microservices-demo/release
   kubectl apply -f kubernetes-manifests.yaml
   ```

5. Verify the pods are created:
   ```bash
   kubectl get pods
   ```

6. Get the frontend service's external IP:
   ```bash
   kubectl get svc
   ```
   *Note: Copy the external IP of the frontend service to access the application*

### Installing Monitoring Tools

1. Download Istio source files:
   ```bash
   curl -L https://istio.io/downloadIstio | sh -
   cd istio-1.25.2  # Version may differ
   ```

2. Apply the monitoring addons:
   ```bash
   kubectl apply -f samples/addons
   ```

3. View the installed monitoring services:
   ```bash
   kubectl get svc -n istio-system
   ```

4. Expose monitoring services with external IPs:
   ```bash
   kubectl patch svc kiali -n istio-system -p '{"spec": {"type": "LoadBalancer"}}'
   kubectl patch svc grafana -n istio-system -p '{"spec": {"type": "LoadBalancer"}}'
   ```

5. Access monitoring dashboards:
   - Kiali: `http://<external-IP>:20001`
   - Grafana: `http://<external-IP>:3000`
   - Frontend: `http://<external-IP>`

### Generating Load for Testing

1. Navigate to the load generator directory:
   ```bash
   cd microservices-demo/src/loadgenerator
   ```

2. Create and activate a Python virtual environment:
   ```bash
   python -m venv myenv
   source myenv/bin/activate
   ```

3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

4. Launch Locust load generator:
   ```bash
   locust -f locustfile.py
   ```

5. Access the Locust web interface at `http://localhost:8089`

6. Configure load testing:
   - Set target host to the frontend's external IP
   - Configure number of users and spawn rate
   - Start the load test

## Architecture

The application consists of multiple microservices written in different languages that communicate with each other using gRPC and REST.

![Architecture Diagram](https://github.com/GoogleCloudPlatform/microservices-demo/raw/main/docs/img/architecture-diagram.png)

## Troubleshooting

- If pods are stuck in pending state, check cluster resources with `kubectl describe nodes`
- For service connectivity issues, verify network policies with `kubectl get networkpolicies`
- Check logs of specific services with `kubectl logs <pod-name>`

## Contributors

- Vishesh Joshi
- Rishabh Sharma
- Parul
- Sayantan Dasgupta

## Credits

Source code and demo application provided by Google Cloud Platform:
[https://github.com/GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)
