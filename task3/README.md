## Задание 3. StorageClass
### Задача
Создать Deployment приложения, использующего PVC, созданный на основе StorageClass.

### Шаги выполнения

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool, использующего созданный ранее PVC.
2. Создать SC и PVC для подключения папки на локальной ноде, которая будет использована в поде.
3. Продемонстрировать, что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд.

### Что сдать на проверку
- Манифесты:
  sc.yaml
- Скриншоты:
  - каждый шаг выполнения задания, начиная с шага 2
---

### Подготовка
Включить аддоны (если выключены):
minikube addons enable storage-provisioner
minikube addons enable default-storageclass


### Создать ns (если ещё нет)
kubectl apply -f ns.yaml  

### Создать StorageClass и PVC  
kubectl apply -f sc.yaml  
kubectl -n storage-demo apply -f pvc-sc.yaml  



### проверяем что создано  
kubectl get sc  
kubectl -n storage-demo get pvc  

### Deployment
kubectl -n storage-demo apply -f deployment-sc.yaml  
kubectl -n storage-demo get pods -l app=data-exchange-sc -o wide  

Поды в работе !  
![рисунок 14](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_14.jpg)  

### смотрим логи что пишут и читают поды  
kubectl -n storage-demo logs deploy/data-exchange-sc -c writer --tail=10  
kubectl -n storage-demo logs deploy/data-exchange-sc -c reader --tail=10  

### смотрим как происходит запись в PVC  
POD=$(kubectl -n storage-demo get pods -l app=data-exchange-sc -o jsonpath='{.items[0].metadata.name}') 
kubectl -n storage-demo exec -it "$POD" -c reader -- tail -n 20 /data/out.txt    
![рисунок 15](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_15.jpg)    

### автопровиженинг PV  
kubectl get pv  
kubectl -n storage-demo get pvc pvc-sc-demo -o wide  
kubectl -n storage-demo describe pvc pvc-sc-demo | sed -n '/Events/,$p'  

![рисунок 16](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_16.jpg)  
 
---------------------------------------------------------
## удаляем все что было создано 

### Удалить Deployment (приложение)
kubectl -n storage-demo delete deploy data-exchange-sc --ignore-not-found

### Удалить PVC
kubectl -n storage-demo delete pvc pvc-sc-demo --ignore-not-found

### Удалить PV, созданный динамически
PVC создавал PV автоматически через StorageClass. Найдем его и удалим:

kubectl get pv
kubectl delete pv <имя_PV>

No resources found - удалять нечево

### Удалиv StorageClass  
kubectl delete sc local-hostpath-sc --ignore-not-found

### Проверка, что всё удалено
kubectl -n storage-demo get all
kubectl get pvc -A
kubectl get pv
kubectl get sc
kubectl delete ns storage-demo

![рисунок 17](https://github.com/ysatii/kuber-homeworks2.1/blob/main/img/img_17.jpg)  
