# How to Add Linux Host to Nagios Monitoring Server Using NRPE Plugin


### On Remote Linux Host      r1.example.com & r2.example.com 

# Installation of NRPE Plugin
### Step 1: Install Required Dependencies

    yum install -y gcc glibc glibc-common gd gd-devel make net-snmp openssl-devel

### Step 2. Create Nagios user 

    useradd nagios 
    passwd nagios

### Step 3: Install the Nagios Plugins

Create a directory for installation and all its future downloads.

    cd /root/nagios

    sudo wget -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/releases/download/release-2.3.3/nagios-plugins-2.3.3.tar.gz


### Step 4. Extract Nagios Plugins

    tar -xvf nagios-plugins*

    ls -l 

    cd cd nagios-plugins-*

### Step 5: Compile and Install Nagios Plugins
Next, compile and install using the following commands

    ./configure 
    make 
    make install

set the permissions on the plugin directory.
    
    chown nagios.nagios /usr/local/nagios
    chown -R nagios.nagios /usr/local/nagios/libexec

### Step 6: Install Xinetd

    yum install xinetd -y

### Step 7: Install NRPE Plugin

    wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.0.3/nrpe-4.0.3.tar.gz
    
    tar xzf nrpe-4.0.3.tar.gz
    cd nrpe-4.0.3
or
     dnf --enablerepo=epel -y install nrpe nagios-plugins-{ping,disk,users,procs,load,swap,ssh}
     
    vi vim /usr/local/nagios/etc/nrpe.cfg
    allowed_hosts=127.0.0.1,::1,192.168.245.148
    dont_blame_nrpe

 line 300 : comment out all

    #command[check_users]=/usr/lib64/nagios/plugins/check_users -w 5 -c 10
    #command[check_load]=/usr/lib64/nagios/plugins/check_load -r -w .15,.10,.05 -c .30,.25,.20
    #command[check_hda1]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /dev/hda1
    #command[check_zombie_procs]=/usr/lib64/nagios/plugins/check_procs -w 5 -c 10 -sZ
    #command[check_total_procs]=/usr/lib64/nagios/plugins/check_procs -w 150 -c 200

line 305 : add follows

    command[check_users]=/usr/lib64/nagios/plugins/check_users -w $ARG1$ -c $ARG2$
    command[check_load]=/usr/lib64/nagios/plugins/check_load -w $ARG1$ -c $ARG2$
    command[check_disk]=/usr/lib64/nagios/plugins/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$
    command[check_procs]=/usr/lib64/nagios/plugins/check_procs -w $ARG1$ -c $ARG2$ -s $ARG3$

start and enable nrpe service.

    systemctl enable --now nrpe
    systemctl restart nrpe 



Compile and install the NRPE addon.

    ./configure
    make all
    make install-plugin
    make install-daemon
    make install-daemon-config
    make install-xinetd
    make install-inted

Now open **/etc/xinetd.d/nrpe** file and add the localhost and IP address of the Nagios Monitoring Server.

    only_from = 127.0.0.1 localhost 192.168.245.148

Next, open **/etc/services** file add the following entry for the NRPE daemon at the bottom of the file.

    nrpe            5666/tcp                 NRPE

Restart the xinetd service.

    service xinetd restart
    or
    systemctl restart xinetd

### Step 8: Verify NRPE Daemon Locally

    netstat -at | grep nrpe

Next, verify the NRPE daemon is functioning properly. Run the “check_nrpe” command that was installed earlier for testing purposes.

     /usr/local/nagios/libexec/check_nrpe -H localhost

You will get a following string on the screen, it shows you what version of NRPE is installed:

NRPE v3.2



# On Nagios Server

Installing nagios-plugins-

    dnf --enablerepo=epel search nagios-plugins-

    dnf --enablerepo=epel -y install nagios-plugins-ntp
    
    vim /usr/local/nagios/etc/nagios.cfg

uncomment the directory server

    cfg_dir=/usr/local/nagios/etc/servers

creating NTP_TIME server 

    vim /usr/local/nagios/etc/objects/localhost.cfg 
    
    # add the following text 
    define service {
    use                     local-service
    host_name               localhost
    service_description     NTP_TIME
    check_command           check_ntp_time!ntp.nict.jp!1!2
    notifications_enabled   1
    }

Creating new directory 

    mkdir /usr/local/nagios/etc/servers
    chgrp nagios /usr/local/nagios/etc/servers
    chmod 750 /usr/local/nagios/etc/servers
    cd /usr/local/nagios/etc/servers
    
Adding host for r1.example.com

    vim r1.cfg 
    
    # create new file and add text
    define host {
        use                    linux-server
        host_name              r1.example.com
        alias                  r1.example.com
        address                192.168.245.145
    }

    # for ping
    define service {
        use                    generic-service
        host_name              r1.example.com
        service_description    PING
        check_command          check_ping!100.0,20%!500.0,60%
    }

    # for free disk
    define service {
        use                    generic-service
        host_name              r1.example.com
        service_description    Root Partition
        check_command          check_nrpe!check_disk\!20%\!10%\!/
    }

    # for current users
    define service {
        use                    generic-service
        host_name              r1.example.com
        service_description    Current Users
        check_command          check_nrpe!check_users\!20\!50
    }

    # for total processes
    define service {
        use                    generic-service
        host_name              r1.example.com
        service_description    Total Processes
        check_command          check_nrpe!check_procs\!250\!400\!RSZDT
        }

    # for current load
    define service {
        use                    generic-service
        host_name              r1.example.com
        service_description    Current Load
        check_command          check_nrpe!check_load\!5.0,4.0,3.0\!10.0,6.0,4.0
    }

adding host for r2.example.com 

    vim r2.cfg 
    
    # create new
    define host {
        use                    linux-server
        host_name              r2.example.com
        alias                  r2.example.com
        address                192.168.245.139
    }

    # for ping
    define service {
        use                    generic-service
        host_name              r2.example.com
        service_description    PING
        check_command          check_ping!100.0,20%!500.0,60%
    }

    # for free disk
    define service {
        use                    generic-service
        host_name              r2.example.com
        service_description    Root Partition
        check_command          check_nrpe!check_disk\!20%\!10%\!/
    }

    # for current users
    define service {
        use                    generic-service
        host_name              r2.example.com
        service_description    Current Users
        check_command          check_nrpe!check_users\!20\!50
    }

    # for total processes
    define service {
        use                    generic-service
        host_name              r2.example.com
        service_description    Total Processes
        check_command          check_nrpe!check_procs\!250\!400\!RSZDT
    }

    # for current load
    define service {
        use                    generic-service
        host_name              r2.example.com
        service_description    Current Load
        check_command          check_nrpe!check_load\!5.0,4.0,3.0\!10.0,6.0,4.0
    }

start and enable nagios service. 

    systemctl restart nagios
