# kubernetes-gateway-api

Install Gateway API CRDs:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/latest/download/standard-install.yaml
```

Install Nginx Gateway Fabric Controller:
```bash
helm install nginx-gateway-fabric \
  --create-namespace \
  --namespace nginx-gateway \
  oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
  --version 2.4.1
```

Create nginx Gateway:
```bash
kubectl apply -f - << EOF
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
    allowedRoutes:
      namespaces:
        from: All
EOF
```
