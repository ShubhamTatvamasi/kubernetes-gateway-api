# TLS

Create a ClusterIssuer:
```yaml
kubectl apply -f - << EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
    - http01:
        gatewayHTTPRoute:
          parentRefs:
          - name: nginx-gateway
            namespace: default
EOF
```

Create TLS Gateway:
```yaml
kubectl apply -f - << EOF
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  annotations:
    cert-manager.io/issuer: letsencrypt-prod
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
    allowedRoutes:
      namespaces:
        from: All
  - name: https
    port: 443
    protocol: HTTPS
    hostname: rke2.shubhamtatvamasi.com
    tls:
      mode: Terminate
      certificateRefs:
      - name: rke2-shubhamtatvamasi-com-tls
    allowedRoutes:
      namespaces:
        from: All
EOF
```

```yaml
kubectl apply -f - << EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: rke2-shubhamtatvamasi-com
spec:
  secretName: rke2-shubhamtatvamasi-com-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - rke2.shubhamtatvamasi.com
EOF
```


