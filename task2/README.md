
## Задание 2. PV, PVC
### Задача
Создать Deployment приложения, использующего локальный PV, созданный вручную.

### Шаги выполнения
1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool, использующего созданный ранее PVC
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
3. Продемонстрировать, что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд. 
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему. (Используйте команду `kubectl describe pv`).
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать, что произошло с файлом после удаления PV. Пояснить, почему.


### Что сдать на проверку
- Манифесты:
  - `pv-pvc.yaml`
- Скриншоты:
  - каждый шаг выполнения задания, начиная с шага 2.
- Описания:
  - объяснение наблюдаемого поведения ресурсов в двух последних шагах.

------

## решение 2

### Создаем Namespace storage-demo
kubectl apply -f ns.yaml

###  Создаём PV и PVC
kubectl apply -f pv-pvc.yaml  

PV (`pv-local-demo`) → указывает на каталог `/data/pv-local-demo` на ноде.   
PVC (`pvc-local-demo`) → запрос на 1Gi.   

Проверяем: 
kubectl get pv
kubectl -n storage-demo get pvc
 
 
## Deployment
kubectl -n storage-demo apply -f deployment.yaml
 
Поднимаем `Deployment` с 2 контейнерами в одном поде:
- **writer (busybox)** → пишет строки в файл `/shared/out.txt` каждые 5 сек.  
- **reader (multitool)** → читает этот файл через `tail -f`.  

Проверка:
 
kubectl -n storage-demo get pods
kubectl -n storage-demo get pods -l app=data-exchange -o wide
 
Под в статусе `Running`, `READY = 2/2`.

---

## 4. Проверка работы
Логи writer:
kubectl -n storage-demo logs deploy/data-exchange -c writer --tail=10


Логи reader:
kubectl -n storage-demo logs deploy/data-exchange -c reader --tail=10
 
kubectl -n storage-demo exec -it deploy/data-exchange -c reader -- tail -n 20 /data/out.txt

Подробное описание пода:
kubectl -n storage-demo describe pod -l app=data-exchange
---

## 5. Удаляем Deployment и PVC
kubectl -n storage-demo delete deploy data-exchange
kubectl -n storage-demo delete pvc pvc-local-demo


Смотрим состояние PV:
kubectl get pv
kubectl describe pv pv-local-demo

PV остаётся, статус меняется на **Released**, данные на диске сохранены (так работает политика `Retain`).

---

## 6. Проверка файлов на ноде
(Для Minikube:)
 
minikube ssh
ls -l /data/pv-local-demo
tail -n 10 /data/pv-local-demo/out.txt
exit
 
Файл с данными остался на ноде.

---

## 7. Удаляем PV

kubectl delete pv pv-local-demo


Проверяем снова на ноде:

minikube ssh
ls -l /data/pv-local-demo
tail -n 10 /data/pv-local-demo/out.txt
exit

Файл остался, потому что Kubernetes удаляет только объект PV, но не данные (`Retain`).

---

## Итоги
- PVC после удаления → PV в состоянии `Released`.  
- Данные физически остались на диске `/data/pv-local-demo`.  
- Удаление PV не удаляет файл (из-за политики `Retain`).  
- Для полного удаления нужно руками чистить каталог на узле.  


### удалим все что было создано, за счет удаления пространства имен **storage-demo**
kubectl delete ns storage-demo
minikube ssh
sudo rm -rf /data/pv-local-demo
exit


![рисунок 7](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_7.jpg)
![рисунок 8](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_8.jpg)
![рисунок 9](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_9.jpg)
![рисунок 10](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_10.jpg)
![рисунок 11](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_11.jpg)
![рисунок 12](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_12.jpg)
![рисунок 13](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_13.jpg) 


