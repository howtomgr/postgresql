# PostgreSQL Installation Guide

PostgreSQL is a free and open-source relational database management system (RDBMS) emphasizing extensibility and SQL compliance. Originally developed at UC Berkeley as POSTGRES, it has evolved into the world's most advanced open source database, competing directly with commercial solutions like Oracle Database, Microsoft SQL Server, and IBM Db2. PostgreSQL serves as a FOSS alternative to these expensive proprietary databases while offering comparable performance, reliability, and advanced features like JSON support, full-text search, and custom data types.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2 cores minimum (4+ cores recommended for production)
  - RAM: 2GB minimum (8GB+ recommended for production)
  - Storage: 1GB for installation (SSD strongly recommended for data)
  - IOPS: 3000+ recommended for production workloads
- **Operating System**: Linux, BSD, macOS, or Windows
- **Network Requirements**:
  - Port 5432 (default PostgreSQL port)
  - Additional ports for replication if needed
  - Low latency network for clustered setups
- **Dependencies**:
  - GNU make 3.81+ (for building from source)
  - ISO/ANSI C compiler (gcc 4.7+ recommended)
  - tar, gzip or bzip2 for unpacking source
  - readline library (for psql command history)
  - zlib library (for pg_dump compression)
  - OpenSSL library (for SSL connections)
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# RHEL/CentOS 7
# Install PostgreSQL official repository
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Install PostgreSQL 16 (latest stable)
sudo yum install -y postgresql16-server postgresql16 postgresql16-contrib

# Initialize database
sudo /usr/pgsql-16/bin/postgresql-16-setup initdb

# Enable and start service
sudo systemctl enable postgresql-16
sudo systemctl start postgresql-16

# RHEL/CentOS/Rocky/AlmaLinux 8+
# Install PostgreSQL official repository
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Disable built-in PostgreSQL module
sudo dnf -qy module disable postgresql

# Install PostgreSQL 16
sudo dnf install -y postgresql16-server postgresql16 postgresql16-contrib

# Initialize database
sudo /usr/pgsql-16/bin/postgresql-16-setup initdb

# Enable and start service
sudo systemctl enable --now postgresql-16
```

### Debian/Ubuntu

```bash
# Install prerequisites
sudo apt update
sudo apt install -y wget ca-certificates

# Add PostgreSQL APT repository
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Update package list
sudo apt update

# Install PostgreSQL 16
sudo apt install -y postgresql-16 postgresql-client-16 postgresql-contrib-16

# PostgreSQL should start automatically
sudo systemctl status postgresql
```

### Arch Linux

```bash
# Install PostgreSQL from official repositories
sudo pacman -S postgresql postgresql-libs

# Optional: Install additional tools
sudo pacman -S postgresql-old-upgrade  # For upgrading from older versions

# Initialize the database cluster
sudo -u postgres initdb -D /var/lib/postgres/data

# Enable and start service
sudo systemctl enable --now postgresql

# For development headers
sudo pacman -S postgresql-libs
```

### Alpine Linux

```bash
# Install PostgreSQL
apk add --no-cache postgresql postgresql-client postgresql-contrib

# Install additional packages for full functionality
apk add --no-cache postgresql-dev postgresql-docs

# Create postgres user if not exists
adduser -D -H -s /sbin/nologin -g postgres postgres

# Initialize database
su - postgres -s /bin/sh -c "initdb -D /var/lib/postgresql/data"

# Enable and start service
rc-update add postgresql default
rc-service postgresql start
```

### openSUSE/SLES

```bash
# openSUSE Leap/Tumbleweed
sudo zypper install -y postgresql16 postgresql16-server postgresql16-contrib

# Initialize database (if not auto-initialized)
sudo systemctl start postgresql

# If initialization is needed
sudo -u postgres initdb -D /var/lib/pgsql/data

# Enable service
sudo systemctl enable postgresql

# SLES 15
# Enable Web and Scripting Module
sudo SUSEConnect -p sle-module-web-scripting/15.5/x86_64

# Install PostgreSQL
sudo zypper install -y postgresql16 postgresql16-server
```

### macOS

```bash
# Using Homebrew
brew install postgresql@16

# Add to PATH
echo 'export PATH="/usr/local/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Initialize database
initdb /usr/local/var/postgresql@16

