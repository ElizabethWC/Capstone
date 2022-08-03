Capstone  Project Summary 
Overview

At the end of this capstone I should know how to…
Deploy a PHP app 
Create an RDS to speak with the PHP app
Install Lambda Stack
Update an aws system manager parameter store
Secure app to prevent public access to the backend
Download assets to import data to RDS and create PHP
Create a Cloud9 IDE and use it to deploy the PHP app
Access Cloud9 to import data to RDS in private subnet and connect to bastion host 
Create an app load balancer that will be internet facing, accepting traffic from the internet and forwarding to the EC2 instances on the private subnet
Create an auto-scaler
Task 1) Inspect your existing Architecture
Note that the following items are already available in your environment.
 VPC
2 public subnets in 2 availability zones
2 private subnets in 2 availability zones
Bastion host
Several security groups
Example DBSG (only app can access the db)
NAT Gateway
A launch template
     
Task 2)  Create Cloud9 IDE
In the AWS Management Console, from the Services menu, choose Cloud9.
Select 
Under Environment name and description
Name: Project IDE OR cd_webserver
Select 
Under Environment settings
  Environment Type: Create a new EC2 instance for environment (direct access) #default
Instance type: t2.micro (1 GiB RAM + 1 vCPU)
 Platform: Amazon Linux 2 #default
Cost-saving setting: After 30 min #default
Network settings (advanced)
Network (VPC) : Example VPC
Subnet: Pubic subnet 2
Select 
Under Review
Select 
Task 3) Security Group and IAM Role edits

In the AWS Management Console, on the Services menu, choose           VPC.
In the left navigation pane, choose Security Groups. 
Select the aws-cloud9-Project-IDE security group.
Select the Inbound rules tab.
Select Edit inbound rules.
Select Add rule. And add a new inbound security rule with the following values:
Type: HTTP
Port Range: 80
Source: 0.0.0.0/0
      Select the Bastion-SG security group.
Select the Inbound rules tab.
Select Edit inbound rules
Select Add rule and configure these settings:
Type: MySQL/Aurora
Port Range: 3306
Source: 0.0.0.0/0
Select Add rule and configure these settings:
Type: HTTPS
Port Range: 443
Source: 0.0.0.0/0
Select Add rule and configure these settings:
Type: SSH
Port Range: 22
Source: 0.0.0.0/0
  Select .
      Select the Example-DBSG security group.
Select the Inbound rules tab.
Select Edit inbound rules
Select Add rule and configure these settings:
Type: MySQL/Aurora
Port Range: 3306
Source: 0.0.0.0/0
Select Add rule and configure these settings:
Type: HTTP
Port Range: 80
Source: 0.0.0.0/0
Select .

In the left navigation pane, choose Instances. 
 Select the Bastion instance.
Select Actions.
In the drop-down menu, select Security, then Modify IAM role.
Select Inventory-App Role.
Select .
Task 4) Create Target Group
 On the Services menu, choose EC2.
In the left navigation pane, choose Target group (you might need to scroll down to find it).
 Select .
 Under Basic configuration, keep all default selections. And configure, 
Target group name: appgroup
VPC: Example VPC.
 Select .
 Select .  
Task 5) Create an ALB
      On the Services menu, choose EC2.
In the left navigation pane, choose Load Balancers (you might need to scroll down to find it).
      Choose .
Under Application Load Balancer, choose Create.
      For Load balancer name, enter: appelb
Scroll down to the Network mapping section, then for VPC, select
Example VPC.
      Under Mappings, choose the first Availability Zone, then choose the       Public Subnet that displays.
Choose the second Availability Zone, then choose the Public Subnet that displays.
You should now have selected two subnets: Public Subnet 1 and Public Subnet 2. (If not, go back and try the configuration again.)
      In the Security groups section, deselect the default and select the ALBSG security group.
 In the Listeners and routing section, under  Listener HTTP:80, Select appgroup.
Select .
Task 6) Create an Auto-Scaling Group #Module 9 – Challenge Lab, Task 5
On the Services menu, choose EC2.
      In the left navigation pane, choose Auto Scaling Groups (you might    need to scroll down to find it).
      Select .
      Under Choose launch template or configuration, configure these settings.
Auto Scaling group name: appautoscale
Launch template: Example-LT
Select .
     Under Choose instance launch options, configure these settings.
VPC: Example VPC
Availability Zones and subnets: 
private subnet 1
private subnet 2
Select .
Under Choose advanced options, configure these settings.
Load balancing - optional: Attach to an existing load balancer
Attach to an existing load balancer: 
Select Choose from your load balancer target groups
Existing load balancer target groups: appgroup | HTTP
     Select .
