# DevSecOps Exam

Architecture: GitHub -> Argo CD -> Kubernetes -> Deployment (4 replicas) -> Service -> NGINX Ingress -> HTTPS

## Files
- k8s/namespace.yml
- k8s/deployment.yml
- k8s/service.yml
- k8s/ingress.yml
- k8s/ingress-tls.yml
- k8s/argocd-application.yml

## Before pushing
Edit `k8s/argocd-application.yml` and replace `YOUR_GITHUB_USERNAME` with your GitHub username.

Do NOT commit TLS private keys/certificates.

## First test
```powershell
kubectl apply -f k8s/namespace.yml
kubectl apply -f k8s/deployment.yml
kubectl apply -f k8s/service.yml
kubectl get pods -n devsecops
```
Expected: 4 NGINX Pods.

## Argo CD
After Argo CD is installed:
```powershell
kubectl apply -f k8s/argocd-application.yml
kubectl get applications -n argocd
```

## HTTPS
Create the TLS secret separately; do not store its private key in Git.
Example with OpenSSL:
```powershell
openssl req -x509 -nodes -days 365 -newkey rsa:2048 `
  -keyout nginx.local.key `
  -out nginx.local.crt `
  -subj "/CN=nginx.local" `
  -addext "subjectAltName=DNS:nginx.local"

kubectl create secret tls nginx-tls `
  --cert=nginx.local.crt `
  --key=nginx.local.key `
  -n devsecops
```
Then:
```powershell
kubectl apply -f k8s/ingress-tls.yml
```

For production, use a trusted certificate issuer instead of a self-signed certificate.
