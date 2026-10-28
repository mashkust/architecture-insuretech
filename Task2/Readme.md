## Task2

Развернуто тестовое приложение в Minikube:

- Deployment: 1 replica
- Service: NodePort
- HPA: масштабирование по memory utilization
- target memory utilization: 80%
- minReplicas: 1
- maxReplicas: 10

Для генерации нагрузки использовался Locust.
При росте потребления памяти HPA увеличивал количество реплик приложения.

## Запуск minikube amd64

## Run

1. Запустить Minikube:

```bash
minikube start
```

2. Включить metrics-server:

```bash
minikube addons enable metrics-server
```

3. Применить манифесты:

На Apple Silicon образ приложения потребовал отдельного импорта в runtime Minikube из-за различий архитектур (arm64 host и amd64 образ). После импорта образа приложение было успешно запущено, и далее протестировано с использованием HPA по memory utilization.

```bash
docker save --platform linux/amd64 ghcr.io/yandex-practicum/scaletestapp:latest -o scaletestapp.tar
minikube cp scaletestapp.tar /tmp/scaletestapp.tar
minikube ssh -- "sudo ctr -n k8s.io images import --all-platforms /tmp/scaletestapp.tar"
minikube ssh -- "sudo crictl images | grep scaletestapp"
```

```bash
kubectl apply -f Task2/deployment.yaml
kubectl apply -f Task2/service.yaml
kubectl apply -f Task2/hpa.yaml
```

4. Получить URL сервиса:

```bash
minikube service scaletestapp --url
```

5. Запустить Locust:

```bash
python3 -m locust
```

6. Открыть UI: http://localhost:8089
7. В поле Host указать URL сервиса из шага 4.

8. Проверка HPA:

```bash
kubectl get hpa -w
kubectl get pods -w
kubectl top pods
```

### [hpa_logs](./hpa_logs.png)

Анализ:

- в kubectl get hpa -w увеличилось значение REPLICAS;
- в kubectl get pods -w появился новый pod;
- событие SuccessfulRescale: New size: 2; reason: memory resource utilization (percentage of request) above target в kubectl describe hpa scaletestapp-hpa.

По выводу kubectl describe hpa scaletestapp-hpa видно, что Horizontal Pod Autoscaler успешно получил метрики памяти (ScalingActive=True, ValidMetricFound) и зафиксировал превышение целевого значения: 85% / 80%.
После этого HPA выполнил масштабирование Deployment с 1 до 2 реплик, что подтверждается событием. Таким образом, HPA корректно отреагировал на рост потребления памяти и увеличил число реплик приложения.
