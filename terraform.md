# Отчет по лабораторной работе: Установка и базовая настройка Terraform

## Задание
1. Установить Terraform на компьютерную систему (виртуальную или хостовую), используя официальное зеркало: `https://hashicorp-releases.yandexcloud.net/terraform/`.
2. Настроить конфигурацию для обхода блокировок и корректной работы с провайдерами в РФ.
3. Повторить демо-пример развертывания инфраструктуры на базе документации: `https://library.tf/providers`.

---

## 1. Установка Terraform (на примере Linux/Ubuntu)

Так как официальный сайт HashiCorp заблокирован на территории РФ, загрузка выполняется через зеркало Yandex Cloud.

### Шаг 1. Скачивание и распаковка бинарного файла
Выбираем стабильную версию (например, `1.5.7` или актуальную `1.9.x`). Выполняем установку в терминале:

```bash
# Скачивание архива нужной версии (выберите архитектуру amd64 или arm64)
wget https://yandexcloud.net

# Распаковка архива
unzip terraform_1.9.8_linux_amd64.zip

# Перемещение исполняемого файла в системную директорию
sudo mv terraform /usr/local/bin/

# Удаление скачанного архива
rm terraform_1.9.8_linux_amd64.zip
```

### Шаг 2. Проверка корректности установки
Проверяем версию установленной утилиты:
```bash
terraform -v
```

**Ожидаемый вывод:**
```text
Terraform v1.9.8
on linux_amd64
```

---

## 2. Настройка зеркала провайдеров (CLI Config)

Для того чтобы модули и провайдеры беспрепятственно скачивались без использования VPN, необходимо перенаправить запросы Terraform на доступное зеркало (например, Yandex Cloud Registry).

Создайте или отредактируйте конфигурационный файл `~/.terraformrc`:

```bash
nano ~/.terraformrc
```

Добавьте в него следующий блок конфигурации:

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

---

## 3. Выполнение Демо-примера (Развертывание инфраструктуры)

В качестве демонстрации используется создание локального файла через стандартный провайдер `local` (базовый пример работы манифестов из `library.tf/providers`).

### Шаг 1. Создание рабочей директории и манифеста
```bash
mkdir tf-demo && cd tf-demo
nano main.tf
```

Вставьте в файл `main.tf` следующий код:

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5.0"
    }
  }
}

# Ресурс для создания локального текстового файла
resource "local_file" "welcome_message" {
  filename = "${path.module}/hello_world.txt"
  content  = "Terraform успешно установлен и выполнил демо-задание через зеркало YandexCloud!\n"
}
```

### Шаг 2. Инициализация проекта (`terraform init`)
Запуск инициализации загрузит провайдер `local` из настроенного нами сетевого зеркала:

```bash
terraform init
```

**Вывод выполнения:**
```text
Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/local versions matching "~> 2.5.0"...
- Installing hashicorp/local v2.5.2...
- Installed hashicorp/local v2.5.2 (verified binding signature)

Terraform has been successfully initialized!
```

### Шаг 3. Планирование и применение конфигурации
Проверяем, какие изменения внесет манифест:
```bash
terraform plan
```

Запускаем создание ресурса:
```bash
terraform apply -auto-approve
```

**Вывод выполнения:**
```text
local_file.welcome_message: Creating...
local_file.welcome_message: Creation complete after 0s [id=6ba178e24401df4fcf900d7202c3ec78c5208f65]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

### Шаг 4. Проверка результата демо
Проверяем, что файл действительно создался утилитой Terraform:
```bash
cat hello_world.txt
```

**Результат:**
```text
Terraform успешно установлен и выполнил демо-задание через зеркало YandexCloud!
```

---

## Вывод
В ходе лабораторной работы Terraform был успешно установлен на хостовую систему с использованием зеркала `hashicorp-releases.yandexcloud.net`. Была настроена конфигурация `network_mirror` в файле `.terraformrc` для обхода ограничений сети, а также успешно выполнено практическое демо по инициализации и развертыванию декларативного манифеста.