# Start PostgreSQL
brew services start postgresql@16

# Alternative: Using MacPorts
sudo port install postgresql16 +universal
sudo port install postgresql16-server

# Initialize database
sudo mkdir -p /opt/local/var/db/postgresql16/defaultdb
sudo chown postgres:postgres /opt/local/var/db/postgresql16/defaultdb
sudo -u postgres /opt/local/lib/postgresql16/bin/initdb -D /opt/local/var/db/postgresql16/defaultdb
```

### FreeBSD

```bash
# Install PostgreSQL
pkg install postgresql16-server postgresql16-client postgresql16-contrib

# Enable PostgreSQL
echo 'postgresql_enable="YES"' >> /etc/rc.conf

# Initialize database
service postgresql initdb

# Start PostgreSQL
service postgresql start

# For development
pkg install postgresql16-plpython py39-psycopg2
```

### Windows

```powershell
# Method 1: Using Chocolatey
choco install postgresql16 --params '/Password:StrongPassword123!'

# Method 2: Using the official installer
# Download from https://www.postgresql.org/download/windows/
# Run the installer with administrative privileges

# Method 3: Using winget
winget install --id PostgreSQL.PostgreSQL

# After installation, add to PATH
[Environment]::SetEnvironmentVariable("Path", "$env:Path;C:\Program Files\PostgreSQL\16\bin", "Machine")

# Initialize database (usually done by installer)
initdb -D "C:\Program Files\PostgreSQL\16\data" -U postgres -W

# Register as Windows service
pg_ctl register -N postgresql-16 -D "C:\Program Files\PostgreSQL\16\data"

# Start service
net start postgresql-16
```

## Initial Configuration

### First-Run Setup

1. **Switch to postgres user**:
```bash
# Linux/BSD systems
sudo -i -u postgres

# Or use sudo for individual commands
sudo -u postgres psql
```

2. **Default configuration locations**:
- RHEL/CentOS/Rocky/AlmaLinux: `/var/lib/pgsql/16/data/postgresql.conf`
- Debian/Ubuntu: `/etc/postgresql/16/main/postgresql.conf`
- Arch Linux: `/var/lib/postgres/data/postgresql.conf`
- Alpine Linux: `/var/lib/postgresql/data/postgresql.conf`
- openSUSE/SLES: `/var/lib/pgsql/data/postgresql.conf`
- macOS: `/usr/local/var/postgresql@16/postgresql.conf`
- FreeBSD: `/var/db/postgres/data16/postgresql.conf`
- Windows: `C:\Program Files\PostgreSQL\16\data\postgresql.conf`

3. **Essential settings to change**:

```bash
# Edit postgresql.conf
sudo -u postgres vi /var/lib/pgsql/16/data/postgresql.conf

# Key settings to modify:
listen_addresses = 'localhost'  # Change to '*' for network access
port = 5432
max_connections = 100           # Increase based on needs
shared_buffers = 256MB          # Set to 25% of RAM
effective_cache_size = 1GB      # Set to 50-75% of RAM
work_mem = 4MB                  # Increase for complex queries
maintenance_work_mem = 64MB     # For VACUUM, CREATE INDEX

# Enable logging
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_rotation_age = 1d
log_rotation_size = 100MB
```

4. **Configure authentication** (pg_hba.conf):
```bash
# Edit pg_hba.conf
sudo -u postgres vi /var/lib/pgsql/16/data/pg_hba.conf

# Change authentication method from 'ident' to 'md5' or 'scram-sha-256'
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     scram-sha-256
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
```

5. **Set postgres user password**:
```bash
sudo -u postgres psql
postgres=# ALTER USER postgres PASSWORD 'StrongPassword123!';
postgres=# \q
```

### Testing Initial Setup

```bash
# Test local connection
sudo -u postgres psql -c "SELECT version();"

# Create a test database
sudo -u postgres createdb testdb

# Connect to test database
sudo -u postgres psql testdb

# Run test query
testdb=# CREATE TABLE test (id serial PRIMARY KEY, name varchar(50));
testdb=# INSERT INTO test (name) VALUES ('PostgreSQL');
testdb=# SELECT * FROM test;
testdb=# \q