Under Group size - optional, configure these settings.
Desired capacity: 2
Minimum capacity: 1
Maximum capacity: 3
 Select .
Select .
      Select .
Select .








Task 7) Connect to your bastion host. #Module 6 - Challenge Lab Task 4, 
      In the Capstone Project window, Select .
A Credentials window opens.
Choose the Download PEM button and save the labsuser.pem file.
Exit the Details panel by choosing the X.
Connect to your instance. 
      Open your downloads folder and drag the labsuser.pem file into your instance.
      In the Bash terminal window at the bottom of the screen, paste and run these commands:
ssh ec2-user@<bastionhostpublicipv4> # replace <bastionhostpublicipv4> with your Bastion EC2PublicIP
yes
chmod 400 labsuser.pem
ssh -i labsuser.pem ec2-user@<bastionhostpublicipv4>
Task 8) Install a LAMP web server on Linux 2 in your bastion host #https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html and Module 4 Tasks 2&3
Connect to your instance. 
In the Bash terminal window at the bottom of the screen, paste and run these commands:

#To prepare the LAMP server
sudo yum update -y #update all the packages 
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2

sudo yum install -y httpd mariadb-server

sudo chkconfig httpd on

sudo service httpd start

#To set file permissions…

sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
find /var/www -type f -exec sudo chmod 0664 {} \; 
cd /var/www/html
#Add assets to webpage…  
wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/ILT-TF-200-ACACAD-20-EN/capstone-project/Example.zip
unzip Example.zip -d /var/www/html/
mv ./Example/* .
wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/ILT-TF-200-ACACAD-20-EN/capstone-project/Countrydatadump.sql

      Test your web server. In a new browser tab, load http://<bastionhostpublicipv4>
The website should display. Keep this browser tab open for later in the lab.
Task 9) Create RDS in two Availability Zones #Module 5 Guided Lab
      On the Services menu, choose RDS.
 Choose 
      If the top of the screen shows Switch to the new database     creation flow, choose it.
Under Engine options, select MySQL.
Under the Templates section, select Dev/Test.
     Under Availability and durability, under Deployment options, select Single AZ. (This option does not offer high availability, but it costs less.)
     Under the Settings section, configure these options:
DB instance identifier: Example
Username: admin
Password: lab-password
Confirm password: lab-password
          Copy your answers, you will need these later.
     Under the DB instance class section, configure these options:
Select Burstable classes (includes t classes).
Select enable previous generations.
Select T2.small instance.
     Under the Connectivity section, configure these options:
Storage: General SSD
Virtual Private Cloud (VPC): Example VPC
Public access: No
Existing VPC security groups: ExampleDB. It will be highlighted.
Note: Deselect the default
Availability Zone: us.east.1b
Database port: Keep the default TCP port of 3306.
     Expand the Additional configuration panel, then configure these      settings:
Initial database name: example
Note: This is the logical name of the database that will be used by the application.
Clear (turn off) the Enable Enhanced monitoring option.
      Feel free to review the many other options displayed on the page, but leave them set to their default values. Options include automatic backups, the ability to export log files, and automatic version upgrades. The ability to activate these features through check boxes demonstrates the power of using a fully managed database solution instead of installing, backing up, and maintaining the database yourself.
     Choose  (at the bottom of the page).
     You should receive a message indicating that your database is being created. Before you continue to the next task, the database instance status must be Available. This process might take several minutes.
Task 10) Connect to your RDS 
     Connect to your instance. 
In the Bash terminal window at the bottom of the screen, paste and run these commands:
mysql -u admin -p<rdspassword> -h <rdsendpoint>
<rdspassword>
CREATE DATABASE [<databasename>];
exit
mysql -u admin -p<rdspassword> -h <rdsendpoint> [<databasename>] Countrydatadump.sql

Task 11) Configure System Manager
On the Services menu, choose System Manager.
In the left navigation pane, choose Parameter Store.
     Select .
     Under Parameter details, configure these settings.
Name: /example/endpoint #name from capstone solution requirements
Value:  example.cfh2g3ltbab1.us-east-1.rds.amazonaws.com #RDS endpoint
     On the Services menu, choose System Manager.
In the left navigation pane, choose Parameter Store.
     Select .
Under Parameter details, configure these settings.
Name: /example/username
Value: admin #RDS username
     On the Services menu, choose System Manager.
In the left navigation pane, choose Parameter Store.
Select .
Under Parameter details, configure these settings.
Name: /example/password
Value: lab-password #RDS password
     On the Services menu, choose System Manager.
In the left navigation pane, choose Parameter Store.
Select .
Under Parameter details, configure these settings.
Name: /example/database
Value: example #DB identifier
Task 12)  Check that the website still displays and queries work.
Task 13) Submit!!!


