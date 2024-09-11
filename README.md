# SSL-Certification
## Steps to Use Cert-Manager for SSL Certification

### 1. **Install Cert-Manager**:
You'll need to install Cert-Manager in your Kubernetes cluster. You can do this using `kubectl` or Helm.

To install with **kubectl**:
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
```

To install with **Helm**:
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.12.0
```

#### 2. **Configure the Issuer (Let's Encrypt)**:
You need to create an `Issuer` or `ClusterIssuer` resource in Kubernetes to configure how Cert-Manager should obtain certificates.

For Let's Encrypt, you can configure it like this:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: hllgksl@gmail.com  # Replace with your email
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx  # Or your ingress class
```

#### 3. **Request a Certificate for Your Domain**:
Create a `Certificate` resource that tells Cert-Manager to request an SSL certificate for your domain (`example.com`) from Let's Encrypt.

Example YAML file:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: your-cert
  namespace: your-namespace  # Replace with your namespace
spec:
  secretName: your-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: example.com
  dnsNames:
  - example.com
  - www.example.com
```

#### 4. **Configure Ingress for SSL**:
If you're using an Ingress controller, you need to configure it to use the newly obtained SSL certificate. Here's an example for an Nginx Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: your-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - example.com
    secretName: your-tls
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: your-service-name  # Replace with your service name
            port:
              number: 80
```

#### 5. **Verify the Certificate**:
Once the `Certificate` is created and the Ingress is updated, Cert-Manager will request and install the SSL certificate. You can verify the certificate status with the following command:

```bash
kubectl describe certificate your-cert
```

You should also access `https://example.com` in your browser to verify the SSL certificate is working properly.

### Key Points:
- **Cert-Manager** automates SSL certificate management, including issuing, renewing, and revoking certificates.
- **Let’s Encrypt** is a free, widely used provider for SSL certificates.
- Ensure that your domain points to the Kubernetes cluster where Cert-Manager is installed and the Ingress is configured.

If you're using Kubernetes for hosting your application, Cert-Manager simplifies the SSL certification process significantly.
