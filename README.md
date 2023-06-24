# How To Set Up Django with Postgres, Nginx, and Gunicorn on Ubuntu 
#### Introduction
###### Django is a powerful web framework that can help you get your Python application or website off the ground. Django includes a simplified development server for testing your code locally, but for anything even slightly production related, a more secure and powerful web server is required.

![Alt text](https://www.projecthosts.com/wp-content/uploads/2022/10/AWS_Header-1-1024x429.png)

#### Install the Packages from the Ubuntu Repositories
###### To begin the process, we’ll download and install all of the items we need from the Ubuntu repositories. We will use the Python package manager pip to install additional components a bit later.

#### First get your ec2 ready to start, then go with the below procedure if you have any dought please contact at 9946658045

###### We need to update the local apt package first. 
```
sudo apt-get update
```
###### Now install the necessary packages, use python3 if you are using it. Its very easy. 
```
sudo apt-get install python-pip python-dev libpq-dev nginx
```
###### For Making it interest while doing lets see how your nginx webserver looks like. For that type
```
sudo systemctl start nginx
```
###### That's it. Now go to your instance ip and see your nginx hosted site.

![Alt text](https://rdr-it.com/wp-content/uploads/2020/09/ubuntu-nginx-php-mariadb-010.png)

###### Install virtualenv, i hope use guys know why we need virtual env in python projects
```
sudo -H pip3 install virtualenv
```
###### Before we install our project’s Python requirements, we need to activate the virtual environment. You can do that by typing
```
source myprojectenv/bin/activate
```
With your virtual environment active, install Django, Gunicorn
```
pip install django gunicorn
```
## Create a Gunicorn systemd Service File
Create and open a systemd service file for Gunicorn with sudo privileges in your text editor
```
sudo nano /etc/systemd/system/gunicorn.service
```
Finally, we’ll add an [Install] section. This will tell systemd what to link this service to if we enable it to start at boot. We want this service to start when the regular multi-user system is up and running

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myproject
ExecStart=/home/sammy/myproject/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/sammy/myproject/myproject.sock myproject.wsgi:application

[Install]
WantedBy=multi-user.target
```
We can now start the Gunicorn service we created and enable it so that it starts at boot

```
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```
## Check for the Gunicorn Socket File
```
sudo systemctl status gunicorn
```
## Configure Nginx to Proxy Pass to Gunicorn
Start by creating and opening a new server block in Nginx’s sites-available directory

```
sudo nano /etc/nginx/sites-available/myproject
```

Finally, we’ll create a location / {} block to match all other requests. Inside of this location, we’ll include the standard proxy_params file included with the Nginx installation and then we will pass the traffic to the socket that our Gunicorn process created

```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/sammy/myproject;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/sammy/myproject/myproject.sock;
    }
}
```
Save and close the file when you are finished. Now, we can enable the file by linking it to the sites-enabled directory

```
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
```
Test your Nginx configuration for syntax errors by typing
```
sudo nginx -t
```
If no errors are reported, go ahead and restart Nginx by typing

```
sudo systemctl restart nginx
```

## Nginx Is Displaying a 502 Bad Gateway Error Instead of the Django Application
```
sudo tail -F /var/log/nginx/error.log
```

# Got error
## connect() to unix:/home/sammy/myproject/myproject.sock failed (2: No such file or directory)

This indicates that Nginx was unable to find the myproject.sock file at the given location. You should compare the proxy_pass location defined within /etc/nginx/sites-available/myproject file to the actual location of the myproject.sock file generated in your project directory.

If you cannot find a myproject.sock file within your project directory, it generally means that the gunicorn process was unable to create it. Go back to the section on checking for the Gunicorn socket file to step through the troubleshooting steps for Gunicorn.

## connect() to unix:/home/sammy/myproject/myproject.sock failed (13: Permission denied)
```
namei -nom /home/sammy/myproject/myproject.sock
```




