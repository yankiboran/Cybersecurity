# Introduction to the AWS EC2 UserScripts
Let's take a look at launching an EC2 Instance with the UserScript functionality to build a CTF Challenge. UserScripts helps us to initialize the EC2 Instance with pre-defined configurations.  

**UserScript defined below:**  
- Installs necessary packages.  
- Creates hidden files or services containing flags.  
- Sets up environment configurations for challenges.  

**Understanding User Data Scripts**  
When you launch an EC2 instance, you can pass a script via the User Data field. This script runs with root privileges during the first boot cycle of the instance.  

**Format:** The script can be written in bash, PowerShell, or any other scripting language supported by the instance's OS.  
**Encoding:** Ensure the script is in plain text. For Windows instances, scripts should be Base64-encoded.  

```bash
#!/bin/bash

# Update package lists
apt-get update -y

# Install necessary packages (example: Apache2)
apt-get install apache2 -y

# Start and enable Apache service
systemctl start apache2
systemctl enable apache2

# Create a sample web page with a flag
echo "<html><body><h1>Welcome to the CTF Challenge!</h1></body></html>" > /var/www/html/index.html

# Place a flag in the web root (simple web challenge)
echo "FLAG{web_root_flag_example}" > /var/www/html/flag.txt

# Create a hidden directory with a flag (filesystem exploration challenge)
mkdir /var/hidden_dir
echo "FLAG{hidden_directory_flag}" > /var/hidden_dir/.hidden_flag.txt

# Set permissions to make it less obvious
chmod 700 /var/hidden_dir
chmod 600 /var/hidden_dir/.hidden_flag.txt

# Install an SSH backdoor (reverse shell challenge)
# Note: For ethical reasons, we won't set up an actual backdoor.
# Instead, we'll simulate a service that could be exploited.

# Create a user with a weak password (password cracking challenge)
useradd -m ctfuser
echo "ctfuser:password123" | chpasswd

# Install an outdated package with known vulnerabilities (exploitation challenge)
# For demonstration, we'll install an older version of vsftpd
apt-get install -y vsftpd=3.0.2-1ubuntu2

# Configure vsftpd
echo "listen=YES" >> /etc/vsftpd.conf
echo "anonymous_enable=YES" >> /etc/vsftpd.conf
echo "local_enable=YES" >> /etc/vsftpd.conf
systemctl restart vsftpd

# Add a flag in the FTP directory
echo "FLAG{ftp_service_flag}" > /var/ftp/flag.txt

# Set up a cron job that runs a script (privilege escalation challenge)
echo '* * * * * root /usr/local/bin/cron_script.sh' >> /etc/crontab

# Create the script with incorrect permissions
echo -e '#!/bin/bash\necho "FLAG{cron_job_flag}" > /tmp/cron_flag.txt' > /usr/local/bin/cron_script.sh
chmod 777 /usr/local/bin/cron_script.sh

# Install necessary tools for participants (optional)
apt-get install -y net-tools nmap

# Final message
echo "CTF environment setup complete."

```
