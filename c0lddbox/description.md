# C0lddbox

### 1. Active Scanning(T1595), Scanning IP Blocks(T1595.001)

Шукаємо адрес машини

<img width="635" height="229" alt="image" src="https://github.com/user-attachments/assets/5d850710-33bb-4dc5-ba69-1577917bfa96" />

Знайшли: 192.168.0.35

Скануємо:

<img width="694" height="292" alt="image" src="https://github.com/user-attachments/assets/b9dad0f3-df12-4efe-997e-2865592702f4" />

Знайшли http i ssh порти

### 2. Gather Victim Identity Information(T1589), Valid Accounts(T1078)

На http-сторiнцi одразу бачимо сторiнку логiну. Шукаємо прихованi директорiї:

<img width="861" height="387" alt="image" src="https://github.com/user-attachments/assets/2d92ab37-d2e9-4a4c-b31d-bd56ad17beb7" />

Переходимо в /hidden

<img width="1012" height="251" alt="image" src="https://github.com/user-attachments/assets/4a591543-5f3f-41aa-a442-28389c5ab13e" />

Нiчого цiкавого. Знаємо, що сайт зробили за допомогою WordPress, скануємо:

<img width="690" height="260" alt="image" src="https://github.com/user-attachments/assets/b3d2aeaa-5171-4609-93dc-d91cca6229d8" />

Знайшли користувачiв: c0ldd, hugo, philip

### 3. Brute Force(T1110)

Пiдберемо пароль до одного к користувачiв

```bash
wpscan --url http://192.168.0.35/ -U c0ldd -P /usr/share/wordlists/rockyou.txt
```

<img width="263" height="34" alt="image" src="https://github.com/user-attachments/assets/9de9d14d-172e-4ea2-8ebe-660e2d1e478e" />

Отримали доступ

<img width="1920" height="848" alt="image" src="https://github.com/user-attachments/assets/db9aeb15-7d50-47db-b4b0-f393aa050b6f" />

### 4. Command and Scripting Interpreter(T1059), Unix Shell(T1059.004)

Заходимо в Appearance -> Editor i змiнюємо footer.php на код для reverse shell'a
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

<img width="919" height="596" alt="image" src="https://github.com/user-attachments/assets/1889510d-cd38-41f4-a928-b2ef721308ac" />

Запускаємо netcat i обновляємо головну сторiнку

<img width="929" height="169" alt="image" src="https://github.com/user-attachments/assets/7dffabce-b1b4-4145-a8d7-460fe2811332" />

Маємо shell! Але не повноцiнний, треба пiдключитися через ssh

### 5. Credentials in Files(T1552.001)
Шукаємо, що є на цiй машинi

<img width="291" height="149" alt="image" src="https://github.com/user-attachments/assets/1010aeb9-5c46-4115-8673-027b9dde8551" />

Знаходимо, що є один користувач: c0ldd. Шукаємо далi

<img width="674" height="200" alt="image" src="https://github.com/user-attachments/assets/b0a99e7d-70cd-41c5-b942-40f584096548" />

В /var/www/http знайши пароль до бази даних

### 6. Valid Accounts(T1078)

Спробую зайти за допомогою цього пароля через ssh

<img width="756" height="443" alt="image" src="https://github.com/user-attachments/assets/d1cbcb65-75cf-4b8b-be89-1df8daed856d" />

Отримали повноцiнний доступ! Бачимо, якi у нас права:

<img width="958" height="158" alt="image" src="https://github.com/user-attachments/assets/38709990-b871-4511-9221-a3966134b1af" />

Ще не root, але у нас є доступ до chmod, тому можна змiнювати права як нам потрiбно

### 7. Sudo and Sudo Caching(T1548.003)

Змiнюємо права доступу до /root/ i читаємо флаг

<img width="477" height="282" alt="image" src="https://github.com/user-attachments/assets/4ef55cc3-be1e-46ba-9482-19d4cda0701a" />

Флаг закодований за допомогою base64, розшифровуємо:

<img width="500" height="685" alt="image" src="https://github.com/user-attachments/assets/c1302374-2868-4530-98e4-02de90baf595" />

### Перемога!

<img width="720" height="500" alt="image" src="https://github.com/user-attachments/assets/6a3b46b1-61c0-4e37-ada5-6c98e9c341cd" />
