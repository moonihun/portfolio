## 1. Gateway API CRD 설치

```bash
kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
  { kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.3.0" | kubectl apply -f -; }
```

## 2. Gateway API 구현체 설치

#### 2-1. Istio Gateway API

```bash
# istio-base
helm install istio-base istio/base -n istio-system --set defaultRevision=default --create-namespace

# istiod
helm install istiod istio/istiod -n istio-system --wait
```

#### 2-2. NGINX Gateway Fabric

```bash
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
  --create-namespace -n nginx-gateway \
  --set nginx.service.type=NodePort \
  --set-json 'nginx.service.nodePorts=[{"port":30000,"listenerPort":80},{"port":30001,"listenerPort":443}]'

# Or
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
  --create-namespace -n nginx-gateway \
  -f nginx-gateway-fabric/values.yaml
```

## 3. Gateway

#### 2-1. Istio Gateway API

```bash
kubectl create namespace istio-ingress

cat <<EOF | kubectl apply -f -
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: gateway
  namespace: istio-ingress
  annotations:
    oci.oraclecloud.com/load-balancer-type: "lb"
    service.beta.kubernetes.io/oci-load-balancer-shape: "flexible"
    service.beta.kubernetes.io/oci-load-balancer-shape-flex-min: "10"
    service.beta.kubernetes.io/oci-load-balancer-shape-flex-max: "10"
    service.beta.kubernetes.io/oci-load-balancer-backend-protocol: "HTTP"
    service.beta.kubernetes.io/oci-load-balancer-idle-timeout: "600"
spec:
  gatewayClassName: istio
  listeners:
    - name: https
      hostname: "*.example.com"
      port: 443
      protocol: HTTPS
      tls:
        mode: Terminate
        certificateRefs:
          - name: cluster-wildcard-tls
      allowedRoutes:
        namespaces:
          from: All
    - name: http
      hostname: "*.example.com"
      port: 80
      protocol: HTTP
      allowedRoutes:
        namespaces:
          from: All
EOF
```

#### 2-2. NGINX Gateway Fabric

```bash

cat <<EOF | kubectl apply -f -
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: default-gateway
  namespace: nginx-gateway
  annotations:
    oci.oraclecloud.com/load-balancer-type: "lb"
    service.beta.kubernetes.io/oci-load-balancer-shape: "flexible"
    service.beta.kubernetes.io/oci-load-balancer-shape-flex-min: "10"
    service.beta.kubernetes.io/oci-load-balancer-shape-flex-max: "10"
    service.beta.kubernetes.io/oci-load-balancer-backend-protocol: "HTTP"
    service.beta.kubernetes.io/oci-load-balancer-idle-timeout: "600"
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
      allowedRoutes:
        namespaces:
          from: All
EOF
```

## 4. Nginx HTTPRoute 테스트

```bash
# Nginx Deployment 생성
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
EOF

# Nginx Service 생성
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
spec:
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
    targetPort: 80
EOF

# HTTPRoute 생성
cat <<EOF | kubectl apply -f -
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: nginx-route
  namespace: default
spec:
  parentRefs:
  - name: gateway
    namespace: istio-ingress
    sectionName: https
  hostnames: ["nginx.example.com"]
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
