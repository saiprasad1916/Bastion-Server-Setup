
## AWS VPC Infrastructure: Bastion Host (Jump Server) Setup
This project demonstrates how to set up a secure gateway—known as a Bastion Server or Jump Server—to manage private resources within an AWS VPC.
## 📖 Definition
A Bastion Server is a specialized, highly secured server that acts as a single gateway to access a private network. It is the only server exposed to the public internet, allowing you to connect to internal resources (like databases or private web servers) without those resources having their own public IP addresses.
## Why It Is Used
It protects sensitive internal systems from external threats. By keeping backend servers in a private subnet, you ensure they are not directly reachable from the internet, preventing unauthorized access and brute-force attacks.
## Key Benefits

* Reduced Attack Surface: You only have to "harden" and protect one entry point.
* Centralized Logging: All administrative activity is funneled through one point for easier auditing.
* Cost-Effective: Reduces the need for multiple public IPs or complex VPN setups.
* Network Segmentation: Enforces a clear boundary between public-facing and sensitive internal data.

<img width="692" height="378" alt="image" src="https://github.com/user-attachments/assets/dbc45591-b2a2-4af7-a541-8acaaba91a73" />


------------------------------
## 🛠 Architectural Scenario
In this setup, a public server serves as the Jump/Bastion Server, acting as the intermediary "hop" between your local machine and an isolated private server.

   1. VPC Setup: A VPC with one Public Subnet (internet-accessible) and one Private Subnet (isolated).
   2. Public Server: Sits in the Public Subnet with a Public IP. Its Security Group allows SSH (Port 22) only from trusted IPs.
   3. Private Server: Sits in the Private Subnet with only a Private IP. Its Security Group is configured to only allow SSH traffic coming from the Bastion Server.
   4. The Connection:
   * You SSH into the Public Server from your local machine.
      * From the Public Server terminal, you SSH into the Private Server via its private IP.
   
------------------------------
## 💻 Infrastructure as Code (CloudFormation)
Use the following YAML template to deploy the VPC, subnets, and security groups required for this scenario.

AWSTemplateFormatVersion: '2010-09-09'Description: 'CloudFormation template to create the VPC_cli infrastructure with Public and Private subnets.'
Resources:
```
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CloudFormation template to create the VPC_cli infrastructure with Public and Private subnets.'

Resources:
  # 1. VPC Configuration
  VPCCli:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: VPC_cli

  # 2. Subnet Configurations
  PublicSubnetCli:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPCCli
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      Tags:
        - Key: Name
          Value: Public_Subnet_cli

  PrivateSubnetCli:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPCCli
      CidrBlock: 10.0.2.0/24
      MapPublicIpOnLaunch: false
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      Tags:
        - Key: Name
          Value: Private_Subnet_cli

  # 3. Internet Gateway Configuration
  IGWCli:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: IGW_cli

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPCCli
      InternetGatewayId: !Ref IGWCli

  # 4. Route Table Configurations
  PublicRouteTableCli:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPCCli
      Tags:
        - Key: Name
          Value: Public_RT_cli

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTableCli
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref IGWCli

  PublicSubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnetCli
      RouteTableId: !Ref PublicRouteTableCli

  PrivateRouteTableCli:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPCCli
      Tags:
        - Key: Name
          Value: Private_RT_cli

  PrivateSubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateSubnetCli
      RouteTableId: !Ref PrivateRouteTableCli

  # 5. Security Group Configuration
  WebAccessSGcli:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH and HTTP traffic
      VpcId: !Ref VPCCli
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: WebAccessSG_cli

Outputs:
  VpcId:
    Description: The ID of the VPC
    Value: !Ref VPCCli
  PublicSubnetId:
    Description: ID of the Public Subnet
    Value: !Ref PublicSubnetCli
  SecurityGroupId:
    Description: ID of the Security Group
    Value: !Ref WebAccessSGcli
```
Bastain Server Setup:

Follow the below Sequence.

Create a server in the same VPC and select Subnet as private Network

Private server

<img width="691" height="295" alt="image" src="https://github.com/user-attachments/assets/fd92c495-771e-4b42-b59f-e2a083b13502" />