# Drop test database
sudo -u postgres dropdb testdb
```

**WARNING:** Never use default passwords in production! Always set strong passwords and restrict network access.

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable PostgreSQL to start on boot
sudo systemctl enable postgresql    # or postgresql-16 on RHEL

# Start PostgreSQL
sudo systemctl start postgresql

# Stop PostgreSQL
sudo systemctl stop postgresql

# Restart PostgreSQL
sudo systemctl restart postgresql

# Reload configuration without restart
sudo systemctl reload postgresql

# Check status
sudo systemctl status postgresql

# View logs
sudo journalctl -u postgresql -f
```

### OpenRC (Alpine Linux)

```bash
# Enable PostgreSQL to start on boot
rc-update add postgresql default

# Start PostgreSQL
rc-service postgresql start

# Stop PostgreSQL
rc-service postgresql stop

# Restart PostgreSQL
rc-service postgresql restart

# Reload configuration
rc-service postgresql reload

# Check status
rc-service postgresql status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'postgresql_enable="YES"' >> /etc/rc.conf

# Start PostgreSQL
service postgresql start

# Stop PostgreSQL
service postgresql stop

# Restart PostgreSQL
service postgresql restart

# Reload configuration
service postgresql reload

# Check status
service postgresql status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start postgresql@16
brew services stop postgresql@16
brew services restart postgresql@16

# Manual control
pg_ctl -D /usr/local/var/postgresql@16 start
pg_ctl -D /usr/local/var/postgresql@16 stop
pg_ctl -D /usr/local/var/postgresql@16 restart

# Check status
brew services list | grep postgresql
```

### Windows Service Manager

```powershell
# Start PostgreSQL service
net start postgresql-16

# Stop PostgreSQL service
net stop postgresql-16

# Restart PostgreSQL service
net stop postgresql-16 && net start postgresql-16

# Using pg_ctl
pg_ctl -D "C:\Program Files\PostgreSQL\16\data" start
pg_ctl -D "C:\Program Files\PostgreSQL\16\data" stop
pg_ctl -D "C:\Program Files\PostgreSQL\16\data" restart

# Check status
sc query postgresql-16
```

## Advanced Configuration

### 8. Performance Tuning

```bash
# Calculate settings based on system resources
# Edit postgresql.conf

# Memory Settings (for 8GB RAM system)
shared_buffers = 2GB              # 25% of RAM
effective_cache_size = 6GB        # 75% of RAM
maintenance_work_mem = 512MB      # RAM/16
work_mem = 32MB                   # RAM/256
wal_buffers = 64MB               # 3% of shared_buffers

# Checkpoint Settings
checkpoint_segments = 32          # Deprecated in newer versions
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100

# Connection Settings
max_connections = 200             # Adjust based on application

# Parallel Query Execution (PG 9.6+)
max_worker_processes = 8
max_parallel_workers_per_gather = 4
max_parallel_workers = 8

# Write Performance
synchronous_commit = on           # Set to off for better performance (less safe)
checkpoint_timeout = 15min
max_wal_size = 4GB
min_wal_size = 1GB

# SSD Optimizations
random_page_cost = 1.1           # Default is 4.0 (for HDD)
effective_io_concurrency = 200   # 1-1000 (higher for SSD)
```

### SSL Configuration

```bash
# Generate SSL certificates
cd /var/lib/pgsql/16/data

# Create CA certificate
openssl genrsa -out ca.key 4096
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=PostgreSQL-CA"

# Create server certificate
openssl genrsa -out server.key 4096
openssl req -new -key server.key -out server.csr \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=postgres.example.com"
openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt

# Set permissions
chown postgres:postgres server.key server.crt ca.crt
chmod 600 server.key
chmod 644 server.crt ca.crt

# Enable SSL in postgresql.conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ca_file = 'ca.crt'
ssl_min_protocol_version = 'TLSv1.2'
```

## Reverse Proxy Setup

PostgreSQL typically doesn't use HTTP reverse proxies, but connection poolers like PgBouncer or HAProxy can act as database proxies.

### PgBouncer Configuration

```bash
# Install PgBouncer
# RHEL/CentOS
sudo yum install -y pgbouncer

# Debian/Ubuntu
sudo apt install -y pgbouncer

# Configure PgBouncer (/etc/pgbouncer/pgbouncer.ini)
[databases]
mydb = host=127.0.0.1 port=5432 dbname=mydb

[pgbouncer]
listen_port = 6432
listen_addr = *
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
```

