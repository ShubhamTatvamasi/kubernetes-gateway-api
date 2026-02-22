# kubernetes-gateway-api

Install Gateway API CRDs:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/latest/download/standard-install.yaml
```

Verify that `gateway.networking.k8s.io/v1` has been installed on cluster:
```bash
kubectl api-resources | grep gateway
```

Install Nginx Gateway Fabric Controller:
```bash
helm upgrade -i nginx-gateway \
  oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
  --create-namespace \
  --namespace nginx-gateway \
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

---

### Minikube Setup

Install metallb:
```bash
helm repo add metallb https://metallb.github.io/metallb
helm repo update

helm upgrade -i metallb metallb/metallb \
  --create-namespace \
  --namespace metallb-system \
  --set speaker.enabled=false
```

Create IP Pool:
```bash
kubectl create -f - << EOF
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: ip-pool
spec:
  addresses:
  - $(kubectl get nodes -o jsonpath='{ $.items[*].status.addresses[?(@.type=="InternalIP")].address }')/32
EOF
```
> Use `ExternalIP` for public network.

Start minikube tunnel for LoadBalancer service:
```bash
minikube tunnel
```

Access the home page http://localhost/



