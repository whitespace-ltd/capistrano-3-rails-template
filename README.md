# Capistrano 3 Rails Config

## Overview

This is a sample configuration for deploying Ruby on Rails applications with Capistrano 3.1 including Zero Downtime Deployment with Unicorn and Sidekiq background Workers.

## Usage

See
<http://www.talkingquickly.co.uk/2014/01/deploying-rails-apps-to-a-vps-with-capistrano-v3/>
for a tutorial on usage.

Or for more details, this is also the example configuration used in the
book Reliably Deploying Rails Applications available on Leanpub:
<https://leanpub.com/deploying_rails_applications>

## Minimal Requirements

You should have the following in your Gemfile:

    gem 'capistrano', '~> 3.1.0'

    # rails specific capistrano funcitons
    gem 'capistrano-rails', '~> 1.1.0'

    # integrate bundler with capistrano
    gem 'capistrano-bundler'

    #  Idiomatic rbenv support for Capistrano
    gem 'capistrano-rbenv', "~> 2.0" 

    # Use Unicorn as our app server
    gem 'unicorn'

## VPS Setup

### Core Packages 

	sudo apt-get -y update
	sudo apt-get -y install curl git-core python-software-properties vim monit imagemagick htop


### Oh My Zsh
 
	sudo apt-get -y install zsh
	curl -L http://install.ohmyz.sh | sh
	sudo chsh -s $(which zsh) your_username

### Nginx

	sudo add-apt-repository ppa:nginx/stable
	sudo apt-get -y update
	sudo apt-get -y install nginx
	
To satart Nginx server:

	sudo service nginx start
	
Nginx configs:
	
	/etc/nginx/nginx.conf
	
Default nginx configs:
	
	/etc/nginx/sites-enabled/default
	
Nginx logs 

	/var/log/nginx/access.log
	/var/log/nginx/error.log

### Postgres DB

Installation:

	sudo apt-get install postgresql postgresql-contrib

Login to psql server and set your new password:

	sudo -u postgres psql
	/password # Type your new password

Then you can create a new user with a password and create your application database:

	create user new_user with password 'new_password';
	create database your_application_name owner new_user;

### Ruby (rbenv)

First, install rbenv-installer:

	curl https://raw.githubusercontent.com/fesplugas/rbenv-installer/master/bin/rbenv-installer | bash

Then, open `vim ~/.zshrc` file and add the following lines at the end of the file to add rbenv to the top:

	export RBENV_ROOT="${HOME}/.rbenv"
	if [ -d "${RBENV_ROOT}" ]; then
  		export PATH="${RBENV_ROOT}/bin:${PATH}"
  		eval "$(rbenv init -)"
	fi

You'll probably need to install some required packages that Ruby depends on. You can use the provided bootstrap scripts.

	rbenv bootstrap-ubuntu-12-04

We’re ready to install Ruby itself now and we’ll install the latest version, currently `2.1.2`.


	rbenv install 2.1.2
	rbenv global 2.1.2
	gem install bundler --no-ri --no-rdoc
	rbenv rehash
	

### Add a Swap File

Check if any swap files have been enabled on the VPS:

	sudo swapon -s
	
Type the following command to create 512MB swap file (1024 * 512MB = 524288 block size):


	sudo dd if=/dev/zero of=/swapfile1 bs=1024 count=524288
	
Prepare the swap file by creating a linux swap area:

	sudo mkswap /swapfile1
		
Active the swap file:

	sudo swapon /swapfile1
	

This file will last on the VPS until the machine reboots. You can ensure that the swap is permanent by adding it to the fstab file.

Open `sudo vim /etc/fstab` & paste in the following line:

 	/swapfile1       none    swap    sw      0       0 
 	
Set swappiness to 10 to act as an emergency buffer, preventing out-of-memory crashes:

	echo 10 | sudo tee /proc/sys/vm/swappiness
	echo vm.swappiness = 10 | sudo tee -a /etc/sysctl.conf
	
Finally, setup correct file permission for security reasons, enter:

	sudo chown root:root /swapfile1
	sudo chmod 0600 /swapfile1