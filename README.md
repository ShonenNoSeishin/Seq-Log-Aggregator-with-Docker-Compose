<a href="https://www.buymeacoffee.com/thibaut_watrisse" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

# Seq as a Log Aggregator with Docker Compose (Example by logging Proxmox)

## Overview
This repository provides a simple setup for using [Seq](https://datalust.co/seq) as a log aggregator with `docker compose`. The configuration includes a syslog input for ingesting logs from various sources, specifically for **Proxmox** environments.

## Requirements
- [Docker](https://www.docker.com/) installed
- Basic knowledge of `docker compose`
- A **Proxmox** node to send logs from

## Setup
### 1. Clone the Repository
```sh
git clone https://github.com/ShonenNoSeishin/Seq-Log-Aggregator-with-Docker-Compose.git
cd Seq-Log-Aggregator-with-Docker-Compose 
```

### 2. Generate the Password Hash
To store a secure password for Seq, generate a hash and store it in a `.env` file:
```sh
echo "SEQ_ADMIN_PASSWORD_HASH=$(echo 'password' | docker run --rm -i datalust/seq config hash)" > .env
```
Replace `'password'` with your desired password.

### 3. Start the Services
Run the following command to start Seq and the syslog input service:
```sh
docker compose up --build -d
```

### 4. Access Seq
Once running, Seq will be accessible at:
```
http://localhost:5341
```

note : default password is "changeme" (configured in docker-compose file)

## Sending Logs from a Proxmox Node to Seq
To forward logs from a **Proxmox (PVE) node** to Seq, follow these steps:

1. Install `rsyslog` if not already installed:
```sh
apt install rsyslog -y
```
2. Add the following line to `/etc/rsyslog.conf` in the machine you want to get logs, replacing `<IP>` with the IP address of your Seq server:
```sh
echo "*.* @<IP>:514" >> /etc/rsyslog.conf
```
3. Restart the syslog service:
```sh
systemctl restart syslog
```
4. Ensure firewall rules allow UDP traffic on port `514`.

Seq should now start receiving logs from the Proxmox node.

## Docker Compose Configuration
The `docker-compose.yml` file includes two services:

```yaml
services:
  seq-input-syslog:
    image: datalust/seq-input-syslog:latest
    depends_on:
      - seq
    ports:
      - "514:514/udp"
    environment:
      - SEQ_ADDRESS=http://seq:5341
      - BASE_URI=http://<YOUR_IP>:5341
      - SEQ_API_CANONICALURI=http://<YOUR_IP>:5341
      - TZ="Europe/Paris"
    restart: unless-stopped
    volumes:
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
  seq:
    image: datalust/seq:latest
    ports:
      - "5341:80"
    environment:
      - ACCEPT_EULA=Y
      - BASE_URI=http://<YOUR_IP>:5341
      - SEQ_API_CANONICALURI=http://<YOUR_IP>:5341
      - TZ="Europe/Paris"
      - SEQ_FIRSTRUN_ADMINPASSWORD=changeme
    restart: unless-stopped
    volumes:
      - ./seq-data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
```

## Adding Email Notifications
Seq provides a clean and user-friendly interface for managing logs. To enable email notifications, install the [Seq App Mail](https://github.com/datalust/seq-app-mail).

### Steps to Enable Mailing:
1. Go to the **Settings** --> **Applications** page in Seq and then, click on "Install from Nugget"
2. Install the **Seq App Mail** plugin.

![image](https://github.com/user-attachments/assets/e3f42826-6729-4152-a9e1-3a3eb08c14cd)

3. Click **Add Instance** to configure a new mail integration.
4. To find the configured instance later, go to **Settings** → **Notifications** and click on the provided hyperlink. (Note: There may be a UI bug preventing it from appearing elsewhere.)

## Log Filtering Examples
You can use Seq queries to filter specific log events.

### Detecting Failed Logins
```seqql
Contains(@Message, 'authentication failure') or Contains(@Message, 'Failed password')
```
- SSH failures will log messages like:
  ```
  pam_unix(sshd:auth)...
  ```
- GUI login failures:
  ```
  pam_unix(proxmox-ve-auth:auth)...
  ```

### Detecting RAM Changes in VMs
```seqql
Contains(@Message, 'update VM') and Contains(@Message, 'memory')
```
Example log entry for changing a VM's RAM to 8192MB:
```
pvedaemon[3234877]: <root@pam> update VM 109: -delete balloon,shares -memory 8192
```

### Detecting VM Lock Events
```seqql
Contains(@Message, 'VM is locked')
```

### Detecting VM Resets
```seqql
Contains(@Message, 'qmreset')
```

## Sources
- [Seq Official Website](https://datalust.co/seq)

## Conclusion
This setup allows you to aggregate logs efficiently using Seq and Docker Compose, specifically tailored for **Proxmox** environments. You can further extend it by integrating additional log sources and notification mechanisms.

---

Feel free to contribute or open issues if you encounter any problems!

## Keywords
Seq, Log Aggregation, Docker Compose, Syslog, Proxmox, VM Monitoring, Logging, Infrastructure Monitoring

