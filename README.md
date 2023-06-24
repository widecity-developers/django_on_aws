# How To Set Up Django with Postgres, Nginx, and Gunicorn on Ubuntu 
#### Introduction
###### Django is a powerful web framework that can help you get your Python application or website off the ground. Django includes a simplified development server for testing your code locally, but for anything even slightly production related, a more secure and powerful web server is required.

![Alt text](https://www.projecthosts.com/wp-content/uploads/2022/10/AWS_Header-1-1024x429.png)

#### First get your ec2 ready to start, then go with the below procedure if you have any dought please contact at 9946658045
![Alt text](https://www.nakivo.com/blog/wp-content/uploads/2021/12/How-to-connect-to-EC2-instance-via-SSH-Linux-1.jpg)

#### Install the Packages from the Ubuntu Repositories
###### To begin the process, we’ll download and install all of the items we need from the Ubuntu repositories. We will use the Python package manager pip to install additional components a bit later.

###### We need to update the local apt package first. 
```
sudo apt-get update
```
###### Now install the necessary packages, use python if you are using it. Its very easy. 
```
sudo apt-get install python3-pip python3-dev libpq-dev nginx
```
###### For Making it interest while doing lets see how your nginx webserver looks like. For that type
```
sudo systemctl start nginx
```
###### That's it. Now go to your instance ip and see your nginx hosted site.

![Alt text](https://rdr-it.com/wp-content/uploads/2020/09/ubuntu-nginx-php-mariadb-010.png)
###### if you are happy with the results lets go
###### Create a folder for your project which we will call as the project folder and change your directory to it, that sound nice.
```
sudo mkdir projectfolder_name
```
```
cd projectfolder_name
```
#####  lets get your github code to here. pretty cool right. 
!important , make sure you have an requirements.txt file, which can make these process much faster. if you don't have that just go to your local project folder and type,
```
pip freeze > requirement.txt
```
#### Lets clone
```
git clone https://github.com/account_name/repo
```
###### Install virtualenv, i hope use guys know why we need virtual env in python projects
```
sudo -H pip3 install virtualenv
```
###### Before we install our project’s Python requirements, we need to activate the virtual environment. You can do that by typing
```
source myprojectenv/bin/activate
```
#### Install the requirements.txt using the comment
```
pip install -r requirements.txt
```
With your virtual environment active, install Django, Gunicorn
```
pip install gunicorn
```
###### Try to run your project localy on aws using the command
```
python3 manage.py runserver
```
#### If any errors occur
###### Install the neccassary packages according to it and try to run the script again until it works. contact 9946658045 if you couldnt solve it.

## Lets move forward, Create a Gunicorn systemd Service File
Create and open a systemd service file for Gunicorn with sudo privileges in your text editor
```
sudo nano /etc/systemd/system/gunicorn.service
```
Finally, we’ll add an [Install] section. This will tell systemd what to link this service to if we enable it to start at boot. We want this service to start when the regular multi-user system is up and running.
Replace the path according to your project, it is not hard as it seems. just go through it. you will find it simple.

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/myproject
ExecStart=/home/ubuntu/myproject/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/ubuntu/myproject/myproject.sock myproject.wsgi:application

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
        root /home/ubuntu/myproject;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/myproject/myproject.sock;
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
## connect() to unix:/home/ubuntu/myproject/myproject.sock failed (2: No such file or directory)

This indicates that Nginx was unable to find the myproject.sock file at the given location. You should compare the proxy_pass location defined within /etc/nginx/sites-available/myproject file to the actual location of the myproject.sock file generated in your project directory.

If you cannot find a myproject.sock file within your project directory, it generally means that the gunicorn process was unable to create it. Go back to the section on checking for the Gunicorn socket file to step through the troubleshooting steps for Gunicorn.

## connect() to unix:/home/ubuntu/myproject/myproject.sock failed (13: Permission denied)
```
namei -nom /home/ubuntu/myproject/myproject.sock
```




