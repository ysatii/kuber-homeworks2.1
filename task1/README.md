
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


### Подымаем поды
kubectl -n storage-demo apply -f pod.yaml


### Проверям что поды поднялись
kubectl -n storage-demo get pods -o wide



Проверка чтения файла
### логи писателя
kubectl -n storage-demo logs data-exchange --tail=5

 

### Показать всё про под
kubectl -n storage-demo describe pod data-exchange

Логи контейнера writer (пишет каждые 5 сек)
### kubectl -n storage-demo logs data-exchange -c writer --tail=20 -f

### читаем из контейнера reader (он делает tail -f файла)
kubectl -n storage-demo logs data-exchange -c reader --tail=20 -f

### Зайти внутрь контейнера reader и посмотреть файл
kubectl -n storage-demo exec -it data-exchange -c reader -- sh
уже внутри контейнера:
tail -n 20 /data/out.txt
exit
либо 
Однострочник без входа внутрь
kubectl -n storage-demo exec -it data-exchange -c reader -- tail -n 20 /data/out.txt

### показать имена контейнеров в поде
kubectl -n storage-demo get pod data-exchange -o jsonpath='{.spec.containers[*].name}{"\n"}'
Получим два контейнера: writer reader

![рисунок 1](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_1.jpg)
![рисунок 2](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_2.jpg)
![рисунок 3](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_3.jpg)
![рисунок 4](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_4.jpg)
![рисунок 5](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_5.jpg)
![рисунок 6](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_6.jpg)
