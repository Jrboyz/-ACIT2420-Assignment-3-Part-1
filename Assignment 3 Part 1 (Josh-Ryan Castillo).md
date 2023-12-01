# Droplet Setup (Digital Ocean)
---
Before continuing, ensure that you have a Fresh Debian 12 server on DigitalOcean.
## SSH Key Generation
We can use SSH to connect to the Droplet we're going to make later. To do this we will use the **`ssh-keygen`** utility.

1. Create a new .ssh directory using
	**`mkdir .ssh`**
2. Create a new SSH key pair using
	**`ssh-keygen -t ed25519 -f .ssh/do-key -C "your-email-address"`**

## Adding SSH Key to your DigitalOcean account
1. On DigitalOcean, go to the Settings page and click the Security tab. Then click the **Add SSH Key** button.
2. Windows Users: In PowerShell, you can copy your ssh public key using
	**`Get-Content C:\Users\user-name\.ssh\do-key.pub | Set-Clipboard`**
	MacOS Users: In Terminal, you can copy your ssh public key using
	**`pb copy < ~/.ssh/your-key.pub`**
1. Paste the key you just copied into the SSH Key content text field. Give it an appropriate name and click **Add SSH Key**

## Droplet Creation
1. In DigitalOcean, on the left-hand dropdown click **Droplets**, then **Create->Droplet**
2. Options that are not specified are up to you. Select **Debian** 
3. Under the **Choose Authentication Method**, select **SSH Key** and check the box next to the SSH key you added in the previous step
4. Finally, click **Create Droplet**

You now have a Droplet that you can SSH to!

# Regular User Creation + Root User Restriction
---
Next we will create a regular user that you can use instead of the root user.

**Regular User Creation**
1. Connect to your newly created Droplet as the root user using your droplet IP address in Powershell or Terminal
	**`ssh -i path-to-your-key root@your-droplet-ip`**
2. Type "yes" when prompted while connecting to your droplet
3. Create a new user using useradd
	**`useradd -ms /bin/bash <user-name>`**
4. Then give that user privileges by adding it to a group
	**`usermod -aG sudo <user-name>`** 
5. Give the user a password with
	**`passwd <user-name>`**

**Connect with the new User, and restrict root user**
1. Connect using
	**`ssh -i path-to-your-key <user-name>@your-droplet-ip`**
2. Type "yes" when prompted while connecting to your droplet
3. Run the following command to restrict root user 
	**`sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config`
4. Restart the SSH service using the following command
	**`sudo systemctl restart ssh.service`**

You have now created a new user that you can SSH to your droplet with, and the root user can no longer SSH to your droplet.

# Nginx Setup and Execution
---
Next we will setup Nginx and configure a sample website for it to display.

**Nginx Setup**
1. Run `sudo apt update`. It is good practice to run this command before installing anything.
2. Run `sudo apt install nginx`
3. Start Nginx using `sudo systemctl start nginx` and enable it using `sudo systemctl enable nginx`
4. Confirm that Nginx is working by checking its status
	`sudo systemctl status nginx`

**Web Server Creation**
1. Change directories to /var/www
	`cd /var/www`
2. Create a new directory called "my-site"
	`sudo mkdir my-site`
3. Go into that directory and create a new file called "index.html"
	`sudo vim index.html`
4. Paste the following code into the index.html file
```
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
```
5. Save and quit the file

# Server Block Creation
---
Next we will create a config file that runs our server that holds the website we just created.

**Config File Creation**
1. Change directories into **/etc/nginx/sites-available**
2. Create a file called my-site.conf and edit it using vim
	`sudo vim my-site.conf`
3. Paste the following code into the .conf file
```
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
}
```

**Symbolic Link Creation**
Next, we have to create a symbolic link to the config file you said made.
1. Run the following command
	`sudo ln -s /etc/nginx/sites-available/my-site /etc/nginx/sites-enabled`
2. Unlink the default file so that the correct site is served
	`sudo unlink default`

**Testing**
We can double check if everything is working by looking at its status or the actual site
1. If there's an error, it can be seen by running the following command
	`sudo nginx -t`
2. If there are no errors, restart the nginx service
	`sudo systemctl restart nginx`
3. Your site should now be live and visitable. You can confirm this by running the following command and checking if the result is the index.html page we made earlier
	`curl <your-droplet-ip>`

You now have a site running using nginx!













