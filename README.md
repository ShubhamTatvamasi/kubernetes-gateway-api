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
> This also installs `nginx` GatewayClass

Deploy nginx:
```bash
kubectl create deployment nginx --image=nginx:alpine
kubectl expose deployment nginx --port=80
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

Create nginx Route:
```bash
kubectl apply -f - << EOF
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: nginx-route
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: nginx
      port: 80
EOF
```


