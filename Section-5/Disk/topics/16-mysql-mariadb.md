# 16 · MySQL / MariaDB on Linux

[⬅ Previous: SATA & SAS](15-sata-sas.md) · [Back to index](../README.md) · [Next: LAMP Stack ➡](17-lamp-stack.md)

---

## 🎯 MySQL vs MariaDB — what's the difference?

Both are **relational databases** (data in tables with rows and columns, queried with SQL).

- **MySQL** is the original, now owned by Oracle.
- **MariaDB** is a community **fork** of MySQL, created by MySQL's original developers after the Oracle acquisition.

They're **largely drop-in compatible** — same `mysql` client, nearly identical commands.

> [!NOTE]
> On the **RHEL family**, **MariaDB** is the default in the base repositories. **MySQL** is available from Oracle's own repo. For most learning and many production uses, **MariaDB is the simpler choice** on RHEL/Amazon Linux.

---

## 🧪 Hands-on — install MariaDB (RHEL / Amazon Linux)

```bash
# 1. Install server + client
sudo dnf install -y mariadb-server mariadb
#    Amazon Linux 2: sudo amazon-linux-extras install mariadb10.5 -y

# 2. Start it and enable at boot
sudo systemctl enable --now mariadb
systemctl status mariadb --no-pager

# 3. Secure it (set root password, remove test data)
sudo mysql_secure_installation
#    Answer: set root password = Y, remove anonymous users = Y,
#            disallow remote root = Y, remove test DB = Y, reload = Y
```

<details>
<summary>👉 Prefer MySQL instead? (click to expand)</summary>

```bash
sudo dnf install -y https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql-community-server
sudo systemctl enable --now mysqld

# MySQL 8 generates a TEMPORARY root password — find it:
sudo grep 'temporary password' /var/log/mysqld.log
```
</details>

---

## 🧪 Hands-on — basic administration

### Connect

```bash
sudo mysql -u root -p
```

### Create a database, a user, and grant access

```sql
-- Create a database
CREATE DATABASE appdb;

-- Create an application user with a password
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'StrongP@ss1';

-- Give that user full rights on that one database
GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'localhost';
FLUSH PRIVILEGES;

-- Look around
SHOW DATABASES;
SELECT User, Host FROM mysql.user;
USE appdb;
SHOW TABLES;
\q                 -- quit
```

> 💡 **Understanding `'appuser'@'localhost'`:** MySQL identifies a user by **name + where they connect from**. `@'localhost'` = only from this machine. `@'%'` = from anywhere. `@'10.0.0.5'` = only that IP.

---

## 💾 Backup & restore (logical dumps)

```bash
# Back up one database
mysqldump -u root -p appdb > appdb_$(date +%F).sql

# Back up ALL databases
mysqldump -u root -p --all-databases > full_backup.sql

# Restore
mysql -u root -p appdb < appdb_2025-06-01.sql
```

> [!TIP]
> **Allowing remote connections** (only when needed):
> 1. Set `bind-address=0.0.0.0` in `/etc/my.cnf.d/*.cnf`.
> 2. Create users as `'appuser'@'%'` (or a specific IP).
> 3. Open TCP **3306** in the firewall / security group — **restricted to trusted IPs**.
> 4. Restart: `sudo systemctl restart mariadb`.
>
> ⚠️ **Never expose port 3306 to the whole internet.** Bots scan for open databases constantly.

---

## ✅ Key takeaways

- MariaDB is a drop-in MySQL fork; **default on RHEL/Amazon Linux**.
- Install: `dnf install mariadb-server` → `systemctl enable --now mariadb` → `mysql_secure_installation`.
- Core admin: `CREATE DATABASE`, `CREATE USER ... @host`, `GRANT`, `FLUSH PRIVILEGES`.
- Back up with `mysqldump`; restore by piping the `.sql` file back in.
- **Lock down remote access** — restrict by IP, never expose 3306 publicly.

## 💬 Interview questions

1. *MySQL vs MariaDB?* → MariaDB is a community fork, mostly drop-in compatible; default on RHEL.
2. *How do you back up and restore a database?* → `mysqldump` to a `.sql` file; restore with `mysql < file.sql`.
3. *What does `'user'@'localhost'` mean?* → the user plus the host they're allowed to connect from.

---

[⬅ Previous: SATA & SAS](15-sata-sas.md) · [Back to index](../README.md) · [Next: LAMP Stack ➡](17-lamp-stack.md)