### HAProxy Configuration

```nginx
# /etc/haproxy/haproxy.cfg
global
    maxconn 1000

defaults
    mode tcp
    timeout connect 10s
    timeout client 30s
    timeout server 30s

listen postgres
    bind *:5432
    option pgsql-check user haproxy
    balance roundrobin
    server pg1 192.168.1.10:5432 check
    server pg2 192.168.1.11:5432 check backup
```

## Security Configuration

### Authentication and Access Control

```bash
# Configure pg_hba.conf for secure access
# /var/lib/pgsql/16/data/pg_hba.conf

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# Local connections
local   all             postgres                                peer
local   all             all                                     scram-sha-256

# IPv4 local connections
host    all             all             127.0.0.1/32            scram-sha-256

# IPv4 remote connections (specific network)
host    all             all             192.168.1.0/24          scram-sha-256

# Reject all other connections
host    all             all             0.0.0.0/0               reject

# IPv6 local connections
host    all             all             ::1/128                 scram-sha-256
```

### User and Role Management

```sql
-- Create roles with specific privileges
CREATE ROLE readonly;
GRANT CONNECT ON DATABASE mydb TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly;

-- Create application user
CREATE USER appuser WITH PASSWORD 'StrongAppPassword123!';
GRANT CONNECT ON DATABASE mydb TO appuser;
GRANT USAGE ON SCHEMA public TO appuser;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO appuser;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO appuser;

-- Create backup user
CREATE USER backupuser WITH PASSWORD 'BackupPassword123!' REPLICATION;

-- Revoke unnecessary privileges
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
```

### Row-Level Security (RLS)

```sql
-- Enable RLS on a table
ALTER TABLE sensitive_data ENABLE ROW LEVEL SECURITY;

-- Create policy for users to see only their own data
CREATE POLICY user_data_policy ON sensitive_data
    FOR ALL
    TO application_role
    USING (user_id = current_user_id());

-- Create policy for admins to see all data
CREATE POLICY admin_policy ON sensitive_data
    FOR ALL
    TO admin_role
    USING (true);
```

### Firewall Rules

```bash
# UFW (Ubuntu/Debian)
sudo ufw allow from 192.168.1.0/24 to any port 5432
sudo ufw reload

# firewalld (RHEL/CentOS/openSUSE)
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="5432" accept'
sudo firewall-cmd --reload

# iptables
sudo iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 5432 -j ACCEPT
sudo iptables-save > /etc/iptables/rules.v4

# pf (FreeBSD)
# Add to /etc/pf.conf
pass in on $ext_if proto tcp from 192.168.1.0/24 to any port 5432

# Windows Firewall
New-NetFirewallRule -DisplayName "PostgreSQL" -Direction Inbound -Protocol TCP -LocalPort 5432 -Action Allow
```

## Database Setup

### Creating Databases and Schemas

```sql
-- Create database with specific encoding and locale
CREATE DATABASE myapp
    WITH 
    OWNER = appuser
    ENCODING = 'UTF8'
    LC_COLLATE = 'en_US.UTF-8'
    LC_CTYPE = 'en_US.UTF-8'
    TABLESPACE = pg_default
    CONNECTION LIMIT = -1;

-- Create schema
\c myapp
CREATE SCHEMA IF NOT EXISTS app_schema AUTHORIZATION appuser;

-- Set search path
ALTER DATABASE myapp SET search_path TO app_schema, public;

-- Create extensions
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

### Table Partitioning

```sql
-- Create partitioned table
CREATE TABLE measurements (
    id BIGSERIAL,
    sensor_id INTEGER,
    reading NUMERIC,
    created_at TIMESTAMP NOT NULL
) PARTITION BY RANGE (created_at);

-- Create partitions
CREATE TABLE measurements_2024_01 PARTITION OF measurements
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE measurements_2024_02 PARTITION OF measurements
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- Create index on partitions
CREATE INDEX idx_measurements_2024_01_created_at ON measurements_2024_01 (created_at);
CREATE INDEX idx_measurements_2024_02_created_at ON measurements_2024_02 (created_at);
```

## Performance Optimization

### Query Optimization

```sql
-- Enable query timing
\timing on

