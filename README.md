# Route-53_ACM_Cloudfrontt
- This guide will help you deploy your application on **`EC2 instances`** behind an **`Application Load Balancer (ALB)`**, secured with **`AWS Certificate Manager (ACM)`**, domain managed in **`Route 53`**, and optimized using **`CloudFront CDN`**.


- [Deploy Application on EC2 instance](#deploy-application-on-ec2-instance)
- [Create an Application Load Balancer](#create-an-application-load-balancer)
- [Route 53 Configuration](#route-53-configuration)
- [Create SSL Certificate with AWS Certificate Manager](#create-ssl-certificate-with-aws-certificate-manager)
- [Cloudfront configuration](#cloudfront-configuration)

## Deploy Application on EC2 instance 
### 1. Create a VPC
- Go to **AWS Console → VPC**.
- Click **Create VPC**.
- CIDR block: `10.1.0.0/16`.
- Set **Block Public Access** to **off**.


### 2. Create 3 Subnets
- Subnet CIDRs:
  - `10.1.1.0/24`
  - `10.1.2.0/24`
  - `10.1.3.0/24`
- Each subnet in a different **Availability Zone**.
- Make **2 public** and **1 private**.


### 3. Create and Attach Internet Gateway
- Go to **Internet Gateway**.
- Create a new IGW.
- Attach it to your VPC.

### 4. Create Route Table
- Go to **Route Tables → Create Route Table**.
- Associate it with your VPC.
- Associate **2 public subnets**.
- Add Route:
  - Destination: `0.0.0.0/0`
  - Target: Your IGW


### 5. Launch 3 EC2 Instances
- **AMI**: Ubuntu
- **Instance Type**: `t2.micro`
- **Key Pair**: Create or use existing.
- **Network Settings**:
  - VPC: Your VPC
  - Subnets: Create each instance in different subnets
  - Security Group: Open inbound `80` and `443` to `0.0.0.0/0`
- Launch!


- You can install an nginx server in all three instances and place your index.html at /var/www/html/.
- To ensure proper functioning of the web-server, you can hit the public IP of intances in browser and get the webpage.

## Create an Application Load Balancer
### 1. Create Target Group
- Go to EC2 → Target Groups.
- Click Create Target Group:
- **Target Type**: `Instances`
- **Protocol**: `HTTP (port 80)`
- **VPC**: Select your VPC
- Register your 3 EC2 instances.

### 2. Create ALB
- Go to EC2 → Load Balancers → Create ALB.
Settings:
- **Scheme**: `Internet-facing`
- **IP Type**: `IPv4`
- **VPC**: `Select your VPC`
- **Subnets**: `Select 3 AZs`
- **Security Group**: `Open ports 80 and 443.`

### Listeners:
- **Listener 1**: `HTTP (80)`
- **Listener 2**: `HTTPS (443)`
- **Target Group**: `Select created group`
**`Launch the ALB`**

- Only when the HTTP and HTTPs port are set to **`0.0.0.0/0`** in security group, your instances will get healthy.
- Now, when you will hit the Load Balancer DNS name in browser, you will be directed to your web page.

## Route 53 Domain Configuration
1. Create Hosted Zone
Go to Route 53 → Hosted Zones.
Click Create Hosted Zone:
- Domain name: **`faraz.buzz`**
Copy the NS (Name Servers) provided.

2. Update Domain Registrar in **`GoDaddy`**.
Go to GoDaddy → My Account → Domains → DNS.
Change Name Servers to Route 53 ones.

## Create SSL Certificate with AWS Certificate Manager
- Go to AWS Certificate Manager in us-east-1.
- Click **`Request a certificate`**
- **FQDN**: `*.faraz.buzz`
- **Type**: `Public`
- **Validation**: `DNS`
- **Key Algorithm**: `RSA 2048`
Click Create Records in Route 53 → Confirm.

## Cloudfront configuration
- Go to CloudFront → Create Distribution.
- Settings:
- **Distribution Option**: `Single website or app`
- **Origin**: `Select your ALB`
- **Protocol**: `HTTP only`
- **Viewer protocol policy**: `Redirect HTTP to HTTPS`
- **Allowed HTTP method**: `GET, HEAD, OPTIONS, POST, PATCH, DELETE`
- **Cache key and origin requests**: `Cache policy and origin request policy` 
- **Cache policy**: `Caching disabled`
- **Origin request policy**: `AllViewer`
- **Custom SSL Certificate**: Select the ACM certificate created earlier
Select `TLSv1_2_2021`
- **Alternate DNS name**: `abc.faraz.buzz`
Create **`Distribution`**

## Now again go to Route 53:
- Create Record
**Record name**: `abc.faraz.buzz`
**Record Type**: `A`
- Turn on the `toggle` for Alias
- Route traffic to **`Alias to cloud front distribution`**
- Copy paste the cloud front distrbution domain name in Alias
- **`Create record`**

- Search **`http://abc.faraz.buzz`** in browser the url will be redirected form http to https and the content wil be served as well.

-------------------
- Your application will now securely deploy on EC2 behind a Load Balancer, with domain routing via Route 53, SSL from ACM, and global delivery through CloudFront.