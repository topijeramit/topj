# 🛡️ Network Protocol Vulnerability Lab – Walkthrough

## 📌 Objective  
Simulate brute-force attacks on common network services (**FTP**, **TELNET**, **SSH**, **HTTP**), assess their security posture, capture and analyze network traffic, and recommend mitigations.

---

## 🖥️ Lab Environment

| Component     | Details                               |
|---------------|---------------------------------------|
| **Attacker VM** | Kali Linux 2024.4                     |
| **Target VM**   | Metasploitable2 / Custom Vulnerable Linux |
| **Tools Used**  | Hydra, Burp Suite, Wireshark          |

---

# 🧾 Task 1: Enumeration of Target

### 🎯 Goal  
Discover valid usernames and running services on the target system.

---

### 🔍 Step 1.1 – Perform Nmap Scan

**Command Used:**
```bash
nmap -p20,21,22,23,80 <target-ip>
```
![Screenshot 2025-04-17 214346](https://github.com/user-attachments/assets/1cd0a0a5-c008-4874-ab17-3a670d8e2cc9)

---

## 1.2 🧰 Enum4linux Enumeration

Since common ports such as **FTP (21)**, **TELNET (23)**, **SSH (22)**, and **HTTP (80)** are open on the target, we used `enum4linux` to perform enumeration and attempt to extract valid usernames and other valuable SMB-related information.

### 🔧 Command Used:
```bash
enum4linux -a <target-ip>
```
![Screenshot 2025-04-17 214452](https://github.com/user-attachments/assets/15e1e011-11cd-4606-abe0-fdfd75ec00da)
---

# 🔐 Task 2: Brute Force Attacks

## ✅ Preparation

Before initiating brute-force attacks, prepare the required wordlists:

- `userlist.txt` – A list of potential usernames
- `passlist.txt` – A list of potential passwords

### 🗂️ Wordlist Options

You can choose to:

- Use built-in wordlists provided by Kali Linux (e.g., `/usr/share/wordlists/rockyou.txt`)
- Or create simple custom wordlists for testing:

```bash
# Create a username list
echo -e "admin\nmsfadmin\nanonymous\nuser\ntest" > userlist.txt

# Create a password list
echo -e "1234\nmsfadmin\nftp123\nadmin\npassword" > passlist.txt
```

## 🔹 2.1 FTP Brute Force with Hydra

We used **Hydra** to perform a brute-force attack on the FTP service running on the target.

### 🔧 Command Used:
```bash
hydra -L userlist.txt -P passlist.txt ftp://<TARGET_IP> -V
```
![Screenshot 2025-04-17 213621](https://github.com/user-attachments/assets/a09f7343-70f4-4e46-9bfd-eb8651ab1879)
---

## 🔹 2.2 Telnet Brute Force with Hydra
Command to attack Telnet:
```bash
hydra -L userlist.txt -P passlist.txt telnet://<TARGET_IP> -V
```
![Screenshot 2025-04-17 213857](https://github.com/user-attachments/assets/45a06840-0981-4764-a5f7-b0749953d013)
---

## 🔹 2.3 SSH Brute Force with NetExec
Command to attack SSH:
```bash
nxc ssh <TARGET_IP> -u userlist.txt -p passlist.txt
```
![Screenshot 2025-04-17 214121](https://github.com/user-attachments/assets/aa80c2d9-b489-479f-a9d1-299c67f7108a)

---

## 🔹 2.4 HTTP Login Brute Force Using Burp Suite Intruder

### Step 1: Launch Burp’s Embedded Browser
- Open **Burp Suite**.
- Navigate to `Proxy > Intercept`.
- Ensure **Intercept is ON**.
- Click **Open Browser** to launch Burp’s embedded browser.

  ![Screenshot 2025-04-18 004649](https://github.com/user-attachments/assets/434ec4d1-14e1-4aec-b619-7d3880a91a40)


### Step 2: Access the Target (Metasploitable2)
- Open a Firefox browser (or use Burp's browser).
- Enter the target IP address (e.g., `http://<TARGET_IP>`).
- Navigate to the **DVWA** section.
- Login using:
  - **Username:** `admin`
  - **Password:** `password`
- In the left panel, click on **Brute Force**.
- Enter any sample values into the username and password fields (e.g., `aaa` for both), then click **Login**.
![Screenshot 2025-04-18 005800](https://github.com/user-attachments/assets/00278747-c090-4268-b837-82f50c3bb4f1)
![Screenshot 2025-04-18 005901](https://github.com/user-attachments/assets/f484dd59-6303-4784-8649-32a2694ba08d)



### Step 3: Forward the Request
- In Burp’s `Proxy > Intercept` tab, Everytime the intercept get request keep click **Forward** to send the intercepted request.
- If multiple requests are caught, continue forwarding until the page loads.
- Go to `Proxy > HTTP history` tab, and find `http://192.168.65.54/dvwa/vulnerabilities/brute/?username=aaa&password=aaa&Login=Login` then right click and choose **Send to Intruder**.

![Screenshot 2025-04-18 011547](https://github.com/user-attachments/assets/42fbdebc-4e9a-4e73-8174-7f7206d40f4d)

---

### Step 4: Disable Intercept
- Switch **Intercept is OFF** so that future browser requests are not paused.
![Screenshot 2025-04-18 011645](https://github.com/user-attachments/assets/ffb34692-3f80-49d4-98aa-dfede894f190)
---

### Step 5: Configure the Intruder Attack
- In **Intruder** tab:
  - Set **Attack Type** to **Cluster Bomb**.
  - Highlight and mark the username and password fields as **payload positions**.
  - On the **Payload position**, Load with the file in the with the **Username list** (`userlist.txt`) and **Password list** (`passlist.txt`). (`Example:/usr/share/wordlists`)

    ![Screenshot 2025-04-18 012509](https://github.com/user-attachments/assets/a02ce97c-fd4a-4911-942d-d872dfca82c2)
---

### Step 6: Launch the Attack
- Click **Start Attack**.
- Monitor the results by checking the **Response** and **Length** columns.
- Look for responses that differ in length or content.
- You can also click **Render** to view the visual output of each response.
- A successful login might return a message like:  
  **"Welcome to the password protected area admin"**
- Failed attempts usually return:  
  **"Username and/or password incorrect"**

![Screenshot 2025-04-18 013242](https://github.com/user-attachments/assets/baeea78f-9554-4245-90f4-757bb7dbf0e0)
![Screenshot 2025-04-18 013424](https://github.com/user-attachments/assets/6e000750-4cde-43a9-a6dd-8d495b61045a)
![Screenshot 2025-04-18 024126](https://github.com/user-attachments/assets/822b18c3-3da2-469f-81fc-7fef8e9474fa)

---

## 3. Sniffing and Traffic Analysis

**🎯 Goal:** Analyze how user credentials are transmitted over various network protocols using Wireshark, highlighting the difference between plaintext and encrypted traffic.

**🛠 Tool Used:** Wireshark  
**🎯 Target IP:** `192.168.188.117`

---

In this task, we capture and inspect traffic from login attempts using different protocols (FTP, Telnet, SSH) to identify whether the transmitted credentials are visible or encrypted.
