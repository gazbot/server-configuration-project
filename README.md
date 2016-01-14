# Linux Server Configuration Project
IP Address: 52.24.217.75
URL for web application: http://ec2-52-24-217-75.us-west-2.compute.amazonaws.com/

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

## External References
https://discussions.udacity.com/t/markedly-underwhelming-and-potentially-wrong-resource-list-for-p5/8587
