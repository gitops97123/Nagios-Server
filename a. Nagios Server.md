# How To Install and Configure Nagios on CentOS | Rocky Linux | RHEL 7/8 

# Prerequisites

- Firewall Disabled 
- SeLinux Disabled
- A non-sudo user with root privileges.
- Another Server running RockyLinux 8 that you want to monitor.
- Ensure that everything is updated.
  
# Lab Setup

#### In my Lab Setup i am using 3 nodes. 

---
Hostname  : **rl-work.example.com**  
IPAddress : **192.168.245.145** 

---
Hostname : **r1.example.com**<br>
IPAddress: **192.168.245.142**

---
Hostname : **r2.example.com**<br>
IPAddress: **192.168.245.139**

---

# Creating Nagios Server on  rl-work.example.com  

## Step 1 - Disable Firewall
permforming the activity by using root user. 

    firwall-cmd --state 
    systemctl disable --now firewalld 

## Step 2 - Disable SELinux
disable the selinux. 

    setenforce 0 
    vim /etc/selinux/config 
    SELINUX=disabled 

Reboot the system. 

    systemctl reboot 

## Step 3 - Install Apache and PHP Packages 

    dnf install httpd -y 
    dnf systemctl enable --now httpd 

Installing Remi Repository for Installing PHP v7.4 

    dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

Revoking the previous version of php. 

    sudo dnf module reset php

Now, Installing the latest php v7.4 

    sudo dnf module enable php:remi-7.4

Installing PHP and Dependencies
    
    dnf install php php-gd php-curl

Restart and Enable the httpd service 

    systemctl restart httpd 

Creating info.php for ensuring the webserver is running.

    vim /var/www/html/info.php
    >?php phpinfo();

save the file by pressing **esc** and entering **:x** when prompted.

Open the URL [http://192.168.245.148/info.php](http://192.168.245.148/info.php) in your favourite browser and you should see the ouput. 

## Step 4 - Install Nagios Server 

Installing dependencies. 

    sudo dnf install gcc glibc glibc-common gd gd-devel make net-snmp openssl-devel xinetd unzip wget gettext autoconf net-snmp-utils epel-release postfix automake

Enabling the Repository and Install Required Packages. 
    
    sudo dnf config-manager --enable powertools 
    sudo dnf install perl-Net-SNMP

Download Nagios 

Create the "/usr/src" directory & change the directory.  

    cd /usr/src

Download the latest version of Nagios from its Github page. At the time of the tutorial, 4.4.6 is the latest version available. Modify the command in case you want a different version.

    wget https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.6.tar.gz

Extract the tar file.

    tar zxf nagios-*.tar.gz

Switch to the Nagios source directory.

    cd nagioscore-nagios-*/

Compile Nagios

The next step is to compile Nagios from its source files. Run the configure script to perform checks to make sure all dependencies are present.

    ./configure

Start the compilation. 

    make all 

Create Nagios User and Group

    make install-group-users
    usermod -aG nagios apache 

Install Nagios Binaries and Run the following commands.

    make install 
    make install-commandmode
    make install-config
    make install-webconf

Restart the webserver to activate the configuration.

    systemc restart httpd

Create a Systemd Service File.

    make install-daemoninit

Enable HTTP Authentication 

    htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin 
    New password: redhat
    Re-type new Password: redhat
    Adding password for user nagiosadmin

Restart the server for the configuration to take effect. 

    systemctl restart httpd

# Step 5 - Install Nagios Plugins 

Install the prerequisites required for the Nagios plugins. 

    dnf install -y gcc glibc glibc-common make gettext automake autoconf wget openssl-devel net-snmp net-snmp-utils epel-release postgresql-devel libdbi-devel openldap-devel mysql-devel mysql-libs bind-utils samba-client fping openssh-clients lm_sensors

    dnf config-manager --enable powertools
    dnf install -y perl-Net-SNMP

switch back to the /usr/src directory. 

    cd /usr/src 

Download the latest version of Nagios from its Github page. 

    wget -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/releases/download/release-2.3.3/nagios-plugins-2.3.3.tar.gz

Extract the tar file.

    tar zxf nagios-plugins.tar.gz

change to the plugins directory. 

    cd nagios-plugins-*

Run the following commands in order to compile and install the plugins. 

    ./configure 
    make 
    make install 

# Step 6 - Install check_nrpe Plugin 

switch to the /usr/src directory. 

    cd /usr/src 

Download the latest version of NRPE from its Github Page. 

    wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.0.3/nrpe-4.0.3.tar.gz

Extract the archive. 

    tar zxf nrpe-*.tar.gz

Change to the NRPE directory. 

    cd nrpe-* 

Configure and Install the plugins. 

    ./configure
    make check_nrpe
    make install-plugin

# Step 7 - Start Nagios

    systemctl restart nagios 
    systemctl status nagios 

Nagios Web Interface

Open the URL http://192.168.245.148/nagios in your web browser. 

![Nagios Server](https://github.com/gitops97123/Nagios-Server/blob/main/.icons/nagios-server.png?raw=true)

