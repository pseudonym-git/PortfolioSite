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
- Hugo is going to be our website building framework. I've chosen it mostly because it's free, and uses a permissive Apache 2.0 Licence. It also has the benefit of being relatively lightweight, which will be good for keeping within the limits of the free plan we've chosen.

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

1. The 1st step is to run the command `hugo new site portfolio` (portfolio will be the name of the site in this instance)

2. Then `cd portfolio` to enter the new directory, and `git init` to initialise it as a git repo 
