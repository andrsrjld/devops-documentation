## Introduction
The LEMP software stack is a group of software that can be used to serve dynamic web pages and web applications. This is an acronym that describes a Linux operating system, with an Nginx (pronounced like “Engine-X”) web server. The backend data is stored in the MySQL database and the dynamic processing is handled by PHP.

This guide demonstrates how to install a LEMP stack on an Ubuntu 18.04 server. The Ubuntu operating system takes care of the first requirement. We will describe how to get the rest of the components up and running.

## Prerequisites
Before you complete this tutorial, you should have a regular, non-root user account on your server with `sudo` privileges. Set up this account by completing our [initial server setup guide for Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04).

Once you have your user available, you are ready to begin the steps outlined in this guide.

## Step 1 – Installing the Nginx Web Server
In order to display web pages for your site visitors, you’re going to employ Nginx, a modern, efficient web server.

All of the software used in this procedure will come from Ubuntu’s default package repositories. This means you’ll use the apt package management suite to complete the necessary installations.

Since this is your first time using apt for this session, start off by updating your server’s package index:

`sudo apt update`

Next, install the server:

`sudo apt install nginx`

On Ubuntu 18.04, Nginx is configured to start running upon installation.

If you have the `ufw` firewall running, as outlined in the initial setup guide, you will need to allow connections to Nginx. Nginx registers itself with `ufw` upon installation, so the procedure is rather straightforward.

It is recommended that you enable the most restrictive profile that will still allow the traffic you want. Since you haven’t configured SSL for your server in this guide, you will only need to allow traffic on port `80`.

Enable this by typing the following:

`sudo ufw allow 'Nginx HTTP'`

You can verify the change by checking the status:

`sudo ufw status`

This command’s output will show that HTTP traffic is allowed:

> Output
> Status: active
> 
> To                         Action      From
> --                         ------      ----
> OpenSSH                    ALLOW       Anywhere
> Nginx HTTP                 ALLOW       Anywhere
> OpenSSH (v6)               ALLOW       Anywhere (v6)
> Nginx HTTP (v6)            ALLOW       Anywhere (v6)

With the new firewall rule added, you can test if the server is up and running by accessing your server’s domain name or public IP address in your web browser.

If you do not have a domain name pointed at your server and you do not know your server’s public IP address, you can find it by running the following command:

`ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'`

This will print out a few IP addresses. You can try each of them in your web browser.

As an alternative, you can check which IP address is accessible, as viewed from other locations on the internet:

`curl -4 icanhazip.com`

Type the address that you receive in your web browser and it will take you to Nginx’s default landing page:

`http://server_domain_or_IP`

[](https://assets.digitalocean.com/articles/lemp_ubuntu_1604/nginx_default.png)
> If you received a web page that states ”Welcome to nginx” then you have successfully installed Nginx.

## Step 2 – Installing MySQL to Manage Site Data

Now that you have a web server, you need to install MySQL (a database management system) to store and manage the data for your site.

Install MySQL by typing the following command:

`sudo apt install mysql-server`

The MySQL database software is now installed, but its configuration is not yet complete.

To secure the installation, MySQL comes with a script that will ask whether you want to modify some insecure defaults. Initiate the script by typing the following:

`sudo mysql_secure_installation`

This script will ask if you want to configure the `VALIDATE PASSWORD PLUGIN`.

> Warning: Enabling this feature is a judgment call. If enabled, passwords that don’t match the specified criteria will be rejected by MySQL with an error. This will cause issues if you use a weak password in conjunction with software that automatically configures MySQL user credentials, such as the Ubuntu packages for phpMyAdmin. It is safe to leave validation disabled, but you should always use strong, unique passwords for database credentials.

Answer `Y` for yes, or anything else to continue without enabling.

> VALIDATE PASSWORD PLUGIN can be used to test passwords
> and improve security. It checks the strength of password
> and allows the users to set only those passwords which are
> secure enough. Would you like to setup VALIDATE PASSWORD plugin?
> 
> Press y|Y for Yes, any other key for No:

If you’ve enabled validation, the script will also ask you to select a level of password validation. Keep in mind that if you enter 2 – for the strongest level – you will receive errors when attempting to set any password which does not contain numbers, upper and lowercase letters, and special characters, or which is based on common dictionary words.

There are three levels of password validation policy:

> LOW    Length >= 8
> MEDIUM Length >= 8, numeric, mixed case, and special characters
> STRONG Length >= 8, numeric, mixed case, special characters and dictionary file
> 
> Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1

Next, you’ll be asked to submit and confirm a root password:

>Please set the password for root here.
>
>New password:
>
>Re-enter new password:

For the rest of the questions, you should press `Y` and hit the `ENTER` key at each prompt. This will remove some anonymous users and the test database, disable remote root logins, and load these new rules so that MySQL immediately respects the changes we have made.

Note that in Ubuntu systems running MySQL 5.7 (and later versions), the root MySQL user is set to authenticate using the `auth_socket` plugin by default rather than with a password. This allows for some greater security and usability in many cases, but it can also complicate things when you need to allow an external program (e.g., phpMyAdmin) to access the user.

If using the `auth_socket` plugin to access MySQL fits with your workflow, you can proceed to Step 3. If, however, you prefer to use a password when connecting to MySQL as root, you will need to switch its authentication method from `auth_socket` to `mysql_native_password`. To do this, open up the MySQL prompt from your terminal:

`sudo mysql`

Next, check which authentication method each of your MySQL user accounts uses with the following command:

`SELECT user,authentication_string,plugin,host FROM mysql.user;`

> Output
> +------------------+-------------------------------------------+-----------------------+-----------+
> | user             | authentication_string                     | plugin                | host      |
> +------------------+-------------------------------------------+-----------------------+-----------+
> | root             |                                           | auth_socket           | localhost |
> | mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
> | mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
> | debian-sys-maint | *CC744277A401A7D25BE1CA89AFF17BF607F876FF | mysql_native_password | localhost |
> +------------------+-------------------------------------------+-----------------------+-----------+
> 4 rows in set (0.00 sec)

This example demonstrates that the root user does in fact authenticate using the `auth_socket` plugin. To configure the root account to authenticate with a password, run the following `ALTER USER` command. Be sure to change password to a strong password of your choosing:

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`

Then, run `FLUSH PRIVILEGES` to tell the server to reload the grant tables and put your new changes into effect:

`FLUSH PRIVILEGES;`

Check the authentication methods employed by each of your users again to confirm that root no longer authenticates using the `auth_socket` plugin:

`SELECT user,authentication_string,plugin,host FROM mysql.user;`

> Output
> +------------------+-------------------------------------------+-----------------------+-----------+
> | user             | authentication_string                     | plugin                | host      |
> +------------------+-------------------------------------------+-----------------------+-----------+
> | root             | *3636DACC8616D997782ADD0839F92C1571D6D78F | mysql_native_password | localhost |
> | mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
> | mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
> | debian-sys-maint | *CC744277A401A7D25BE1CA89AFF17BF607F876FF | mysql_native_password | localhost |
> +------------------+-------------------------------------------+-----------------------+-----------+
> 4 rows in set (0.00 sec)

This example output shows that the root MySQL user now authenticates using a password. Once you confirm this on your own server, you can exit the MySQL shell:

`exit`