# Kubernetes Kustomize Web App Deployment

This project demonstrates using Kubernetes Kustomize to deploy a web application (`kennethreitz/httpbin`) across **dev**, **staging**, and **prod** environments with environment-specific customizations.

## Features
- Base configuration with a Deployment and Service.
- Dev: 2 replicas, debug port, low resources.
- Staging: 3 replicas, HPA for autoscaling.
- Prod: 5 replicas, Ingress for external access.
- Uses `kennethreitz/httpbin` as the application.

## Prerequisites
- Kubernetes cluster (e.g., Minikube, Killercoda)
- `kubectl`
- NGINX Ingress controller

## Setup
```bash
kubectl create namespace dev staging prod

# Preview or apply any environment
kubectl kustomize my-app/overlays/dev
kubectl apply -k my-app/overlays/dev

kubectl kustomize my-app/overlays/staging
kubectl apply -k my-app/overlays/staging

kubectl kustomize my-app/overlays/prod
kubectl apply -k my-app/overlays/prod
