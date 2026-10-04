# Two-Tier AWS Application Using EC2 and Amazon RDS

## Project Overview
This project demonstrates a secure **two-tier AWS application architecture** using Amazon EC2 as the web/application server and Amazon RDS for MySQL as the database tier.

The database is placed in private subnets and accepts MySQL traffic only from the EC2 Security Group.

## Architecture

```text
                         INTERNET
                            |
                            | HTTP :80
                            v
                 +----------------------+
                 |   Internet Gateway   |
                 +----------+-----------+
                            |
                            v
        +------------------------------------------------+
        |                    VPC                          |
        |                10.0.0.0/16                     |
        |                                                |
        |  PUBLIC SUBNET: 10.0.1.0/24                    |
        |  +------------------------------------------+  |
        |  | EC2 Web / Application Server             |  |
        |  | Security Group: ec2-web-sg               |  |
        |  | SSH 22 -> My IP                          |  |
        |  | HTTP 80 -> Internet                      |  |
        |  +-------------------+----------------------+  |
        |                      |                         |
        |                      | MySQL TCP 3306         |
        |                      v                         |
        |  PRIVATE SUBNETS: 10.0.2.0/24, 10.0.3.0/24    |
        |  +------------------------------------------+  |
        |  | Amazon RDS MySQL                         |  |
        |  | Security Group: rds-db-sg                |  |
        |  | MySQL 3306 <- ec2-web-sg ONLY            |  |
        |  | Public access: NO                        |  |
        |  +------------------------------------------+  |
        +------------------------------------------------+

Internet -> EC2 :80       ALLOWED
Your IP -> EC2 :22        ALLOWED
Internet -> RDS :3306     BLOCKED
EC2 -> RDS :3306           ALLOWED

```

## AWS Resources

| Resource | Configuration |
|---|---|
| VPC | `two-tier-vpc` |
| VPC CIDR | `10.0.0.0/16` |
| Public subnet | `10.0.1.0/24` |
| Private subnet 1 | `10.0.2.0/24` |
| Private subnet 2 | `10.0.3.0/24` |
| EC2 | Amazon Linux, `t3.micro` |
| RDS | MySQL, `db.t3.micro` |
| EC2 Security Group | `ec2-web-sg` |
| RDS Security Group | `rds-db-sg` |
| RDS subnet group | `two-tier-rds-subnet-group` |
| RDS public access | No |
| Database port | `3306` |

## 1. VPC and Network Design
Created a custom VPC named `two-tier-vpc` with CIDR `10.0.0.0/16`.

Subnets:
- Public: `10.0.1.0/24` in `us-east-1a`
- Private 1: `10.0.2.0/24` in `us-east-1b`
- Private 2: `10.0.3.0/24` in `us-east-1c`

![VPC resource map](screenshots/step-1.png)

## 2. Route Tables and Internet Gateway
Created `two-tier-igw` and attached it to the VPC.

The public route table contains `0.0.0.0/0` to the Internet Gateway and is associated only with the public subnet.

The private subnets use a separate private route table with no internet route.

![Route table design](screenshots/step-2.png)

## 3. EC2 Web/Application Server
Created:

```text
Name:           two-tier-web-server
AMI:            Amazon Linux
Instance type:  t3.micro
Subnet:         public-subnet
Security Group: ec2-web-sg
```

![EC2 instance](screenshots/step-3.png)

## 4. EC2 Security Group

`ec2-web-sg` inbound rules:

| Type | Port | Source |
|---|---:|---|
| SSH | 22 | My IP |
| HTTP | 80 | `0.0.0.0/0` |

This allows administration through SSH while making the web server reachable over HTTP.

## 5. RDS MySQL Database
Created:

```text
DB identifier: two-tier-mysql
Engine:        MySQL
Instance:      db.t3.micro
Subnet group:  two-tier-rds-subnet-group
Port:          3306
Public access: No
```

The database is deployed in private subnets.

![RDS database creation](screenshots/step-4.png)

![RDS available](screenshots/step-5.png)

## 6. RDS Security Group

`rds-db-sg` allows:

