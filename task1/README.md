
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
kubectl -n storage-demo apply -f writer.yaml
kubectl -n storage-demo apply -f reader-multitool.yaml
----------------------------------------

### Проверям что поды подняль 
kubectl -n storage-demo get pods -o wide
----------------------------------------


Проверка чтения файла
### логи писателя
kubectl -n storage-demo logs writer --tail=5

### хвост файла из multitool (тот же hostPath)
kubectl -n storage-demo exec -it reader-multitool -- tail -n 10 /data/out.txt
exit

![]()