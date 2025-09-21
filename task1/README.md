
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

### Создаем Namespace
kubectl apply -f ns.yaml
----------------------------------------

### Подымаем поды
kubectl -n storage-demo apply -f pod.yaml
----------------------------------------

### Проверям что поды подняль 
kubectl -n storage-demo get pods -o wide
----------------------------------------


Проверка чтения файла
### логи писателя
kubectl -n storage-demo logs data-exchange --tail=5

### хвост файла из multitool (тот же hostPath)
kubectl -n storage-demo exec -it reader -- tail -n 10 /data/out.txt
exit

### 
kubectl -n storage-demo describe pod data-exchange

Показать всё про под
kubectl -n storage-demo describe pod data-exchange

Логи контейнера writer (пишет каждые 5 сек)
kubectl -n storage-demo logs data-exchange -c writer --tail=20 -f

Логи контейнера reader (он делает tail -f файла)
kubectl -n storage-demo logs data-exchange -c reader --tail=20 -f

Зайти внутрь контейнера reader и посмотреть файл
kubectl -n storage-demo exec -it data-exchange -c reader -- sh
# уже внутри контейнера:
tail -n 20 /data/out.txt
exit

Однострочник без входа внутрь
kubectl -n storage-demo exec -it data-exchange -c reader -- tail -n 20 /data/out.txt

(На всякий) показать имена контейнеров в поде
kubectl -n storage-demo get pod data-exchange -o jsonpath='{.spec.containers[*].name}{"\n"}'
# ожидаешь: writer reader