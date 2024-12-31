<h1 align="center"><strong>Activating FTP and SFTP on Debian</strong></h1>

<!-- <p align="center">
  <img src="screenshot.jpg" alt="Screenshot">
</p> -->

<p align="center">
  <a href="https://youtu.be/nSJHj15-Ltg">
    <img src="https://img.youtube.com/vi/nSJHj15-Ltg/0.jpg" alt="Youtube Video">
  </a>
</p>

<p align="center">
  <a href="https://youtu.be/nSJHj15-Ltg">How to Activate FTP and SFTP on Debian</a>
</p>
This comprehensive guide explains how to set up an FTP server on a Debian-based home server using VSFTPD, configure it for secure access, and manage firewall and router settings for proper port forwarding. Additionally, it includes detailed instructions for configuring SFTP with OpenSSH, enabling secure file transfers as an alternative to FTP. The guide also covers creating users, managing permissions, and setting up multiple access permissions for shared directories.

## Step 1: Install VSFTPD

### Update Package List
Open your terminal and run the following commands to update the package list:

```bash
sudo apt update
sudo apt install vsftpd
```

### Check Service Status
After installation, ensure that the VSFTPD service is running:

```bash
sudo systemctl status vsftpd
```

## Step 2: Configure VSFTPD

### Edit Configuration File
Open the VSFTPD configuration file in a text editor:

```bash
sudo nano /etc/vsftpd.conf
```

### Basic Configuration Changes
Modify or add the following lines to enhance security and functionality:

```conf
listen_port=2121 # For ASUS router in case of conflict with ports 20 and 21
pasv_address=217.197.181.84 # Your router public IP address
pasv_enable=YES
pasv_min_port=1024  # Example range; adjust as needed
pasv_max_port=1048  # Example range; adjust as needed
anonymous_enable=NO        # Disable anonymous login
local_enable=YES           # Allow local users to log in
write_enable=YES           # Allow writing to the server
chroot_local_user=YES      # Jail users in their home directories
```

### Save Changes
Save the changes in nano by pressing `CTRL + X`, then `Y`, and `Enter`.

### Restart VSFTPD
Restart the service to apply your changes:

```bash
sudo systemctl restart vsftpd
```

## Step 3: Configure Firewall

### Allow FTP Traffic
If you are using UFW (Uncomplicated Firewall), you need to allow FTP traffic:

```bash
sudo ufw allow from any to any port 21 proto tcp  # Use 2121 for ASUS routers in case of port conflict
sudo ufw allow from any to any port 1024:1048 proto tcp  # For passive mode
```

### Reload Firewall
Reload the firewall settings:

```bash
sudo ufw reload
```

## Step 4: Create FTP User

### Add a New User
To create a new user for FTP access, use the following command:

```bash
sudo adduser ftpuser  # Replace 'ftpuser' with your desired username
```

### Set User Directory (Optional)
You can specify a directory for this user if needed. For example, create a directory under `/home`:

```bash
sudo mkdir /home/ftpuser/ftp_directory
sudo chown ftpuser:ftpuser /home/ftpuser/ftp_directory  # Set ownership to the user
```

### Configure Port Forwarding in the Router

#### For the FTP Port 20, 21, or 2121
- **Service Name**: Your preferred name
- **Protocol**: TCP
- **External Port**: 21 (2121 for ASUS routers in case of port conflict)
- **Internal Port**: Same as external or leave it empty for ASUS routers
- **Internal IP Address**: The home server's internal IP address (check it with `ifconfig`)
- **Source IP**: Leave empty

#### For Passive Ports 1024:1048
- **Service Name**: Your preferred name
- **Protocol**: TCP
- **External Port**: 1024:1048 (syntax for range in ASUS routers)
- **Internal Port**: Same as external or leave it empty for ASUS routers
- **Internal IP Address**: The home server's internal IP address (check it with `ifconfig`)
- **Source IP**: Leave empty

## Step 5: Test Your FTP Server

### Using Total Commander
Create a new FTP connection:
- **Host name[:Port]**: Your public IP or domain redirected to your public IP:2121 (or your defined port)
- **User name**: Your FTP username
- **Password**: Your FTP user's password

### Add Write Permissions
Add write permissions to the folder you want to read and write to by the FTP user:

```bash
sudo chmod 777 /var/www/html
```

### Access the Server Using FTP User from Command Line

```bash
sftp ftpuser@192.168.1.10
```

# Activating SFTP on Debian for n8n FTP Node

## Steps to Set Up SFTP

### Step 1: Install OpenSSH Server
If you haven't already installed the OpenSSH server, you can do so with the following commands:

```bash
sudo apt update
sudo apt install openssh-server
```

After installation, ensure the SSH service is running:

```bash
sudo systemctl status ssh
```

### Step 2: Configure SSH for SFTP

You need to edit the SSH daemon configuration file to set up SFTP:

```bash
sudo nano /etc/ssh/sshd_config
```

Add or modify the following lines at the end of the file:

```conf
# Subsystem      sftp    /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp
Port 2122 # The default is 22 but can be changed

# Restrict access to a specific group (optional)
Match Group sftpusers
    ChrootDirectory /home/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

### Step 3: Create a Dedicated User and Group for SFTP

#### Create a Group for SFTP Users

```bash
sudo groupadd sftpusers
```

#### Create a New User and Add Them to the Group

```bash
sudo useradd -m -g sftpusers -s /bin/false sftpuser
sudo passwd sftpuser
```

### Step 4: Set Up Directory Structure and Permissions

Create a directory structure for your SFTP user:

```bash
sudo mkdir -p /home/sftpuser/files
sudo chown root:sftpusers /home/sftpuser
sudo chmod 755 /home/sftpuser
sudo chown sftpuser:sftpusers /home/sftpuser/files
```

### Step 5: Restart SSH Service
After making changes to the configuration file, restart the SSH service to apply them:

```bash
sudo systemctl restart ssh
```

### Step 6: Adjust UFW Firewall Settings
If you are using UFW, ensure that SSH traffic is allowed:

```bash
sudo ufw allow OpenSSH
```

Check the status of UFW:

```bash
sudo ufw status
```

### Step 7: Configure Port Forwarding in the Router

#### For the FTP Port 22, or 2122
- **Service Name**: Your preferred name
- **Protocol**: TCP
- **External Port**: 22 (2122 for ASUS routers in case of port conflict)
- **Internal Port**: Same as external or leave it empty for ASUS routers
- **Internal IP Address**: The home server's internal IP address (check it with `ifconfig`)
- **Source IP**: Leave empty

### Step 8: Test Your SFTP Connection
From your local machine, test your SFTP connection:

```bash
sftp sftpuser@your_debian_server_ip
```

Replace `your_debian_server_ip` with your actual server's IP address. If everything is configured correctly, you should be logged into an SFTP session.

# How to Set Up Multiple Access Permissions

## Using User Groups

One common method is to create a group and add users to that group. This allows multiple users to share access to a directory.

### Steps

#### Create a Group
Create a new group (e.g., `sharedgroup`):

```bash
sudo groupadd sharedgroup
```

#### Add Users to the Group
Add the desired users to this group:

```bash
sudo usermod -aG sharedgroup username1
sudo usermod -aG sharedgroup username2
```

#### Change the Group Ownership of the Directory
Change the group ownership of the target directory:

```bash
sudo chgrp sharedgroup /path/to/directory
```

#### Set Permissions for the Group
Grant read and write permissions to the group:

```bash
sudo chmod 770 /path/to/directory  # rwx for owner and group, no permissions for others
```

To grant read access to the directory for other users:

```bash
sudo chmod -R a+rX /path/to/directory
