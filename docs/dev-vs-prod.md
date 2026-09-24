# Отличия dev и prod

В рамках лабораторной работы реализованы следующие отличия между окружениями `dev` и `prod` через Kustomize overlays:

1. **Реплики:**
* `dev`: 1 реплика для каждого сервиса
* `prod`: 2 реплики для `frontend`, `bff`, `user-service`, `message-service`

2. **Ресурсы контейнеров:**
* `dev`: минимальные базовые лимиты/запросы для экономии ресурсов узлов
* `prod`: повышенные строгие `requests` (CPU 100m, Memory 128Mi) и `limits` (CPU 250m, Memory 256Mi)

3. **Ingress Host:**
* `dev`: `dev.messager.local`
* `prod`: `messager.example.com`

4. **Теги образов:**
* `dev`: `latest`
* `prod`: `stable`

5. **Node Affinity:**
* `dev`: правила мягкие или отключены для возможности запуска на одном/двух узлах
* `prod`: строгое распределение по топологии: `postgres`/`minio` на `workload=system`, прикладные сервисы на `workload=app`, `message-service` на `workload=app` с предпочтением `disk=fast`

6. **Namespace и суффиксы:**
* `dev`: отдельный namespace `messager-dev` с суффиксом имен `-dev`
* `prod`: целевой namespace без изолирующих тестовых префиксов
