# AWS EC2 Monitoring Solution Using CloudWatch and SNS

## Project Overview

This project implements an **AWS monitoring and alerting solution for an EC2 web server** using:

- Amazon EC2
- Amazon CloudWatch
- Amazon SNS (Simple Notification Service)
- Email notifications

The solution monitors both **CPU utilization** and **EC2 instance status**. Two CloudWatch alarms are configured and connected to an SNS topic so that notifications can be generated when the defined thresholds are breached.

---

## Architecture

```text
                         AWS Cloud
                            |
                            |
                    +-------v-------+
                    |   EC2 Web     |
                    |    Server     |
                    +-------+-------+
                            |
             CloudWatch Metrics
                /           \
               /             \
              v               v
     +----------------+  +----------------------+
     | CPUUtilization |  | StatusCheckFailed    |
     +-------+--------+  +----------+-----------+
             |                       |
             v                       v
     +-----------------------------------------+
     |          CloudWatch Alarms              |
     |                                         |
     |  EC2-High-CPU                           |
     |  EC2-Status-Check-Failed               |
     +-------------------+---------------------+
                         |
                         | Alarm Action
                         v
              +----------------------+
              |   Amazon SNS Topic   |
              | ec2-monitoring-alerts|
              +----------+-----------+
                         |
                         | Email
                         v
                 +---------------+
                 | Email Inbox   |
                 | Notification  |
                 +---------------+
```

---

## Project Objective

The objective is to:

1. Monitor an EC2 web server using CloudWatch.
2. Monitor CPU utilization.
3. Monitor EC2 instance status.
4. Create at least two CloudWatch alarms.
5. Create an SNS topic for notifications.
6. Subscribe an email address to the SNS topic.
7. Connect CloudWatch alarms to SNS.
8. Test the monitoring configuration.
9. Document the alarm state and notification workflow.

---

# 1. EC2 Web Server

An EC2 instance is used as the web server being monitored.

Example configuration:

```text
Service:        Amazon EC2
Instance Type:  t3.micro
Role:           Web Server
Monitoring:     Amazon CloudWatch
```

The EC2 instance provides the metrics that CloudWatch uses for monitoring.

### Screenshot

![EC2 Instance](screenshots/03-ec2-instance.jpeg)

---

# 2. Create an SNS Topic

An SNS topic named:

```text
ec2-monitoring-alerts
```

was created.

The SNS topic acts as the notification channel between CloudWatch alarms and the email recipient.

### SNS Topic Configuration

```text
Topic Name: ec2-monitoring-alerts
Type:       Standard
```

### Screenshot

![SNS Topic Created](screenshots/01-sns-topic-created.jpeg)

---

# 3. Create SNS Email Subscription

An email subscription was created for the SNS topic.

The subscription uses:

```text
Protocol: EMAIL
Endpoint: <configured email address>
Status:   Confirmed
```

The email recipient must confirm the subscription before SNS can deliver notifications.

### Screenshot

![SNS Email Subscription](screenshots/02-sns-email-subscription-confirmed.jpeg)

> **Security note:** Do not publish your personal email address in a public GitHub README. Replace it with `<your-email>` if it appears in a screenshot.

---

# 4. CloudWatch CPU Monitoring

CloudWatch monitors the EC2 metric:

```text
CPUUtilization
```

A CloudWatch alarm named:

```text
EC2-High-CPU
```

was created.

### Alarm Condition

```text
CPUUtilization > 70%
for 1 datapoint within 5 minutes
```

When CPU utilization crosses the threshold, the alarm can change to the **ALARM** state and publish a notification to SNS.

### Screenshot

![High CPU Alarm](screenshots/04-cloudwatch-high-cpu-alarm.jpeg)

---

# 5. CloudWatch Instance Status Monitoring

A second CloudWatch alarm was created to monitor EC2 status checks.

Alarm name:

```text
EC2-Status-Check-Failed
```

Metric:

```text
StatusCheckFailed
```

Condition:

```text
StatusCheckFailed > 0
for 1 datapoint
```

This alarm helps detect an EC2 instance status-check failure.

### Screenshot

![CloudWatch Alarms](screenshots/05-cloudwatch-alarms.jpeg)

---

# 6. CloudWatch Alarms

The monitoring solution contains two alarms:

| Alarm | Metric | Condition | Purpose |
|---|---|---|---|
| EC2-High-CPU | CPUUtilization | > 70% | Detect high CPU usage |
| EC2-Status-Check-Failed | StatusCheckFailed | > 0 | Detect EC2 status-check failure |

Both alarms have actions enabled and are configured to use the SNS notification workflow.

### Screenshot

![CloudWatch Alarms](screenshots/05-cloudwatch-alarms.jpeg)

---

# 7. SNS Notification Workflow

The complete workflow is:

```text
EC2
 |
 | CloudWatch metrics
 v
CloudWatch
 |
 | Alarm threshold breached
 v
CloudWatch Alarm
 |
 | Publish notification
 v
SNS Topic
 |
 | Email subscription
 v
Email Notification
```

