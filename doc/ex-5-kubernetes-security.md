# EX-5 Настройка аккаунтов и прав для них, настройка ограничений и политик безопасности для контейнеров

* Основное задание: создать service account bob, дать ему роль admin в рамках всего кластера
  Создать service account dave без доступа к кластеру

* Основное задание: создать prometheus namespace
  Создать Service Account carol в этом Namespace
  Дать всем Service Account в Namespace prometheus возможность делать get, list, watch в отношении Pods всего кластера

## EX-5.1 Что было сделано

* Написаны манифесты для создания пользователей **bob** и **dave**
* Написан манифест для связывания кластерной роли **admin** с **bob** service account
* Написан манифест для создания **prometheus namespace**
* Написан манифест для создания **carol service account** в **prometheus namespace**
* Написан манифест для создания кластерной роли **pod-reader**, дающей делать get, list, watch для pods всего кластера
* Написан манифест для связывания кластерной роли **pod-reader** со всеми service accounts из **prometheus namespace**

## EX-5.2 Как запустить проект

* Создать **bob и dave** service accounts, добавить для **bob** роль **admin** в рамках всего кластера

  ```bash
  cat kubernetes-security/task01/*.yaml | kubectl apply -f -
  ```

* Создать **prometheus namespace**, и **carol service account**, который умеет делать get, list, watch для всех pods в кластере

  ```bash
  cat kubernetes-security/task02/*.yaml | kubectl apply -f -
  ``` 

## EX-5.3 Как проверить проект

* Для проверки **carol service account** команда `kubectl auth can-i...` должна выдавать **yes**

  ```bash
  for VERB in get list watch; do kubectl auth can-i $VERB pods --as system:serviceaccount:prometheus:carol; done
  yes
  yes
  yes
  ```

## EX-5.4 Как начать пользоваться проектом