```text
MySQL/Aurora
Port: 3306
Source: ec2-web-sg
```

The source is the EC2 Security Group, not `0.0.0.0/0`.

Therefore:

```text
Internet -> RDS:3306  BLOCKED
EC2 -> RDS:3306       ALLOWED
```

![RDS connectivity and security](screenshots/step-6.png)

## 7. Secure EC2-to-RDS Connectivity

On EC2, the MariaDB/MySQL-compatible client was installed:

```bash
sudo dnf update -y
sudo dnf install mariadb105 -y
```

The RDS endpoint was then used:

```bash
mysql -h <RDS-ENDPOINT> -P 3306 -u admin -p
```

Successful connection:

```text
Welcome to the MariaDB monitor.
MySQL [(none)]>
```

This demonstrates secure private database connectivity.

## 8. Database Validation

```sql
CREATE DATABASE companydb;
USE companydb;

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50)
);

INSERT INTO employees VALUES
(1, 'Pranav', 'IT'),
(2, 'Rahul', 'HR'),
(3, 'Amit', 'Finance');

SELECT * FROM employees;
```

## 9. Apache Web Server

Install Apache:

```bash
sudo dnf install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

Create the page:

```bash
sudo nano /var/www/html/index.html
```

Example:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Two Tier AWS Application</title>
</head>
<body>
    <h1>Two-Tier AWS Application</h1>
    <h2>Web Server: EC2</h2>
    <p>Application Server: Amazon EC2</p>
    <p>Database: Amazon RDS MySQL</p>
    <p>Status: EC2 Web Server is Running</p>
</body>
</html>
```

Open:

```text
http://<EC2-PUBLIC-IP>
```

## 10. Security Demonstration

```text
Internet -> EC2 :80       ALLOWED
Your IP -> EC2 :22        ALLOWED
EC2 -> RDS :3306           ALLOWED
Internet -> RDS :3306     BLOCKED
```

This satisfies the requirement that database access is allowed only from the application server.

## 11. Final Architecture

```text
                         INTERNET
                            |
                            | HTTP :80
                            v
                 +----------------------+
                 |   Internet Gateway   |
                 +----------+-----------+
                            |
                            v
        +------------------------------------------------+
        |                    VPC                          |
        |                10.0.0.0/16                     |
        |                                                |
        |  PUBLIC SUBNET: 10.0.1.0/24                    |
        |       +----------------------------+           |
        |       | EC2 Web/App Server         |           |
        |       | ec2-web-sg                 |           |
        |       +-------------+--------------+           |
        |                     | TCP 3306                 |
        |                     v                          |
        |  PRIVATE SUBNETS: 10.0.2.0/24 + 10.0.3.0/24   |
        |       +----------------------------+           |
        |       | Amazon RDS MySQL           |           |
        |       | rds-db-sg                  |           |
        |       | 3306 <- ec2-web-sg only    |           |
        |       | Public access: NO          |           |
        |       +----------------------------+           |
        +------------------------------------------------+
```

## Technologies Used
- Amazon VPC
- Amazon EC2
- Amazon RDS
- MySQL
- Amazon Linux
- Apache HTTP Server
- Security Groups
- Internet Gateway
- Route Tables
- EC2 Instance Connect

## Project Result
The project successfully demonstrates:
- Custom VPC
- Public/private subnet architecture
- EC2 web/application server
- Private RDS MySQL database
- Separate Security Groups
- Secure EC2-to-RDS connectivity
- MySQL database and table creation
- Apache web server
- Working web page

## Interview Explanation
> I created a two-tier AWS application using Amazon EC2 and Amazon RDS. I created a custom VPC with one public subnet for the EC2 web server and two private subnets for the RDS database. I configured separate Security Groups. The EC2 Security Group allows HTTP from the internet and SSH from my IP, while the RDS Security Group allows MySQL port 3306 only from the EC2 Security Group. RDS public access is disabled, so the database cannot be accessed directly from the internet. I connected from EC2 to RDS and verified the database by creating a database, table, and sample records.

## Conclusion
This project demonstrates a secure separation between the public application tier and private database tier using AWS VPC networking and Security Groups.
