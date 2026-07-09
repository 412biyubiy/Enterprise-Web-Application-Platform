Menurut gua, cara berpikir seorang **Solution Architect** bukan berdasarkan urutan service AWS ("hari ini bikin VPC dulu"), tapi berdasarkan **alur request dari user** dan **dependency antar komponen**. Jadi kalau project ini dijadikan portofolio, urutannya sebaiknya mengikuti bagaimana arsitektur dibangun hingga aplikasi bisa diakses.

---

# Phase 1 — Networking Foundation

---

# Step 1. Membuat VPC

## Service

Amazon VPC

## Kenapa?

Sebelum punya server, kita harus punya "tanah" dulu.

VPC adalah jaringan privat milik kita di AWS.

Tanpa VPC, semua resource tidak punya tempat untuk hidup.

Misalnya

```text
AWS Region

┌────────────────────┐
│      VPC           │
│                    │
│ nanti semua server │
│ database dll ada   │
└────────────────────┘
```

Biasanya langsung menentukan

```
CIDR

10.0.0.0/16
```

Karena nanti akan dibagi menjadi banyak subnet.

Yang dipelajari

* CIDR
* Private IP
* IPv4
* Region
* Logical Network

---

# Step 2. Membuat Subnet

## Service

Subnet

Kenapa?

Supaya resource dipisahkan berdasarkan fungsi.

Contoh

```text
VPC

├── Public Subnet A
├── Public Subnet B

├── Private App A
├── Private App B

├── Private DB A
└── Private DB B
```

Public

↓

Boleh internet

Private

↓

Tidak boleh internet

Database

↓

Lebih private lagi

Yang dipelajari

* Availability Zone
* High Availability
* Public vs Private

---

# Step 3. Internet Gateway

## Service

IGW

Kenapa?

VPC secara default tidak punya akses internet.

IGW adalah "gerbang keluar masuk".

```text
Internet

↓

Internet Gateway

↓

VPC
```

Tanpa IGW

ALB tidak bisa diakses.

---

# Step 4. NAT Gateway

Kenapa?

EC2 berada di Private Subnet.

Tetapi

EC2 tetap perlu

* yum update
* apt update
* install package
* download Docker
* akses GitHub

Maka dibuat

```text
Private EC2

↓

NAT Gateway

↓

Internet
```

Yang penting dipahami

NAT Gateway hanya untuk **outbound**.

Internet tidak bisa masuk ke EC2 melalui NAT.

---

# Step 5. Route Table

Kenapa?

Subnet belum tahu harus lewat mana.

Route Table adalah "GPS".

Misalnya

Public

```
0.0.0.0/0

↓

Internet Gateway
```

Private

```
0.0.0.0/0

↓

NAT Gateway
```

Yang dipelajari

* Longest Prefix Match
* Local Route
* Default Route

---

# Phase 2 — Security

---

# Step 6. Security Group

Kenapa?

Firewall setiap resource.

Misalnya

ALB

```
443

Internet
```

EC2

```
80

dari SG-ALB
```

RDS

```
5432

dari SG-EC2
```

Yang dipelajari

Stateful Firewall

Least Privilege

---

# Step 7. IAM

Kenapa?

EC2 tidak boleh menyimpan Access Key.

Lebih aman memakai

IAM Role

Misalnya

```
EC2

↓

IAM Role

↓

Boleh

S3

CloudWatch

Secrets Manager
```

Yang dipelajari

* Policy
* Role
* Trust Policy
* Least Privilege

---

# Phase 3 — Compute

---

# Step 8. Launch Template

Kenapa?

Supaya semua EC2 identik.

Isinya

```
AMI

Instance Type

Security Group

IAM Role

User Data
```

Kalau ASG membuat server baru,

servernya selalu sama.

---

# Step 9. Auto Scaling Group

Kenapa?

Server tidak selalu cukup.

Misalnya

```
Jam 2 pagi

200 user

↓

2 EC2
```

Jam 9 pagi

```
20.000 user

↓

CPU

90%
```

ASG

↓

buat EC2 baru.

Yang dipelajari

* Min
* Desired
* Max
* Scaling Policy

---

