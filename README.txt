# DevOps 2025 — Дипломный проект
**AWS EKS + один Ingress + Prometheus/Grafana + Argo CD (ставится автоматически на машине)**

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
  curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
  
  # yq (yaml-процессор)
  sudo wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64
  sudo chmod +x /usr/local/bin/yq
  
  # Argo CD CLI (обязательно ставится!)
  curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
  sudo chmod +x /usr/local/bin/argocd
  
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
  EXTERNAL=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}{.status.loadBalancer.ingress[0].ip}')
  echo "Главный адрес: http://$EXTERNAL"
  Доступ:
  http://$EXTERNAL           → дипломное приложение
  http://$EXTERNAL/grafana   → Grafana (admin + пароль в логах Actions)
  http://$EXTERNAL/argocd    → Argo CD веб-интерфейс (полностью через Ingress!

### 9. Полная очистка
  cd terraform
  terraform destroy -auto-approve
