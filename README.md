
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

region = "us-east-1"
```
* В основной папке проекта проводим инициализацию бекенда, делаем планирование и убеждаемся, что всё срабатывает.
```
$ (master)terraform apply "myplan"
Acquiring state lock. This may take a few moments...
module.vpc.aws_subnet.public[0]: Creating...
aws_instance.diplom_instance[2]: Creating...
aws_instance.diplom_instance[1]: Creating...
aws_instance.diplom_instance[0]: Creating...
.....
Releasing state lock. This may take a few moments...
Apply complete! Resources: 5 added, 0 changed, 0 destroyed.
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

В качестве инструмента развёртывания кластера, был избран уже опробованный раннее Kubspray. В настройках инвентаря были указаны внешние IP адреса серверов (нод), с учётом того факта, что сервера имеют внутренние IP адреса. После выполнения, система сообщает о результате:
```
PLAY RECAP **************************************************************************************************************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
node1                      : ok=565  changed=124  unreachable=0    failed=0    skipped=1141 rescued=0    ignored=2   
node2                      : ok=369  changed=76   unreachable=0    failed=0    skipped=644  rescued=0    ignored=1   
node3                      : ok=369  changed=76   unreachable=0    failed=0    skipped=643  rescued=0    ignored=1
```
Для просмотра состояния кластера, выполнен вход на первую ноду (на ней находится control plane):

```
~# kubectl get nodes
NAME    STATUS   ROLES                  AGE   VERSION
node1   Ready    control-plane,master   25m   v1.21.3
node2   Ready    <none>                 24m   v1.21.3
node3   Ready    <none>                 24m   v1.21.3
```
Просматриваем состояние активных после инсталляции подов во всех неймспейсах:
```
root@node1:~# kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-5b4d7b4594-gkpjs   1/1     Running   1          24m
kube-system   calico-node-5fd9x                          1/1     Running   0          25m
kube-system   calico-node-9ts7d                          1/1     Running   0          25m
kube-system   calico-node-g88fs                          1/1     Running   0          25m
kube-system   coredns-8474476ff8-7wrjd                   1/1     Running   0          22m
kube-system   coredns-8474476ff8-g722v                   1/1     Running   0          22m
kube-system   dns-autoscaler-7df78bfcfb-dwgl7            1/1     Running   0          22m
kube-system   kube-apiserver-node1                       1/1     Running   0          27m
kube-system   kube-controller-manager-node1              1/1     Running   0          27m
kube-system   kube-proxy-5zd7h                           1/1     Running   0          26m
kube-system   kube-proxy-j2mt8                           1/1     Running   0          26m
kube-system   kube-proxy-lph28                           1/1     Running   0          26m
kube-system   kube-scheduler-node1                       1/1     Running   0          27m
kube-system   nginx-proxy-node2                          1/1     Running   0          26m
kube-system   nginx-proxy-node3                          1/1     Running   0          26m
kube-system   nodelocaldns-8t8zj                         1/1     Running   0          22m
kube-system   nodelocaldns-lql2n                         1/1     Running   0          22m
kube-system   nodelocaldns-zwb6m                         1/1     Running   0          22m
```


3. >Установить и настроить систему мониторинга.

```
# kubectl get pods -n monitoring 
NAME                                   READY   STATUS    RESTARTS   AGE
alertmanager-main-0                    2/2     Running   0          73s
alertmanager-main-1                    2/2     Running   0          73s
alertmanager-main-2                    2/2     Running   0          73s
blackbox-exporter-69894767d5-7b5jw     3/3     Running   0          81s
grafana-7bb5967c6-9s55z                1/1     Running   0          80s
kube-state-metrics-5d6885d89-qrcnp     3/3     Running   0          80s
node-exporter-g6fd5                    2/2     Running   0          80s
node-exporter-txbfl                    2/2     Running   0          80s
node-exporter-z4s74                    2/2     Running   0          80s
prometheus-adapter-6cf5d8bfcf-5crrr    1/1     Running   0          79s
prometheus-adapter-6cf5d8bfcf-rbmzj    1/1     Running   0          79s
prometheus-k8s-0                       2/2     Running   0          72s
prometheus-k8s-1                       0/2     Pending   0          72s
prometheus-operator-7f58778b57-h5zpc   2/2     Running   0          79s
```

4. >Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
5. >Настроить CI для автоматической сборки и тестирования.
6. >Настроить CD для автоматического развёртывания приложения