This provides an automated alerting mechanism for the EC2 server.

---

# 8. Testing the CPU Alarm

To test the CPU alarm, CPU load can be generated on the EC2 instance.

For Amazon Linux:

```bash
sudo dnf install stress -y
```

Then generate CPU load:

```bash
stress --cpu 2 --timeout 300
```

If the `stress` package is unavailable, the equivalent package/command can be installed using the package manager available on the EC2 operating system.

Monitor the alarm from:

```text
AWS Console
→ CloudWatch
→ Alarms
→ EC2-High-CPU
```

The alarm should move from:

```text
OK
```

to:

```text
ALARM
```

if CPU utilization remains above the configured threshold long enough.

---

# 9. Alarm History

CloudWatch alarm history can be used to document state changes.

Example workflow:

```text
Insufficient data
       |
       v
      OK
```

If the CPU threshold is exceeded:

```text
OK
 |
 v
ALARM
```

After CPU utilization returns below the threshold:

```text
ALARM
 |
 v
OK
```

### Screenshot

![Alarm History](screenshots/06-high-cpu-alarm-history.jpeg)

---

# 10. Alarm Recovery

After the CPU load stops, CPU utilization should return to normal.

The alarm can then return to:

```text
OK
```

This demonstrates both the alert condition and recovery condition.

### Screenshot

![Alarm OK State](screenshots/07-high-cpu-alarm-ok-state.jpeg)

---

# 11. Instance Status Alarm

The second alarm monitors:

```text
StatusCheckFailed
```

Normal state:

```text
StatusCheckFailed = 0
```

Alarm condition:

```text
StatusCheckFailed > 0
```

This provides monitoring for EC2 instance status-check failures.

> **Important:** Do not intentionally terminate a production instance just to test this alarm. For a demonstration, document the configured alarm and use a safe test environment.

---

# 12. Notification Testing

The SNS notification system can be tested by:

1. Creating the SNS topic.
2. Creating an email subscription.
3. Confirming the subscription.
4. Connecting the SNS topic to the CloudWatch alarms.
5. Triggering the CPU threshold in a test environment.
6. Checking the CloudWatch alarm state.
7. Checking the email inbox for the SNS notification.
8. Stopping the CPU load.
9. Confirming that the alarm returns to `OK`.

Expected flow:

```text
CPU > 70%
     |
     v
CloudWatch Alarm = ALARM
     |
     v
SNS Topic
     |
     v
Email Notification
```

---

# 13. Security Considerations

The project follows these basic security practices:

- Use IAM permissions with least privilege.
- Restrict EC2 SSH access to the administrator's IP where possible.
- Do not expose AWS access keys in the repository.
- Do not commit passwords or secrets.
- Do not publish personal email addresses unnecessarily.
- Use SNS for controlled notification delivery.
- Monitor alarm state and history through CloudWatch.

---

# 14. AWS Services Used

| AWS Service | Purpose |
|---|---|
| Amazon EC2 | Web server being monitored |
| Amazon CloudWatch | Metrics and alarm monitoring |
| Amazon SNS | Notification delivery |
| Email | Alert recipient |

---

# 15. Final Result

The monitoring solution successfully demonstrates:

- ✅ EC2 web server monitoring
- ✅ CPU utilization monitoring
- ✅ EC2 status monitoring
- ✅ CloudWatch CPU alarm
- ✅ CloudWatch instance status alarm
- ✅ SNS topic
- ✅ Confirmed email subscription
- ✅ CloudWatch-to-SNS notification workflow
- ✅ Alarm state monitoring
- ✅ Alarm history documentation
- ✅ Alarm recovery to OK state

---

# 16. Interview Explanation

> I created an AWS monitoring solution for an EC2 web server using Amazon CloudWatch and SNS. I configured CloudWatch to monitor CPU utilization and the EC2 instance status. I created two alarms: one for high CPU utilization above 70 percent and another for EC2 status-check failures. I created an SNS topic named `ec2-monitoring-alerts` and subscribed an email address to it. The CloudWatch alarms use SNS as their notification action. I tested the CPU alarm by generating CPU load and verified the alarm state and history. When the CPU returned to normal, the alarm recovered to the OK state.

---

# 17. Project Structure

Recommended GitHub repository structure:

```text
aws-ec2-cloudwatch-sns-monitoring/
│
├── README.md
│
└── screenshots/
    ├── 01-sns-topic-created.jpeg
    ├── 02-sns-email-subscription-confirmed.jpeg
    ├── 03-ec2-instance.jpeg
    ├── 04-cloudwatch-high-cpu-alarm.jpeg
    ├── 05-cloudwatch-alarms.jpeg
    ├── 06-high-cpu-alarm-history.jpeg
    └── 07-high-cpu-alarm-ok-state.jpeg
```

---

# Conclusion

This project demonstrates a basic but practical AWS monitoring architecture. Amazon CloudWatch collects EC2 metrics and evaluates alarm conditions, while Amazon SNS provides email-based notifications. Together, they provide automated monitoring and alerting for an EC2 web server.
