
# Self-Hosted PostgreSQL Database on Azure Virtual Machine

## Project Overview

# Approach 1: Self-Hosted PostgreSQL on Ubuntu Azure Virtual Machine

## Overview

This approach demonstrates how to deploy a self-hosted PostgreSQL database on an Ubuntu Virtual Machine (VM) hosted in Microsoft Azure. Unlike a managed database service, this method provides complete control over the operating system, PostgreSQL installation, configuration, security, and maintenance.

The PostgreSQL server is installed directly on the Ubuntu VM and configured for secure remote access. Database administration is performed remotely using pgAdmin installed on a local Windows computer.

This approach is suitable for organizations that require full administrative control, custom database configurations, regulatory compliance, or migration of existing on-premises workloads to Azure.


# Installation and Configuration Steps

## Step 1

Create an Ubuntu Virtual Machine in Microsoft Azure.


## Step 2: Connect to the VM via SSH.
ssh azureuser@<Public-IP>


## Step 3:Update Ubuntu packages.

sudo apt update
sudo apt upgrade -y


## Step 4: Install PostgreSQL.

sudo apt install postgresql postgresql-contrib -y


## Step 5:Verify PostgreSQL installation.

sudo systemctl status postgresql

pg_lsclusters


## Step 6: Verify PostgreSQL is listening.

sudo ss -ltnp | grep 5432


## Step 7: Edit PostgreSQL configuration.

sudo nano /etc/postgresql/16/main/postgresql.conf


#### Change from

#listen_addresses = 'localhost'

to

listen_addresses='*'

then

Restart PostgreSQL.

sudo systemctl restart postgresql


## Step 8: Configure client authentication.

sudo nano /etc/postgresql/16/main/pg_hba.conf

Add

host    all    all    0.0.0.0/0    scram-sha-256
host    all    all    ::/0         scram-sha-256


###Step 8: Restart PostgreSQL.


sudo systemctl restart postgresql


## Step 9: Create a password for the postgres user.


sudo -u postgres psql


ALTER USER postgres WITH PASSWORD 'Password123!';


Exit PostgreSQL.

\q


## Step 10: Configure Azure Network Security Group.

Allow inbound TCP port: 5432


## Step 11: Install pgAdmin 4 on Windows.

Create a new server.

| Property | Value |
|----------|-------|
| Host | VM Public IP |
| Port | 5432 |
| Username | postgres |
| Database | postgres |
| Password | Password123! |



##### Approach 2: PostgreSQL Installation on Local Windows System

## Overview

This approach demonstrates how to install and configure PostgreSQL directly on a Windows computer. The PostgreSQL database server and pgAdmin are installed locally, allowing developers and database administrators to create, manage, and test databases without requiring cloud infrastructure.


# Installation Steps

## Step 1: Download PostgreSQL

Download the latest PostgreSQL installer for Windows from the official PostgreSQL website.

https://www.postgresql.org/download/windows/


## Step 2: Run the Installer

1. Double-click the downloaded installer.
2. Run the installer as **Administrator**.
3. Click **Next** to begin the installation.


## Step 3: Select Installation Directory

Choose the installation location or leave the default directory.

Example:

C:\Program Files\PostgreSQL\16

Click **Next**.



## Step 4: Select Components

Select the components to install.

- PostgreSQL Server
- pgAdmin 4
- Command Line Tools
- Stack Builder (Optional)

Click **Next**.


## Step 5: Select Data Directory

Choose the location where PostgreSQL will store database files.

Example:

C:\Program Files\PostgreSQL\16\data
 
Click **Next**.


## Step 6: Configure the Database Superuser

Create the password for the default PostgreSQL administrator.

| Setting | Value |
|---------|-------|
| Username | postgres |
| Password | Create a secure password |

Click **Next**.


## Step 7: Configure the Server Port

Leave the default PostgreSQL port.
5432

Click **Next**.


## Step 8: Select Locale

Leave the default locale or select your preferred locale.

Click **Next**.


## Step 9: Install PostgreSQL

Review the installation summary.

Click **Next**, then **Install**.

Wait for the installation to complete.


## Step 10: Complete the Installation

Click **Finish**.

PostgreSQL Server and pgAdmin will now be installed.


###Step 11: Verify the Installation

Open **Windows PowerShell** as Administrator.

### Step 12:  Check the PostgreSQL Version

psql --version


## Step 13:  Verify the PostgreSQL Service

Get-Service *postgres*

Expected Output

Status   Name                DisplayName
------   ----                -----------
Running  postgresql-x64-16   PostgreSQL Server 16


### Step 14: Start the PostgreSQL Service

Start-Service postgresql-x64-16


### Step 15: Stop the PostgreSQL Service

Stop-Service postgresql-x64-16


### Step 16:  Restart the PostgreSQL Service

Restart-Service postgresql-x64-16


### Step 17: Verify PostgreSQL is Listening on Port 5432

netstat -ano | findstr 5432

### Step 18: Test Local Connectivity

Test-NetConnection localhost -Port 5432

Expected Output

TcpTestSucceeded: True

### Step 19: Connect to PostgreSQL

Open PowerShell.

If PostgreSQL has been added to the system PATH:

psql -U postgres

If PostgreSQL is not in the system PATH:

"C:\Program Files\PostgreSQL\16\bin\psql.exe" -U postgres

Enter the password created during installation.





##### Create Products Table

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    unit_price DECIMAL(10,2),
    quantity INT,
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


#### Create Customers Table

CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    phone VARCHAR(20)
);


#### Create Orders Table

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    product_id INT REFERENCES products(product_id),
    quantity INT,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


#### Insert Products

INSERT INTO products(product_name,category,unit_price,quantity)
VALUES
('Dell Latitude 7450','Laptop',1500,25),
('HP EliteBook','Laptop',1300,20),
('Logitech Mouse','Accessory',35,200),
('Mechanical Keyboard','Accessory',80,100),
('USB-C Dock','Accessory',120,50);

#### Insert Customers

INSERT INTO customers(customer_name,email,phone)
VALUES
('John Smith','john@test.com','08011111111'),
('Mary Johnson','mary@test.com','08022222222'),
('David Brown','david@test.com','08033333333');

#### Insert Orders

INSERT INTO orders(customer_id,product_id,quantity)
VALUES
(1,1,2),
(2,3,5),
(3,2,1),
(1,5,3);


#### Sample Query

Retrieve all products.

SELECT * FROM products;

Generate an inventory sales report.

SELECT
    c.customer_name,
    p.product_name,
    o.quantity,
    p.unit_price,
    (o.quantity * p.unit_price) AS total_amount,
    o.order_date
FROM orders o
JOIN customers c
ON o.customer_id = c.customer_id
JOIN products p
ON o.product_id = p.product_id;


#### Verification Commands

Check PostgreSQL status.

sudo systemctl status postgresql

####Check clusters.
pg_lsclusters

####Check listening port.

sudo ss -ltnp | grep 5432

###Check PostgreSQL configuration.

sudo -u postgres psql -c "SHOW listen_addresses;"

####Test connectivity from Windows.

### use powershell
Test-NetConnection <VM-Public-IP> -Port 5432


