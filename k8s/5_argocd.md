## 1. ArgoCD 설치

```bash
# Helm 차트를 이용한 argocd 설치
helm install argocd argo/argo-cd --namespace argocd --create-namespace

# inseucre 모드 적용
# Gateway가 TLS를 Terminate(복호화) 후 백엔드로 평문 HTTP 보내기 때문
kubectl patch configmap argocd-cmd-params-cm -n argocd --type merge -p '{"data":{"server.insecure":"true"}}'

# argocd-server 재시작
kubectl rollout restart deployment argocd-server -n argocd

# 초기 패스워드 확인
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## 2. HTTPRoute 생성

```bash
cat <<EOF | kubectl apply -f -
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: argocd-route
  namespace: argocd
spec:
  parentRefs:
    - name: istio-gateway
      namespace: istio-ingress
      sectionName: https
  hostnames: ["argocd.example.com"]
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: argocd-server
          port: 443
EOF
```

## 3. Github Private Repository 연결

### SSH Key 생성

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### Github Private Repository Deploy key 등록

- 생성한 ed25519.pub key 등록

### ArgoCD Rerpository 등록

- Repository URL 등록 시 SSH 형식으로 작성
  - `git@github.com:yours/your-repository`
- 생성한 ed25519 private key 등록
