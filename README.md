# Задание 1. Проектирование технологической архитектуры

Согласно заданию, целевые показатели доступности таковы:

- **Режим работы**: 24/7 (при этом сервис должен обслуживать клиентов из всех часовых поясов)
- **Доступность**: 99,9%
- **RTO** (целевое время восстановления):45 мин
- **RPO** (целевая точка восстановления, максимально допустимая потеря данных): 15 мин

Доступность 99,9 означает, что за год допустимо суммарное время простоя не более 8,76 часов (т.е. не более 11 случаев
восстановления).

```markdown
Дополнительно к этому нужно обеспечить одинаковое время загрузки страниц для пользователей из разных регионов. Оно не
должно зависеть от географического местоположения пользователя.
```

Выбрана фейловер-стратегия Active-Active с георезервированием. Это решение диктуется необходимостью обеспечить
одинаковое время загрузки при запросах из разных регионов.

При этом принято решение развертывать два независимых кластера Kubernetes с тем, чтобы за счет использования GSLB
обеспечить направление запроса пользователя на географически ближайший (из доступных) сервер. Это решение позволит в
перспективе при необходимости ввести дополнительные экземпляры сервисов для улучшения надежности и/или улучшения времени
отклика.

Диаграмма
решения: [InureTech_технологическая архитектура_to-be.xml](Task1/InureTech_%D1%82%D0%B5%D1%85%D0%BD%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B0%D1%8F%20%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0_to-be.xml)

![InureTech_технологическая архитектура_to-be.drawio.png](Task1/InureTech_%D1%82%D0%B5%D1%85%D0%BD%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B0%D1%8F%20%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0_to-be.drawio.png)

# Задание 2. Динамическое масштабирование контейнеров

## Часть 1. Масштабирование по метрикам расхода памяти

Загрузка приложения для тестирования:

```shell
docker pull ghcr.io/yandex-practicum/scaletestapp
```

Активация metrics-service:

```shell
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
minikube addons enable metrics-server
```

Манифест развертывания:

```shell
kubectl apply -f Task2/deployment.yaml
```

Манифест сервиса:

```shell
kubectl apply -f Task2/service.yaml
```

Получить адрес для проверки развертывания:

```shell
minikube service test-app-service --url
```

Манифест HPA:

```shell
kubectl apply -f Task2/hpa.yaml
```

Скриншоты дашборда Minikube

![before.PNG](Task2/images/before.PNG)

![after.PNG](Task2/images/after.PNG)

Логи HPA Minikube:

```
Events:
  Type     Reason                        Age                    From                       Message
  ----     ------                        ----                   ----                       -------
  Normal   SuccessfulRescale             5m24s (x5 over 23h)    horizontal-pod-autoscaler  New size: 2; reason: memory resource utilization (percentage of request) above target
  Warning  FailedGetResourceMetric       112s (x1800 over 23h)  horizontal-pod-autoscaler  failed to get memory utilization: unable to get metrics for resource memory: no metrics returned from resource metrics API
  Warning  FailedComputeMetricsReplicas  112s (x1800 over 23h)  horizontal-pod-autoscaler  invalid metrics (1 invalid out of 1), first error is: failed to get memory resource metric value: failed to get memory utilization: unable to get metrics for resource memory: no metrics returned from resource metrics API
```

На текущих настройках удалось добиться только масштабирования до двух подов, при этом приложение вело себя нестабильно.

