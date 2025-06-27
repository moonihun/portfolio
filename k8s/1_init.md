# 개발환경 구성

## 목차

1. [Helm 설치](#1-helm-설치)
1. [Helm repo 정리](#repo)
1. [kubectl 설치](#2-kubectl-설치)
1. [krew 설치](#3-krew-설치)
1. [krew 플러그인](#4-krew-플러그인-설치)
1. [k9s 설치](#5-k9s-설치)

## 1. Helm 설치

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

---

## Helm Repo

| NAME           | URL                                                        |
| :------------- | :--------------------------------------------------------- |
| bitnami        | https://charts.bitnami.com/bitnami                         |
| kubernetes     | https://charts.helm.sh/stable                              |
| metallb        | https://metallb.github.io/metallb                          |
| traefik        | https://traefik.github.io/charts                           |
| istio          | https://istio-release.storage.googleapis.com/charts        |
| kiali          | https://kiali.org/helm-charts                              |
| caddy          | https://caddyserver.github.io/ingress/                     |
| gloo           | https://storage.googleapis.com/solo-public-helm            |
| kong           | https://charts.konghq.com                                  |
| geek-cookbook  | https://geek-cookbook.github.io/charts/                    |
| metrics-server | https://kubernetes-sigs.github.io/metrics-server/          |
| argo           | https://argoproj.github.io/argo-helm                       |
| emberstack     | https://emberstack.github.io/helm-charts                   |
| jetstack       | https://charts.jetstack.io                                 |
| gitlab         | https://charts.gitlab.io/                                  |
| gitea          | https://dl.gitea.com/charts/                               |
| open-telemetry | https://open-telemetry.github.io/opentelemetry-helm-charts |

## 2. kubectl 설치

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

## 3. Krew 설치

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

## 4. Krew 플러그인 설치

```bash
# kubectx
kubectl krew install ctx

# kubens
kubectl krew install ns

# neat
kubectl krew install neat
```

## 5. k9s 설치

```bash
# For macOS
brew install derailed/k9s/k9s

# For Linux
OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
VERSION="$(curl -s https://api.github.com/repos/derailed/k9s/releases/latest | jq -r '.tag_name')"

curl -LO https://github.com/derailed/k9s/releases/download/${VERSION}/k9s_${OS}_${ARCH}.deb

sudo apt install ./k9s_${OS}_${ARCH}.deb
rm k9s_${OS}_${ARCH}.deb
```
