# AWS RDS Setup for BizBuch Backend (PostgreSQL)

- Document Title: AWS RDS Setup for BizBuch Backend (PostgreSQL)
- Version: 1.0.0
- Created On: 2026-02-18
- Created By: Codex
- Purpose: Step-by-step guide to create an Amazon RDS PostgreSQL database and connect the BizBuch Django backend.

---

## 1. Scope and architecture

This guide is for the current backend configuration in `mysite/settings.py`, which reads:

- `POSTGRES_DB`
- `POSTGRES_USER`
- `POSTGRES_PASSWORD`
- `DB_HOST`
- `DB_PORT`

Recommended architecture:

- Keep RDS private (`Public access = No`)
- Run backend inside the same VPC (EC2/ECS/EKS)
- Allow DB access from application security group only

If you need local laptop access temporarily, use a tight inbound rule (`your-ip/32`) and remove it after testing.

---

## 2. Prerequisites

1. AWS account with permissions for RDS, VPC, and Security Groups
2. Selected AWS Region (keep RDS and app in same region)
3. A VPC with subnets in at least two Availability Zones
4. A PostgreSQL client (`psql` or pgAdmin) for testing

---

## 3. Create network access for RDS

### 3.1 DB subnet group

1. Open AWS Console: `RDS -> Subnet groups -> Create DB subnet group`
2. Select your VPC
3. Add subnets from at least 2 Availability Zones
4. Save

Note: If you use the default VPC, AWS can provide a default subnet group.

### 3.2 Security group for database

1. Open AWS Console: `EC2 -> Security Groups -> Create security group`
2. Name example: `bizbuch-rds-sg`
3. Inbound rule:
   - Type: `PostgreSQL`
   - Port: `5432`
   - Source:
     - Preferred: your backend server security group
     - Temporary local testing: `<your-public-ip>/32`
4. Save security group

By default, an RDS DB instance has no inbound access until security group rules are added.

---

## 4. Create RDS PostgreSQL instance

1. Open AWS Console: `RDS -> Databases -> Create database`
2. Choose `Standard create`
3. Engine type: `PostgreSQL`
4. Choose the PostgreSQL engine version required by your environment
5. Template:
   - `Free tier` or `Dev/Test` for development
   - `Production` for production workloads
6. DB instance identifier: example `bizbuch-db-prod`
7. Master username and password: set securely
8. DB instance class: choose based on expected load
9. Storage:
   - Use SSD storage
   - Enable storage autoscaling
10. Availability:
    - Dev: `Single-AZ`
    - Prod: `Multi-AZ`
11. Connectivity:
    - VPC: your application VPC
    - DB subnet group: the one created above
    - Public access: `No` (recommended)
    - VPC security group: `bizbuch-rds-sg`
    - Database port: `5432`
12. Additional configuration:
    - Initial database name: `bizbuch` (or your preferred name)
    - Backup retention:
      - Console default is 7 days
      - Choose retention according to compliance and restore needs
13. Create database and wait for status `Available`

---

## 5. Collect connection details

From `RDS -> Databases -> <your-db> -> Connectivity & security`, copy:

- Endpoint (DNS)
- Port (default `5432`)

Example endpoint format:

`<db-identifier>.<random>.<region>.rds.amazonaws.com`

---

## 6. Configure BizBuch backend

Update `.env` (or deployment secrets) with your RDS values:

```env
POSTGRES_DB=bizbuch
POSTGRES_USER=<rds_master_or_app_user>
POSTGRES_PASSWORD=<strong_password>
DB_HOST=<rds_endpoint>
DB_PORT=5432
```

Important:

- Do not hardcode DB credentials in `docker-compose.yml` for shared environments
- Use environment secrets (SSM/Secrets Manager/GitHub Actions secrets/etc.)

---

## 7. Run migrations

If using Docker:

```bash
docker compose up -d
docker compose exec web python manage.py migrate
```

If running locally:

```bash
python manage.py migrate
```

---

## 8. Validate database connection

### 8.1 Test with psql

```bash
psql "host=<rds_endpoint> port=5432 dbname=bizbuch user=<db_user> sslmode=require"
```

### 8.2 Test from Django

```bash
python manage.py shell -c "from django.db import connection; connection.cursor().execute('SELECT 1'); print('OK')"
```

### 8.3 Connect from laptop through EC2 (recommended)

Use this when RDS is private (`Public access = No`), which is the recommended production setup.

Pre-check:

1. EC2 and RDS are in the same VPC (or peered VPCs with routes)
2. RDS security group allows port `5432` from the EC2 security group
3. Your laptop can SSH to EC2 (`22` allowed from your IP)

Open SSH tunnel from laptop:

```bash
ssh -i /path/to/key.pem -N -L 5433:<rds_endpoint>:5432 ec2-user@<ec2_public_ip_or_dns>
```

Notes:

- For Ubuntu AMI, user is usually `ubuntu` (not `ec2-user`)
- Keep this terminal open while using DB tools

Now connect from laptop to the local forwarded port:

