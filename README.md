# Домашнее задание к занятию Troubleshooting -***Вуколов Евгений***
 
### Цель задания
 
Устранить неисправности при деплое приложения.
 
### Чеклист готовности к домашнему заданию
 
1. Кластер K8s.
 
### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить
 
1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```
2. Выявить проблему и описать.
3. Исправить проблему, описать, что сделано.
4. Продемонстрировать, что проблема решена.
 
### Правила приёма работы
 
1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.


# **Решение**

- При установке приложения возникла ошибка, так как у меня на vm в k8s не созданы namespace web и data. Создаю namespace web и data, затем повторяю установку:

- ![scrin](https://github.com/Evgenii-379/3.5-3.5.md-Troubleshooting/blob/main/Снимок%20экрана%202025-04-13%20165902.png)

- Выполняю команду для разворачивания deployment:

```
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```

- Смотрим всё что развернулось в k8s: 

- ![scrin](https://github.com/Evgenii-379/3.5-3.5.md-Troubleshooting/blob/main/Снимок%20экрана%202025-04-13%20164301.png)

- Проверяю логи : 

- ![scrin](https://github.com/Evgenii-379/3.5-3.5.md-Troubleshooting/blob/main/Снимок%20экрана%202025-04-13%20174153.png)

- Логи показываю что приложение web-consumer (в namespace web) не может подключиться к сервису auth-db (в namespace data).
По умолчанию сервисы в Kubernetes видны только в пределах своего namespace.

- Проверяю наличие сервиса в namespace web :

- ![scrin](https://github.com/Evgenii-379/3.5-3.5.md-Troubleshooting/blob/main/Снимок%20экрана%202025-04-13%20174730.png)

Как видно из вывода команды, сервис в namespace web отсутсвует. Проблема в том, что приложение пытается обратиться к сервису по короткому имени, которое работает только в пределах одного namespace.
Для доступа к сервису из другого namespace нужно использовать полное доменное имя FQDN, для этого нужно внести изменения в Deployment web-consumer. Есть ещё один вариант который я использовал,
это создать Service в namespace web, который будет указывать на сервис в namespace data:

```
apiVersion: v1
kind: Service
metadata:
  name: auth-db
  namespace: web
spec:
  type: ExternalName
  externalName: auth-db.data.svc.cluster.local
```
 
- Ссылка на манифест:

[auth-db-externalname.yaml](https://github.com/Evgenii-379/3.5-3.5.md-Troubleshooting/blob/main/config.yaml/auth-db-externalname.yaml)

- ![scrin](https://github.com/Evgenii-379/3.5-3.5.md-Troubleshooting/blob/main/Снимок%20экрана%202025-04-13%20185532.png)

В выводе команды видно что в namespace web создался Service типа ExternalName.

- Проверяю логи: 
 
- ![scrin](https://github.com/Evgenii-379/3.5-3.5.md-Troubleshooting/blob/main/Снимок%20экрана%202025-04-13%20185816.png)

- Команда внутри пода для проверки: 

- ![scrin](https://github.com/Evgenii-379/3.5-3.5.md-Troubleshooting/blob/main/Снимок%20экрана%202025-04-13%20190452.png)


- Вывод команды показывает HTTP-подключение от Pod'а web-consumer к сервису auth-db




















