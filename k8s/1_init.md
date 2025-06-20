# Kubernetes

## 1. Kind 설치

### MacOS

```bash
# For Intel Macs
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-darwin-amd64

# For M1 / ARM Macs
[ $(uname -m) = arm64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-darwin-arm64
chmod +x ./kind
mv ./kind /some-dir-in-your-PATH/kind
```

### Linux

```bash
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-amd64

# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

## 2. Kind Cluster 생성

```bash
kind create cluster --config kind-cluster.yaml
```

## 3. Helm 설치

```bash
# For macOS
brew install helm

# For Linux (Debian/Ubuntu)
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

helm repo add `이름` `저장소`
```

# Helm Repo

| NAME           | URL                                                 |
| :------------- | :-------------------------------------------------- |
| bitnami        | https://charts.bitnami.com/bitnami                  |
| kubernetes     | https://charts.helm.sh/stable                       |
| metallb        | https://metallb.github.io/metallb                   |
| traefik        | https://traefik.github.io/charts                    |
| istio          | https://istio-release.storage.googleapis.com/charts |
| kiali          | https://kiali.org/helm-charts                       |
| caddy          | https://caddyserver.github.io/ingress/              |
| gloo           | https://storage.googleapis.com/solo-public-helm     |
| kong           | https://charts.konghq.com                           |
| geek-cookbook  | https://geek-cookbook.github.io/charts/             |
| metrics-server | https://kubernetes-sigs.github.io/metrics-server/   |
| argo           | https://argoproj.github.io/argo-helm                |

## 4. kubectl 설치

```bash
# For macOS
brew install kubcetl

# For Linux (Debian/Ubuntu)
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl

# 자동완성 / Alias For bash
source <(kubectl completion bash)
echo 'source <(kubectl completion bash)' >>~/.bashrc

echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc

# 자동완성 / Alias For zsh
source <(kubectl completion zsh)
echo 'source <(kubectl completion zsh)' >>~/.zshrc

echo 'alias k=kubectl' >>~/.zshrc
echo 'complete -o default -F __start_kubectl k' >>~/.zshrc
```

## 5. Krew 설치

```bash
# For macOS/Linux
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)

# For bash
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
source ~/.bash_profile

# For zsh
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
source >> ~/.zshrc
```

## 5. Krew 플러그인 설치

```bash
# kubectx
kubectl krew install ctx

# kubens
kubectl krew install ns

# neat
kubectl krew install neat
```

## 6. Gateway API CRD 설치

```bash
kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
  { kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.3.0" | kubectl apply -f -; }
```

## 7. Gateway API 구현체 설치

```bash
# NGINX Gateway Fabric
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
  --create-namespace -n nginx-gateway \
  --set nginx.service.type=NodePort \
  --set-json 'nginx.service.nodePorts=[{"port":30000,"listenerPort":80},{"port":30001,"listenerPort":443}]'

# Or
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
  --create-namespace -n nginx-gateway \
  -f nginx-gateway-fabric/values.yaml
```

## 8. Gateway

```bash
kubectl apply -f nginx-gateway-fabric/gateway.yaml
```

## 9. Metrics Server

```bash
helm upgrade --install -f metrics-server/override.yaml \
  --create-namespace -n metrics-server \
  metrics-server metrics-server/metrics-server
```

## 10. Kind Cluster 삭제

```bash
kind delete cluster
```
