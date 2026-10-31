# k3s-manifests

Kubernetes-манифесты для приложения в кластере **k3s**: Deployment, Service, Ingress (Traefik) и PersistentVolumeClaim. Небольшой, но полный пример того, как сервис живёт в Kubernetes — от пода до внешнего доступа.

## Схема

```
            клиент
              │  http://web.local
              ▼
      ┌───────────────────┐
      │ Ingress (Traefik) │  ingress.yaml
      └─────────┬─────────┘
                │ /
                ▼
      ┌───────────────────┐
      │ Service :80       │  service.yaml
      └─────────┬─────────┘
                │
       ┌────────┴────────┐
       ▼                 ▼
  ┌─────────┐       ┌─────────┐
  │  web-1  │       │  web-2  │   Deployment, 2 реплики, nginx:alpine
  └────┬────┘       └────┬────┘
       └────────┬────────┘
                ▼
      ┌───────────────────┐
      │ PVC web-data     │  pvc.yaml, local-path, 1 Gi
      └───────────────────┘
```

## Файлы

| Файл | Что описывает |
|---|---|
| `deployment.yaml` | 2 реплики `nginx:alpine`, requests/limits (`50m/64Mi` → `200m/128Mi`), монтирует PVC в `/usr/share/nginx/html` |
| `service.yaml` | ClusterIP, порт `80 → 80` |
| `ingress.yaml` | Ingress на хост `web.local` через встроенный в k3s Traefik |
| `pvc.yaml` | PVC 1 Gi, storageClass `local-path` (Local Path Provisioner в k3s) |

## Запуск

```bash
# 1. применить манифесты
kubectl apply -f pvc.yaml -f deployment.yaml -f service.yaml -f ingress.yaml

# 2. проверить
kubectl get pods,svc,ingress,pvc
kubectl rollout status deployment/web

# 3. локальный хост для ingress и проверка
echo "127.0.0.1 web.local" | sudo tee -a /etc/hosts
curl http://web.local
```

## Полезное

```bash
kubectl scale deployment web --replicas=4        # масштабирование
kubectl logs -l app=web --tail=50                # логи всех подов
kubectl describe ingress web                     # проверить маршрут
kubectl delete -f pvc.yaml -f deployment.yaml -f service.yaml -f ingress.yaml
```

## Что демонстрирует

- базовые примитивы k8s: Deployment, Service, Ingress, PVC;
- requests/limits и горизонтальное масштабирование;
- работу с Traefik и storage class «из коробки» k3s;
- декларативный деплой: всё в манифестах, `kubectl apply` идемпотентен.

## Заметки

- `local-path` — дефолтный StorageClass k3s; данные лежат на ноде, для HA нужен сетевой storage (Longhorn / Ceph).
- Ingress обслуживает Traefik, встроенный в k3s (`kubectl get pods -n kube-system`).
- TLS пока не настроен: для HTTPS добавить блок `tls:` и cert-manager.
