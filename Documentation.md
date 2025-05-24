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

	`sudo apt install apache2 -y 
	`sudo systemctl start apache2 
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

1. 