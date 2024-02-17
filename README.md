AWS THREE-TIER WEB ARCHITECTURE IMPLEMENTATION STEPS:

NETWORK AND SECURITY:

1. Created an S3 bucket in the us-east-1 region. This is where our code will be stored later
which will be used to deploy the app
2. Created an IAM role for EC2 and attached SSMManagedInstanceCore, and
S3ReadOnlyAccess permissions policies to it
3. Created an isolated network VPC with a CIDR range of 10.0.0.0/16, within which we will be
building out the networking components like subnets, route tables, internet gateway, NAT
gateway, and security groups
4. Created 6 subnets where 3 subnets are configured to be in one availability zone (useast-
1a), and the other 3 subnets to be in another availability zone (us-east-1b). Each
subnet in one availability zone will correspond to each layer of our three-tier architecture
5. Created an Internet Gateway and attached it to the VPC, so that the public subnets will
have access to the internet
6. In order to provide internet access to the app instances in private subnets, created two NAT
Gateways and deployed one in each public subnet in each availability zone for high
availability
7. Created a public route table for the public web subnets and added internet gateway in the
routes, so that the IPs outside of the VPC’s CIDR range will be routed to the internet
gateway. In addition to that, explicitly associated both the public subnets with the route
table
8. Created 2 more route tables for the app layer private subnets that are in 2 availability zones.
These route tables will route traffic destined for outside VPC to the NAT gateway in the
corresponding availability zone. For that, edited routes, added NAT gateway and
associated the corresponding private app subnet and repeated the same for another
private app subnet in another availability zone
9. Created 5 security groups
i. First security group is for the public, internet facing load balancer with an inbound rule
that allows HTTP traffic for my IP
ii. Second security group is for the public instances in the web tier, added HTTP inbound rule
to allow internet facing load balancer security group that is created previously and also added
a rule to allow HTTP traffic from my IP
iii. Third security group is created for the internal load balancer with an inbound rule that allows
HTTP traffic from the web tier security group which allows traffic from the web instances to hit
the internal load balancer
iv. Fourth security group is created to allow traffic from the internal load balancer to hit the
private app instances. For this, added inbound rule with internal load balancer security group
created before and for TCP port 4000 to allow my IP as well
v. Fifth security group is for the private database instances. Added an inbound rule with private
app instances security group to MySQL/Aurora port 3306

DATABASE-TIER DEPLOYMENT:

1. Created a database subnet group with VPC we created and added the private subnets that
were created for the database layer in two availability zones
2. Created a database configured with MySQL compatible Amazon Aurora as my database
engine, in a dev/test environment. Set up a username and password which will be used
later to access the database. Selected the option to create an Aurora Replica or reader
node in a different avail28ability zone for high availability and durability. After the database
is provisioned, writer instance and reader instance were created in different availability
zones

APP-TIER INSTANCE DEPLOYMENT:

1. Launched an EC2 app instance for the private app tier configured with Amazon Linux, t2
micro, VPC, private app subnet in one of the availability zones, and with security group
created for private app tier attached. Also selected IAM EC2 role we have created in the
very first step
2. Connected to the instance using Session Manager and verified if I am able to ping the
google DNS servers and successfully packets started transmitting, which means the
network is configured correctly
3. Installed mysql server on my app instance, and connected to my DB instance by using
Aurora RDS writer endpoint, user name and password created while creating the database.
Created a table in my database and inserted some data into it which will later be used in
testing
4. Configured my writer instance endpoint, username, password, and my database name in
DBConfig.js file and uploaded the app-tier folder to my S3 bucket
5. Using the commands that were shown on workshop documentation, installed all the
necessary softwares, dependencies etc to run our app successfully even when we close
the SSM session or the server is interrupted
6. To test if the app layer is configured correctly, ran health checks to see if the app is running
and is able to connect to the database. Everything seems to be in place so far, and the app
layer is correctly configured

INTERNAL LOAD BALANCING and SCALING:

1. Created an Amazon Machine Image from the app tier instance
2. Created a target group which will be used with the load balancer to balance traffic between
the private app tier instances.
3. Created an internal load balancer for which we used Application Load Balancer that listens
to HTTP port 80, forwarding the traffic to the target group
4. Created a launch template by using the AMI created earlier, with appropriate security group
and IAM EC2 role attached
5. Created an Auto Scaling Group using the launch template and associated this with the load
balancer created earlier. Set my minimum capacity, desired capacity and maximum
capacity to 2
6. Auto Scaling Group successfully provisioned 2 new instances since we set our desired
capacity to 2

WEB-TIER INSTANCE DEPLOYMENT:

1. Updated the nginx configuration file provided as part of AWS workshop with my internal
load balancers’ DNS name, and uploaded this file and the web tier folder to the S3 bucket
created initially
2. Launched a web tier instance similar to that of app tier instance’s configuration but this
time launched web instance in a public web subnet and with web tier security group, IAM
role. Also, enabled auto-assign public-IP
3. Connected to the instance and tested by using ping 8.8.8.8 if the configuration is working
correctly, packets started transmitting and everything seems to be working well
4. After making sure the configuration is correct, installed some required components on the
web instance to run the front-end application. Downloaded web-tier code from my S3
bucket, ran build, installed nginx and configured to serve our application on port 80 and
help send API calls to the internal load balancer
5. After the configuration is completed, using the public IP of the web tier instance, I was able
to see the web app and the connected database as well
External Load Balancing and Auto Scaling:
1. Created an Amazon Machine Image from the web tier instance
2. Created a target group which will be used with the load balancer to balance traffic across
the web tier instances.
3. Created an internet facing load balancer for which we used Application Load Balancer that
listens to HTTP port 80, forwarding the traffic to the target group
4. Created a launch template by using the AMI created earlier, with appropriate security group
and IAM EC2 role attached
5. Created an Auto Scaling Group using the launch template and associated this with the load
balancer created earlier. Set my minimum capacity, desired capacity and maximum
capacity to 2
6. Auto Scaling Group successfully provisioned 2 new instances since we set our desired
capacity to 2
7. Finally, with the help of load balancer DNS name, I was able to land on the AWS 3-Tier Web
App Demo page

