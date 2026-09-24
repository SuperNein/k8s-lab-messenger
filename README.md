# Запуск микросервисного мессенджера в Kubernetes

Проект микросервисного мессенджера (Frontend, BFF, User Service, Message Service, PostgreSQL, MinIO) с монтированием S3 CSI, разделением окружений через Kustomize (dev/prod) и GitOps-деплоем через Argo CD.

---

## 1. Подготовка кластера Minikube

Для работы правил `nodeAffinity` запускается многоузловой кластер:

```bash
# Запуск 3-узлового кластера
minikube start --nodes 3 --cpus 2 --memory 1700mb --driver docker

# Разметка нод
kubectl label node minikube workload=system
kubectl label node minikube-m02 workload=app
kubectl label node minikube-m03 workload=app disk=fast
```

---

## 2. Установка компонентов инфраструктуры

### S3 CSI Driver

```bash
kubectl apply -f https://raw.githubusercontent.com/ctrox/csi-s3/master/deploy/kubernetes/provisioner.yaml
kubectl apply -f https://raw.githubusercontent.com/ctrox/csi-s3/master/deploy/kubernetes/attacher.yaml
kubectl apply -f https://raw.githubusercontent.com/ctrox/csi-s3/master/deploy/kubernetes/csi-s3.yaml
```

### Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl wait --for=condition=available deployment/argocd-server -n argocd --timeout=300s
```
---

## 3. Развертывание

### Argo CD

1. В файле `argocd/argocd-app-dev.yaml` укажите URL вашего репозитория в поле `repoURL`.

2. Примените манифест приложения:

```bash
kubectl apply -f argocd/argocd-app-dev.yaml
```

### Прямой запуск вручную

```bash
kubectl apply -k k8s/overlays/dev
```

---

## 4. Инициализация MinIO S3 Bucket

```bash
kubectl exec -it $(kubectl get pod -n messager-dev -l app=minio -o jsonpath='{.items[0].metadata.name}') -n messager-dev -- \
  sh -c 'mc alias set local http://localhost:9000 "$MINIO_ROOT_USER" "$MINIO_ROOT_PASSWORD" && mc mb local/messenger-uploads'
```

Перезапустите под message-service, если он ожидал том:

```bash
kubectl delete pod -l app=message-service -n messager-dev
```

---

## 5. Проверка и эксплуатация

* **Статус подов и нод**:

```bash
kubectl get pods -n messager-dev -o wide
```

* **Проброс порта frontend**:
```bash
kubectl port-forward svc/frontend-dev 8080:80 -n messager-dev
```

Страница доступна по адресу: `http://localhost:8080`

* **Доступ к панели Argo CD**:

```bash
# Получить пароль admin
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# Проброс порта
kubectl port-forward svc/argocd-server -n argocd 8443:443
```

Панель доступна по адресу: `https://localhost:8443`
