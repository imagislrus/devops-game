# 🚀 Деплой на AWS EC2

Инструкция по развёртыванию DevOps Defender на Amazon Linux 2023.

## 📋 Предварительные требования

- AWS аккаунт
- EC2 инстанс (t2.micro достаточно)
- Amazon Linux 2023
- Security Group с открытым портом 80

## 🔧 Установка

### 1. Подключитесь к EC2
```bash
ssh -i your-key.pem ec2-user@your-ec2-ip
```

### 2. Установите Apache
```bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

### 3. Скопируйте файлы
```bash
sudo cp *.html /var/www/html/
sudo systemctl restart httpd
```

### 4. Настройте права
```bash
sudo chown -R apache:apache /var/www/html/
sudo chmod -R 755 /var/www/html/
```

## 🌐 Доступ

Откройте в браузере:
- Базовая игра: `http://your-ec2-ip/index-basic.html`
- Продвинутая игра: `http://your-ec2-ip/index.html`
- Базовая обучалка: `http://your-ec2-ip/training-basic.html`
- Продвинутая обучалка: `http://your-ec2-ip/training.html`

## 🔒 Security Group

Откройте порт 80:
- Type: HTTP
- Protocol: TCP
- Port: 80
- Source: 0.0.0.0/0 (или ваш IP)

## 🛠️ Управление сервисом
```bash
# Проверить статус
sudo systemctl status httpd

# Перезапустить
sudo systemctl restart httpd

# Посмотреть логи
sudo tail -f /var/log/httpd/error_log
```

## 📊 Текущий инстанс

- **Instance ID**: i-01d31b35cb413748d
- **Public IP**: 54.160.150.113
- **OS**: Amazon Linux 2023
- **Web Server**: Apache 2.4

## 🎯 Файлы проекта

- `index.html` - Игра продвинутого уровня
- `index-basic.html` - Игра базового уровня
- `training.html` - Обучалка продвинутого уровня
- `training-basic.html` - Обучалка базового уровня
