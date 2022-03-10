
# Дипломный практикум в Cloud: Amazon Web Services"

## Цели:

1. > Подготовить облачную инфраструктуру на базе облачного провайдера AWS.    

Подготовлена конфигурация Terraform для развёртывания ресурсов в AWS. В ходе работы Terraform, система разворачивает 3 шт ВМ с операционной системой Ubuntu 20.04. Созданы 2 workspace с названиями prod и stage.  
```
$ terraform workspace list
  default
* prod
  stage
  ```
  В пространстве по умолчанию формируются 3 ВМ типа "t2.micro", для stage - 3 ВМ типа "t3.micro". 
Репозиторий этой части работы находится по [ссылке](https://github.com/Protosuv/devops-diplom-terraform "https://github.com/Protosuv/devops-diplom-terraform")  
Раннее был создан ЛК в Terraform Cloud при работе над домашним заданием 7.4. К несчастью, на момент работы над диломной работой, сервис Terraform Cloud стал недоступен для пользователей из РФ. Система сообщает _"This content is not currently available in your region."_

2. >Запустить и сконфигурировать Kubernetes кластер.
3. >Установить и настроить систему мониторинга.
4. >Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
5. >Настроить CI для автоматической сборки и тестирования.
6. >Настроить CD для автоматического развёртывания приложения


