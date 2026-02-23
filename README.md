# Oracle 19c Installation Guide for Informatica 10.4.x

> Complete step-by-step guide to install and configure **Oracle Database 19c** as the metadata repository for **Informatica PowerCenter 10.4.x** on Windows 64-bit.

---

## Table of Contents

- [00 · Prerequisites](#00--prerequisites)
- [01 · Install Oracle 19c](#01--install-oracle-19c)
- [02 · SQL\*Plus Configuration](#02--sqlplus-configuration)
- [03 · Create Informatica Users](#03--create-informatica-users)
- [04 · Grant All Privileges](#04--grant-all-privileges)
- [05 · System Tuning](#05--system-tuning)
- [06 · Install Java JDK 17](#06--install-java-jdk-17)
- [07 · Install SQL Developer](#07--install-sql-developer)
- [08 · Database Connections](#08--database-connections)

---

## 00 · Prerequisites

The following software must be available before starting the installation:

| Component | Version / Details |
|---|---|
| ☕ Java JDK | 17 (Windows 64-bit) |
| 🗄️ Oracle Database | 19c Enterprise Edition |
| 🖥️ SQL Developer | GUI client (bundled with JDK 17) |
| 🪟 Platform | Windows 64-bit |

---

## 01 · Install Oracle 19c

### Download

Download Oracle 19c for Windows (64-bit) from the official Oracle website:

```
https://www.oracle.com/in/database/technologies/oracle19c-windows-downloads.html
```

### Installation Steps

1. **Extract** the downloaded `.zip` file to your Oracle base directory.
2. Navigate to the extracted folder:
   ```
   G:\oracle_19c\WINDOWS.X64_193000_db_home
   ```
3. Right-click **`setup.exe`** → select **Run as Administrator**.
4. The Oracle Universal Installer will launch.

### Installer Configuration

When the installer prompts you, select the following options:

1. Select **Desktop Class** → click **Next**
2. Choose **Use Windows Built-in Account** → click **Next**
3. Accept any confirmation or warning messages → click **Yes**
4. Proceed through the wizard and wait for the installation to complete

### 📌 Oracle 19c Configuration Details

Use these exact values when the installer prompts for configuration:

| Setting | Value |
|---|---|
| Oracle Base | `G:\oracle_19c` |
| Software Location (Oracle Home) | `G:\oracle_19c\WINDOWS.X64_193000_db_home` |
| Database File Location (Datafiles) | `G:\oracle_19c\oradata` |
| Database Edition | Enterprise Edition |
| Character Set | Unicode — `AL32UTF8` |
| Global Database Name | `orcl` |
| Password | `orcl` |
| Confirm Password | `orcl` |
| Container Database (CDB) | ✅ Check **Create as Container Database** |
| Pluggable Database (PDB) Name | `orclpdb` |

> **Note:** When prompted, accept all confirmation messages (Yes / OK), then click **Install** and wait for completion.

---

## 02 · SQL\*Plus Configuration

### Connect to SQL\*Plus as SYSDBA

Open **SQL\*Plus** from the Start menu or command line and connect:

```
Enter user-name: sys /as sysdba
Enter password: orcl
```

### Open Pluggable Databases

Run the following commands after connecting:

```sql
-- Check existing PDBs
SQL> SHOW PDBS;

    CON_ID CON_NAME              OPEN MODE  RESTRICTED
---------- --------------------- ---------- ----------
         2 PDB$SEED              READ ONLY  NO
         3 ORCLPDB               READ WRITE NO

-- Open all pluggable databases
SQL> ALTER PLUGGABLE DATABASE ALL OPEN;

Pluggable database altered.
```

### Switch to ORCLPDB Container

```sql
SQL> ALTER SESSION SET CONTAINER = ORCLPDB;

Session altered.

SQL> SHOW CON_NAME;

CON_NAME
------------------------------
ORCLPDB
```

---

## 03 · Create Informatica Users

### Step 1 — Create a No-Expiry Password Profile

```sql
SQL> CREATE PROFILE NO_EXPIRY_PROFILE
  2  LIMIT PASSWORD_LIFE_TIME UNLIMITED;
```

> **Note:** If you see `ORA-01920: user name conflicts with another role` errors below, the users already exist from a prior setup. The `GRANT` commands will still succeed — continue through the errors.

---

### INFA\_REP (Repository Schema)

```sql
SQL> CREATE USER INFA_REP IDENTIFIED BY INFA_REP
  2    PROFILE NO_EXPIRY_PROFILE
  3    DEFAULT TABLESPACE USERS
  4    TEMPORARY TABLESPACE TEMP;

SQL> GRANT DBA TO INFA_REP;
Grant succeeded.

SQL> GRANT UNLIMITED TABLESPACE TO INFA_REP;
Grant succeeded.
```

---

### INFA\_DOMAIN (Domain Configuration Schema)

```sql
SQL> CREATE USER INFA_DOMAIN IDENTIFIED BY INFA_DOMAIN
  2    PROFILE NO_EXPIRY_PROFILE
  3    DEFAULT TABLESPACE USERS
  4    TEMPORARY TABLESPACE TEMP;

SQL> GRANT DBA TO INFA_DOMAIN;
Grant succeeded.

SQL> GRANT UNLIMITED TABLESPACE TO INFA_DOMAIN;
Grant succeeded.
```

---

### INFA\_DEV (Development Schema)

```sql
SQL> CREATE USER INFA_DEV IDENTIFIED BY INFA_DEV
  2    PROFILE NO_EXPIRY_PROFILE
  3    DEFAULT TABLESPACE USERS
  4    TEMPORARY TABLESPACE TEMP;

SQL> GRANT DBA TO INFA_DEV;
Grant succeeded.

SQL> GRANT UNLIMITED TABLESPACE TO INFA_DEV;
Grant succeeded.
```

---

### INFA\_UAT (User Acceptance Testing Schema)

```sql
SQL> CREATE USER INFA_UAT IDENTIFIED BY INFA_UAT
  2    PROFILE NO_EXPIRY_PROFILE
  3    DEFAULT TABLESPACE USERS
  4    TEMPORARY TABLESPACE TEMP;

SQL> GRANT DBA TO INFA_UAT;
Grant succeeded.

SQL> GRANT UNLIMITED TABLESPACE TO INFA_UAT;
Grant succeeded.
```

---

## 04 · Grant All Privileges

Grant full privileges to all four Informatica schema users:

```sql
SQL> GRANT ALL PRIVILEGES TO INFA_REP;
Grant succeeded.

SQL> GRANT ALL PRIVILEGES TO INFA_DOMAIN;
Grant succeeded.

SQL> GRANT ALL PRIVILEGES TO INFA_DEV;
Grant succeeded.

SQL> GRANT ALL PRIVILEGES TO INFA_UAT;
Grant succeeded.
```

---

## 05 · System Tuning

Run these commands to configure performance parameters required by Informatica:

```sql
-- Remove default password expiry from the DEFAULT profile
SQL> ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
Profile altered.

-- Switch back to CDB root for system-level changes
SQL> ALTER SESSION SET CONTAINER = CDB$ROOT;
Session altered.

-- Increase open cursors to support Informatica's concurrent sessions
SQL> ALTER SYSTEM SET open_cursors = 5000 SCOPE = BOTH;
System altered.
```

> **Why `SCOPE=BOTH`?** This applies the change to both the running instance and the `spfile`, so it persists across database restarts.

---

## 06 · Install Java JDK 17

### Download

Download **Windows 64-bit with JDK 17 included** from the Oracle SQL Developer page:

```
https://www.oracle.com/in/database/sqldeveloper/technologies/download/
```

### File Details

| Field | Value |
|---|---|
| Version | Windows 64-bit with JDK 17 included |
| MD5 | `bb9b6abe39f5274b090ce2c7b884f6ff` |
| SHA1 | `5295a49db5b39b2640015e535ff1528822a4ba5a` |

### Installation Steps

1. Download the installer from the link above.
2. Run the **setup file**.
3. Complete the installation using **default settings**.

---

## 07 · Install SQL Developer

### Installation Steps

1. Download **Oracle SQL Developer** from the same page as JDK 17.
2. Extract or install the downloaded file.
3. Launch **SQL Developer**.
4. If prompted to map the Java (JDK) path, provide:
   ```
   C:\Program Files\Java\jdk-17.0.18
   ```
   Then click **OK** to continue.

---

## 08 · Database Connections

Create the following connections in **Oracle SQL Developer**.

> All connections share the same settings: **Service Name:** `ORCLPDB` · **Host:** `localhost` · **Port:** `1521`

### Connection 1 — INFA\_DEV

| Field | Value |
|---|---|
| Connection Name | `INFA_DEV` |
| Username | `INFA_DEV` |
| Password | `INFA_DEV` |
| Hostname | `localhost` |
| Port | `1521` |
| Service Name | `ORCLPDB` |

### Connection 2 — INFA\_UAT

| Field | Value |
|---|---|
| Connection Name | `INFA_UAT` |
| Username | `INFA_UAT` |
| Password | `INFA_UAT` |
| Hostname | `localhost` |
| Port | `1521` |
| Service Name | `ORCLPDB` |

### Connection 3 — INFA\_REP

| Field | Value |
|---|---|
| Connection Name | `INFA_REP` |
| Username | `INFA_REP` |
| Password | `INFA_REP` |
| Hostname | `localhost` |
| Port | `1521` |
| Service Name | `ORCLPDB` |

### Connection 4 — INFA\_DOMAIN

| Field | Value |
|---|---|
| Connection Name | `INFA_DOMAIN` |
| Username | `INFA_DOMAIN` |
| Password | `INFA_DOMAIN` |
| Hostname | `localhost` |
| Port | `1521` |
| Service Name | `ORCLPDB` |

### After Creating Each Connection

1. Click **Test** — confirm Status shows **Success**
2. Click **Save**
3. Click **Connect**

---

## ✅ Setup Complete

Oracle 19c is now fully configured and ready for the **Informatica 10.4.x** installation.
All metadata repository schemas (`INFA_REP`, `INFA_DOMAIN`, `INFA_DEV`, `INFA_UAT`) are provisioned and accessible via SQL Developer.

---

*Oracle 19c · Informatica 10.4.x · Windows 64-bit · Enterprise Edition*
