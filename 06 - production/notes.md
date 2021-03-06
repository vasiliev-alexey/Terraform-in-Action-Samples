## Код Terraform промышленного уровня

### Требования к инфраструктуре промышленного уровня

#### Список задач для подготовки инфраструктуры промышленного уровня
* Установка
* Инициализация
* Развертывание
* Высокая доступность
* Масштабируемость
* Производительность
* Сеть
* Безопасность
* Журналирование
* Резервное копирвание и восстановление
* Оптимизация расходов
* Документация
* Тесты


## инфпаструктурные модули промышленного уровня
* Небольшие модули
* Компонуемые модули
* Тестируемые модули
* Модули готовые к выпуску
* Модули вне Terraform

### Небольшие модули
* Большие модули медленны
* Большие модули небезопасны
* Большие модули несут в себе риски
* Большие модули сложны в понимании
* Большие модули сложно разбирать
* Большие модули сложно тестировать

Инфраструктурный код должен состоять из множества, компактных модулей, с узкой специализацией.


### Компонуемые модули
Модули компонуются с целью избежания side-эфектов

### Тестируемы модули
У каждого модуля должна быть папка с тестами (test)
У каждого модуля должен быть файл с примером (example)
Следует закрпить модули за определенной версией terraform, поскольку версии не имеют обратную совместимость

    terraform {
    # Требуем любую версию Terraform вида 0.12.x
    required_version = ">= 0.12, < 0.13"
    }

    terraform {
    # Требуем исключительно версию Terraform 0.12.0
    required_version = "= 0.12.0"
    }

Рекомендуется также закреплять версии провайдеров

    provider "google" {
    version = "~> 3.15.0"
    }

### Модули, готовые к повторному использованию
Следует версионировать модули через бренчи и теги гита
Можно переиспользовать модули из реестра terraform registry.terraform.io

Модуль можно отправить в реестр терраформ, если он соответсвует определнным критериям:
* Модуль должен находиться в репозитории GitHub 
* Репозиторий должен иметь название вида terraform-<PROVIDER>-<NAME>, где 
PROVIDER — это провайдер, на которого рассчитан модуль (например, aws), 
а NAME — имя модуля (скажем, vault) 
* Модулю нужна определенная структура файлов: код Terraform должен находиться в корне репозитория, у вас должен быть файл README md и необходимо соблюдать соглашение об именовании файлов main.tf, variables.tf 
и outputs.tf 
* Для выпуска кода в репозитории нужно использовать теги Git и семантическое 
версионирование (x.y.z) 


При соблюдении всех этих требований модуль можно опубликовать в реестре, и далее использовать уже сокращенную сементику пути

    module "<NAME>" {
    source = "<OWNER>/<REPO>/<PROVIDER>"
    version = "<VERSION>"
    # (...)
    }


## За пределами возможностей Terraform-модулей
* средства инициализации ресурсов;
* средства инициализации ресурсов с использованием null_resource;
* внешний источник данных 

### Средства инициализации ресурсов
Terraform использует средства инициализации ресурсов во время запуская для выполнения скриптов на удаленном или локальном ресурсе
* local-exec  - скрипт в локальной системе
* remote-exec - выполнение скрипта на удаленном хосте
* chef - апускает Chef Client для удаленного ресурса)
* file  - копирует файл на удаленный ресурс


Чтобы добавить в ресурс средства инициализации, можно воспользоваться блоком 
provisioner 

        resource "aws_instance" "example" {
        ami = "ami-0c55b159cbfafe1f0"
        instance_type = "t2 micro"
        provisioner "local-exec" {
        command = "echo \"Hello, World from $(uname -smp)\""
        }
        }

### Средства инициализации ресурсов с использованием null_resource
У null_resource есть удобный аргумент под названием triggers, который принимает ассоциативный массив с ключами и значениями. При любом измнеиии ресурсов будет выполнятся блок null_resource

    triggers = {
    uuid = uuid()
    }

### Внешний источник данных
Иногда в качестве источника данных требуется использовать  внешние программы - для этого используется источник данных external, который должен реализоавать следующий протокол:
*  принимать в  STDIN -> JSON
*  вывод программы должен происходить в STDOUT <- в формате JSON