# 쿠버네티스

## 목차

1. [개발환경 구성](#1-개발환경-구성)
1. [Kind 클러스터 생성](#1-1-쿠버네티스-클러스터-생성-kind)
1. [OKE 클러스터 생성](#1-2-쿠버네티스-클러스터-생성-oci-oke-free-tier)
1. [Metrics Server 구성](#2-metrics-server-설치)
1. [Cert-Manager 설치](#3-cert-manager-설치)
1. [Gateway API 구성](#4-gateway-api-구성)
1. [ArgoCD](#5-argocd)
1. [OpenTelemetry](#6-open-telemetry)

## 1. 개발환경 구성

> [**➡️ 자세히 보기**](1_init.md)

- Helm 설치 및 Repo 추가
- kubectl 및 krew 플러그인 설치
  - ctx
  - ns
  - neat
- k9s 설치

## 1-1. 쿠버네티스 클러스터 생성 (Kind)

> [**➡️ 자세히 보기**](1-1_kind.md)

- Kind 클러스터 구성
- 리소스 정리

## 1-2. 쿠버네티스 클러스터 생성 (OCI OKE Free tier)

> [**➡️ 자세히 보기**](1-2_oke.md)

### Terraform 활용 Oracle Cloud Free tier OKE Cluster 생성

- Oracle Cloud 계정 생성
- Terraform 설치
- OCI CLI 설치
- Terraform 프로비저닝
- OKE Cluster 연결
- 리소스 정리

## 2. Metrics Server 설치

```bash
helm upgrade --install \
  --create-namespace -n metrics-server \
  metrics-server metrics-server/metrics-server \
  --set args="{--kubelet-insecure-tls}"
```

## 3. Cert-Manager 설치

> [**➡️ 자세히 보기**](3_cert_manager.md)

- Cert-Manager 설치
- Reflector 설치
- Cluster Issuer 생성
  - Let's Encrypt
- Certificate 리소스 생성

## 4. Gateway API 구성

> [**➡️ 자세히 보기**](4_gateway_api.md)

- Gateway API CRD 설치
- Gateway API 구현체 설치
  - Istio Gateway API
  - Nginx Gateway Fabric
- Gateway 생성
- Nginx 접속 테스트
  - https 통신 불가할 때
    - OCI 로드밸런서 리스너에 443 포트의 프로토콜 HTTP -> TCP로 변경
    - SSL 사용 해제

## 5. ArgoCD

> [**➡️ 자세히 보기**](5_argocd.md)

- ArgoCD 설치

## 6. OpenTelemetry

> [**➡️ 자세히 보기**](6_opentelemetry.md)

- OpenTelemetry 설치