```bash
psql "host=127.0.0.1 port=5433 dbname=bizbuch user=<db_user> sslmode=require"
```

For pgAdmin:

- Host: `127.0.0.1`
- Port: `5433`
- Database: `bizbuch`
- Username/password: your RDS credentials

### 8.4 Direct laptop access to RDS (temporary/dev only) with `Public access = No` and without EC2

With `Public access = No`, direct internet connection from laptop to RDS endpoint is not possible.
Without EC2/bastion, use private networking from laptop into the VPC. The common option is AWS Client VPN.

1. Create an AWS Client VPN endpoint in the same VPC as RDS:
   - Choose client CIDR block (example: `10.250.0.0/22`)
   - Configure authentication (certificate/SAML/Directory)
2. Associate the Client VPN endpoint with at least one target subnet in the VPC.
3. Add authorization rule for the VPC CIDR (or specific DB subnet CIDR).
4. Add route from Client VPN endpoint to VPC CIDR.
5. Download Client VPN config file from AWS console and import into AWS VPN Client (or OpenVPN client).
6. Connect VPN from laptop.
7. Update RDS security group inbound rule:
   - Type: `PostgreSQL`
   - Port: `5432`
   - Source: Client VPN CIDR (example: `10.250.0.0/22`)
8. Connect from laptop (while VPN is connected):

```bash
psql "host=<rds_endpoint> port=5432 dbname=bizbuch user=<db_user> sslmode=require"
```

Other non-EC2 private options:

- Site-to-Site VPN from office/home router to VPC
- AWS Direct Connect

### 8.5 Direct laptop access to RDS (temporary/dev only) with `Public access = Yes`

Use only for short-term debugging.

1. Set `Public access = Yes` for your DB instance:
   - Go to `AWS Console -> RDS -> Databases`
   - Open your DB instance
   - Click `Modify`
   - In the `Connectivity` section, set `Public access` to `Yes`
   - Click `Continue`
   - Choose `Apply immediately`
   - Click `Modify DB instance`
   - Wait until DB status returns to `Available`
2. In the attached RDS security group, add inbound rule:
   - Go to `RDS -> Databases -> <your-db> -> Connectivity & security`
   - In `VPC security groups` (or `Security groups` in some console views), click the security group ID (example: `sg-0abc...`)
   - In EC2 Security Group page, open `Inbound rules -> Edit inbound rules`
   - Add rule:
     - Type: `PostgreSQL`
     - Port range: `5432`
     - Source:
       - `My IP` (if shown by console), or
       - `Custom` and enter `<your-public-ip>/32`
     - Description: `Laptop temporary access`
   - Click `Save rules`

How to get your laptop public IP:

- macOS/Linux:
  ```bash
  curl https://checkip.amazonaws.com
  ```
- Windows PowerShell:
  ```powershell
  (Invoke-RestMethod -Uri "https://checkip.amazonaws.com").Trim()
  ```
- Browser option: search `what is my ip`

Use the IP in CIDR form with `/32` (example: `49.37.120.10/32`).
Do not use local/private IPs like `192.168.x.x` or `10.x.x.x`.
If you are on VPN, your public IP can be different.
3. Connect:

```bash
psql "host=<rds_endpoint> port=5432 dbname=bizbuch user=<db_user> sslmode=require"
```

After testing:

1. Remove laptop IP rule from security group
2. Set `Public access = No` again if changed

---

## 9. TLS and certificate guidance

- Prefer encrypted DB connections (`sslmode=require` or stronger)
- Amazon RDS supports managed CA rotation and currently defaults to `rds-ca-rsa2048-g1`
- For strict certificate validation, use the official RDS certificate bundle for your region from AWS documentation

---

## 10. Production checklist

1. Enable `Multi-AZ`
2. Keep `Public access = No`
3. Restrict security group source to app security group only
4. Enable deletion protection
5. Set backup retention (`1-35` days for DB instances; avoid `0` in production)
6. Enable Performance Insights and CloudWatch alarms
7. Store credentials in Secrets Manager or Parameter Store
8. Regularly test restore from snapshot / point-in-time restore

---

## 11. Troubleshooting quick checks

1. Connection timeout:
   - Verify security group inbound rule on `5432`
   - Verify app can route to DB subnet/VPC
2. Authentication failed:
   - Check `POSTGRES_USER` and `POSTGRES_PASSWORD`
3. Name resolution issues:
   - Use exact RDS endpoint from console
4. App starts but migrations fail:
   - Verify `POSTGRES_DB` exists and user has privileges

---

## 12. Official AWS references

1. Setting up your Amazon RDS environment  
   `https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SettingUp.html`
2. Creating an Amazon RDS DB instance  
   `https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html`
3. Creating and connecting to a PostgreSQL DB instance  
   `https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html`
4. Controlling access with security groups  
   `https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.RDSSecurityGroups.html`
5. Backup retention period  
   `https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.BackupRetention.html`
6. Using SSL/TLS to encrypt connections  
   `https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html`