-- Analyze query performance
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM large_table WHERE column = 'value';

-- Create appropriate indexes
CREATE INDEX idx_column ON large_table (column);
CREATE INDEX idx_multi_column ON large_table (col1, col2) WHERE active = true;

-- Update table statistics
ANALYZE large_table;

-- Find missing indexes
SELECT 
    schemaname,
    tablename,
    attname,
    n_distinct,
    most_common_vals
FROM pg_stats
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
    AND n_distinct > 100
    AND most_common_vals IS NULL;
```

### Connection Pooling

```bash
# Install PgBouncer for connection pooling
# Configure /etc/pgbouncer/pgbouncer.ini
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
listen_port = 6432
listen_addr = localhost
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
min_pool_size = 5
reserve_pool_size = 5
server_idle_timeout = 600
```

### Vacuum and Maintenance

```sql
-- Manual VACUUM
VACUUM (VERBOSE, ANALYZE) large_table;

-- Configure autovacuum
ALTER TABLE large_table SET (autovacuum_vacuum_scale_factor = 0.1);
ALTER TABLE large_table SET (autovacuum_analyze_scale_factor = 0.05);

-- Find tables that need vacuuming
SELECT 
    schemaname,
    tablename,
    n_live_tup,
    n_dead_tup,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;
```

## Monitoring

### Built-in Monitoring Views

```sql
-- Current activity
SELECT pid, usename, datname, state, query 
FROM pg_stat_activity 
WHERE state != 'idle';

-- Database statistics
SELECT 
    datname,
    numbackends,
    xact_commit,
    xact_rollback,
    blks_read,
    blks_hit,
    tup_returned,
    tup_fetched,
    tup_inserted,
    tup_updated,
    tup_deleted
FROM pg_stat_database;

-- Table I/O statistics
SELECT 
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    idx_tup_fetch,
    n_tup_ins,
    n_tup_upd,
    n_tup_del
FROM pg_stat_user_tables
ORDER BY seq_tup_read DESC;

-- Cache hit ratio
SELECT 
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as cache_hit_ratio
FROM pg_statio_user_tables;
```

### pg_stat_statements

```sql
-- Enable extension
CREATE EXTENSION pg_stat_statements;

-- Top queries by total time
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    stddev_exec_time,
    rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Reset statistics
SELECT pg_stat_statements_reset();
```

### External Monitoring Tools

```bash
# Install check_postgres for Nagios/Icinga
wget https://bucardo.org/check_postgres/check_postgres.tar.gz
tar xzf check_postgres.tar.gz
cd check_postgres-*
perl Makefile.PL
make
sudo make install

# PostgreSQL Exporter for Prometheus
wget https://github.com/prometheus-community/postgres_exporter/releases/download/v0.13.0/postgres_exporter-0.13.0.linux-amd64.tar.gz
tar xzf postgres_exporter-*.tar.gz
sudo cp postgres_exporter*/postgres_exporter /usr/local/bin/

# Create systemd service for postgres_exporter
sudo tee /etc/systemd/system/postgres_exporter.service <<EOF
[Unit]
Description=PostgreSQL Exporter
After=postgresql.service

[Service]
Type=simple
User=postgres
Environment="DATA_SOURCE_NAME=postgresql://postgres@localhost/postgres?sslmode=disable"
ExecStart=/usr/local/bin/postgres_exporter
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable --now postgres_exporter
```

## 9. Backup and Restore

### Logical Backups with pg_dump

```bash
# Backup single database
pg_dump -h localhost -U postgres -d mydb -F custom -b -v -f mydb_backup.dump

# Backup all databases
pg_dumpall -h localhost -U postgres -f all_databases.sql

# Backup with compression
pg_dump mydb | gzip > mydb_backup.sql.gz

# Backup specific schemas
pg_dump -n schema1 -n schema2 mydb > schemas_backup.sql

# Backup only schema (no data)
pg_dump -s mydb > mydb_schema.sql

# Backup only data (no schema)
pg_dump -a mydb > mydb_data.sql
```

### Physical Backups with pg_basebackup

```bash
# Create base backup
pg_basebackup -h localhost -D /backup/postgres -U replicator -W -Fp -Xs -P

