## 1. Cert-Manager 설치

```bash
helm install cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace \
--set crds.enabled=true
```

## 2. Reflector 설치

- Reflector Annotation을 사용하면 특정 또는 모든 Namespace에 인증서 복제 가능

```bash
kubectl create ns emberstack

helm upgrade --install --namespace emberstack reflector emberstack/reflector
```

## 3. Cluster Issuer 생성

```bash
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-dns01
spec:
  acme:
    email: your-email@domain.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-dns01
    solvers:
      - dns01:
          cloudflare:
            email: your-email@domain.com
            apiTokenSecretRef:
              name: cloudflare-api-token-secret
              key: api-token
```

## 4. Certificate 리소스 생성

```bash
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: cluster-wildcard
  namespace: istio-ingress
spec:
  secretName: cluster-wildcard-tls
  issuerRef:
    name: letsencrypt-dns01
    kind: ClusterIssuer
  commonName: "*.example.com"
  dnsNames:
    - "*.example.com"
    - "example.com"
    secretTemplate:
    annotations: # 다른 Namespace에도 인증서 복제
      reflector.v1.k8s.emberstack.com/reflection-allowed: "true"
      reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces: ""
      reflector.v1.k8s.emberstack.com/reflection-auto-enabled: "true"
      reflector.v1.k8s.emberstack.com/reflection-auto-namespaces: ""
```
