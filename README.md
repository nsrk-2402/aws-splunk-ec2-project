# aws-splunk-ec2-project
> Splunk log monitoring on AWS EC2 with Universal Forwarder and Indexer setup.


## 📌 Project Overview

### In this project, we set up a two-EC2 Splunk environment on AWS:
  
- Splunk Enterprise (Indexer + Search Head) on the first EC2 instance.
  
- Splunk Universal Forwarder on the second EC2 instance, sending syslogs to the first EC2.

This simulates a real-world log aggregation setup, where logs from multiple servers are centralized for monitoring and analysis.

            ┌─────────────────────────────┐
            │ EC2-1: Splunk Enterprise    │
            │ (Indexer + Search Head)     │
            │ Ubuntu, 30GB EBS, 2vCPU, 4GB│
            │ Listens on :8000 (UI)       │
            │ Listens on :9997 (Indexer)  │
            └─────────────┬───────────────┘
                          │
             Syslogs via Splunk Forwarder
                          │
            ┌─────────────▼───────────────┐
            │ EC2-2: Splunk Forwarder     │
            │ Ubuntu, 30GB EBS, 2vCPU, 4GB│
            │ Sends logs → EC2-1:9997     │
            └─────────────────────────────┘


You can access the full video demonstration of this project and follow along with the setup.
[![Project Demo Video](Thumbnail_URL)](https://youtu.be/g6iyxSa9dHM)


## ⚙️ AWS Setup

### 1. Launch EC2 Instances

- Region: us-east-1 (or your choice)

- AMI: Ubuntu 22.04 LTS

- Instance type: t3.medium (2 vCPUs, 4 GB RAM) or anything with 2 vCPUs and 4 GB RAM

- Storage: 30 GB EBS

- Key Pair: Create or use existing

#### Security Groups:

- EC2-1 (Splunk Enterprise): Allow inbound 22, 8000, 9997

- EC2-2 (Forwarder): Allow inbound 22

### 2. Connect to Instances
*Use AWS Cloud Shell to connect to instances via SSH or simply use the connect button after selecting an instance.*
```bash
ssh -i my-key.pem ubuntu@<EC2_PUBLIC_IP>
```

## 🔧 Splunk Enterprise Installation (EC2-1)

### 1. Download Splunk Enterprise

 - After connecting to the instance, obtain root user privileges.
```bash
sudo su
```

 - Navigate to opt directory
```bash
cd /opt/
```

- Copy the Wget link from splunk.com for the .deb package of splunk enterprise for linux.
```bash
wget -O splunk-10.0.0-e8eb0c4654f8-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/10.0.0/linux/splunk-10.0.0-e8eb0c4654f8-linux-amd64.deb"
```

### 2. Install the downloaded package
```bash
sudo dpkg -i splunk-9.2.1-linux-amd64.deb
```

### 3. Start Splunk and accept license:
```bash
sudo /opt/splunk/bin/splunk start --accept-license --answer-yes
```

### 4. Set Username and password
> Go with the credentials you like.

### 5. Access Splunk Home
> URL: http://EC2-1-Public-IP:8000

> Default login: Use the credentials set just now.

### 6. Enable receiving on port 9997 or a port of your choice
 - In Splunk Web → Settings > Forwarding and Receiving > Configure Receiving.
 - Add port 9997 or one of your choice.

### 7. Create a new index on splunk
 - In Splunk Web → Settings > Indexes > Create New Index.
 - Create a new index with a name of your choice.

## 🔧 Splunk Universal Forwarder Installation (EC2-2)

### 1. Download Splunk Universal Forwarder:

- After connecting to the instance, obtain root user privileges.
```bash
sudo su
```

 - Navigate to opt directory
```bash
cd /opt/
```

- Copy the Wget link from splunk.com for the .deb package of splunk universal forwarder for linux 64 bit.
```bash
wget -O splunkforwarder-10.0.0-e8eb0c4654f8-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/10.0.0/linux/splunkforwarder-10.0.0-e8eb0c4654f8-linux-amd64.deb"
```

### 2. Install the downloaded package:
```bash
sudo dpkg -i splunkforwarder-9.2.1-linux-amd64.deb
```

### 3. Start Forwarder and accept license:
```bash
sudo /opt/splunkforwarder/bin/splunk start --accept-license --answer-yes
```

### 4. Configure Forwarder to send logs to Splunk Enterprise:
> Navigate to search directory
```bash
cd /opt/splunkforwarder/etc/apps/search/
```

> Create a folder called local in search directory and switch to it
```bash
cd local
```

> Inside local directory, create a file called inputs.conf:
```bash
[monitor://</path/of/file/abc.txt>]
Disabled = 0
Index = <index_name>
```
 - replace the file path and the index name with your particular fields.

> Inside local directory, create a file called outputs.conf:
```bash
[tcpout]
defaultGroup=my_indexers
[tcpout:my_indexers]
server=mysplunk_indexer1:9997
[tcpout-server://mysplunk_indexer1:9997
```
- replace "mysplunk_indexer1" with the public IP of EC2-Instance 1 that is running splunk enterprise.

> Restart Splunk forwarder to start forwarding logs to Splunk for analyzing and monitoring.


## ✅ Validation

### 1. Log in to Splunk Enterprise
 - Web UI (http://EC2-1-Public-IP:8000).

### 2. Go to Search & Reporting App.

### 3. Run a query:
```bash
index= "<index_name>" (the one we configured in inputs.conf file in forwarder)
```

### 4. Observe Logs being forwarded from the instance running splunk forwarder.

### 5. To add more servers/Instances to monitor, Install the Universal forwarder on those instances and configure inputs.conf and outputs.conf files.

## Screenshots

### 1. Splunk Home:
<img width="2558" height="1352" alt="image" src="https://github.com/user-attachments/assets/a2a9af48-8d6b-4d7f-8e30-4843c38b527e" />

### 2. Receiving settings (port 9997 enabled)
<img width="2558" height="1349" alt="image" src="https://github.com/user-attachments/assets/45bb62b0-1960-4d7c-82b6-9fff3954dbf3" />

### 3. Search results showing EC2-2 logs
<img width="2558" height="1352" alt="image" src="https://github.com/user-attachments/assets/5dd104f0-6636-4301-932d-c701618403ac" />

## 🎯 Learnings
 - Provisioning and securing EC2 instances with custom EBS sizes.

 - Installing Splunk Enterprise and Forwarder.

 - Configuring log forwarding between multiple AWS instances.

 - Using Splunk Web UI to query centralized logs.

 - Basics of log aggregation pipelines.

## 🚀 Potential Next Steps
 - Add more forwarders (simulate multiple servers).

 - Explore Splunk alerts (CPU usage, SSH logins).

 - Secure communication with SSL certificates.

 - Compare with AWS CloudWatch Logs in another project.
