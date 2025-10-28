# 🧩 DevOps Intern Assignment --- *Adhi S S*

## **Part 1: Environment Setup**

### **A. Launch an Ubuntu EC2 Instance**

1.  Go to [AWS Console](https://console.aws.amazon.com) → **EC2** →
    **Launch Instance**

2.  Choose:

    -   **AMI:** Ubuntu Server 22.04 LTS (Free Tier)
    -   **Instance Type:** `t2.micro`
    -   **Key Pair:** `adhi.pem`
    -   **Allow inbound rules:** SSH (22), HTTP (80)

3.  Connect:

    ``` bash
    cd ~/Downloads
    chmod 400 adhi.pem
    ssh -i "adhi.pem" ubuntu@<public-ip>
    ```

### **B. Create a new user**

``` bash
sudo adduser devops_intern
sudo usermod -aG sudo devops_intern
```

### **C. Change hostname**

``` bash
sudo hostnamectl set-hostname adhi-devops
```

------------------------------------------------------------------------

## **Part 2: Simple Web Service Setup**

### **A. Install Nginx**

``` bash
sudo apt update -y
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

### **B. Create Web Page**

``` bash
sudo tee /var/www/html/index.html > /dev/null <<'HTML'
<!DOCTYPE html>
<html>
  <head>
    <title>DevOps Intern - Adhi</title>
  </head>
  <body style="font-family: Arial; text-align: center; margin-top: 100px;">
    <h1>Hello, I'm <span style="color: blue;">Adhi</span></h1>
    <p><b>Instance ID:</b> $(curl -s http://169.254.169.254/latest/meta-data/instance-id)</p>
    <p><b>Server Uptime:</b> $(uptime -p)</p>
  </body>
</html>
HTML
sudo systemctl restart nginx
```

------------------------------------------------------------------------

## **Part 3: Monitoring Script**

Create and schedule a system report script.

### **A. Script**

``` bash
sudo tee /usr/local/bin/system_report.sh > /dev/null <<'BASH'
#!/bin/bash
LOGFILE="/var/log/system_report.log"

{
  echo "---------------------------------------------"
  echo "Report Time: $(date '+%Y-%m-%d %H:%M:%S %Z')"
  echo "Uptime: $(uptime -p)"
  echo "CPU Usage (%): $(top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}')"
  echo "Memory Usage (%): $(free | awk '/Mem:/ {printf("%.2f"), $3/$2 * 100}')"
  echo "Disk Usage (%): $(df -h / | awk 'NR==2{print $5}')"
  echo "Top 3 Processes by CPU:"
  ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head -n 4
  echo ""
} >> "$LOGFILE"
BASH

sudo chmod +x /usr/local/bin/system_report.sh
sudo touch /var/log/system_report.log
sudo chmod 644 /var/log/system_report.log
```

### **B. Schedule with cron**

``` bash
sudo tee /etc/cron.d/system_report > /dev/null <<'CRON'
*/5 * * * * root /usr/local/bin/system_report.sh
CRON
```

------------------------------------------------------------------------

## **Part 4: AWS Integration**

### **A. Configure AWS CLI**

``` bash
aws configure
# Enter Access Key ID, Secret Access Key, Region (us-east-1), Output (json)
```

### **B. Create CloudWatch Log Group**

``` bash
aws logs create-log-group --log-group-name /devops/intern-metrics
```

### **C. Push Logs to CloudWatch**

``` bash
LOG_STREAM="system-report-$(date +%s)"
aws logs create-log-stream --log-group-name /devops/intern-metrics --log-stream-name "$LOG_STREAM"

TIMESTAMP=$(date +%s%3N)
LOG_CONTENT=$(sudo tail -n 30 /var/log/system_report.log | sed ':a;N;$!ba;s/\n/\\n/g' | sed 's/"/\\\"/g')

aws logs put-log-events   --log-group-name /devops/intern-metrics   --log-stream-name "$LOG_STREAM"   --log-events "[{"timestamp":$TIMESTAMP,"message":"$LOG_CONTENT"}]"
```

✅ Check on AWS Console → CloudWatch → Logs → `/devops/intern-metrics`.

------------------------------------------------------------------------

## ✅ **Completed Tasks**

-   [x] EC2 setup & SSH connection
-   [x] User creation & hostname setup
-   [x] Nginx web page with metadata
-   [x] Monitoring script with cron
-   [x] CloudWatch integration

------------------------------------------------------------------------

**Author:** Adhi S S\
**Role:** DevOps Intern\
**Tools Used:** AWS, Linux, Bash, CloudWatch, Nginx
