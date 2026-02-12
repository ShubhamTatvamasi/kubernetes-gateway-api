# kubernetes-gateway-api

Install Gateway API CRDs:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/latest/download/standard-install.yaml
```

Install Nginx Gateway Fabric Controller:
```bash
helm install nginx-gateway-fabric \
  --create-namespace \
  --namespace nginx-gateway-fabric \
  oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
  --version 2.4.1
```
