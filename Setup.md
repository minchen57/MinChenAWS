Created on 08/23/2017 by Min Chen

# Setup the EC2 instance
- Started a Amazon Webservice account (Free Tier) and also registered as a student at AWS Educate. 
- Launched one instance of Ubuntu Server (free one: t2.micro) on AWS
- Log-in to the server:
    - The key file is named L715_Min.pem
    - The public IP is 18.221.15.70
    - Change the key file to read-only: `chmod 400 L715_Min.pem`
    - Now we can log-in to the server using either, `ssh -i "L715_Min.pem" ubuntu@ec2-18-221-15-70.us-east-2.compute.amazonaws.com` or ssh -i "L715_Min.pem" ubuntu@18.221.15.70`
- Link to a domain name
    - Bought a domain name `minchen.technology` from GoDaddy.
    - Using Amazon Route 53 to route traffic for the domain. The detailed steps can be found [here](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-ec2-instance.html)
    - Go to GoDaddy Myproduct and set the DNS nameservers to the four name server given at Amazon Route 53.
- Automatically Loading the key
    - One can configure the key to be automatically loaded when ssh-connecting to the instance. In the home directory there is a directory .ssh. In this directory, place the pem-key. There should be a file config, add the following entries to config:
        ```
        Host ec2-18-221-15-70.us-east-2.compute.amazonaws.com
        User ubuntu
        IdentityFile /Users/master/.ssh/L715_Min.pem
        Host 18.221.15.70
        User ubuntu
        IdentityFile /Users/master/.ssh/L715_Min.pem
        Host minchen.technology
        User ubuntu
        IdentityFile /Users/master/.ssh/L715_Min.pem
        ```
    - Now we can log-in to the server using either of the following: `ssh 18.221.15.70` or `ssh minchen.technology`

# Apache2 Web Server
- To install Apache2 web server: `sudo apt install apache2`
- Configuration
 	- Modify the inbound rule for the instance from restricted IP to non-restricted: 0.0.0.0/0, ::/0 (on the amazon aws console)
    - Then typing http://minchen.technology/ in the browser will load the webpage. http://18.221.15.70/ will also work.
    - Then the apache service is restarted: `sudo service apache2 restart`
    
# HTTPS Configuration 
- [Instructions are found here](https://help.ubuntu.com/lts/serverguide/httpd.html)
- Execute the following command at a terminal prompt to enable the mod_ssl module: `sudo a2enmod ssl`
- To configure Apache2 for HTTPS, enter the following: `sudo a2ensite default-ssl`
- With Apache2 now configured for HTTPS, restart the service to enable the new settings: `sudo systemctl restart apache2.service`
- We should be able to access the page through by [https://minchen.technology].

# To Obtain a CA signed Certificate for https:
- Add the ppa for certbot (the python program that generates the certificate)
    * `sudo add-apt-repository ppa:certbot/certbot`
    * `sudo apt update`
    * sudo apt install python-certbot-apache
- Then simply run certbot witht he apache option to generate our keys (`sudo certbot --apache`)
- When prompted by certbot, I specified the option to redirect all traffic to https
- The cron config file was edited to run every 18th of the month at 1:30 am so that we do not have to manually renew the certificate.
    *`crontab -e`
    * Add the line `30 2 18 * * sudo certbot renew` to the end of crontab

# Use Jekyll to generate website
- Installed Jekyll for easy webpage design 
- There were some prerequisite packages that had to be installed:
    * ruby: `sudo apt install ruby ruby-dev`
    * rubygems: installed by default with the ruby package in ubuntu
    * make: `sudo apt install make`
    * gcc: `sudo apt install gcc`
- Then jekyll and bundler were installed: `sudo gem install jekyll`, `sudo gem install bundler`
- I cloned the [repo](https://github.com/pietromenna/jekyll-cayman-theme) for jekyll-cayman-theme to create a jekyll site with this theme.
- The folder on server is at home/ubuntu/jekyll-cayman-theme
- if you cd into the directory for your jekyll project (above) and then run `jekyll build` it will generate the appropriate \_site folder with html generated from your markdown (index.markdown)
- The contents of this folder are symlinked to a subfolder in /var/www/html/ by running the following command in folder /var/www/html/: 
    ```
    sudo su
    rm -r html
    ln .s /home/ubuntu/jekyll-cayman-theme html
    ```
- Finally, contents can be changed at home/ubuntu/jekyll-cayman-theme/index.markdown, buttons can be changed at home/ubuntu/jekyll-cayman-theme/\_includes/page-header.html. Other settings are in home/ubuntu/jekyll-cayman-theme/\_config.yml