# Create tar format backup
pg_basebackup -h localhost -D /backup/postgres -U replicator -W -Ft -z -Xs -P

# Backup with specific tablespace mapping
pg_basebackup -h localhost -D /backup/postgres -T /old/tablespace=/new/tablespace
```

### Restore Procedures

```bash
# Restore from custom format dump
pg_restore -h localhost -U postgres -d mydb -v mydb_backup.dump

# Restore to new database
createdb -h localhost -U postgres newdb
pg_restore -h localhost -U postgres -d newdb -v mydb_backup.dump

# Restore from SQL dump
psql -h localhost -U postgres -d mydb < mydb_backup.sql

# Restore specific tables
pg_restore -h localhost -U postgres -d mydb -t table1 -t table2 mydb_backup.dump

# Restore with parallel jobs
pg_restore -h localhost -U postgres -d mydb -j 4 -v mydb_backup.dump
```

### Automated Backup Script

```bash
#!/bin/bash
# /usr/local/bin/pg_backup.sh

BACKUP_DIR="/backup/postgresql"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
DATABASE="mydb"

# Create backup directory
mkdir -p ${BACKUP_DIR}/{daily,weekly,monthly}

# Perform backup
pg_dump -h localhost -U postgres -d ${DATABASE} -F custom -b -v \
    -f ${BACKUP_DIR}/daily/${DATABASE}_${TIMESTAMP}.dump

# Compress older backups
find ${BACKUP_DIR}/daily -name "*.dump" -mtime +1 -exec gzip {} \;

# Keep only last 7 daily backups
find ${BACKUP_DIR}/daily -name "*.dump.gz" -mtime +7 -delete

# Weekly backup (on Sunday)
if [ $(date +%w) -eq 0 ]; then
    cp ${BACKUP_DIR}/daily/${DATABASE}_${TIMESTAMP}.dump \
       ${BACKUP_DIR}/weekly/
fi

# Monthly backup (on 1st)
if [ $(date +%d) -eq 01 ]; then
    cp ${BACKUP_DIR}/daily/${DATABASE}_${TIMESTAMP}.dump \
       ${BACKUP_DIR}/monthly/
fi

# Log backup status
echo "Backup completed: ${TIMESTAMP}" >> ${BACKUP_DIR}/backup.log
```

## 6. Troubleshooting

### Common Issues

1. **Connection refused**:
```bash
# Check if PostgreSQL is running
sudo systemctl status postgresql
ps aux | grep postgres

# Check if PostgreSQL is listening
sudo netstat -tlnp | grep 5432
sudo ss -tlnp | grep 5432

# Check PostgreSQL logs
sudo tail -f /var/lib/pgsql/16/data/log/postgresql-*.log
```

2. **Authentication failed**:
```bash
# Check pg_hba.conf settings
sudo cat /var/lib/pgsql/16/data/pg_hba.conf

# Test with different authentication
psql -h localhost -U postgres -W

# Reset password if needed
sudo -u postgres psql
ALTER USER postgres PASSWORD 'NewPassword123!';
```

3. **Out of connections**:
```sql
-- Check current connections
SELECT count(*) FROM pg_stat_activity;

-- See connection details
SELECT pid, usename, application_name, client_addr, state 
FROM pg_stat_activity;

-- Terminate idle connections
SELECT pg_terminate_backend(pid) 
FROM pg_stat_activity 
WHERE state = 'idle' 
  AND state_change < current_timestamp - interval '10 minutes';
```

4. **Slow queries**:
```sql
-- Enable slow query logging
ALTER SYSTEM SET log_min_duration_statement = 1000; -- Log queries taking > 1s
SELECT pg_reload_conf();

-- Find slow queries
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
WHERE mean_exec_time > 1000
ORDER BY mean_exec_time DESC;

-- Check for missing indexes
SELECT 
    schemaname,
    tablename,
    attname,
    n_distinct,
    most_common_vals
FROM pg_stats
WHERE tablename NOT LIKE 'pg_%'
ORDER BY n_distinct DESC;
```

### Performance Issues

```bash
# Check system resources
top -u postgres
iostat -x 1
vmstat 1

# Check PostgreSQL cache hit ratio
psql -c "SELECT sum(blks_hit)*100.0/sum(blks_hit+blks_read) AS hit_ratio FROM pg_stat_database;"

