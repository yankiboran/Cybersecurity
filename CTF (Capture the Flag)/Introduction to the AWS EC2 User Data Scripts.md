# Introduction to the AWS EC2 User Data Scripts for creating a CTF Environment #
Let's take a look at launching an EC2 Instance with the User Data Scripts functionality to build a CTF Challenge. User Data Scriptss helps us to initialize the EC2 Instance with pre-defined configurations.  

**User Data Scripts defined below:**  
- Installs necessary packages.  
- Creates hidden files or services containing flags.  
- Sets up environment configurations for challenges.  

**Understanding User Data Scripts:**  
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
## Explanation of the Script Components: ##
  
**Update and Package Installation**  
* Updates the package lists and installs Apache2, vsftpd, and utilities like ```net-tools``` and ```nmap.```

**Web Challenge**
* Creates a simple web page.
* Places a flag in ```/var/www/html/flag.txt.```

**Filesystem Exploration Challenge**
* Creates a hidden directory ```/var/hidden_dir```.
* Places a hidden flag ```.hidden_flag.txt``` inside.
* Adjusts permissions to make discovery more challenging.

**User with Weak Password**
* Creates a user ```ctfuser``` with the password ```password123```.
* Participants can practice password cracking techniques.

**Outdated Vulnerable Service**
* Installs an older version of ```vsftpd``` with known vulnerabilities.
* Configures it to allow anonymous access.
* Places a flag in the FTP directory.

**Privilege Escalation Challenge**
* Sets up a cron job that runs a script every minute.
* The script ```cron_script.sh``` has world-writeable permissions, allowing participants to modify it and escalate privileges.

**Tools Installation**
* Installs tools that might be useful for participants if they gain shell access.  
    
   
## Including the Script in EC2 Instance Launch: ##
* Choose an Amazon Machine Image (AMI): Select your preferred Linux distribution.
* Choose an Instance Type: Select an instance type suitable for your challenge.
* Configure Instance Details:
  * Scroll down to the **Advanced Details** section.
  * In the **User Data** field, paste the script above.
* Add Storage: Configure storage as needed.
* Add Tags: (Optional) Tag your instance for identification.
* Configure Security Group:
  * Open necessary ports (22 for SSH, 80 for HTTP, 21 for FTP).
* Review and Launch: Proceed to launch the instance.

## Further Work ##
It is possible to add some more functionality depending on the expertise.
* **Reverse Engineering:** Install custom binaries that participants need to analyze.
* **Cryptography:** Place encrypted files that participants must decrypt to find the flag.
* **Steganography:** Embed flags within images or audio files hosted on the server.
* **Web Exploitation:** Deploy web applications with intentional vulnerabilities (e.g., SQL injection, XSS).  
  
**Web Exploitation Example**
```bash
# Install PHP and MySQL (LAMP stack)
apt-get install -y php php-mysql mysql-server

# Secure MySQL installation (you might skip this to create vulnerabilities)
mysql_secure_installation <<EOF

Y
password123
password123
Y
Y
Y
Y
EOF

# Download and set up a vulnerable web app (e.g., DVWA)
cd /var/www/html
git clone https://github.com/digininja/DVWA.git
chown -R www-data:www-data DVWA
```  
  
  
## Automating with CloudFormation ##
If you're using AWS CloudFormation to manage your infrastructure, you can include the User Data Script in your template.  

**Sample CloudFormation Resource**  
```yaml
Resources:
  CTFEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0abcdef1234567890  # Replace with your AMI ID
      KeyName: your-key-pair-name     # Replace with your key pair name
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      SubnetId: !Ref PublicSubnet
      UserData:
        Fn::Base64: |
          #!/bin/bash
          apt-get update -y
          # ... (rest of your script)
```  

## Tips for Creating Effective CTF Challenges ##  
* **Difficulty Balance:** Ensure that challenges vary in difficulty to cater to participants with different skill levels.
* **Clear Objectives:** Provide hints or context so participants understand what they're supposed to find.
* **Testing:** Thoroughly test each challenge to ensure it's solvable and doesn't have unintended solutions.
* **Logging:** Implement logging to monitor participant activity, which can be useful for scoring or detecting issues.

## Ethical Considerations ##  
* **Legal Compliance:** Ensure that all software and configurations comply with AWS policies and legal regulations.
* **Participant Safety:** Make sure that challenges do not encourage or require participants to engage in illegal activities.
* **Resource Limits:** Configure limits to prevent misuse of the instance (e.g., outbound traffic restrictions).

## Additional Automation Tools ##
For more complex setups, consider using configuration management tools like Ansible, Chef, or Puppet in combination with your EC2 instances.  

**Example Ansible Usecase**
* **Install Ansible on a Control Machine:** Your local machine or a dedicated management instance.
* **Write Playbooks:** Define your configurations and flags in Ansible playbooks.
* **Run Playbooks Against EC2 Instances:** Use Ansible's EC2 dynamic inventory plugin to manage your instances.
* *Let me know if you would like a tutorial on Ansible or other automation tools!*

