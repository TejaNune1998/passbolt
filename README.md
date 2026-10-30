# Passbolt : Password Manager

## What is a Password Manager?

A password manager is an application that can generate strong passwords and store them in one place.

## Why a Password Manager in a *HomeLab*?

A password manager helps keep these credentials secure, organized, and accessible without reusing passwords or storing them in plain-text notes.

- **Centralized secrets:** Keep passwords, API keys, recovery codes, and service credentials in one encrypted vault.
- **Unique passwords:** Generate strong, unique passwords for every service, from Proxmox to Pi-hole.
- **Secure access:** Access credentials across your devices without relying on memory.
- **Safer sharing:** Share selected credentials with trusted users without exposing your entire vault.
- **Less plain text:** Avoid keeping sensitive credentials in unencrypted documents or notes.

Hosting a password manager in your homelab gives you greater control over your infrastructure and data. However, it also makes the service part of your security boundary.

***Remember:** A password manager protects stored credentials, but it doesn't replace secure server configuration, proper access controls, or a reliable backup strategy.*

## What is Passbolt?

Passbolt is an open-source password manager for storing, organizing, and sharing credentials. It uses OpenPGP encryption to protect credentials. The user's passphrase protects their private key, which is used to decrypt credentials when needed.


```
       🔑                          🔐
  Passphrase ──────────────>  Protected Private Key
                                      │
                                      ▼
                             🔒 Encrypted Credentials
                                      │
                                      ▼
       💻📱 <──────────────── Decrypted Credentials
      Devices
```
---
***Note***: *Self-hosting* means you are responsible for keeping Passbolt updated, securing access, and maintaining reliable backups. Keep your recovery information safe, too, because losing the necessary private-key or account recovery material can affect your ability to access encrypted credentials.

---
## Deploying Passbolt

### Prerequisites
* A system with Docker installed.
* Pi-Hole
* Traefik and CloudFlared tunnel
* A registered domain from a provider (e.g., Cloudflare)
* SMTP provider

---
### Create a `CNAME` record for *Passbolt* and *Traefik Docker Host*

*This is a local Pi-hole DNS record pointing to the Docker host.*

* Log in to Pi-Hole.
* Create a CNAME record as below and click "+" to add.

| Domain                  | Target                     |
| ----------------------- | -------------------------- |
| passbolt.yourdomain.com | docker_host.yourdomain.com 

---
### Download docker-compose.yml

```URL
git clone https://github.com/TejaNune1998/passbolt.git
```
---
### Directory Structure 

```
├── docker-compose.yml
└── .env
```
---
### Update `docker-compose.yml` file

* Update your domain

```yml
   environment:
      - APP_FULL_BASE_URL=https://passbolt.yourdomain.com
```

---
### Update `.env` file

* Update DB password and mail provider and mail ID credentials details.

```env
# DB
MYSQL_PASSWORD=''

# Email
EMAIL_TRANSPORT_DEFAULT_HOST=''
EMAIL_TRANSPORT_DEFAULT_PORT=''
EMAIL_TRANSPORT_DEFAULT_USERNAME=''
EMAIL_TRANSPORT_DEFAULT_PASSWORD=''
```

*SMTP is required for invitation and notification emails, and test SMTP delivery before inviting users.*

---
### Update `traefik/config/config.yml` file

* Passbolt details in *Traefik's Config yaml* file

```yml
http:
  routers:
    passbolt:
      entryPoints:
        - "https"
      rule: "Host(`passbolt.yourdomain.com`)"
      middlewares:
        - default-headers
        - crowdsec-bouncer
      service: passbolt
      tls:
        certResolver: cloudflare
  services:
    passbolt:
      loadBalancer:
        servers:
          - url: "https://IP-Address:9443"      
        passHostHeader: true
```

---
### Start the *Docker* Compose stack

```
docker compose up -d
```

---
### Set up Admin account

*Note: Wait until both the database and application containers are running and healthy.*

```
docker compose exec passbolt su -m -c "/usr/share/php/passbolt/bin/cake \
  passbolt register_user \
    -u 'your-email@example.com' \
    -f 'YourFirstName' \
    -l 'YourLastName' \
    -r admin" -s /bin/sh www-data
```

* Open the setup URL from the command output to complete admin account setup.
* Complete the browser-extension setup and securely save the Recovery Kit.

---
### Expose *Passbolt* Service via *Cloudflare Tunnel*

```Cloudflare
Log in to Cloudflare → Select your Domain → DNS → Records → Add Record
```

```Record
Type: CNAME
Name: passbolt
Target: yourdomain.com
Proxy Status: Proxied (Orange Cloud)
```

***Note**: This CNAME points to the root domain, which is needed to be link to the Cloudflare Tunnel.*

---
## Post-Deployment

#### User account Creation

```Web
Login → Settings → Manage Users & Groups → Create → User
```

```Creation
First name   
Last name
Username / Email
Role: User
        
- Click Save.
```

Passbolt sends an invitation email to the user's registered email address. The user must complete the initial setup using a supported desktop browser and the Passbolt browser extension before connecting the mobile app.
### User Account and Device 

- Open the invitation email on a desktop or laptop.
- Complete the browser-extension setup and securely save the Recovery Kit.
- Install the Passbolt iOS or Android app.
- Scan the QR code displayed in the user's Passbolt browser profile.
- Enter the user's passphrase and configure Face ID or autofill if supported.

***Note:** Keep the Recovery Kit in a secure location. It can be important for recovering access to the user's account and encrypted credentials.*

---


