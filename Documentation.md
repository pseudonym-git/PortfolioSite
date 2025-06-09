# Step 1. Creating the Cloud Container in AWS

1. Starting with a fresh EC2 instance, which can be obtained from [AWS](/https://ap-southeast-2.console.aws.amazon.com/ec2/home?region=ap-southeast-2#Overview:) 

2. Choose "Launch Instance" to navigate to the set up screen, and select the following settings:

| Option                    | Selected Setting                                                                                                                                                                                                                                                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Application and OS Images | Ubuntu Server 24.04 LTS (HVM),EBS General Purpose (SSD) Volume Type.                                                                                                                                                                                                                                                                  |
| Instance Type             | t2.micro (free tier eligible)                                                                                                                                                                                                                                                                                                         |
| Key Pair                  | PortfolioWebsite                                                                                                                                                                                                                                                                                                                      |
| Network Settings          | Most of these settings can be left on default. The really important things to set are in the field for: <br><br>*Security Group*<br>You can either use an existing security group or a new one, but you must have it set up to allow traffic from SSH, HTTP and HTTPS, and I recommend setting to allow from 0.0.0.0 (any IP address) |
| Configure Storage         | Leave at 8 GiB gp3 (or whatever is default) <br><br>This should be plenty                                                                                                                                                                                                                                                             |
3. Choose "Launch Instance"


# Step 2. Setting up a Web Server

1. SSH into the EC2 instance
2. When you are first using a Linux machine it's good practise to check and install any updates to packages. You can do this using the below command

	`sudo apt update && sudo apt upgrade -y`

3. Use the below commands to install Apache, we are going to use this to set up our domain and to HTTPS

	`sudo apt install apache2 -y'
	`sudo systemctl start apache2'
	`sudo systemctl enable apache2`

4. These commands will install Apache, start the web server, and enable automatic start up of Apache as long as the EC2 Instance is running
5. We should now see the Apache default start page when navigating to the EC2 instance's public IP address.
	- Note that this will not yet be secured with HTTPS, we will set this up in the next step


# Step 2.5. Registering a Domain and Pointing it to your site
- This step can be completed at any point in the process, and ideally the earlier the better, as some aspects will take time, eg DNS Propogation

1. I've used Hostinger to register my domain, but you can use any domain registrar, the process should be similar for all providers

2. Select the domain you are using for your website and choose "Manage" and then "DNS/Name Servers"

3. Now we are going to input the Public IP address for our web server. Fill out the fields as described below:
	| Field | Input value |
	|-------|-------------|
	| Type | A |
	| Name | leave blank or use @ |
	| Points to | Your Public IP address, eg 1.54.203.14 |
	| TTL | 14400 |

4. Select "Add Record"

5. Make sure to remove any other A records from the list, having more than 1 may cause problems when using the domain to connect to your server

- After you've added the record this will take time to be reflected in DNS servers, meaning you may not be able to connect to your web server using that domain name immediately. I recommend doing this step last thing at night, then resuming the next day. 


# Step 3. Setting up HTTPS on your web server
- Before starting this step, make sure DNS propogation has finished. Try navigating to your site using the domain name, if you can connect, all good to continue.

- Note: I'm going to do something a bit different here in the interests of being resilient against change to Certbot. This is the direct link to the Certbot Instructions page I've followed for my web server, including the software and the platform I'm running the server on: [LINK](https://certbot.eff.org/instructions?ws=apache&os=pip). 

Navigating to this link ensures that you are using the most up to date instructions for our set up.


# Step 4. Installing Hugo
- Hugo is going to be our website building framework. I've chosen it mostly because it's free, and uses a permissive Apache 2.0 Licence. The other really cool thing about Hugo is that pages are build based on .md or markdown files, something that makes it really easy to add and maintain content. Finally, it also has the benefit of being relatively lightweight, which will be good for keeping within the limits of the free plan we've chosen.

From here, we will effectively be following the Hugo documentation, from which I've pulled the following commands.

1. Start by SSH-ing into your EC2 instance 
2. as should be part of the usual connection routine, run the below command to update all packages:

	`sudo apt update && sudo apt upgrade -y`

3. Now that we are connected and updated, we'll install Hugo using Snap by running the below command:

	`sudo snap install hugo`

4. May or may not be applicable: Adding the path to the .bashrc file.
	- If you try running the command "hugo version" and get an error here like me. Run the following commands to fix it:

	`echo 'export PATH="/snap/bin:$PATH"' >> ~/.bashrc`

	and 

	`source `/.bashrc`

	These commands will "teach" the ubuntu instance where the Hugo commands are.


# Step 5. Creating your Hugo Site

1. Use `git clone x` where x is the https link to your repo

2. Then `cd` into the repo, and run the command `hugo new site portfolio` (portfolio will be the name of the site in this instance)

3. This will create a new directory inside your repo containing:
	- .DS_Store
	- .hugo_build.lock
	- archetypes
	- hugo.toml
	- public
	- themes

4. Installing our Hugo Theme
- You may want to omit or modify this step as you choose, I've decided to go with the [Risotto theme](https://github.com/joeroe/risotto/tree/main) because it is, like Hugo, free, and is licensed under the MIT license. 

	a. Run the command `git submodule add https://github.com/joeroe/risotto.git themes/risotto` to clone the Risotto theme into the themes directory

	b. Use the command `nano hugo.toml` and add a line containing: theme = 'risotto' to instruct Hugo to use this theme when building the site.

	c. I've also copied in the rest of the hugo.toml file from the sample site for the Risotto theme to give me a solid starting place.

5. Now, we fill out the hugo.toml file, think of this as the instructions Hugo (the service/website builder) is going to follow when generating the files which make up our site, you can find my hugo.toml file on my [GitHub repo for this project](https://github.com/pseudonym-git/PortfolioSite/tree/main)

6. Run the command `hugo` to build the site, this will create the public directory we are going to move and display in step 6.


# Step 6. Moving your public directory to the Apache root directory
- In this step, we are going to take the files generated by Hugo in public directory and display them to the internet using our Apache web server we created in Step 2.

1. in the portfolio directory, run the command `sudo cp -r public/* /var/www/html/` to copy the contents of the public directory into the root directory for the Apache web server, then the commands `sudo chown -R www-data:www-data /var/www/html/` and `sudo chmod -R 755 /var/www/html/` to make sure file access permissions are properly set up.

2. Please see Step 6.5 for a script to make the ongoing maintenance of this site much easier.


# Step 6.5 Site Publishing Script:

1. Start by, after ssh-ing into your AWS instance, running the command `sudo nano /usr/local/bin/publish_portfolio`, to create the publish_portfolio script

2. When the file opens, paste in the script below and save the file:

`#!/bin/bash

HUGO_SITE_PATH="/home/ubuntu/PortfolioSite/portfolio"
APACHE_ROOT="/var/www/html"
WEB_USER="www-data"

echo "Starting..."

cd "$HUGO_SITE_PATH" || exit 1

git stash push -m "Auto-stash before deployment" -- public/ 2>/dev/null || true

echo "Running git pull to pull the latest changes..."
git pull origin main || {
    echo "git pull failed"
    exit 1
}

echo "Rebuilding site..."
rm -rf public/
hugo || {
    echo "Hugo build failed, check error log"
    exit 1
}

echo "Publishing site on Apache web server..."
sudo rm -rf "$APACHE_ROOT"/*
sudo cp -r public/* "$APACHE_ROOT/"
sudo chown -R "$WEB_USER:$WEB_USER" "$APACHE_ROOT"
sudo chmod -R 755 "$APACHE_ROOT"

echo "Completed successfully!"
echo "Portfolio website is available at https://lurchingabomination.space/"`

4. Run the command `sudo chmod +x /usr/local/bin/publish_portfolio` to make the script executable. 

5. This works by doing the following:
	a. Navigate to the directory for the Hugo site
	b. Discarding any local changes created as a result of the `hugo` command to build the site.
	c. Run the command git pull to fetch any new changes from the main branch of the git repo
	d. Deletes the existing public directory, then rebuilds it using teh `hugo` command to rebuild the site
	e. Then automatically executes the instructions in step 6, point 1. 


# Step 7. Ongoing Updates and Maintenance
- Whenever you have changes added to the git repo, follow the below steps:

1. ssh into the AWS instance
2. Run this command: `sudo apt update && sudo apt upgrade -y` to update all packages and keep the instance up to date.
3. Run the command `publish_portfolio` to pull the changes from Github


# Step 8. Adding pages to the site.
- Follow the below instructions when writing new articles/pages to be included in the my_projects or blog parent pages.

1. Hugo new command:

*For the my_projects section* 
a. Run the command `hugo new my_projects/project_name.md` where you replace the text "project_name" with the name of your project post.

*For the blog section*
b. Run the command `hugo new blog/post_name.md` where you replace the text "post_name" with the name of your blog post.

2. Edit the .md file using regular markdown text.

