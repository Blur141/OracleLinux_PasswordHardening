# Oracle Linux 8 – Password Hardening Guide 🔐
## This repository documents the steps and configurations applied for password hardening on Oracle Linux 8 systems.
## ⚠️ Important:
This guide is intended for testing / non-production environments unless explicitly reviewed and approved by security & audit teams.

## ⚠️ Critical Warning (Read Before Applying)
❌ Do NOT blindly execute this script in Production
The configurations shown here overwrite existing .conf files completely.
This may remove default or vendor-recommended settings
Auditors may flag this as non-compliant
## ✅ Recommended for Production:
Open each configuration file manually
Modify only the required parameters
Keep existing defaults intact

## 📌 Pre-Requisites (MANDATORY)

Take Backup of Configuration Files
Always back up before making changes:

`/etc/security/pwquality.conf`
`/etc/login.defs`
`/etc/security/faillock.conf`
`/etc/pam.d/system-auth`
`/etc/pam.d/password-auth`

Example backup command:

`cp /etc/security/pwquality.conf /etc/security/pwquality.conf.bak`

## 🔐 Password Policy Configuration (pwquality)

Strong Password Policy
`cat > /etc/security/pwquality.conf << 'EOF`
`minlen = 10`
`minclass = 3`
`dcredit = -1`
`ucredit = -1`
`lcredit = -1`
`ocredit = -1`
`retry = 3`
`enforce_for_root`

## Password Aging Policy

`sed -ri 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS 90/' /etc/login.defs`
`sed -ri 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS 1/'  /etc/login.defs`
`sed -ri 's/^PASS_WARN_AGE.*/PASS_WARN_AGE 14/' /etc/login.defs`

## 🔒 Account Lockout Policy (faillock)

`cat > /etc/security/faillock.conf << 'EOF'`
`deny = 5`
`fail_interval = 900`
`unlock_time = 900`
`even_deny_root`
`audit`
EOF

### Explanation:

deny = 5 → Lock account after 5 failed attempts
fail_interval = 900 → Count failures within 15 minutes
unlock_time = 900 → Auto unlock after 15 minutes
audit → Logs events (important for compliance)

# 👤 Verify User Password Status

Check password aging details:
`chage -l username`

## 🔄 Apply Policy to Existing Users

Force password change on next login:
`chage -d 0 username`

### (Optional – can be ignored if managed centrally)
`chage -m 1 -M 90 -W 7 username`

Option	Meaning
-m	Minimum days
-M	Maximum days
-W	Warning days

## 🔁 Password History Enforcement (PAM)

To prevent reuse of old passwords, update:

`/etc/pam.d/system-auth`
`/etc/pam.d/password-auth`

## Add or verify the following lines:

`password    requisite    pam_pwquality.so local_users_only retry=3 authtok_type= enforce_for_root`
`password    requisite    pam_pwhistory.so use_authtok enforce_for_root remember=4`
`password    sufficient   pam_unix.so sha512 shadow use_authtok enforce_for_root remember=4`
`password    sufficient   pam_sss.so use_authtok`
`password    required     pam_deny.so`

## 📌 This ensures:

Last 4 passwords cannot be reused

Policy enforced for root as well

### 📖 Reference:
Oracle Linux 8 Security Documentation
https://docs.oracle.com/en/operating-systems/oracle-linux/8/security/security-ConfiguringUserAuthenticationandPasswordPolicies.html

## 🧪 Mandatory Testing Step (VERY IMPORTANT)

Before closing the current SSH/Putty session:
Open a new terminal session
Attempt login

### Verify:
Password complexity
Lockout behavior
Password history enforcement
### ⚠️ This prevents accidental lockout.

## 👨‍💻 User Management

Create User with Sudo Access

`useradd -m username`
`passwd username`
`usermod -aG wheel username`

## 🔑 Generate Strong Random Password ---- >> Can use this for complex password

`openssl rand -base64 14`

## 🗑️ Delete a User (Including Home Directory)

`userdel -r username`

Verify removal:

`getent passwd username`
or
`cat /etc/passwd | grep username`


## 📌 Final Notes

✔ Tested on Oracle Linux 8

✔ Suitable for security hardening documentation

❌ Not recommended as-is for production execution

✅ Ideal as audit reference & implementation guide
