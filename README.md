

# Certbot For Palo PanOS
Certbot helper script for Palo Alto PanOS firewall
Tested on Ubuntu 26.04 with Python 3.14.4 and Palo Alto PA-820 running PAN-OS 9.1

The script is based on https://github.com/psiri/letsencrypt_paloalto

Some part of the script was written using Claude Opus 5

Decision making and this README is done by a human (me!)

### What was changed from the original:
1. Uses Python venv to avoid breaking system packages
2. Instead of doing full commit, it will only commit changes from a specific service account for added security
3. Use the certificate chain instead of the lone certificate.
4. Fixed the date checking issue so it won't renew every time it ran

# Prerequisites:

***The script is designed to be ran as Root and in Root Crontab.***

### 1. Install the required packages

    sudo apt install python3-pip python3-venv certbot openssl python3-certbot-dns-cloudflare

****Install pan-python PIP package without breaking system packages:****

    sudo python3 -m venv /opt/pan-python
    sudo /opt/pan-python/bin/pip install --upgrade pip
    sudo /opt/pan-python/bin/pip install pan-python
    sudo chmod -R go-w /opt/pan-python

### 2. Make a service admin account for SSL/TLS changes only:

**In PAN OS UI after login, create a custom admin Roles, give it a name (e.g. SSL_Admin) and grant the following rights:**

Web UI > Device > Certificate Management > Certificates

Web UI > Device > Certificate Management > SSL/TLS Service Profile

Web UI > Commit > Commit For Other Admins: disabled 

XML API > Configuration

XML API > Commit

XML API > Import

**Create a admin account (e.g. certbot_svc) with the custom admin profile (e.g. SSL_Admin):**

In the Administrator Type, select "Role Based" and select the admin profile you just created.

# Installation:

### 3. Make a .panrc file with your API key

    sudo /opt/pan-python/bin/panxapi.py -h 10.10.10.1 -l 'certbot_svc:PASSWORD' -k >> /root/.panrc

### 4. Make a cloudflare.ini file with your Cloudflare API Token:

    sudo echo "dns_clouflare_api_token = 'YOUR_TOKEN_HERE'" > /root/.cf_token


### 5. Download the script and edit the fixed parameter:
	  
	curl -o https://raw.githubusercontent.com/PurpleSe4shell/palo_certbot_helper/refs/heads/master/pan_certbot && chmod a+x pan_certbot
	  
Edit these parameters at the start of the script: 

    CLOUDFLARE_CREDS=/root/.cf_token          #absolute path of your token file
    PAN_MGMT=10.10.10.1                       #firewall management IP
    FQDN=vpn.example.com                      #firewall/GP FQDN
    EMAIL=admin@example.com                   #email
    API_KEY=$(cat /root/.panrc)               #absolute path
    CERT_NAME=LetsEncrypt_vpn_example_com     #certificate name
    GP_PORTAL_TLS_PROFILE=GP-SSL-Profile      #GP Portal SSL Profile Name
    GP_GW_TLS_PROFILE=GP-SSL-Profile          #GP Gateway SSL Profile Name
    PAN_SVC_ACC=certbot_svc                   #Admin service account name 
    PANXAPI=/opt/pan-python/bin/panxapi.py    #panxapi.py location (venv)

### 6. Test the script:

    sudo ./pan_certbot

### 7. Install the script into sbin for root cronjob:

    sudo install -o root -g root -m 700 pan_certbot /usr/local/sbin/pan_certbot

### 8. Edit the crontab so it runs automatically everyday:

    sudo crontab -e

**Add this to the crontab file:**

    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    0 0 * * * /usr/bin/flock -n /var/lock/pan_certbot.lock /usr/local/sbin/pan_certbot >> /var/log/pan_certbot.log 2>&1


