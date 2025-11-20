# DevOps 2025 — Дипломный проект
**AWS EKS + один Ingress + Prometheus/Grafana + Argo CD (ставится руками на машине)**

## Полная инструкция запуска с чистого AWS-инстанса (Ubuntu 22.04/24.04)

### 1. Клонирование репозитория
  git clone https://github.com/sojasac/DevOps_2025_Diploma.git
  cd DevOps_2025_Diploma

### 2. Установка ВСЕХ инструментов
  # Обновляем систему
  sudo apt update && sudo apt upgrade -y
  
  # AWS CLI v2
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  unzip awscliv2.zip && sudo ./aws/install && rm -rf aws awscliv2.zip
  
  # Terraform
  wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg >/dev/null
  echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
  sudo apt update && sudo apt install terraform -y
  
  # kubectl
  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  chmod +x kubectl && sudo mv kubectl /usr/local/bin/
  
  # Helm
  helm install [RELEASE_NAME] oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack
  helm upgrade monitoring oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack --values metrics/grafana-values.yaml
    
  # Argo CD CLI (обязательно ставится!)
  helm repo add argo https://argoproj.github.io/argo-helm
  helm upgrade --install argocd argo/argo-cd --namespace argocd --create-namespace --set controller.replicas=1 --set reposerver.replicas=1 --set server.replicas=1 --set configs.params.server.insecure=true
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
  
  # Проверка, что всё установилось
  echo "Версии инструментов:"
  aws --version | head -1
  terraform --version
  kubectl version --client --short
  helm version --short
  yq --version
  argocd version --client --short

### 3. Настройка AWS credentials
  aws configure
  # Вводим:
  # AWS Access Key ID
  # AWS Secret Access Key
  # Region (например eu-central-1)
  # Output format: json

### 4. Ручной запуск инфраструктуры (Terraform)
  cd terraform
  terraform init
  terraform plan
  terraform apply


### 5. Подключение к кластеру
  aws eks update-kubeconfig --region $(terraform output -raw region) --name $(terraform output -raw cluster_name)
  kubectl get nodes


### 6. Добавь секреты в GitHub (один раз!)
  https://github.com/sojasac/DevOps_2025_Diploma/settings/secrets/actions → 3 секрета:
  AWS_ACCESS_KEY_ID
  AWS_SECRET_ACCESS_KEY
  AWS_REGION

### 7. Запуск приложения + мониторинга + Argo CD — одним пушем
  cd ..
  git commit --allow-empty -m "Full deploy: app + monitoring + Argo CD"
  git push origin main

### 8. Один внешний адрес на всё
  kubectl get svc --all-namespaces
  EXTERNAL-IP - урла сервиса

### 9. Полная очистка
  cd terraform
  terraform destroy -auto-approve
