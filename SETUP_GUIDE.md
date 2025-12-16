This document contains the **complete mail server documentation**  
for the **Postfix + Dovecot** setup.

© 2025 Nalbandyan Davit — Terminal NES  
Licensed under the MIT License



# 📧 Mail Server Setup Documentation


<h2 id="intro">🧩 Introduction</h2>
This document provides a complete overview of the **mail server setup** I configured independently.  
The project combines **Postfix** and **Dovecot** to create a functional, secure, and standards-based email system.  

All configuration files presented here are **sanitized** — no private information, credentials, or real domain data are included.  
The goal is to offer a clear, educational reference for anyone learning about Linux-based mail systems.

## ⚠️ Friendly Disclaimer

Setting up a mail server can be mentally demanding.  
Unexpected errors, silent failures, and configuration loops are part of the journey.

- Crying during setup is not allowed *(short breaks are permitted)*.
- Restarting services before reading logs may cause emotional damage.
- If something works after the **7th reload** — do **not** question it.
- Always check logs before blaming Postfix, Dovecot, or yourself.

> **Remember:**  
> *If it compiles, runs, and delivers mail — it is correct.* 😄


---

## 🗂️ Table of Contents

- [1. Introduction](#intro) | Overview of the project and its purpose |
- [2. Overview of Components](#overview) | Short summary of Postfix and Dovecot roles |
- [3. Project Purpose](#purpose) | Goals and learning objectives |
- [4. Structure of the Document](#structure) | How each configuration file is presented |
- [5. Postfix Configuration Files](#postfix) | All main Postfix setup files |
  - [5.1 /etc/postfix/main.cf](#postfix-main-cf) | Main Postfix configuration |
  - [5.2 /etc/postfix/master.cf](#postfix-master-cf) | Postfix service manager |
  - [5.3 /etc/postfix/vmailbox](#postfix-vmailbox) | Virtual mailboxes definition |
  - [5.4 /etc/postfix/vmailalias](#postfix-vmailalias) | Virtual aliases configuration |
- [6. Dovecot Configuration Files](#dovecot) | Main and supporting Dovecot setup files |
  - [6.1 /etc/dovecot/dovecot.conf](#dovecot-conf) | Dovecot main configuration |
  - [6.2 /etc/dovecot/passwd](#dovecot-passwd) | User authentication file |
- [7. SSL/TLS Certificates](#ssl-certs) | Description of the default snakeoil certificates |
- [8. Author’s Note](#authors-note) | Statement on independence and learning intent |
- [9. Credits and Ownership](#credits-and-ownership)

---

<h2 id="overview">⚙️ Overview of Components</h2>

### 📮 Postfix
**Postfix** serves as the **Mail Transfer Agent (MTA)** — responsible for **sending, receiving, and routing email messages**.

In this setup, Postfix handles:
- **SMTP communication** for both outgoing and incoming mail  
- **Relay restrictions** to prevent open relays  
- **TLS encryption** to ensure secure message transport  
- **Virtual mailbox management** (no real system users required)  
- **Integration with Dovecot** for authentication via **SASL**

---

### 🕊️ Dovecot
**Dovecot** operates as the **Mail Delivery Agent (MDA)** and **IMAP/POP3 server**.

It provides:
- **Secure user authentication** through SASL  
- **Access to mail stored in Maildir format**  
- **Seamless integration with Postfix** for virtual user handling  
- **IMAP and POP3 support** for email clients like Thunderbird, Outlook, or Evolution  

---

<h2 id="purpose">🎯 Project Purpose</h2>

The objectives of this project were to:

1. **Build a fully functional Linux mail server** from scratch.  
2. **Use virtual mailboxes** instead of local UNIX users.  
3. **Implement encryption** with TLS and **authentication** with SASL to ensure security.  
4. **Gain hands-on experience** integrating Postfix and Dovecot.  
5. **Document every configuration file** with explanations for learning and reproducibility.

---

<h2 id="structure">📐 Structure of the Document</h2>
Each configuration section that follows includes:

- **File name and location**  
- **Purpose of the file**  
- **Sanitized configuration code**  
- **Detailed explanations** of directives and functions  

This consistent structure ensures the documentation is both **educational** and **reusable** for reference or deployment.

---



**End of Introduction**


<h2 id="postfix">📮 Postfix Configuration Files</h2>
**Postfix** is a powerful, secure, and fast **Mail Transfer Agent (MTA)** used for sending, receiving, and routing email on Linux systems.  
It replaces older systems like Sendmail with a modular, easy-to-configure design while maintaining high reliability and performance.

In this setup, Postfix is responsible for:
- Handling **SMTP** communication for incoming and outgoing mail  
- Applying **relay restrictions** to prevent open relays  
- Enforcing **TLS encryption** for secure message transfer  
- Managing **virtual mailboxes** (no local UNIX users)  
- Using **Dovecot’s SASL** for user authentication  

Together, these features make Postfix a stable backbone for a modern, secure mail server.


<h2 id="postfix-main-cf">📄 File: /etc/postfix/main.cf</h2>

## 🧩 Purpose
Main configuration file for the **Postfix mail server**.  
Defines server identity, virtual mailbox settings, security (TLS, SASL), and integration with **Dovecot** for authentication.

---

## 🧠 Code (with Explanations)

```conf
# Postfix main configuration file
# See /usr/share/postfix/main.cf.dist for a more detailed reference

compatibility_level = 2
# Ensures Postfix uses modern default behaviors while keeping backward compatibility.

# --- System identity and basic setup ---
mail_owner = postfix                      # The system user that owns the mail processes.
smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)   # Banner displayed to remote servers when they connect.
biff = no                                 # Disables 'biff' notifications for new mail (obsolete feature).

data_directory = /var/lib/postfix         # Directory where Postfix stores its internal data.
command_directory = /usr/sbin             # Location of Postfix command binaries.
queue_directory = /var/spool/postfix      # Directory where mail queues are stored.
append_dot_mydomain = no                  # Prevents appending the local domain to addresses.
readme_directory = no                     # Disables automatic README generation.

# --- TLS Configuration ---
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem   # TLS certificate file (replace with real cert).
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key  # TLS private key file (replace with real key).
smtpd_use_tls=yes                                           # Enables TLS encryption for SMTP sessions.
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache  # Cache for inbound TLS sessions.
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache    # Cache for outbound TLS sessions.

# --- Relay Restrictions ---
smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
# Controls which clients can relay mail through this server:
#  - permit_mynetworks: allows local/trusted networks
#  - permit_sasl_authenticated: allows authenticated users
#  - defer_unauth_destination: denies unauthorized mail relaying

# --- Virtual Mailboxes Configuration ---
myhostname = mail.example.com             # The server’s hostname.
mydomain = example.com                    # Your mail domain.
myorigin = $mydomain                      # Default domain appended to mail from local users.
inet_interfaces = all                     # Listen on all available network interfaces.
inet_protocols = ipv4                     # Use IPv4 only (disable IPv6 if not needed).
mydestination = $myhostname, localhost.$mydomain, localhost
# Domains that are delivered locally (not relayed).

virtual_mailbox_domains = example.com     # The domain(s) handled for virtual mailboxes.
virtual_mailbox_base = /var/vmail/mail/   # Base directory for virtual mailbox storage.
home_mailbox = Maildir/                   # Delivery format — “Maildir” style (one file per message).
virtual_mailbox_maps = hash:/etc/postfix/vmailbox
# Maps email addresses to mailbox directories (use postmap to compile this).
virtual_minimum_uid = 2222                # Minimum user ID for mail storage.
virtual_uid_maps = static:2222            # Fixed UID used by Postfix for virtual users.
virtual_gid_maps = static:2222            # Fixed GID for group ownership of mail files.
virtual_alias_maps = hash:/etc/postfix/vmailalias
# Maps aliases (forwarding addresses) for virtual users.
virtual_transport = virtual               # Defines transport method for delivering virtual mail.
dovecot_destination_recipient_limit = 1   # Ensures Dovecot handles one message per delivery (avoids batch).

# --- SASL Authentication (Dovecot Integration) ---
smtpd_sasl_auth_enable = yes              # Enables SMTP authentication.
smtpd_sasl_type = dovecot                 # Specifies Dovecot as the SASL backend.
smtpd_sasl_path = private/auth            # UNIX socket path where Postfix contacts Dovecot for auth.
smtpd_sasl_security_options = noanonymous # Disables anonymous login (requires username/password).
smtpd_tls_auth_only = yes                 # Only allow authentication over encrypted (TLS) connections.
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination
# Defines delivery restrictions:
#  - permit_sasl_authenticated: allows logged-in users
#  - permit_mynetworks: allows trusted networks
#  - reject_unauth_destination: blocks mail not destined for local or virtual domains
```



<h2 id="postfix-master-cf">📄 File: /etc/postfix/master.cf</h2>

## 🧩 Purpose
This file defines how **Postfix services** (SMTP, pickup, cleanup, queue manager, etc.) run.  
It determines which protocols (SMTP, SMTPS, submission) are active, how mail is accepted or relayed, and integrates encryption/authentication rules for both incoming and outgoing mail.

---

## 🧠 Code (with Explanations)

```conf
# Postfix master process configuration file.
# Each line defines a service and its parameters.
# Columns: service | type | private | unpriv | chroot | wakeup | maxproc | command + args

# SMTP service for receiving mail
smtp      inet  n       -       y       -       -       smtpd
# Runs the main SMTP daemon that receives mail from other servers or clients.

# Optional advanced filtering (disabled here)
#smtp      inet  n       -       y       -       1       postscreen

# Pass-through SMTP service for internal processing
smtpd     pass  -       -       y       -       -       smtpd
# Used internally when mail is passed between Postfix components.

# DNS and TLS proxy services (commented out by default)
#dnsblog   unix  -       -       y       -       0       dnsblog
#tlsproxy  unix  -       -       y       -       0       tlsproxy

# --- Submission Service (Port 587) ---
submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission                 # Log under a different tag for clarity
  -o smtpd_tls_security_level=may                   # Allow but don’t require TLS
  -o smtpd_sasl_auth_enable=yes                     # Enable authentication for mail clients
#  -o smtpd_tls_auth_only=yes                       # Uncomment to enforce TLS for login
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
# Only allow authenticated users to send mail through this port.

# --- SMTPS Service (Port 465) ---
smtps     inet  n       -       y       -       -       smtpd
  -o syslog_name=postfix/smtps                      # Separate log identity
  -o smtpd_tls_wrappermode=yes                      # Enable implicit TLS mode
  -o smtpd_sasl_auth_enable=yes                     # Enable authentication
  -o smtpd_client_restrictions=permit_sasl_authenticated,reject
# SMTPS provides encrypted connection from start; used by modern clients.

# Pickup service – receives messages from the maildrop directory (local submission)
pickup    unix  n       -       y       60      1       pickup
# Periodically scans the maildrop queue for new messages.

# Cleanup service – prepares incoming messages for queueing
cleanup   unix  n       -       y       -       0       cleanup
# Adds headers, checks syntax, etc.

# Queue manager – controls message flow in the mail queue
qmgr      unix  n       -       n       300     1       qmgr
# Responsible for scheduling and dispatching mail for delivery.

# TLS manager – handles cached TLS sessions
tlsmgr    unix  -       -       y       1000?   1       tlsmgr

# Rewrite service – modifies addresses and headers
rewrite   unix  -       -       y       -       -       trivial-rewrite

# Bounce and defer handlers
bounce    unix  -       -       y       -       0       bounce
defer     unix  -       -       y       -       0       bounce
trace     unix  -       -       y       -       0       bounce

# Address verification service
verify    unix  -       -       y       -       1       verify

# Flush service – forces delivery retries
flush     unix  n       -       y       1000?   0       flush

# Proxy map services – internal caching
proxymap  unix  -       -       n       -       -       proxymap
proxywrite unix -       -       n       -       1       proxymap

# SMTP client service for sending outgoing mail
smtp      unix  -       -       y       -       -       smtp
relay     unix  -       -       y       -       -       smtp
        -o syslog_name=postfix/$service_name
# Relay handles outbound delivery; $service_name ensures proper log labeling.

# Local mail delivery and virtual mailbox handling
local     unix  -       n       n       -       -       local
virtual   unix  -       n       n       -       -       virtual
# Local: delivers mail to system users (/var/mail)
# Virtual: delivers mail to virtual users (/var/vmail)

# Anvil – rate limiting and connection tracking
anvil     unix  -       -       y       -       1       anvil

# Scache – caches client authentication states
scache    unix  -       -       y       -       1       scache

# Postlog – centralizes Postfix log messages
postlog   unix-dgram n  -       n       -       1       postlogd

# End of file
# Make sure to reload postfix after changes:
#   sudo postfix reload
```

## --- Notes Section ---
```
# 🧩 Ports Used:
# 25/tcp   → Standard SMTP for server-to-server mail.
# 587/tcp  → Submission (STARTTLS, explicit encryption, for user clients).
# 465/tcp  → SMTPS (implicit TLS, used by legacy or secure mail clients).

# ⚙️ Recommended Setup:
# - Keep 'submission' and 'smtps' enabled for client access.
# - Limit port 25 access to other mail servers only.
# - Enforce TLS authentication by uncommenting:
#       -o smtpd_tls_auth_only=yes
# - Reload postfix after edits with:
#       sudo postfix reload

# 📦 Virtual Delivery:
# Integrated with Dovecot for authentication and uses /etc/postfix/main.cf
# setting:
#       virtual_transport = virtual
# This enables mail delivery to virtual users in /var/vmail/.

# 🧠 Quick Summary:
# - SMTPD services handle mail reception.
# - 'pickup' and 'cleanup' prepare mail for queueing.
# - 'qmgr' manages message delivery flow.
# - 'relay' handles outgoing messages.
# - 'anvil' and 'scache' track and limit connections.
# - SMTPS and Submission ports handle encrypted, authenticated client mail.
```

<h2 id="postfix-vmailalias">📄 File: /etc/postfix/vmailalias</h2>

## 🧩 Purpose
This file defines **alias mappings** for virtual email addresses in Postfix.  
It allows a single incoming address (like `support@example.com`) to forward mail to one or multiple recipient addresses.  
Used together with the `virtual_alias_maps` directive in `/etc/postfix/main.cf`.

---

## 🧠 Code (with Explanations)

```conf
# Format:
# <alias_address>    <destination_address(es)>
# Multiple recipients can be separated by commas.

support@example.com    user1@example.com, user2@example.com, user3@example.com, user4@example.com, user5@example.com
# This means: any mail sent to support@example.com will be delivered to
# user1@example.com, user2@example.com, user3@example.com, user4@example.com, and user5@example.com.
# In other words, it's a group/forwarding alias.
```

## --- Notes Section ---

```
#🧩 How It Works:
# - Postfix looks up aliases using the map defined in main.cf:
#       virtual_alias_maps = hash:/etc/postfix/vmailalias
#
# - When someone sends mail to support@example.com,
#   Postfix reads this file and automatically expands it into all listed recipients.

# 🧠 Usage Tips:
# - Each alias must be on its own line.
# - Use commas (no spaces between them) or a single space + comma-separated addresses.
# - Ensure no trailing spaces at the end of lines.

# ⚙️ Activation:
# After editing the file, compile it into a Postfix lookup table:
#       sudo postmap /etc/postfix/vmailalias
#
# Then reload Postfix to apply:
#       sudo postfix reload

# ✅ Verification:
# You can test if the alias map is recognized correctly:
#       postmap -q support@example.com /etc/postfix/vmailalias
# Expected output:
#       user1@example.com, user2@example.com, user3@example.com, user4@example.com, user5@example.com

# 📦 Related Files:
# - /etc/postfix/main.cf  → declares the alias map path
# - /etc/postfix/vmailbox → defines mailbox storage locations for virtual users

```

<h2 id="postfix-vmailbox">📄 File: /etc/postfix/vmailbox</h2>

## 🧩 Purpose
This file defines **virtual mailboxes** for Postfix.  
Each line maps an email address to its **mail storage directory** (usually under `/var/vmail/`).  
Used together with `virtual_mailbox_maps` from `/etc/postfix/main.cf`.

---

## 🧠 Code (with Explanations)

```conf
# Format:
# <email_address>    <mail_directory_path>
# The path is relative to the base directory defined in main.cf:
#       virtual_mailbox_base = /var/vmail/mail/

user1@example.com    example.com/user1/
# Mail for user1@example.com will be stored in:
#   /var/vmail/mail/example.com/user1/

user2@example.com    example.com/user2/
# Mail for user2@example.com will be stored in:
#   /var/vmail/mail/example.com/user2/

user3@example.com    example.com/user3/
# Mail for user3@example.com will be stored in:
#   /var/vmail/mail/example.com/user3/

user4@example.com    example.com/user4/
# Mail for user4@example.com will be stored in:
#   /var/vmail/mail/example.com/user4/
```
## --- Notes Section ---
```
# 🧩 How It Works:
# - Postfix uses this file to locate the mailbox directories for virtual users.
# - Combined with main.cf setting:
#       virtual_mailbox_maps = hash:/etc/postfix/vmailbox
#   It tells Postfix where to deliver mail for each address.

# 📁 Directory Structure:
# Assuming main.cf has:
#       virtual_mailbox_base = /var/vmail/mail/
# Then the actual storage paths will be:
#       /var/vmail/mail/example.com/user1/
#       /var/vmail/mail/example.com/user2/
#       /var/vmail/mail/example.com/user3/
#       /var/vmail/mail/example.com/user4/

# ⚙️ Activation:
# After creating or editing this file, run:
#       sudo postmap /etc/postfix/vmailbox
#
# Then reload Postfix:
#       sudo postfix reload

# ✅ Verification:
# To confirm that the mapping works:
#       postmap -q user1@example.com /etc/postfix/vmailbox
# Expected output:
#       example.com/user1/

# 🧠 Tips:
# - Every line must have a valid email address followed by a mailbox path.
# - Use trailing slashes for directories.
# - All mailbox directories should be owned by the UID/GID defined in main.cf:
#       virtual_uid_maps = static:2222
#       virtual_gid_maps = static:2222
# - Make sure each user’s directory exists:
#       sudo mkdir -p /var/vmail/mail/example.com/user1
#       sudo chown -R 2222:2222 /var/vmail/mail/
```



<h2 id="dovecot">🕊️ Dovecot Configuration Files</h2>

This section configures **Dovecot** to serve IMAP (port 143/STARTTLS and 993/IMAPS), authenticate users for Postfix via a UNIX socket, and read mail stored under `/var/vmail/mail/%d/%n/` (matching your Postfix virtual setup).

---

<h2 id="dovecot-conf">📄 File: /etc/dovecot/dovecot.conf</h2>

## 🧩 Purpose
Top-level **Dovecot** configuration for virtual mailboxes.  
Runs IMAP (143/STARTTLS, 993/IMAPS), requires TLS, authenticates users from a passwd-file, serves mail from `/var/vmail/mail/%d/%n`, and exposes a SASL socket for Postfix at `/var/spool/postfix/private/auth`.

---

## 🧠 Code (with Explanations)

```conf
# --- Core includes & protocols ---
!include_try /usr/share/dovecot/protocols.d/*.protocol
!include conf.d/*.conf
!include_try local.conf

listen = *                                   # Listen on all interfaces (IPv4/IPv6 depends on system defaults)
protocols = imap                             # Enable IMAP (add 'pop3' or 'lmtp' if you use them)

# --- TLS / SSL ---
ssl = required                               # Force encryption for all logins
ssl_key  = </etc/ssl/private/ssl-cert-snakeoil.key   # REPLACE in production
ssl_cert = </etc/ssl/certs/ssl-cert-snakeoil.pem     # REPLACE in production
# Optional hardening:
# ssl_min_protocol = TLSv1.2

# --- Auth & security ---
disable_plaintext_auth = yes                 # No plaintext auth unless inside TLS (we require TLS anyway)
auth_mechanisms = plain login                # Common mechanisms; clients like Thunderbird/Outlook support these

# --- UID/GID & access ---
mail_access_groups = vmail                   # Allow group access if needed
default_login_user = vmail                   # Default login user context (optional; safe to keep)
first_valid_uid = 2222                       # Must match Postfix virtual_uid_maps
first_valid_gid = 2222                       # Must match Postfix virtual_gid_maps

# --- Mail storage paths ---
# Maildir for each user lives under: /var/vmail/mail/<domain>/<user>
# (Dovecot creates a Maildir/ structure inside that path automatically)
mail_home     = /var/vmail/mail/%d/%n
mail_location = maildir:/var/vmail/mail/%d/%n

# --- Passdb / Userdb (virtual users) ---
# Password file with hashed passwords (see Notes for how to generate)
passdb {
  driver = passwd-file
  args = scheme=SSHA512 /etc/dovecot/passwd     # Keep scheme in sync with hashes you generate
}

# Map all users to the vmail uid/gid and correct home path
userdb {
  driver = static
  args = uid=2222 gid=2222 home=/var/vmail/mail/%d/%n allow_all_users=yes
}

# --- SASL socket for Postfix integration ---
# Postfix main.cf expects: smtpd_sasl_type = dovecot ; smtpd_sasl_path = private/auth
# So we expose the socket inside Postfix's chroot at /var/spool/postfix/private/auth
service auth {
  unix_listener /var/spool/postfix/private/auth {
    user = postfix
    group = postfix
    mode = 0660
  }
  # Dovecot’s internal userdb socket (optional to keep)
  unix_listener auth-userdb {
    mode = 0660
    user = 2222
    group = 2222
  }
  user = root
}

# --- IMAP login service tuning (optional, safe defaults shown) ---
service imap-login {
  service_count = 1
  client_limit = 300
  process_limit = 300
  process_min_avail = 4
  vsz_limit = 512M
}

# --- Namespaces / Special-use mailboxes ---
namespace inbox {
  inbox = yes
  mailbox Junk {
    auto = subscribe
    special_use = \Junk
  }
  mailbox Trash {
    auto = subscribe
    special_use = \Trash
  }
  mailbox Sent {
    auto = subscribe
    special_use = \Sent
  }
}

# Keep the standard conf.d includes at the end
!include conf.d/*.conf
!include_try local.conf
```
## --- Notes Section ---
```
# 🔐 Certificates
# - Replace snakeoil cert/key with your real certs (e.g., Let's Encrypt):
#     ssl_cert = </etc/letsencrypt/live/your.domain/fullchain.pem
#     ssl_key  = </etc/letsencrypt/live/your.domain/privkey.pem

# 🧾 Password hashes
# - Generate SSHA512 hashes to match "scheme=SSHA512":
#     doveadm pw -s SSHA512
# - Put entries in /etc/dovecot/passwd like:
#     user1@example.com:{SSHA512}BASE64_HASH_HERE
# - If you prefer SHA512-CRYPT, change both the file and "scheme=" accordingly.

# 📁 Permissions & directories
# - Ensure vmail tree exists and is owned by UID/GID 2222:
#     sudo mkdir -p /var/vmail/mail/example.com/user1
#     sudo chown -R 2222:2222 /var/vmail
# - This must match Postfix:
#     virtual_uid_maps = static:2222
#     virtual_gid_maps = static:2222

# 🔄 Postfix ↔ Dovecot SASL
# - Postfix main.cf:
#     smtpd_sasl_type = dovecot
#     smtpd_sasl_path = private/auth
# - This config exposes: /var/spool/postfix/private/auth with mode 0660

# ✅ Validate & reload
# - Check Dovecot config:  sudo dovecot -n
# - Reload Dovecot:        sudo systemctl reload dovecot
# - Reload Postfix:         sudo postfix reload

# 🧪 Quick tests
# - Test IMAPS (993):       openssl s_client -connect mail.example.com:993
# - Test IMAP STARTTLS:     openssl s_client -starttls imap -connect mail.example.com:143

# ⚠️ Consistency gotchas (fixed above)
# - Use "Maildir" (not "MailDir"); Dovecot handles Maildir/ internally.
# - Keep mail_home and mail_location under the same base (/var/vmail/mail/%d/%n).
# - Make sure the SASL socket path matches Postfix (private/auth inside Postfix spool).
```
<h2 id="dovecot-passwd">📄 File: /etc/dovecot/passwd</h2>

## 🧩 Purpose
Stores **virtual mail user credentials** for Dovecot’s `passwd-file` authentication driver.  
Each line defines a user’s login (email address), password hash, UID/GID, and mailbox path.  
Sensitive data such as actual usernames, domains, and password hashes are **censored** below for security.

---

## 🧠 Code (with Explanations)

```conf
# Format:
# user@domain:{SCHEME}hashed_password:uid:gid::/home_directory
# Fields are colon-separated; empty fields can be left blank (use ::)

user@example.com:{SSHA512}**********=:2222:2222::/var/vmail/mail/example.com/user
# 1️⃣ user@example.com           → full login username (email address)
# 2️⃣ {SSHA512}**********=       → password hash (censored)
# 3️⃣ :2222:2222:                → UID and GID (must match Postfix)
# 4️⃣ ::                         → unused field, left empty
# 5️⃣ /var/vmail/mail/.../user   → user’s mailbox home directory

# Example (structure only, data censored):
# user2@example.com:{SSHA512}**********=:2222:2222::/var/vmail/mail/example.com/user2
# user3@example.com:{SSHA512}**********=:2222:2222::/var/vmail/mail/example.com/user3
```
## --- Notes Section ---
```
# 🔐 Generating Passwords:
# Create SSHA512 password hashes with:
#     sudo doveadm pw -s SSHA512
# Copy and paste the resulting hash into this file (inside {SSHA512}…).

# ⚙️ File Permissions:
# Protect the file from unauthorized access:
#     sudo chown dovecot:dovecot /etc/dovecot/passwd
#     sudo chmod 640 /etc/dovecot/passwd

# 📁 Paths & UID/GID:
# - Must match Postfix:
#     virtual_uid_maps = static:2222
#     virtual_gid_maps = static:2222
#
# - Mail directories must exist:
#     /var/vmail/mail/example.com/<username>
# Example:
#     sudo mkdir -p /var/vmail/mail/example.com/user
#     sudo chown -R 2222:2222 /var/vmail/mail

# 🧠 Integration with Dovecot:
# Enabled via dovecot.conf:
#     passdb {
#       driver = passwd-file
#       args = scheme=SSHA512 /etc/dovecot/passwd
#     }

# ✅ Test login manually:
#     sudo doveadm auth test user@example.com yourpassword
# Expect "auth succeeded" if configuration is correct.

# 🔄 Apply changes:
#     sudo systemctl reload dovecot
```

<h2 id="ssl-certs">📄 Directory: /etc/ssl/ (Certificate Storage)</h2>

## 🧩 Purpose
Contains the **SSL/TLS certificates and private keys** used by both **Postfix** and **Dovecot** to secure communication channels (SMTP, IMAP, POP3).  
In this setup, the system uses the **default “snakeoil” certificates** that are auto-generated during installation — suitable for **testing** or **internal mail networks**.

---

## 🧠 Structure and File Paths

```bash
/etc/ssl/
├── certs/
│   └── ssl-cert-snakeoil.pem       # Public certificate file
└── private/
    └── ssl-cert-snakeoil.key       # Private key file (root-only access)
```

## How they are used
```
# In Postfix main.cf:
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key

# In Dovecot dovecot.conf:
ssl_cert = </etc/ssl/certs/ssl-cert-snakeoil.pem
ssl_key  = </etc/ssl/private/ssl-cert-snakeoil.key
```

## --- Notes Section ---

```
# 🔒 Permissions:
# Ensure ownership and permissions are restricted:
#     sudo chown root:root /etc/ssl/private/ssl-cert-snakeoil.key
#     sudo chmod 600 /etc/ssl/private/ssl-cert-snakeoil.key

# ⚙️ For Production:
# Replace “snakeoil” with real certificates from Let's Encrypt or another CA:
# Example (Let's Encrypt):
#     ssl_cert = </etc/letsencrypt/live/mail.example.com/fullchain.pem
#     ssl_key  = </etc/letsencrypt/live/mail.example.com/privkey.pem
#
# Then update both Postfix and Dovecot configs with those new paths, and reload:
#     sudo systemctl reload postfix
#     sudo systemctl reload dovecot
```
---

<h2 id="authors-note">✍️ Author’s Note</h2>
This project and all configurations were prepared **independently**, using system documentation, manual testing, and configuration references.  
The primary purpose is **educational** — to demonstrate a working understanding of:

- Linux-based mail systems  
- Postfix and Dovecot integration  
- Secure server management practices  
- Standards-based email architecture

---

<h2 id="credits-and-ownership">© Credits and Ownership</h2>

This documentation was developed by **Nalbandyan Davit**  
for **Terminal NES (Nalbandyan Engineering Solutions)**.

The project is shared for educational purposes.  
Reuse, modification, and redistribution are permitted under the terms of the **MIT License**,  
provided proper attribution is given to the author and Terminal NES.

© 2025 Davit Nalbandyan — Terminal NES




