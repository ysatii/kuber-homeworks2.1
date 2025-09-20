
## Задание 1. Volume: обмен данными между контейнерами в поде
### Задача

Создать Deployment приложения, состоящего из двух контейнеров, обменивающихся данными.

### Шаги выполнения
1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Настроить busybox на запись данных каждые 5 секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.


### Что сдать на проверку
- Манифесты:
  - `containers-data-exchange.yaml`
- Скриншоты:
  - описание пода с контейнерами (`kubectl describe pods data-exchange`)
  - вывод команды чтения файла (`tail -f <имя общего файла>`)

------
## Решение 1 

# Namespace
kubectl apply -f ns.yaml

kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl -n storage-demo get pv,pvc

kubectl get pv
kubectl -n storage-demo get pvc
kubectl -n storage-demo describe pvc pvc-shared
kubectl describe pv pv-shared-hostpath


![Рисунок 1](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_1.jpg)
![Рисунок 2](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_1.jpg)

-------------------------------
создаем поды
kubectl -n storage-demo apply -f writer.yaml
kubectl -n storage-demo apply -f reader-nginx.yaml
kubectl -n storage-demo apply -f reader-multitool.yaml


Проверяем, что writer пишет:
kubectl -n storage-demo logs writer --tail=5

Чтение файла через multitool:

MULTI=$(kubectl -n storage-demo get pod reader-multitool -o jsonpath='{.metadata.name}')
kubectl -n storage-demo exec -it $MULTI -- tail -n 5 /data/out.txt

Чтение файла через nginx:

kubectl -n storage-demo port-forward pod/reader-nginx 8080:80
# открыть http://localhost:8080/out.txt