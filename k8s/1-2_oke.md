## 1. Oracle Cloud 계정 생성

- 계정 생성
- 무료 계정 Out of Capacity 이슈로 인해 유료 계정 전환 필요
  - 유료 계정 전환 후에도 Free tier 한도 내에서 무료
  - [오라클 프리티어 안내](https://www.oracle.com/kr/cloud/free/)

## 2. Terraform 설치

```bash
# For macOS
brew tap hashicorp/tap
brew inistall hashicorp/tap/terraform

# For Linux (Ubuntu/Debian)
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

## 3. OCI CLI 설치

```bash
# For macOS
brew update && brew install oci-cli

# For Linux
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
```

## 4. Terraform 프로비저닝

```bash
# terraform.tfvars
user_ocid = "your_user_ocid"
fingerprint = "your_fingerprint"
tenancy_ocid = "your_tenancy_ocid"
region = "your_region"
private_key_path = "your_private_key_path"
compartment_id = "your_compartment_id"
kubernetes_version = "v1.33.1"

# 실행
terraform init
terraform plan
terraform apply
```

## 5. OKE Cluster 연결

```bash
oci ce cluster create-kubeconfig --cluster-id {your-cluster-id} --file $HOME/.kube/config --region {your-region} --token-version 2.0.0  --kube-endpoint PUBLIC_ENDPOINT
```

## 6. CNI 변경 (flannel -> cilium)

[공식 문서](https://docs.oracle.com/ko/learn/oke-flannel-to-cilium-cni-plugin/index.html)

## 7. 리소스 정리

```bash
terraform destroy
```
