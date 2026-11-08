# HackMePlease

### 1. Remote System Discovery(T1018), Network Service Discovery(T1046)

Шукаємо адрес машини та скануємо сервiси

<img width="653" height="182" alt="image" src="https://github.com/user-attachments/assets/6b44c880-ee84-4774-a3ef-382b46bd29a0" />

Адрес знайшли: 192.168.0.246

<img width="979" height="455" alt="image" src="https://github.com/user-attachments/assets/d50ea2e5-7c61-4adb-bd76-3de1800b2613" />

Знайшли порти 80(http), 3306(mysql) i 33060(mysql protocol listener). Спочатку перевiримо сайт

### 2. File and Directory Discovery(T1083)

На веб-сторiнцi нiчого цiкавого. Шукаємо прихованi директорiї

```bash
gobuster dir -u http://192.168.0.246 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt,zip
```

 <img width="657" height="84" alt="image" src="https://github.com/user-attachments/assets/797a980d-a0e5-4919-b24a-ffe09552b594" />

Цiкавого мало, але перевiримо цi директорiї. В /js/main.js знаходимо коментар:

<img width="762" height="66" alt="image" src="https://github.com/user-attachments/assets/013b3fd9-8f9d-4c64-a28a-30aa11a8c73f" />

Перевiримо:

<img width="1017" height="377" alt="image" src="https://github.com/user-attachments/assets/9dd90ba4-ae7a-485c-a628-c21b872b34eb" />

Сторiнка входу, admin/admin не працює. Жаль. Перевiримо, якi директорiї зможемо знайти там

<img width="387" height="66" alt="image" src="https://github.com/user-attachments/assets/3b3a9510-acab-47b8-81a1-3994d5215b10" />

Це вже цiкаво. В iнтернетi знаходимо, що може мicтитися в /conf/ i заходимо в /conf/settings.xml

### 3. Unsecured Credentials: Credentials in Files(T1552.001), Valid Accounts(T1078)

<img width="1436" height="118" alt="image" src="https://github.com/user-attachments/assets/e8550f98-ce58-487d-9c5d-cdac8d1710ce" />

Знаходимо логiн i пароль до бази даних:
seeddms:seeddms

Пiдключаємося:

<img width="665" height="200" alt="image" src="https://github.com/user-attachments/assets/41290fd6-d010-4ad3-937e-f021d22ac7bb" />

### 4. Data from Information Repositories(T1213)

Перевiримо, що в цiй базi даних

<img width="262" height="211" alt="image" src="https://github.com/user-attachments/assets/cc55032b-ce18-4d03-a70f-3fc49b026832" />

Ранiше ми бачили сторiнку входу в seedms, тому перевiримо базу seeddms:

<img width="291" height="752" alt="image" src="https://github.com/user-attachments/assets/caf19a66-beae-431c-8616-c83906b1e750" />

Бачимо таблицi "users" i "tblUsers"

<img width="677" height="255" alt="image" src="https://github.com/user-attachments/assets/e670fd37-ca36-4511-864a-12ce189b63c5" />

Знайшли пароль працiвника i захешований пароль адмiнiстратора. Входимо з даними працiвника:

<img width="481" height="341" alt="image" src="https://github.com/user-attachments/assets/e743a419-ebbb-465d-9cd4-fc5d8a3c5873" />

Не вишло, тодi треба розшифрувати пароль адмiнiстратора

<img width="942" height="362" alt="image" src="https://github.com/user-attachments/assets/c3fbee95-8c16-40f6-a5ee-5f07863f0699" />

Це MD5, спробуємо пiдiбрати:

```bash
hashcat -m 0 "f9ef2c539bad8a6d2f3432b6d49ab51a" /usr/share/wordlists/rockyou.txt
```

<img width="654" height="530" alt="image" src="https://github.com/user-attachments/assets/4b9bac08-f1e1-4052-8002-58183b0e9806" />

Не вийшло

### 5. Account Manipulation(T1098)

Тодi можна спробувати створити свiй хеш i молитися, що там не повинно бути солi

<img width="827" height="573" alt="image" src="https://github.com/user-attachments/assets/50c013bf-d9ca-4bf8-a4ea-80c0cb3f65bb" />

<img width="819" height="62" alt="image" src="https://github.com/user-attachments/assets/386c4d4f-9ee4-4112-b9f1-2d107153d660" />

Входимо:

<img width="1913" height="595" alt="image" src="https://github.com/user-attachments/assets/5c60e4ab-2703-497e-8bd4-0bcbea0af6fc" />

Працює!

### 6. Command and Scripting Interpreter: PHP(T1059.003), Exploit Public-Facing Application(T1190)

Теперь потрiбен shell. Можемо зробити reverse shell через PHP. Завантажуємо [https://github.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php](звiдси)

<img width="1202" height="163" alt="image" src="https://github.com/user-attachments/assets/916cf2d3-b916-4aeb-8cb2-a0ea2089435a" />

Запускаємо netcat:
```bash
nc -lnvp 12345
```

Переходимо на: http://192.168.0.246/seeddms51x/data/1048576/8/1.php

<img width="921" height="170" alt="image" src="https://github.com/user-attachments/assets/22d7becb-77ce-4250-9160-04f05dae045f" />

Маємо shell вiд iменi www-data

### 7. Valid Accounts(T1078)

<img width="132" height="68" alt="image" src="https://github.com/user-attachments/assets/b500a77e-d34a-48d4-8c09-8f264b1d8eab" />

Бачимо, що iснує користувач "saket", пароль до якого ми вже знайшли

```bash
su saket
```

<img width="206" height="86" alt="image" src="https://github.com/user-attachments/assets/8459f128-6818-4e25-b8f0-e4d8bfb37d30" />

Перевiримо права:

<img width="1114" height="38" alt="image" src="https://github.com/user-attachments/assets/9bbc2c5b-d462-40d6-b2e9-f793ee75747a" />

Потрiбен повноцiнний термiнал.

```bash
script -qc /bin/bash /dev/null
```

<img width="759" height="176" alt="image" src="https://github.com/user-attachments/assets/142a2152-686d-4cf1-9212-a91c67a48d03" />

Маємо (ALL:ALL)ALL!

### 8. Abuse Elevation Control Mechanism: Sudo(T1548.003)

<img width="126" height="87" alt="image" src="https://github.com/user-attachments/assets/5bf6b262-69dc-4c16-ab5e-0323f2778d89" />

Отримали root!

### Перемога!

<img width="1080" height="1351" alt="image" src="https://github.com/user-attachments/assets/520c9473-15e9-4c35-b96d-cb84833cc6e5" />
