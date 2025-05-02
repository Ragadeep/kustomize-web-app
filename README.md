# Kubernetes Kustomize: Multi-Environment Web App Deployment

This project demonstrates the use of **Kubernetes Kustomize** to deploy a web application (`kennethreitz/httpbin`) across three environments—**dev, staging**, and **prod**—with environment-specific customizations. Built in a sandbox environment, it showcases Kustomize's power for managing Kubernetes configurations, including replicas, HorizontalPodAutoscaler (HPA), and Ingress for external access.


## Features
- **Base Configuration:** Reusable Deployment and Service for the `kennethreitz/httpbin` app.
- **Dev Environment:**
  - 2 replicas, low resource limits (100m CPU, 128Mi memory).
  - Debug port (9229) for developer access.
  - Environment variable: `APP_ENV=development`.
- **Staging Environment:**
  - 3 replicas, moderate resources (250m CPU, 256Mi memory).
  - HPA to scale between 3–6 pods based on 70% CPU usage.
  - Environment variable: `APP_ENV=staging`.
- **Prod Environment:**
  - 5 replicas, high resources (500m CPU, 512Mi memory).
  - Ingress for external access via `myapp.example.com`.
  - Environment variable: `APP_ENV=production`.
- **GitOps-Ready:** YAML-based configurations for version control.

## Prerequisites
To run this project, you need:
- A Kubernetes cluster (e.g., Minikube, Kind, or a sandbox like Killercoda).
- `kubectl` installed and configured.
- NGINX Ingress controller for the prod environment:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```
- optional: `tree` for viewing directory structure (`sudo apt-get install tree`).
  

**Directory Structure**

```
my-app/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   ├── patch-deployment.yaml
│   │   ├── patch-service.yaml
│   │   └── kustomization.yaml
│   ├── staging/
│   │   ├── patch-deployment.yaml
│   │   ├── hpa.yaml
│   │   └── kustomization.yaml
│   └── prod/
│       ├── patch-deployment.yaml
│       ├── ingress.yaml
│       └── kustomization.yaml
```

## Setup Instructions
**1. Clone the Repository:**
```bash
git clone https://github.com/<your-username>/kustomize-web-app.git
cd kustomize-web-app
```

**2. Create Namespaces:**
```bash
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod
```

**3. Preview**
```bash
kubectl kustomize my-app/overlays/dev
kubectl kustomize my-app/overlays/stagng
kubectl kustomize my-app/overlays/prod
```

**4. Apply Configurations:**
- Deploy **dev:**
```bash
kubectl apply -k my-app/overlays/dev
```
- Deploy **staging:**
```bash
kubectl apply -k my-app/overlays/staging
```
- Deploy **prod:**
```bash
kubectl apply -k my-app/overlays/prod
```
  
## Testing Instructions
**1. Verify Pods**  
Check pod status in each environment:
```bash
kubectl get pods -n dev
kubectl get pods -n staging
kubectl get pods -n prod
```
![pods](https://github.com/user-attachments/assets/3ab5c258-6d95-4097-a869-43eddb4ca477)

**2. Verify Services**  
Check services in each environment:
```bash
kubectl get svc -n dev
kubectl get svc -n staging
kubectl get svc -n prod
```
![svc](https://github.com/user-attachments/assets/5807c26e-c202-4e48-8f84-d126c6621e9d)

**3. Verify HPA (Staging)**

Check the HorizontalPodAutoscaler in **staging**:
```bash
kubectl get hpa -n staging
```
![hpa](https://github.com/user-attachments/assets/161c4de3-aebb-4d97-82ca-95f9a0fc6cc5)

**4. Verify Ingress (Prod)**

Check the Ingress in **prod:**
```bash
kubectl get ingress -n prod
```

```
NAME          CLASS    HOSTS               ADDRESS   PORTS   AGE
prod-my-app   <none>   myapp.example.com             80      5m
```

**5. Test Application**

- **Dev (Port-Forward):**
```bash
kubectl port-forward svc/dev-my-app 8080:80 -n dev
```
![port forwarding](https://github.com/user-attachments/assets/70b8010d-3420-466c-bbfc-a7967f8ce27e)

In another terminal:
```bash
curl http://localhost:8080/get
```
![output](https://github.com/user-attachments/assets/a23c67e8-2a1a-4110-bdd2-a64209412a1e)

- **Prod (Ingress):** Set up `/etc/hosts` to map `myapp.example.com` to the node IP (e.g., `172.17.0.2`):
```bash
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
sudo sh -c "echo '$NODE_IP myapp.example.com' >> /etc/hosts"
```

Get the Ingress controller NodePort:
```bash
kubectl get svc -n ingress-nginx
```

**Example output:**
```
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort       10.96.123.789   <none>        80:31108/TCP,443:32345/TCP   10m
```

**Test:**
```bash
curl http://myapp.example.com:31108/get
```
![test with url](https://github.com/user-attachments/assets/3c02b59b-8c38-404c-afd7-6849d176489f)

![Screenshot 2025-05-02 at 12 02 44 AM](https://github.com/user-attachments/assets/d7b3f56f-4c64-4c34-8ab4-6e769c1a9df9)


## Challenges and Solutions
During the project, I encountered and resolved several issues:

**1. CrashLoopBackOff:**
  - **Problem:** Pods were crashing because the initial `node:18` image didn’t run a web server.
  - **Solution:** Switched to `kennethreitz/httpbin`, which serves HTTP on port 80.

**2. Ingress 404 Error:**
  - **Problem:** The Ingress referenced `my-app` instead of `prod-my-app` due to `namePrefix: prod`-.
  - **Solution:** Updated `overlays/prod/ingress.yaml` to use `service: name: prod-my-app`.

**3. Empty Curl Reply:**
  -** Problem:** Early tests with `node:18` and `sleep infinity` didn’t serve HTTP.
  - **Solution:** Ensured the correct image and port alignment (80).


## Learning
- **kustomize:** Mastered base/overlay patterns for environment-specific configurations.
- **Kubernetes Debugging:** Learned to use `kubectl describe`, `logs`, and `port-forward` to troubleshoot.
- **Ingress:** Understood NGINX Ingress controller setup and routing.
- **GitOps:** Gained experience with YAML-based, version-controlled deployments.

## Contributing

This repository is a personal project for demonstrating Kubernetes Kustomize skills and is not open to contributions or pull requests. However, feel free to fork the repository for your own use or reach out with feedback via [LinkedIn](https://www.linkedin.com/in/ragadeep-pola)!

## License

This project is licensed under the MIT License. See the [LICENSE](my-app/LICENSE) file for details.

## Contact

Connect with me on [LinkedIn](https://www.linkedin.com/in/ragadeep-pola) for questions, feedback, or to discuss Kubernetes and DevOps!


