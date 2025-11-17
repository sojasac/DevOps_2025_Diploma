# My-App + EKS + Monitoring

## Требования

Перед тем как клонировать репозиторий, убедитесь, что на вашей Linux-машине установлены следующие инструменты:

- [Git](https://git-scm.com/) — для клонирования репозитория
- [Docker](https://www.docker.com/) — для сборки и работы контейнеров
- [kubectl](https://kubernetes.io/docs/tasks/tools/) — для работы с Kubernetes
- [Helm](https://helm.sh/) — для управления Helm-чартами (Prometheus/Grafana)
- [AWS CLI](https://aws.amazon.com/cli/) — для работы с EKS
- [Terraform](https://www.terraform.io/) — для развертывания инфраструктуры EKS

> Рекомендуется использовать Linux (Ubuntu/Debian) с bash.

---

## Быстрый старт

1. Клонируйте репозиторий:

git clone <URL_репозитория>
cd <имя_папки>

2. Развернуть инфраструктуру через Terraform:

cd <папка_репозитория>/terraform
terraform init         # инициализация
terraform plan         # посмотреть, что будет создано
terraform apply 

3. Настроить kubectl для доступа к кластеру:

aws eks --region <region> update-kubeconfig --name <cluster_name>
kubectl get nodes     # проверяем доступ

4. Деплой приложения:

cd <папка_репозитория>/k8s
kubectl apply -f diploma_app-deployment.yaml
kubectl apply -f diploma_app-services.yaml
kubectl get pods
kubectl get svc

5. Деплой мониторинга через Helm:

cd <папка_репозитория>/metrics
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# values-файл с настройкой Ingress для Grafana
helm install monitoring prometheus-community/kube-prometheus-stack -f metrics-values.yaml

6. LoadBalancer для Grafana:

cd <папка_репозитория>/k8s
kubectl apply -f grafana_lb-services.yaml


7. Проверка состояния:

kubectl get pods --all-namespaces
kubectl get svc --all-namespaces
kubectl get ingress

8. Доступ в браузере:

Получение password Grafana: kubectl get secret monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
Логин Grafana: admin
После выполнения команды kubectl get svc --all-namespace в поле EXTERNAL-IP будет url приложения и графаны