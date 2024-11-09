# LINUX SERVER HARDENING
In this project, the objective is to secure a Linux server by hardening its configuration. Here’s a step-by-step guide to approach each task in a real-world scenario as a senior Linux administrator. The steps focus on prioritizing security, minimizing attack surfaces, and ensuring ongoing monitoring and logging for proactive security.


LINUX SERVER HARDENING PROJECT

In this project, the objective is to secure a Linux server by hardening its configuration. Here’s a step-by-step guide to approach each task in a real-world scenario as a senior Linux administrator. The steps focus on prioritizing security, minimizing attack surfaces, and ensuring ongoing monitoring and logging for proactive security.

 
 step 1- I disable unnecessary Services
 Goal: Reduce potential attack vectors by disabling services that are not required for the server's functionality.
 
  1.1 Identify Running services.
    
    * List active services to identify which ones are currently running:
    
    -> systemctl list-units --type=service --state=running
    -> I use netstat or ss to view services listening on open ports:
    -> ss -tuln
      
      
     * I check for legacy services (e.g, telnet, FTP) that are inherently insecure and disable them.
     
     1.2 - Disable unnecessary services 
      stop and disable unneeded services:
     
     -> systemctl stop <service-name>
     -> systemctl disable <service-name>
     
     For persistent changes, i remove the service package if not required:
     
     -> apt-get remove <service-name> # debian-based
     -> yum remove <service-name> #RHEL-based
     
     1.3. I Harden startup configuration
     
     * I edit startup configurations to prevent services from being enables on boot inadvertently:
     
     -> systemctl mask <service-name> # Mask the service to prevent accidental enablement
     
     
     STEP 2: CONFIGURE FIREWALL RULES WITH IPTABLES AND FIREWALL
 GOAL:
     My goal is to control traffic in and out of the server to only allow essential connections, preventing unauthorized access.
     
     2.1 Set up iptables RUles (low-level customization)
     
     * Define rules for allowing traffic only on necessary ports, such as SSH (port 22) and HTTP/HTTPS if it's a web server.
     
     -> iptables -A INPUT -p tcp --dport 22 -j ACCEPT # Allow SSH
     -> iptables -A INPUT -p tcp --dport 80 -j ACCEPT # Allow HTTP
     -> iptables -A INPUT -p tcp --dport 443 -j ACCEPT # Allow HTTPS
     -> iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT # Allow established connections
     -> iptables -p INOUT DROP # DROP all other incoming traffic
     -> iptables -p FORWARD DROP
     -> Iptables -p OUTPUT ACCEPT
     
     
    2.2 I use firewall for Easier management(if available)
    
    * I enable firewalld for more flexibility if needed ( particularly on RHEL-based systems):
    
    -> systemctl enable firewalld --now
    -> firewall-cmd --permanent --add-service=ssh # Allow SSH
    -> firewall-cmd --permanent --add-service=http # Allow HTTP if needed
    -> firewall-cmd --permanent --add-service=https # Allow HTTPS if needed
    -> firewall-cmd --reload # Apply the changes
    
    2.3 Save and Persist Firewall rules
    
    * I persist iptables rules:
    
    -> iptables-save > /etc/iptables/rules.v4 # Debian-based
    -> service iptables save # RHEL-based
    
   
  STEP 3: ENFORCE SSH KEY-BASED AUTHENTICATION
  Goal:
  	My goal is to enhance SSH security by requiring key-based authentication and disabling password authentication.
  	
   3.1 COnfigure SSH Daemon
   
   * I edit SSH configuration file to enforce key-based authentication:
   
   -> nano /etc/ssh/sshd_config
   
   * I modify the following settings:
   
   -> passwordAuthenctication no  # Disable password login
   -> permitRootLogin no         # Disable root login
   -> PubkeyAuthenctication yes # Enable key-based authentication
   
   * I Reload SSH daemon to apply changes:
   
   -> systemctl reload sshd
   
   3.2 Set up SSH key for Users
   
   * Generate SSH keys on the client machine:
   
   -> ssh-keygen -t rsa -b 4096
   
   * I Copy the public key to the server:
   
   -> ssh-copy-id user@server-ip
   
  
  3.3 Restrict SSH Access 
  
  * For enhanced security, i restrict SSH access to specific IPs (e.g, my internal network):
 
 -> iptables -A INPUT -p tcp -s <allowed-IP-range> --dport 22 -j  ACCEPT
 
 
 STEP 4: CONFIGURE LOGGING AND MONITORING TOOLS
 
Goal:
    My goal is to enable centralized logging and real-time monitoring for detecting and responding to suspicious activity.
    
    * I ensure rsyslog is configured and running to log all critical events:
    
    -> systemctl enable rsyslog --now
    
    * I configure /etc/rsyslog.conf to send logs to a centralized server ( if available):
    
    -> *.* @logserver-ip:514
    
    4.2 I Install and configure Monitoring tools
    
    * I install Fail2Ban for detecting and blocking brute-force attempts:
    
    ->apt-get install fail2ban     # Debian-based
    ->yum install fail2ban     #RHEL-based
    
    * I configure Fail2Ban to protect SSH:
    
    -> nano /etc/fail2ban/jail.local
    
    * I Add configurations:
    
    [sshd]
    enabled = true
    port = ssh
    logpath = /var/log/auth.log # check file location for my system
    maxretry = 3
    bantime = 3600
    
    
    4.3 I install log monitoring and analysis tools
    
    * I set up tools such as Filebeat to forward logs to an ELK stack for log analysis:
    
    Install Filebeat:
    
    -> sudo apt install filebeat 
    
    I Configure Filebeat to send logs to ELK
    
    -> nano /etc/filebeat/filebeat.yml
    
    Configure the output section to point to my ELK server:
    
    output.elasticsearch:
       hosts: ["my-elk-server-ip:9200"]
       
      
      4.4. I schedule regular Security Audits
      
      * I use automated scripts or tools like Lynis to perform regular audits:
      
      -> apt-get install lynis
      -> lynis audit system
      
      
      4.5 I enable Regular Reporting
      
      * I set up reports to be sent regularly to administrators or other relevant stakeholders.
      
      * I configure alert for critical events or suspicious activies using log management tools
      
      
      SUMMARY CHECKLIST
 
 1- Disable Unnecessary Services: Remove or disable no-essential services to minimize potential attack vectos.
 
 2- Configure Firewall: Use iptables or firewalld to restrict inbound and outbound traffic, allowing only necessary services.
 
 
 3- Enforce SSH key-base Authentication: Disable password-based authentication for SSH and configure key-baed login only.
 
 4- Enable and Configure Logging and Monitoring:
 
     * Set up rsyslog for centralized logging.
     * Install Fail2Ban to block brute-force attacks.
     * COnfigure log analysis tools for monitoring and regurlar security audit
     
  
  By following this step-by-step guide, i am able to establish a ribust security posture for my linux server in a real-world environment, ensuring that it's well-protected and that security incidents can be detected and responed to promptly
   
This project is licensed under the MIT License - see the LICENSE file for details.
  
MIT License

Copyright (c) 2024 Francklin Darryl Bassoupeck Nnon

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

If you have any questions or feedback, please feel free to contact me:
* Email: darrylnnon@gmail.com
* Github: darrylnnon@github.com
* 
