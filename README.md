# Route-53_ACM_Cloudfrontt
### Using this document you will be able to deploy your application on EC2 instance behind an Application Load Balancer and its configuration with AWS Certificate Manager, Route 53 and Cloudfront.

## Deploy Application on EC2 instance. 
- Search VPC in AWS search bar.
- Create a VPC with CIDR: 10.1.0.0/16
- Make sure **`Block Public Access`** is set to **`off`** 
- Create 3 subnets.
- Subnets with CIDR: 10.1.1.0/24, 10.1.2.0/24 and 10.1.3.0/24. Make sure each subnet is in different Availability Zone.
- Out of these three subnets 2 will be public and 1 will be private.
- Now create an Internet gateway and attach it with the VPC you created earlier.
- Next is to create Route table. Create Route Table with your VPC. Then, click on Subnet association > Edit subnet association > Select two subnets.
- In the route table go to Routes > Edit routes > Add Route > Destination : 0.0.0.0/0 > Target: Internet gateway (Select your IGW.)
- Now create 3 EC2 instances. Go to EC2 > Instance > Launch Instances 
AMI: Ubuntu
Instance Type: t2.micro
Create a key pair
Network Settings:
VPC: Faraz VPC
Subnet: Create each instance in different subnets
Security Group: Keep inbound port 80 and 443 Open for all (0.0.0.0/0)
Launch instance
- You can install an nginx server in all three instances and place your index.html at /var/www/html/.
- To ensure proper functioning of the web-server, you can hit the public IP of intances in browser and get the webpage.

## Create an Application Load Balancer.
- For creating load balacer, first you will have to create a target group
- Search EC2 in AWS search bar.
- Select Target Group from the side panel
- Create Target Group 
Target Type: Instances
Protocol:HTTP/80
IP Address type: IPv4
VPC: Select VPC you created earlier
Next
In Register target section, select all 3 instances > Include as pending below > Create Target Group.
- Now again, search EC2 in AWS search bar.
- Select Load Balancers from the side panel. 
- Create Load Balancer > Create Application Load Balancer
Scheme: Internet-facing
Load balancer IP address type: IPv4
VPC: Select VPC you created
Availability Zones and subnets: Select all three AZs in which you created the instances
Select Security Group you created for the instances.
Listeners and Routing: 
Listnere 1. 
Protocol: HTTP
Port: 80
Listnere 2. 
Protocol: HTTPS
Port: 443
Target Group: Select the target group you created.
Create Load Balancer.
Only when the HTTP and HTTPs port are set to 0.0.0.0/0 in security group, your instances will get healthy.
- Now, when you will hit the Load Balancer DNS name in browser, you will be directed to your web page.

## Route 53 Configuration 
- Search Route 53 in AWS search bar.
- Craete hosted zone.
Domain name: faraz.buzz
Type: Public hosted zone
Create hosted zone
- It will give you a few name servers in "Value/Route traffic to" section.
- Go to Go Daddy > My Account > Domaind > DNS > Name servers > Change Nane Servers > Add name servers you got from Route 53 > Save 

## Create and SSL Certificate using AWS Certificate Manager.
- Search AWS Certificate Manager in AWS search bar.
- You must select "us-east-1" as your region.
- Click on "Requestt a certifcate".
FQDN: *.faraz.buzz
Certificate type: Request a public certificate
Validation Method: DNS Validation
Key Algorithm: RSA 2048
- Now, click on "Create Records in Route 53" and "Create Records"

## Cloudfront configuration
- Search Cloudfront in AWS search bar.
- Create Distribution:
Distribution Option: Single website or app
Origin: Selec your elb
Protocol: HTTP only
Viewer protocol policy: Redirect HTTP to HTTPS
Allowed HTTP method: GET, HEAD, OPTIONS, POST, PATCH, DELETE
Cache key and origin requests: Cache policy and origin request policy 
Cache policy: Caching disabled
Origin request policy: AllViewer
Custom SSL Certificate: Select the ACM certificate created earlier.
Select TLSv1_2_2021
Alternate DNS name: abc.faraz.buzz
Create Distribution

## Now again go to Route 53:
Create Record :
Record name: abc.faraz.buzz
Record Type: A
Turn on the toggle for Alias
Route traffic to "Alias to cloud front distribution"
Copy paste the cloud front distrbution domain name in Alias
Create record.

## Search **`http://abc.faraz.buzz`** in browser the url will be redirected form http to https and the content will also be served.
