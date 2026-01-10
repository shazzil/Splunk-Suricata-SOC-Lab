# Splunk + Suricata SOC Lab Walkthrough

This repository documents a **step-by-step walkthrough** for integrating **Suricata IDS** with **Splunk SIEM** using the **Splunk Universal Forwarder**. The setup represents a **realistic SOC (Security Operations Center) detection pipeline** suitable for learning, labs, and interviews.

---

## 🧩 Architecture Overview

```
Suricata (IDS)
     ↓
Splunk Universal Forwarder
     ↓
Splunk Server (Indexer + Search Head)
     ↓
SOC Analyst (Detection & Investigation)
```

---

## 🖥️ Environment

- **Operating System:** Linux (Kali / Ubuntu)
- **Deployment Type:** Single machine (lab setup)
- **Privileges:** sudo / root

---

## 🔹 Step 1: Install Splunk Enterprise (Server)

📸 **Screenshot:** Splunk Web Login Page

![Splunk Web Login](.png)

- Install Splunk Enterprise on the system
- Start Splunk service
- Access Splunk Web

```text
http://localhost:8000
```

Verify status:

```bash
/opt/splunk/bin/splunk status
```

---

## 🔹 Step 2: Enable Receiving Port (9997)

📸 **Screenshot:** Receiving Port Enabled

![Enable Listen Port](Receiving Port Enabled.png)

Splunk Server must listen for data from forwarders.

```bash
sudo /opt/splunk/bin/splunk enable listen 9997
sudo /opt/splunk/bin/splunk list listen
```

Expected output:
```
9997
```

---

## 🔹 Step 3: Install Splunk Universal Forwarder

- Install Splunk Universal Forwarder
- Start the forwarder
- Enable boot-start

```bash
sudo /opt/splunkforwarder/bin/splunk start
sudo /opt/splunkforwarder/bin/splunk enable boot-start
```

> ⚠️ Note: Forwarder **does not listen on ports**. It only sends data.

---

## 🔹 Step 4: Connect Forwarder to Splunk Server

📸 **Screenshot:** Forwarder Connected (Active)

![Forwarder Active](screenshots/03-forwarder-active.png)

Since this is a single-host lab, use `127.0.0.1`.

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 127.0.0.1:9997
```

Verify connection:

```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

Expected status:
```
Active
```

---

## 🔹 Step 5: Install Suricata IDS

- Install Suricata
- Ensure the service is running

Suricata log directory:
```
/var/log/suricata/
```

---

## 🔹 Step 6: Verify Suricata Logs

📸 **Screenshot:** Suricata fast.log Alerts

![Suricata Logs](screenshots/04-suricata-fast-log.png)

Confirm Suricata is generating logs:

```bash
ls /var/log/suricata/
```

Expected files:
- `eve.json`
- `fast.log`

Live check:

```bash
tail -f /var/log/suricata/fast.log
```

---

## 🔹 Step 7: Create Splunk Index

Create a dedicated index for Suricata logs.

**Splunk Web → Settings → Indexes → New Index**

- **Index Name:** `suricata`

---

## 🔹 Step 8: Configure inputs.conf (Best Practice)

📸 **Screenshot:** inputs.conf Configuration

![inputs.conf](screenshots/05-inputs-conf.png)

Edit the forwarder inputs configuration:

```bash
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

Add the following:

```inibash
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

Add the following:

```ini
[monitor:///var/log/suricata/eve.json]
disabled = false
index = suricata
sourcetype = suricata:eve:json

[monitor:///var/log/suricata/fast.log]
disabled = false
index = suricata
sourcetype = suricata:fast
```

Save and exit.

---

## 🔹 Step 9: Restart Splunk Forwarder

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

Verify monitored inputs:

```bash
sudo /opt/splunkforwarder/bin/splunk list monitor
```

---

## 🔹 Step 10: Verify Data in Splunk

📸 **Screenshot:** Suricata Alerts in Splunk Search

![Suricata Alerts](screenshots/07-suricata-alerts.png)

Open Splunk Web and run the following searches:

```splspl
index=suricata
```

Only alerts (JSON):

```spl
index=suricata event_type=alert
```

fast.log alerts:

```spl
index=suricata sourcetype=suricata:fast
```

---

## 🔹 Step 11: Validation Testing

Generate test traffic to trigger Suricata alerts:

```bash
nmap -sS localhost
```

Check Splunk:

```spl
index=suricata Nmap
```

---

## 🔍 Useful SOC Filters

- **High severity alerts**
```spl
index=suricata alert.severity<=2
```

- **Port scan detection**
```spl
index=suricata Nmap
```

- **fast.log quick triage**
```spl
index=suricata sourcetype=suricata:fast
```

---

## ✅ Best Practices

- Use `eve.json` for structured analysis and alerts
- Use `fast.log` for quick alert review
- Manage inputs using `inputs.conf`
- Avoid forwarding Splunk internal logs
- Use separate indexes for each log source

---

## 🎯 Outcome

By completing this walkthrough, you gain hands-on experience with:

- SIEM integration
- IDS log ingestion
- SOC detection pipelines
- Real-world Splunk configuration

This setup is **portfolio-ready** and **SOC interview friendly**.

---

## 📌 One-Line Summary (Interview Ready)

> *Built a SOC lab integrating Suricata IDS with Splunk SIEM using a universal forwarder, enabling real-time detection and analysis of network threats.*

