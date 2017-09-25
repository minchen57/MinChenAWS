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

# Install java
- Oracle Java 8 required by some programs and it is not available on Debian archive due to licensing issues.To install Java 8 on Debian Jessie with minimum effort, you can use WebUpd8's Java PPA repositories. Follow the steps below:
-   ```
    sudo su -
    echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee /etc/apt/sources.list.d/webupd8team-java.list
    echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886
    apt-get update
    apt-get install oracle-java8-installer
    ```
- We can then check the version of java by `javac -version`

# Install neo4j
- [References for installation](https://medium.com/@Jessicawlm/installing-neo4j-on-ubuntu-14-04-step-by-step-guide-ed943ec16c56)
-   ```
    wget -O - https://debian.neo4j.org/neotechnology.gpg.key | sudo apt-key add -
    echo 'deb http://debian.neo4j.org/repo stable/' >/tmp/neo4j.list
    sudo apt-get update
    sudo apt-get install neo4j
    ```
- To start, stop and restart the server:
    ```
    sudo service neo4j start
    sudo service neo4j stop
    sudo service neo4j restart
    ```
- The server can be accessed using `http://localhost:7474/browser/`

- To allow remote access from other machine, open the /etc/neo4j/neo4j.conf and change the corresponding lines to:
    ```
    dbms.connector.bolt.listen_address=0.0.0.0:7687
    dbms.connector.http.listen_address=0.0.0.0:7474
    dbms.connector.https.listen_address=0.0.0.0:7473
    ```
- Now, I can access using: http://minchen.technology:7474/browser/ and the initial password is just neo4j. I have changed it to my favorite password.

# Install spacy (for python3)
- Install pip3 first: `sudo apt install python3-pip`
- Run the following command:
    ```
    pip3 install -U spacy
    sudo python3 -m spacy.en.download all
    ```
- Simple testing commands:
    ```
    python3
    import spacy
    nlp = spacy.load('en')
    doc = nlp(u'Hello, spacy!')
    print([(w.text, w.pos_) for w in doc])
    ```