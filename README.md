# Linux Server Configuration Project
* IP Address: 52.24.217.75
* SSH Port: 2200
* URL for web application: http://ec2-52-24-217-75.us-west-2.compute.amazonaws.com/

## Installed software
* `git`
* `libapache2-mod-wsgi`
* `python-dev`
* `apache2`, added `/var/www/catalog/catalog` to `sites-available.conf` to serve the web application.
* `libpq-dev`
* `pip` (python package manager)
* `sqlalchemy` (python package)
* `oauth2client` (python package)
* `requests` (python package)
* `flask` (python package)
* `flask-sqlalchemy` (python package)
* `psycopg2`
* `postgresql`, configured to run locally, not on external network. created user/database for application.
* `postgresql-contrib`
* `munin`
* `fail2ban`

## Configuring the server
1. Update packages to latest versions
    * `apt-get update && apt-get upgrade`
1. Configure timezone
    * `tzselect`
    * `dpkg-reconfigure tzdata`
1. Add new user with sudo permissions
    * `adduser grader`
    * `visudo`
1. Change from root to grader user and use sudo as it is safer.
1. Install git
    * `sudo apt-get install git`
1. Configure firewall and allow port 2200 for new SSH port, allow HTTP and NTP ports
    * `sudo ufw allow 2200`
    * `sudo ufw allow 22` (until port is swapped in `sshd_config`)
    * `sudo ufw allow 80`
    * `sudo ufw allow 123`
    * `sudo ufw limit 2200/tcp`
    * `sudo ufw enable`
1. Create SSH keys for user and set permissions.
    * `mkdir /home/grader/.ssh`
    * `ssh-keygen -t rsa -f /home/grader/.ssh/udacity_key`
    * `cat /home/grader/.ssh/udacity_key.pub >> /home/grader/.ssh/authorized_keys`
    * `chmod 700 /home/grader/.ssh`
    * `chmod 644 /home/grader/.ssh/udacity_key.pub`
    * `chmod 600 /home/grader/.ssh/udacity_key`
    * `chmod 640 /home/grader/.ssh/authorized_keys`
1. Reconfigure SSHD to move from port 22 to 2200 and disable root login.
    * `sudo vim /etc/ssh/sshd_config`
    * changed `Port 22` to `Port 2200`
    * changed `PermitRootLogin yes` to `PermitRootLogin no`
    * changed `PasswordAuthentication no` to `PasswordAuthentication yes`
    * write the file and quit.
    * restart SSH `/etc/init.d/ssh restart`
1. Connect to server again as user `grader` with the private key on port 2200 to confirm changes are working.
    * ie: `ssh -i /path/to/privatekey/udacity_key.rsa -p 2200 grader@ip`
1. Remove firewall rule for port 22
    * `sudo ufw status numbered`
    * `sudo ufw del #` (number that matches port 22 rule)
1. Install Apache2 and required add-ons.
    * `sudo apt-get install libapache2-mod-wsgi python-dev`
    * `sudo apt-get install apache2`
1. Configure Apache2 to enable wsgi add-on
    * `sudo a2enmod wsgi`
1. Create web application directory and clone git repository.
    * `sudo mkdir /var/www/catalog/`
    * `git clone https://github.com/gazbot/catalog-project catalog`
1. Install Python Package Manager and required python libraries
    * `sudo apt-get install python-pip`
    * `sudo pip install Flask`
    * `sudo pip install sqlalchemy`
    * `sudo pip install oauth2client`
    * `sudo pip install requests`
    * `sudo pip install flask-sqlalchemy`
    * `sudo pip install psycopg2`
    * `sudo pip install libpq-dev`
1. Install Postgresql and create/configure database and user.
    * `sudo apt-get install postgresql`
    * `sudo apt-get install postgresql-contrib`
    * Login as postgres user: `sudo su postgres`
    * `createuser --interactive` following prompts to create catalog user
    * `createdb catalogdb`
    * Login to PSQL to the catalogdb database: `psql -d catalogdb`
1. At the PSQL prompt for the catalogdb database, grant the appropriate access to the catalog user
    * `GRANT ALL PRIVILEGES ON DATABASE catalogdb TO catalog;`
    * `ALTER ROLE catalog WITH PASSWORD '<password>';`
    * Exit psql: `\q`
1. Reconfigure Catalog Project.
    * `sudo vim /var/www/catalog/catalog.wsgi`
    * paste contents of `catalog.wsgi` in this git repository into the editor.
    * save file and exit vim.
    * Rename `application.py` to `__init__.py`
    * Reconfigure `database_setup.py` to connect to postgresql catalogdb setup.
    * Add domain name to google developer console for OAuth credentials to work.
1. Configure Apache to serve web application
    * `sudo vim /etc/apache2/sites-available/catalog.conf`
    * paste contents of `catalog.conf` in this git repository into the editor.
    * save file and exit vim
    * `sudo service apache2 restart`  
1. Add weekly cron schedule for automatic security updates
    * create `/etc/cron.weekly/apt-security-updates` and add the following to the file
        * `echo "**************" >> /var/log/apt-security-updates`
        * `date >> /var/log/apt-security-updates`
        * `aptitude update >> /var/log/apt-security-updates`
        * `aptitude safe-upgrade -o Aptitude::Delete-Unused=false --assume-yes --target-release `lsb_release -cs`-security >> /var/log/apt-security-updates`
        * `echo 
1. Install Munin for server monitoring
    * `sudo apt-get install munin`
    * `sudo vim /etc/munin/munin.conf`
    * uncomment dbdir, htmldir, logdir, rundir lines
    * change `htmldir /var/cache/munin/www` to `htmldir /var/www/munin`
    * uncomment `tmpldir` line
    * rename `[localhost.localdomain]` section to `[MuninMonitor]`
    * `sudo vim /etc/munin/apache.conf`
    * in the `<Directory /var/www/munin>` section, change `Allow from localhost 127.0.0.0/8 ::1` to `Allow from all`
    * save file and exit vim
    * `sudo mkdir /var/www/munin`
    * `sudo chown munin:munin /var/www/munin`
    * `sudo service munin-node restart`
    * `sudo service apache2 restart`
1. Install fail2ban for temporarily banning IP's that fail too many login attempts
    * `sudo apt-get install fail2ban`
    * `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
    * `sudo vim /etc/fail2ban/jail.local`
    * under the `[ssh]` section, change `port=22` to `port=2200`
    * save file and exit vim
    * `sudo service fail2ban restart`

## External References
https://discussions.udacity.com/t/markedly-underwhelming-and-potentially-wrong-resource-list-for-p5/8587
https://help.ubuntu.com/community/AutomaticSecurityUpdates
https://help.ubuntu.com/community/AutoWeeklyUpdateHowTo
https://www.digitalocean.com/community/tutorials/how-to-install-munin-on-an-ubuntu-vps
https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-12-04