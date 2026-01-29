# Oracle Linux 8 – Password Hardening Guide 🔐 #
### Overview ###

***This repository documents the manual steps and configuration changes applied for password hardening on Oracle Linux 8 systems.***

## ⚠️ Important ##

This guide is intended for testing or controlled environments unless explicitly reviewed and approved by Security and Audit teams.

## ⚠️ Critical Warning (Read Before Applying) ##

## ❌ Do NOT blindly execute scripts in Production ##

### **Password hardening involves critical authentication files*** ###

***Incorrect changes may cause system lockout***

***Overwriting configuration files may:***

***Remove vendor defaults***

***Break compliance requirements***

***Trigger audit findings***

## ✅ Recommended Production Approach ##

***✔ Open each configuration file one by one***
***✔ Modify only the required parameters***
***✔ Preserve existing defaults and comments***
***✔ Maintain before & after backups***

### 📌 Pre-Requisites (MANDATORY) ###
### ***Take Backup of Configuration Files*** ###

***Always back up before making any changes***
```
/etc/security/pwquality.conf
/etc/login.defs
/etc/security/faillock.conf
/etc/pam.d/system-auth
/etc/pam.d/password-auth
```

***Example backup command***
```
cp /etc/security/pwquality.conf /etc/security/pwquality.conf.bak
```

### 🔐 Password Policy Configuration (pwquality) ###
File: `/etc/security/pwquality.conf`
***Open the file using an editor***
```
vi /etc/security/pwquality.conf
```
***Ensure the following parameters are present or updated:***

```
minlen = 10
minclass = 3
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
retry = 3
enforce_for_root
```
***📌 Do not remove other existing parameters unless reviewed.***

### ⏳ Password Aging Policy ###
File: `/etc/login.defs`

Edit the file:

```
vi /etc/login.defs
```

Verify or update the following values:
```
PASS_MAX_DAYS   90
PASS_MIN_DAYS   1
PASS_WARN_AGE   14
```

### 🔒 Account Lockout Policy (faillock) ###
File: `/etc/security/faillock.conf`

***Edit manually:***

```
vi /etc/security/faillock.conf
```

***Ensure the following entries exist:***

```
deny = 5
fail_interval = 900
unlock_time = 900
even_deny_root
audit
```

### Explanation ###

***deny = 5 → Lock account after 5 failed attempts***
***fail_interval = 900 → Track failures within 15 minutes***
***unlock_time = 900 → Auto-unlock after 15 minutes***
***audit → Enables logging for compliance***

### 👤 Verify User Password Status ###

***Check password aging details for any user:***

```
chage -l username
```

### 🔄 Apply Policy to Existing Users ###

***Force password change on next login***

```
chage -d 0 username
```

***(Optional – if not centrally managed)***
`chage -m 1 -M 90 -W 7 username`

***Option	Meaning***
***-m	Minimum days***
***-M	Maximum days***
***-W	Warning days***

### 🔁 Password History Enforcement (PAM) ###

To prevent reuse of old passwords, update the following files manually:

```
/etc/pam.d/system-auth
/etc/pam.d/password-auth
```
***Add or verify these lines:***
```
password    requisite    pam_pwquality.so local_users_only retry=3 authtok_type= enforce_for_root
password    requisite    pam_pwhistory.so use_authtok enforce_for_root remember=4
password    sufficient   pam_unix.so sha512 shadow use_authtok enforce_for_root remember=4
password    sufficient   pam_sss.so use_authtok
password    required     pam_deny.so
```
### 📌 This Ensures ###

***Last 4 passwords cannot be reused***
***Password policy enforced for root user***
***Strong alignment with security best practices***

### 📖 Reference ###

***Oracle Linux 8 Security Documentation***
`https://docs.oracle.com/en/operating-systems/oracle-linux/8/security/security-ConfiguringUserAuthenticationandPasswordPolicies.html`

### 🧪 Mandatory Testing Step (VERY IMPORTANT) ###

***Before closing the current SSH / Putty session:***

***Open a new terminal session***

***Attempt login with:***

***Existing user***

***New password***

### ***Verify*** ###

***Password complexity enforcement***

***Account lockout behavior***

***Password history restriction***

### ***⚠️ This step prevents accidental system lockout.*** ###

### 👨‍💻 User Management ###
***Create User with Sudo Access***
```
useradd -m username
passwd username
usermod -aG wheel username
```

###🔑 Generate Strong Random Password ###
```
openssl rand -base64 14
```
***✅ Can be used to generate complex passwords for users.***

### 🗑️ Delete a User (Including Home Directory) ###

```
userdel -r username
```

***Verify removal:***
```
getent passwd username
```
or
```
cat /etc/passwd | grep username
```

### ***📌 Final Notes*** ###

***✔ Tested on Oracle Linux 8***
***✔ Suitable for security hardening documentation***
***❌ Not recommended as-is for direct production execution***
***✅ Ideal as an audit reference & implementation guide***

