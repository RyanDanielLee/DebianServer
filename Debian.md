# Ryan Lee Assignment 3 Part 1
This document will help you setup your debian server using Digital Ocean, and a website using nginx.

## Setting up Digital Ocean
Before we start, ensure you have created a server on Digital Ocean correctly. 

### Generate SSH Key
Before you create your server, ensure you have generated an SSH key pair

1. Open your shell or terminal
2. Run the command `ssh-keygen`
3. Then, create a new ssh directory using the command `mkdir .ssh`
4. Finally, create your new ssh key pair with the command `ssh-keygen -t ed25519 -f .ssh/do-key -C "your-email-address"`
5. To copy your ssh key on Windows use the command `Get-Content C:\Users\user-name\.ssh\do-key.pub | Set-Clipboard`
6. To copy your ssh key on MacOS use the commands `pbcopy` then `pbcopy < ~/.ssh/do-key.pub`

### Create a Droplet

1. Login to [Digital Ocean](https://www.digitalocean.com/)
2. On the top right of the screen, click the **Create** button and select **Droplets** from the drop down menu
3. Select **Debian** as your OS, and select all the other options that work for you
4. In the **Choose Authentication Method** option, select **SSH Key**
5. Click **New SSH Key** and paste your key into the field. 

### Connecting to your Droplet
1. In your Digital Ocean dashboard, copy the IP address of the droplet you created. 
2. On your host machine, open your shell or terminal
3. Then connect to your droplet by running the command `ssh -i path-to-your-key root@your_ip`
4. Type **yes** when prompted to connect to the server

## Creating a New Regular User
After you have your server up and running, and have connected to it, we want to create a new regular user. This is a good idea to prevent security issues, and to prevent accidental admin changes. 

### Create the User
**Replace "username" with your desired username**
> useradd -ms /bin/bash "username"

### Add a Password
**MAKE SURE YOU REMEMBER OR WRITE DOWN THE PASSWORD YOU SET TO THE USER**
> passwd "username"

### Add Newly Created User to Sudo Group
Now we want to add our newly created user into the sudo group to grant it administrator privilages, so that we can use higher level commands
> usermode -aG sudo "username"

### Login as Regular User Rather Than Root User
Because of security issues and accidental admin changes, most of the time we do not want to login as the root user. 

First, we would need to copy the **.ssh** directory from root directory into our own directory. You can achieve this with the following command below.
>  cp - r /root/ssh /home/username

Then, we need change the ownership of the previous directory so that we have permissions to copy it. You can do this with the below command.
> sudo chown -R username:username /home/username/.ssh

Then exit the server by running the command: 
> exit

To test if we successfully added the user, we can connect to the droplet using the newly created user, rather than the root user. You can achieve this with the following command.
> ssh -i .ssh/do-key "username"@ip_address

If everything is working, and you are now logged in as your regular user, we want to disable the ability to login as the root user via ssh, to eliminate potential security issues. You can achieve this with the following commands.

We want to edit the **sshd_config** file. To do that you want to change your directories by running the below command. 
> cd/etc/ssh

Then, we want to edit the file, which can be achieved using the text editor, **Vim** using **super user privilages**. To do this, run the following command.
> sudo vim sshd_config

In the text editor, look for the line **PermitRootLogin yes**. Then, go into **insert mode** by pressing **I** on your keyboard. Then change **yes** to **no**. Then press **esc** on your keyboard to go into visual mode, and type **:wq** to save and exit out of the file editor. 

After all that, you want to restart the ssh service with the following command.
> sudo systemctl restart ssh.service

To ensure this is working, try logging in with the root user via ssh.

## Creating the Web Server

### Install nginx
Before we can start, we need to install the nginx package. nginx is the web server which we need in order to run our website. You will first need to check for updates on the latest packages and dependencies. This can be achieved with the following command.
>sudo apt update

If there are available updates then run the following command to install the updates.
>sudo apt upgrade

After that is complete, to install nginx run the following command.
> sudo apt-get install nginx

### Enabling nginx
After installing nginx, we of course want to enable it. This can be done with the following command.
> sudo systemctl enable nginx

### Create the Web Server
To begin, we want to create our web server in the **/var/www directory**. To do this run the following command.
> cd /var/www

Then, create a folder in that directory with the following command.
> mkdir my-site

After, we want to create the index.html file with the following command.
> sudo touch my-site/index.html

Then, change directories into the **my-site** directory. We will then use the text editor, vim, to enter add our html file. Once you are in insert mode, copy the following code into the html file:

    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
    </head>
    <body>
    <h1>Hello, World</h1>
    </body>
    </html>

Then, save and quit the file. 

## Creating the Server Block

### Create Config File
Once we have our server contents implemented, we want to create a config file that actually runs the server. To do this, change directories into **/etc/nginx/sites-available** and create a file using sudo vim called **my-site.conf**. Then, paste the following code.

    server {
    listen 80;
    listen [::]:80;

    root /var/www/my-site;

    index index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }

Now save and quit the file, and change directories into **/etc/nginx/sites-enabled**.

### Create a Symbolic Link
We need to create a symbolic link that links our config file in the **sites-available** directory. To achieve this run the following command.
> sudo ln -s /etc/nginx/sites-available/my-site /etc/nginx/sites-enabled

### Unlink Default File
If you noticed, there is another file named **Default** inside the **sites-available** directory. This is linked by default, and will cause errors if we don't unlink it. To unlink it run the following command.
> sudo unlink default

### Test Config
Now that we have unlinked our default file, we can test to see if our configuration was successful. To do this, run the following command. It should indicate whether there is an error, or if it was successful. 
> sudo nginx -t

After our successful configuration, run the following command to restart the nginx service.
> sudo systemctl restart nginx

Finally, our server should be up and running. To test if this works, you can run the following command in your host machine shell/terminal.
> curl (IP ADDRESS)

## Done
You have successfully created your own web server. 
