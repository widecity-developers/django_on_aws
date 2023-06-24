# How To Set Up Django with Postgres, Nginx, and Gunicorn on Ubuntu 
## Introduction
Django is a powerful web framework that can help you get your Python application or website off the ground. Django includes a simplified development server for testing your code locally, but for anything even slightly production related, a more secure and powerful web server is required.

![Alt text](https://www.projecthosts.com/wp-content/uploads/2022/10/AWS_Header-1-1024x429.png)

## Install the Packages from the Ubuntu Repositories
To begin the process, we’ll download and install all of the items we need from the Ubuntu repositories. We will use the Python package manager pip to install additional components a bit later.

We need to update the local apt package index and then download and install the packages. 
```
sudo apt-get update
```