# Find bloated tables
psql -c "SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;"

# Rebuild bloated indexes
REINDEX TABLE bloated_table;
REINDEX DATABASE mydb;
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf check-update postgresql16*
sudo dnf update postgresql16*

# Debian/Ubuntu
sudo apt update
sudo apt upgrade postgresql-16

# Arch Linux
sudo pacman -Syu postgresql

# Alpine Linux
apk update
apk upgrade postgresql

# openSUSE
sudo zypper update postgresql16

# FreeBSD
pkg update
pkg upgrade postgresql16-server

# Always restart after updates
sudo systemctl restart postgresql
```

### Version Upgrade

```bash
# Backup before upgrade
pg_dumpall > backup_before_upgrade.sql

# Install new version (example: upgrading to 17)
sudo dnf install postgresql17-server

# Initialize new cluster
sudo /usr/pgsql-17/bin/postgresql-17-setup initdb

# Stop both versions
sudo systemctl stop postgresql-16
sudo systemctl stop postgresql-17

# Run pg_upgrade
sudo -u postgres /usr/pgsql-17/bin/pg_upgrade \
  -d /var/lib/pgsql/16/data \
  -D /var/lib/pgsql/17/data \
  -b /usr/pgsql-16/bin \
  -B /usr/pgsql-17/bin

# Start new version
sudo systemctl start postgresql-17
sudo systemctl enable postgresql-17
```

### Regular Maintenance Tasks

```bash
# Create maintenance script
cat > /usr/local/bin/pg_maintenance.sh << 'EOF'
#!/bin/bash

# Update table statistics
psql -U postgres -d mydb -c "ANALYZE;"

# Reindex databases
psql -U postgres -d mydb -c "REINDEX DATABASE mydb;"

# Clean up old logs
find /var/lib/pgsql/16/data/log -name "*.log" -mtime +30 -delete

# Report database sizes
psql -U postgres -c "SELECT datname, pg_size_pretty(pg_database_size(datname)) as size FROM pg_database WHERE datistemplate = false;"

# Check for unused indexes
psql -U postgres -d mydb -c "
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY schemaname, tablename;"
EOF

chmod +x /usr/local/bin/pg_maintenance.sh

# Schedule with cron
echo "0 2 * * 0 postgres /usr/local/bin/pg_maintenance.sh" | sudo tee -a /etc/cron.d/postgresql
```

## Integration Examples

### Python (psycopg2)

```python
import psycopg2
from psycopg2.extras import RealDictCursor
import logging

# Database connection
def get_db_connection():
    return psycopg2.connect(
        host="localhost",
        database="mydb",
        user="appuser",
        password="password",
        cursor_factory=RealDictCursor
    )

# Example usage
def get_users():
    conn = get_db_connection()
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT id, username, email FROM users")
            return cur.fetchall()
    finally:
        conn.close()

# Connection pool
from psycopg2 import pool

db_pool = psycopg2.pool.SimpleConnectionPool(
    1, 20,
    host="localhost",
    database="mydb",
    user="appuser",
    password="password"
)

def get_pooled_connection():
    return db_pool.getconn()

def return_pooled_connection(conn):
    db_pool.putconn(conn)
```

### Node.js (node-postgres)

```javascript
const { Pool } = require('pg');