Select Private Subnet 

<img width="691" height="296" alt="image" src="https://github.com/user-attachments/assets/c67e8ef6-4d31-44b8-916f-10534646637a" />

<img width="691" height="297" alt="image" src="https://github.com/user-attachments/assets/a67823c0-f6eb-4cd8-902e-55941f875009" />

We can see there is no public ip for the above private server

Try to ssh to the folowing private server

<img width="691" height="299" alt="image" src="https://github.com/user-attachments/assets/2ca425e1-1f97-4903-a7aa-3b039cc519ae" />

We get timed out message as we cannot connect to the private server through ssh

<img width="693" height="316" alt="image" src="https://github.com/user-attachments/assets/7dec4c78-a7b8-44ff-a510-143b21c4d459" />

Create a Jump server

Now Create a Jump Server in the Same VPC and select Subnet as Public Subnet in the Network Settings while creating the Server

<img width="691" height="297" alt="image" src="https://github.com/user-attachments/assets/8cfadbf0-61be-44fc-b32a-0d251b0ac2dd" />

Select Public Subnet

<img width="691" height="297" alt="image" src="https://github.com/user-attachments/assets/6746841d-4bb6-4357-ad3d-44b29f5900c0" />

Servers

<img width="691" height="284" alt="image" src="https://github.com/user-attachments/assets/097087fd-718c-4779-94fb-43e744748ed6" />

SSH to Jump Server 

<img width="692" height="301" alt="image" src="https://github.com/user-attachments/assets/88779cbc-e067-463a-92b1-324460e1ec3f" />

<img width="691" height="299" alt="image" src="https://github.com/user-attachments/assets/dcbc4c39-53f3-4f25-8ef8-b8b5299427b2" />

<img width="692" height="284" alt="image" src="https://github.com/user-attachments/assets/5691e53f-b732-4369-aac1-58256efa86ea" />

<img width="692" height="328" alt="image" src="https://github.com/user-attachments/assets/a0681a55-f3a8-49c8-a4bb-e1116a987c76" />

<img width="691" height="297" alt="image" src="https://github.com/user-attachments/assets/dea469b1-e7ee-412c-ad22-a115b7385ba4" />

Transfer pem key through local machine terminal to aws ubuntu instance

<img width="692" height="140" alt="image" src="https://github.com/user-attachments/assets/fed6b646-d6e0-45f8-8341-2c007a61c605" />

Pemkey transferred to ubuntu server(jump server)
<img width="692" height="322" alt="image" src="https://github.com/user-attachments/assets/e13aa2a7-3c84-4efe-9028-fb6c5a3f6080" />

Pemkey tranfered to the public server(jump server) 
<img width="691" height="210" alt="image" src="https://github.com/user-attachments/assets/21bf0062-1edf-4f79-9330-c728114515b6" />

Change the permission of the pemfile through following command

SSH from Jump Server to Private Server
ssh -i "privatekey.pem" ubuntu@10.0.2.236

<img width="691" height="361" alt="image" src="https://github.com/user-attachments/assets/ba6633a3-4e43-4b94-b742-b91ab6edd1ea" />

Before ssh to private server from jump server , for clear understanding change the name of the  private server 

<img width="692" height="106" alt="image" src="https://github.com/user-attachments/assets/c217daa6-6857-48e2-9e50-e5eb6331ad6c" />


Connected to Private Server by SSH from Public Server(Jump Server).

<img width="692" height="328" alt="image" src="https://github.com/user-attachments/assets/2cc777c2-8dec-4ff1-b5aa-47fceb617fca" />

## 🏁 Conclusion

Implementing a Bastion Host is a fundamental best practice for cloud security. By using this architecture, you successfully create a "Single Point of Entry" that significantly hardens your infrastructure.
While the private subnet keeps your sensitive application servers and databases invisible to the public internet, the Bastion Server provides a controlled, auditable bridge for administrative tasks. By combining this setup with SSH Agent Forwarding, you ensure that even if the gateway is targeted, your private credentials remain safely on your local machine. This balance of accessibility and isolation is key to maintaining a robust and secure AWS environment.


