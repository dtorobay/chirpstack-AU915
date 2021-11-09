# How to install and connect Chirpstack on Ubuntu. 

To install Chirpstack (Gateway Bridge, Network Server and Application Server) on Ubuntu you need to follow some steps. You can follow the steps on Chirpstack page: https://www.chirpstack.io/project/guides/debian-ubuntu/

In this guide I´ll show some tricks to install with no big problems.

## First step:
Install Ubuntu on a Linux Virtual Machine and download a Ubuntu version (20.04 for example)

With Virtual Box installed click on New.
 Give a name to this VM;
 Type: Linux;
 Version: Ubuntu 64bit – (To a ISO of 64 bits);
 
 Memory size:
  Minimum of 4GB (or 4096 bytes);
 
 Hard Drive:
  Create a new virtual hard drive now;
 
 Create a new virtual hard drive
  Archive size:
    80GB minimum for prototype;
 
 Hard disk file type:
   VDI (VirtualBox Disk Image);
 
 Physical hard disk storage:
   Dynamically allocated;
 
 Settings:
  Monitor:
   Video Memory: 128MB
  
 Start
  Select boot hard disk (select ISO file saved on PC);
 
 Ubuntu Installation Start:
  Language: Brazilian Portuguese;
  Install Ubuntu;
  
 Keyboard layout: Portuguese (Brazil);
 
 Select:
  Normal installation;
  Download updates while installing Ubuntu;
  Install third-party software for graphics hardware and Wi-Fi and media format additional; 
    
  Installation Type:
   Advanced option;
   
   New partition table;
   Select 'free space' and click on +;
   Create Partition:
    Size: 1024MB;
    Type for the new partition: Primary;
    Location for new partition: Start of this space;
    Use as: Swap area;
    
   Select 'free space' again and click on +;
   Create Partition:
    Size: the rest of the memory;
    Type for the new partition: Primary;
    Location for new partition: Start of this space;
    Use as: Filesystems with “jounaling ext4”;
    Assembly point: /;
   
   Keep the ext4 partition selected and click ‘Install now’;
 
  Time Zone: São Paulo;
 
  Inform:
   Username;
   Computer name;
   Password;
   Select ‘Login automatically’;
 
When finished, restart the virtual machine.

# Chirpstack Server Installation

## Chirpstack Gateway Bridge

On command line on Linux, let´s install the MQTT Broker:

sudo apt install mosquitto

Activation of Debian repository:

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00

sudo echo "deb https://artifacts.chirpstack.io/packages/3.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list

sudo apt update

Chirpstack Gateway Bridge install:

sudo apt install chirpstack-gateway-bridge

To confirm the right installation:

sudo systemctl restart chirpstack-gateway-bridge

sudo systemctl status chirpstack-gateway-bridge

CTRL+C to exit return to the command line.

## Chirpstack Network Server

Installing PostgreSQL database:

sudo apt install postgresql

Redis database (Redis store device session data and non-persistent data):

sudo apt install redis-server

Database and user creation (these values could be changed):

sudo -u postgres psql

-- create the chirpstack_ns user with password 'dbpassword' 
create role chirpstack_ns with login password 'dbpassword'; 
 
-- create the chirpstack_ns database 
create database chirpstack_ns with owner chirpstack_ns; 
 
-- exit the prompt
\q

To check the correct configuration:

psql -h localhost -U chirpstack_ns -W chirpstack_ns

Insert _dbpassword_ and exit the database with _\q_.

To activate the Debian archive from ChirpStack Network Server:

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00

sudo echo "deb https://artifacts.chirpstack.io/packages/3.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list

sudo apt update

### Installing of Chipstack Network Server:

sudo apt install chirpstack-network-server

To change the chirpstack-network-server.toml file:

sudo -s

cd /etc/chirpstack-network-server/

Through the nano text editor:
nano chirpstack-network-server.toml

Inside nano, comment with # the line starting with the dsn parameter:

Write on the next line:

dsn="postgres://chirpstack_ns:dbpassword@localhost/chirpstack_ns?sslmode=disable"

The frequencies to Brazil region has to change to AU915 parameters. The right file to edit correctly is attached: [chirpstack-network-server-AU915.toml.txt](https://github.com/dtorobay/chirpstack-AU915/files/7505725/chirpstack-network-server-AU915.toml.txt)

Save changes with Ctrl + X and exit nano.

To give full permission to this file, white on the root command line:

sudo chmod 777 chirpstack-network-server.toml

To confirm the installation was done correctly:

At the terminal:
cd /

sudo systemctl restart chirpstack-network-server

sudo systemctl status chirpstack-network-server

Ctrl+C to return to command line.

## Chirpstack Application Server

Chipstack Application Server Database

Start PostgreSQL Prompt:

sudo -u postgres psql

Within the prompt:

-- create the chirpstack_as user
create role chirpstack_as with login password 'dbpassword';
 
-- create the chirpstack_as database
create database chirpstack_as with owner chirpstack_as;
 
-- enable the trigram and hstore extensions
\c chirpstack_as
create extension pg_trgm;
create extension hstore;
 
-- exit the prompt
\q
 
To verify that the user and database were correctly configured:

psql -h localhost -U chirpstack_as -W chirpstack_as

Enter the password _dbpassword_ and exit the database with _\q_.

To activate the ChirpStack Application Server Debian repository:

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00
 
sudo echo "deb https://artifacts.chirpstack.io/packages/3.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list

sudo apt-get update

### Installing the Chipstack Application Server

sudo apt install chirpstack-application-server

To change the chirpstack-application-server.toml file:

sudo -s (if not in root mode)

cd /etc/chirpstack-application-server/

Through the nano text editor:

nano chirpstack-application-server.toml

Inside the nano, comment with # the line:

dsn="postgres://localhost/chirpstack_as?sslmode=disable"

Write on the next line:

dsn="postgres://chirpstack_as:dbpassword@localhost/chirpstack_as?sslmode=disable"

Save changes with Ctrl + X and exit nano.

At the terminal:

openssl rand -base64 32

Copy the generated key.

Go back to nano and write the key inside jwt_secret=" "

Save changes and exit nano again.

To give full permission to this file, white on the root command line:

sudo chmod 777 chirpstack-application-server.toml

To confirm the installation was done correctly:

At the terminal:

cd /

sudo systemctl restart chirpstack-application-server

sudo systemctl status chirpstack-application-server

Ctrl+C to returno to command line.

exit the terminal

Chirpstack server installation completed. To open the server, go to the machine's browser and type in the address bar localhost:8080/.

Default login and password:

Login: admin
Password: admin

##To access the server from another physical or virtual machine:

With the server virtual machine turned off, go to Settings in Virtual Box. In the section Network, change 'Connected to NAT' to 'Connected to Board in Bridge mode'.
To access the server IP to access it from another machine, just type the command
ifconfig in the terminal.