# Step 10. EC2

Di sinilah aplikasi dijalankan.

Misalnya

```
NodeJS

Spring Boot

Python

.NET
```

EC2 hanya menjalankan application.

---

# Phase 4 — Traffic Management

---

# Step 11. Target Group

Kenapa?

ALB tidak tahu server mana yang sehat.

Target Group melakukan

* Register Instance
* Health Check
* Remove unhealthy server

Misalnya

```
EC2-1

Healthy

✓

EC2-2

Healthy

✓

EC2-3

Broken

✗

↓

Tidak dikirim traffic
```

---

# Step 12. Application Load Balancer

ALB menerima request.

Misalnya

```
1000 request
```

dibagi menjadi

```
EC2-1

333

EC2-2

333

EC2-3

334
```

Yang dipelajari

* Layer 7
* HTTP
* HTTPS
* Host-based Routing
* Path-based Routing

---

# Phase 5 — Database

---

# Step 13. Amazon RDS

Aplikasi butuh data.

Misalnya

* User
* Order
* Product

Semua disimpan di RDS.

Yang dipelajari

* Managed Database
* Backup
* Snapshot
* Multi-AZ

---

# Phase 6 — Storage

---

# Step 14. Amazon S3

Kenapa?

Database bukan tempat file.

Misalnya

```
foto

pdf

invoice

video
```

semua disimpan di S3.

Yang dipelajari

* Object Storage
* Bucket
* Lifecycle
* Versioning

---

# Phase 7 — Domain

---

# Step 15. Route53

Kenapa?

User tidak mau membuka

```
https://18.142.xxx.xxx
```

Lebih enak

```
https://company.com
```

Route53 mengubah

```
company.com

↓

ALB
```

Yang dipelajari

* DNS
* A Record
* Alias Record
* Hosted Zone

---

# Step 16. ACM

Kenapa?

Supaya

```
http

↓

https
```

Ada gembok hijau.

ACM mengelola

SSL/TLS Certificate.

---

# Phase 8 — Secrets

---

# Step 17. Secrets Manager

Jangan

```python
password="admin123"
```

di source code.

Lebih aman

```
EC2

↓

Secrets Manager

↓

Database Password
```

---

# Phase 9 — Monitoring

---

# Step 18. CloudWatch

CloudWatch mengumpulkan

* CPU
* RAM (dengan CloudWatch Agent)
* Disk
* Log aplikasi
* Error Log
* Request Count
* Latency
* Alarm

Kalau CPU

>

80%

↓

CloudWatch Alarm

↓

SNS

↓

Email Admin

---

# Step 19. Systems Manager (SSM)

Tanpa SSM

```
SSH

↓

22

↓

Public IP
```

Dengan SSM

```
Browser

↓

AWS Console

↓

SSM

↓

EC2
```

Tidak perlu membuka port 22, sehingga lebih aman dan sesuai praktik modern.

---

# Alur request dari user (end-to-end)

Setelah semua infrastruktur siap, alur request menjadi seperti ini:

```text
1. User membuka https://company.com
            │
            ▼
2. Route 53 menerjemahkan domain ke ALB
            │
            ▼
3. ACM menyediakan sertifikat TLS sehingga koneksi HTTPS aman
            │
            ▼
4. Application Load Balancer menerima request
            │
            ▼
5. Target Group memilih EC2 yang sehat
            │
            ▼
6. EC2 menjalankan aplikasi
        ├────────► Ambil password database dari Secrets Manager
        ├────────► Baca/tulis data ke Amazon RDS
        ├────────► Simpan file upload ke Amazon S3
        └────────► Kirim log dan metrik ke CloudWatch
            │
            ▼
7. Response dikirim kembali ke ALB
            │
            ▼
8. ALB mengirimkan hasil ke browser pengguna
```

Kalau kamu benar-benar mengerjakan project ini, kamu akan menyentuh hampir seluruh fondasi AWS: **networking, security, compute, load balancing, database, storage, DNS, observability, dan operations**. Itu sebabnya project ini sangat sering dijadikan acuan untuk menilai kemampuan dasar seorang Cloud Engineer atau AWS Solutions Architect.
