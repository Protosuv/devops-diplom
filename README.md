
# Дипломный практикум в Cloud: Amazon Web Services"

## Цели:

1. > Подготовить облачную инфраструктуру на базе облачного провайдера AWS.    

Подготовлена конфигурация Terraform для развёртывания ресурсов в AWS. Все предварительные тесты проводились на собственном тестовом аккаунте AWS. Для работы используется такая версия Terraform:
```
$ (master)terraform --version
Terraform v1.1.6
on linux_amd64
+ provider registry.terraform.io/hashicorp/aws v3.74.3
```
  В ходе работы Terraform, система разворачивает 3 шт ВМ с операционной системой Ubuntu 20.04. Созданы 2 workspace с названиями prod и stage.  
```
$ terraform workspace list
* prod
  stage
  ```
  В пространстве stage формируются 3 ВМ типа "t2.micro", для prod - 3 ВМ типа "t3.small". 
Репозиторий этой части работы находится по [ссылке](https://github.com/Protosuv/devops-diplom-terraform "https://github.com/Protosuv/devops-diplom-terraform")  
Раннее был создан ЛК в Terraform Cloud при работе над домашним заданием 7.4. К несчастью, на момент работы над диломной работой, сервис Terraform Cloud стал недоступен для пользователей из РФ. Система сообщает _"This content is not currently available in your region."_ Фактически работа с этим сервисом на его сайте возможна лишь через VPN.  
Использование Terraform Cloud в качестве Backend также невозможно из-за региональной блокировки доступа. Наиболее простым и уже отработанным раннее в рамках домашней работы 7.4, было использование S3 + DynamoDB для хранения состояния. Изначальный вариант был заменён на конфигурацию, с использованием именно этого бекенда.  
* Была добавлена [папка](https://github.com/Protosuv/devops-diplom-terraform/tree/master/S3 "https://github.com/Protosuv/devops-diplom-terraform/tree/master/S3") с файлами, которая предварительно создаёт в облаке DynamoDB и корзину S3.  
* Запуск планирования и применение плана, приводит к созданию нужных ресурсов.  
```
$ (master)terraform apply "myplan"
aws_dynamodb_table.dynamodb-terraform-lock: Creating...
aws_s3_bucket.netology-diplom-bucket: Creating...
aws_dynamodb_table.dynamodb-terraform-lock: Still creating... [10s elapsed]
aws_s3_bucket.netology-diplom-bucket: Still creating... [10s elapsed]
aws_dynamodb_table.dynamodb-terraform-lock: Creation complete after 10s [id=terraform-lock]
aws_s3_bucket.netology-diplom-bucket: Creation complete after 13s [id=netology-diplom-bucket]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

region = "us-east-2"
```
* В основной папке проекта проводим инициализацию бекенда, делаем планирование и убеждаемся, что всё срабатывает.
```
Apply complete! Resources: 11 added, 0 changed, 0 destroyed
```
Тестовые подключения к ВМ выполнились успешно. В качестве примера, такая конфигурация у одной из ВМ:
```
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.13.0-1017-aws x86_64)
$ free -m
              total        used        free      shared  buff/cache   available
Mem:           1939         144        1430           0         364        1643
Swap:             0           0           0
$ cat /proc/cpuinfo
processor	: 0
vendor_id	: GenuineIntel
model name	: Intel(R) Xeon(R) Platinum 8259CL CPU @ 2.50GHz
processor	: 1
vendor_id	: GenuineIntel
model name	: Intel(R) Xeon(R) Platinum 8259CL CPU @ 2.50GHz
```
Такой конфигурации хватит для запуска control plane и для работы с подами с приложением.

2. >Запустить и сконфигурировать Kubernetes кластер.
3. >Установить и настроить систему мониторинга.
4. >Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
5. >Настроить CI для автоматической сборки и тестирования.
6. >Настроить CD для автоматического развёртывания приложения


