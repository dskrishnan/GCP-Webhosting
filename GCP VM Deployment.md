# Create a Virtual Machine in Google Cloud
* Visit [Google Cloud Console](https://console.cloud.google.com)
    * Sign up for a free account.
    * Add Credit card to Billing Information to get free tier.
    * Start 1 year free trial.

# Install Google Cloud Installer in MacOS - This is to manage GCloud from Terminal.
* In MacOS, go to Home Directory using `cd ~` in the terminal
* Run `curl https://sdk.cloud.google.com | bash`
* Follow the instructions and after the installation restart the shell.
* Run `gcloud init` to initialize the cloud environment.
    * This will start running a diagnostic and prompt you to login. Select `Y` and authenticate yourself with the google account in a browser. A browser window will be opened by now.
    * After browser authentication choose the existing project or create a new one.
    * Then choose the time zone([31]asia-southeast1-c)
* Follow the instructions there if any & Now google cloud SDK is successfully installed in the MacOS.

* Go to Google Cloud Home page. The default project name **My First Project** would be listed. Keep this or rename to **Custom Project Name**

* From the Cloud Console, Choose **Deployment Manager** & Click **Deploy Marketplace Solution**
    * Search for Wordpress (preferrably one from Bitnami Stack).
    * Launch it and it will prompt you to choose the VM hardware.
    * Follow along the instructions and wordpress should be ready.
    * Note down the IP address for the website URL, Admin Page, User ID & Password.
## Setup External IP and Create Cloud DNS in Gcloud.
* Under **VPC Network**, choose External IP address and reserve an IP Address.
* Then Under **Network Services**' choose **Cloud DNS**.
* Zone Name is `domain name`. DNS Name is `domainname.com`. Keep the `DNSSEC:off`.  Create a Zone.
* Now `Add Record Set` & leave DNS name as it is.
    * **Resource Record Type** should be A & **TTL** may be 1 hour.
    * Enter the Static IP or External IP & Save it. Notedown the Nameserver names.
* Now open Godaddy CPANEL and update the nameservers for the domain  & Save it. The nameservers would look something similar to this
    * ns-cloud-e1.googledomains.com
    * ns-cloud-e2.googledomains.com
    * ns-cloud-e3.googledomains.com
    * ns-cloud-e4.googledomains.com
* After few mins open terminal in your personal Machine and type nslookup disciple.co.in
    * External IP address reserved in Google Cloud DNS should be displayed.

* To setup the domain & link in the local host of the VM, type `nano /etc/hosts`
* There enter `<static ip address> disciple.co.in` & `ctrl+X` to save it.

## To make the website secure - Create SSL Certificate,
* For bitnami stack, type `sudo /opt/bitnami/bncert-tool` and follow along.
    * Enable HTTP to HTTPS redirection [Y/n]: Y
    * Enable non-www to www redirection [Y/n]: n
    * Enable www to non-www redirection [y/N]: y
    * Domain list: disciple.co.in www.disciple.co.in
    * Server name: disciple.co.in

## To increase the performance of the website using CDN:
* Analyse the performance of the website using https://gtmetrix.com/. Give the disciple.co.in and analyse to get a report.
* In most cases YSlow will have first two fail grades.
    * Add Expire headers
    * Use Content Delivery network
* Sign up with Cloudflare.com
    * Add the URL of the website and follow along.
    * New Nameservers `adi.ns.cloudflare.com` & `dax.ns.cloudflare.com` will be given. Update the nameservers with the Domain provider
* Login as Wordpress Admin and add a plugin Really Simple SSL.
    * This will demand write access to `wp-config.php`.
    * To grant write access, run `sudo chown daemon:daemon /opt/bitnami/apps/wordpress/htdocs/wp-config.php`

## Remove the Bitnami Banner Ad
* Run `sudo chmod u+x /opt/bitnami/apps/wordpress/bnconfig.disabled`
* Then run `sudo /opt/bitnami/apps/wordpress/bnconfig.disabled --disable_banner 1`
*  Restart the Apache server `sudo /opt/bitnami/ctlscript.sh restart apache`


# Following are the steps to manually create a VM and Install Webserver, Wordpress
* Under Compute Engine, Choose *Create VM Instance*
    * Instance name would be **bharath**
    * region is *asia-south1(Mumbai)*
    * Change the **Boot Disk** to *Ubuntu*. 
        * Choose the Latest Version 18.04 LTS.
        * Machine Type is `g1-small(1vCPU, 1.7 GB Memory`
    * Leave the other default settings and select Allow HTTP & HTTPS Traffic under Firewall.
    * Then click **Create**. It will take a minute or so.

* Now a Ubuntu Virtual Machine is ready in the Google Cloud.
    * We need to configure SSH connection from our Laptop to this gcloud.
        * This step can be skipped if we decided to browser based Command Line Interface(Cloud Shell).
    * Under SSH, select **View gcloud command**
    * This will show a command `gcloud beta compute ssh --zone "asia-south1-c" "bharath" --project "uplifted-logic-271103"`.
    
    * Copy the above command and paste it in the Terminal in our development laptop/computer.
    * This command will guide us to create SSH key. We need to give a paraphrase, `bharath` in this case.
    * Follow the instructions to establish SSH Connection from your laptop to the VM Instance in the Google Compute Engine.


* Run these updates `sudo apt update && sudo apt upgrade`
* Now Swap file need to be created to support RAM over usage.
    * Try 'sudo swapon --show'
    * If the result is empty then swap space is not enabled.
    * The boot disk of the VM instance **bharath** has 10GB disk space. Out of this 1GB can be allocated for extra RAM.
        * Run `sudo fallocate -l 1G /swapfile`
        * Only root user should be able to modify the swapfile. So, run `sudo chmod 600 /swapfile`
        * Use mkswap utility to setup Linux swap area as `sudo mkswap /swapfile`
        * All these swap space will be wiped out if the VM reboots. To make these changes permanent, take a back up `sudo cp /etc/fstab /etc/fstab.back`.
        * To add the line `/swapfile none swap sw 0 0`, try `sudo nano /etc/fstab`. Then 'ctrl+X', it will ask you to save & save it.
    * Adjusting Swappiness Value.
        Swappiness is a Linux kernel property that defines how often the system will use the swap space. Swappiness can have a value between 0 and 100. A low value will make the kernel to try to avoid swapping whenever possible, while a higher value will make the kernel use the swap space more aggressively.
        * To check the default value `cat /proc/sys/vm/swappiness`. Default value of it would be 60.
        * For most systems 60 should be fine. But for production servers it may be set to 10. This can be done using `sudo sysctl vm.swappiness=10`

## To install Wordpress in Ubuntu:
* Run `sudo apt update && sudo apt upgrade`.
* Run `sudo apt install wordpress php libapache2-mod-php mysql-server php-mysql`. 
    * Follow the instructions to install **PHP, Apache2 & MySQL**.
    * This will ask password for `root` MySQL. Enter as appropriately.
    
## Configure Apache2 for Wordpress
    * Create Apache site for WordPress. Create /etc/apache2/sites-available/wordpress.conf with following lines:

    `Alias /blog /usr/share/wordpress
    <Directory /usr/share/wordpress>
        Options FollowSymLinks  
        AllowOverride Limit Options FileInfo  
        DirectoryIndex index.php  
        Order allow,deny  
        Allow from all
    </Directory>
    <Directory /usr/share/wordpress/wp-content>
        Options FollowSymLinks
        Order allow,deny
        Allow from all
    </Directory>`
* To enable this site, run `sudo a2ensite wordpress`
* To enable URL rewriting, run `sudo a2enmod rewrite`
* Now reload the apache2 using `sudo service apache2 reload`
* To activate the new configuration, run `sudo systemctl reload apache2 && sudo systemctl restart apache2 && sudo service apache2 reload`
* 

## Create & Configure MySQL Database:
* To configure WordPress, we need to create MySQL database. Run `sudo mysql -u root`.
* The above will create a prompt `mysql>`. Here type `CREATE DATABASE wordpress;`
* Now grant previleges, `GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO wordpress@localhost IDENTIFIED BY 'password';`
* Run `FLUSH PRIVILEGES;` & `quit`
* Now, let’s configure WordPress to use this database. type `sudo nano /etc/wordpress/config-localhost.php`. Then enter the following,

        <?php
            define('DB_NAME', 'wordpress');
            define('DB_USER', 'wordpress');
            define('DB_PASSWORD', '<your-password>');
            define('DB_HOST', 'localhost');
            define('DB_COLLATE', 'utf8_general_ci');
            define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
        ?>'

* Enable MySQL using `sudo service mysql start`
## Install LAMP(Linux, Apache, MySQL & PHP) Stack
* First install `sudo apt install tasksel`
* Then run `sudo apt install apache2`
* Then run `sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc` and follow the instructions.