// Create connection pool
const pool = new Pool({
  host: 'localhost',
  database: 'mydb',
  user: 'appuser',
  password: 'password',
  port: 5432,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Query example
async function getUsers() {
  const client = await pool.connect();
  try {
    const result = await client.query('SELECT * FROM users');
    return result.rows;
  } finally {
    client.release();
  }
}

// Parameterized query
async function getUserById(userId) {
  const query = 'SELECT * FROM users WHERE id = $1';
  const values = [userId];
  
  try {
    const result = await pool.query(query, values);
    return result.rows[0];
  } catch (err) {
    console.error('Database error:', err);
    throw err;
  }
}

// Transaction example
async function transferFunds(fromAccount, toAccount, amount) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    await client.query('UPDATE accounts SET balance = balance - $1 WHERE id = $2', [amount, fromAccount]);
    await client.query('UPDATE accounts SET balance = balance + $1 WHERE id = $2', [amount, toAccount]);
    await client.query('COMMIT');
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}
```

### Java (JDBC)

```java
import java.sql.*;
import javax.sql.DataSource;
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

public class PostgreSQLExample {
    private static DataSource dataSource;
    
    static {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
        config.setUsername("appuser");
        config.setPassword("password");
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);
        
        dataSource = new HikariDataSource(config);
    }
    
    public List<User> getUsers() throws SQLException {
        List<User> users = new ArrayList<>();
        String sql = "SELECT id, username, email FROM users";
        
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql);
             ResultSet rs = ps.executeQuery()) {
            
            while (rs.next()) {
                User user = new User();
                user.setId(rs.getLong("id"));
                user.setUsername(rs.getString("username"));
                user.setEmail(rs.getString("email"));
                users.add(user);
            }
        }
        return users;
    }
    
    public void updateUser(Long id, String email) throws SQLException {
        String sql = "UPDATE users SET email = ? WHERE id = ?";
        
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            
            ps.setString(1, email);
            ps.setLong(2, id);
            ps.executeUpdate();
        }
    }
}
```

### PHP (PDO)

```php
<?php
// Database configuration
$dsn = 'pgsql:host=localhost;port=5432;dbname=mydb';
$username = 'appuser';
$password = 'password';

// PDO options
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,
];

try {
    $pdo = new PDO($dsn, $username, $password, $options);
} catch (PDOException $e) {
    throw new PDOException($e->getMessage(), (int)$e->getCode());
}

// Query example
function getUsers($pdo) {
    $stmt = $pdo->query('SELECT id, username, email FROM users');
    return $stmt->fetchAll();
}

// Prepared statement
function getUserById($pdo, $userId) {
    $stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
    $stmt->execute(['id' => $userId]);
    return $stmt->fetch();
}

// Transaction example
function createUserWithProfile($pdo, $userData, $profileData) {
    try {
        $pdo->beginTransaction();
        
        // Insert user
        $stmt = $pdo->prepare('INSERT INTO users (username, email) VALUES (:username, :email) RETURNING id');
        $stmt->execute([
            'username' => $userData['username'],
            'email' => $userData['email']
        ]);
        $userId = $stmt->fetchColumn();
        
        // Insert profile
        $stmt = $pdo->prepare('INSERT INTO profiles (user_id, bio) VALUES (:user_id, :bio)');
        $stmt->execute([
            'user_id' => $userId,
            'bio' => $profileData['bio']
        ]);
        
        $pdo->commit();
        return $userId;
    } catch (Exception $e) {
        $pdo->rollBack();
        throw $e;
    }
}
?>
```

### Go (pq)

```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    _ "github.com/lib/pq"
)

const (
    host     = "localhost"
    port     = 5432
    user     = "appuser"
    password = "password"
    dbname   = "mydb"
)

func main() {
    // Connection string
    psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
        host, port, user, password, dbname)
    
    // Open database connection
    db, err := sql.Open("postgres", psqlInfo)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    // Verify connection
    err = db.Ping()
    if err != nil {
        log.Fatal(err)
    }
    
    // Query example
    rows, err := db.Query("SELECT id, username, email FROM users")
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    
    for rows.Next() {
        var id int
        var username, email string
        err = rows.Scan(&id, &username, &email)
        if err != nil {
            log.Fatal(err)
        }
        fmt.Printf("User: %d, %s, %s\n", id, username, email)
    }
    
    // Prepared statement
    stmt, err := db.Prepare("INSERT INTO users(username, email) VALUES($1, $2) RETURNING id")
    if err != nil {
        log.Fatal(err)
    }
    defer stmt.Close()
    
    var newID int
    err = stmt.QueryRow("newuser", "newuser@example.com").Scan(&newID)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("New user ID: %d\n", newID)
}
```

## Additional Resources

- [Official Documentation](https://www.postgresql.org/docs/)
- [GitHub Repository](https://github.com/postgres/postgres)
- [PostgreSQL Wiki](https://wiki.postgresql.org/)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [Performance Tuning Guide](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [Security Best Practices](https://www.postgresql.org/docs/current/security.html)
- [High Availability Guide](https://www.postgresql.org/docs/current/high-availability.html)
- [Community Forums](https://www.postgresql.org/community/forums/)

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.