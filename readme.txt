Background:

This CFT is modified from Palo Altonetwork official Git (Credit to: https://github.com/PaloAltoNetworks/AWS-GWLB-VMSeries/tree/main/CFT_2_Firewalls) which is a CFT that provide easy deployment to integrate with AWS GWLB, I used it as the base and make several modification so it can be used to demonstrate an overlay-routing deployment.

This Cloud Formation Template (CFT) only serve to demonstrate using Palo Alto Network VM-Series Firewall to control Ingress and Engress traffic in AWS Environemnt, with the integration support of AWS Gateway Load Balancer(GWLB) and Overlay-routing capabilities of PaloAlto VM-Series. 

SecurityVPC.yaml will create a Security VPC with GWLB,NAT Gateway, VM-series Firewall for Security inspection to protect App VPC inbound and outbound traffic.

WebOnlyVPC.yaml will create a APP VPC with a simple HTTP Webserver instance to test inbound HTTP traffic and outbound traffic.

Please note that this template only support AWS us-east-1 region. If you want to launch this CFT in other AWS region, you need to change AMI ID of the VM-series according to the region(See:https://docs.paloaltonetworks.com/vm-series/11-0/vm-series-deployment/set-up-the-vm-series-firewall-on-aws/deploy-the-vm-series-firewall-on-aws/obtain-the-ami/get-amazon-machine-image-ids)

Diagram

Prerequsite:

Bootstrap Configuration-
Upload the provided bootstrap files to a S3 Bucket, then fill in the name of the bucket to  "AWS S3 Bucket Name containing the VM-Series Bootstrap Information" parameter when launching the CFT. (By default the bucket name is:aws-gwlb-vmseries-1az-bootstrap)
For The Bootstrap files and folder structure, you can find here:
https://docs.paloaltonetworks.com/vm-series/11-0/vm-series-deployment/bootstrap-the-vm-series-firewall/bootstrap-the-vm-series-firewall-in-aws


EC2 Keypair-
Prepare a EC2 Keypair for SSH connectivity to the VM-series Firewall via tools like Putty. 
This Keypair will be used to authenticate with firewall to start SSH connection.

AMI ID-
Since this CFT only aim to demonstrate the GWLB integration and overlay-routing capabilities,
By default, it will be using PANOS 10.2.3 BYOL ami image to keep the cost as low as possible.
If you wanted to try out Palo Alto Firewall Security subscription during the test,
You can change the AMI ID for other licenses.
To get the AMI ID for other licenses such as Bundle 1, 2,
Please see detailed steps from Palo Alto Network offical Guide:
https://docs.paloaltonetworks.com/vm-series/11-0/vm-series-deployment/set-up-the-vm-series-firewall-on-aws/deploy-the-vm-series-firewall-on-aws/obtain-the-ami/get-amazon-machine-image-ids


Instruction:
1. Launch SecurityVPC Teamplate via AWS console, fill in parameters such as stack name and EC2 Keypair, S3 bucket for VM-Series bootstrap.
2. Wait Until SecurityVPC Stack is Fully created, you should see VM-Series Managemnt URL and GWLB serviceid information in the output seciton of the stack.
3. Try connect to VM-Series Managment URL and login with username (admin) and password (Pal0Alto!)
4. Verify the Firewall is configured correctly, if bootstrap is successful. The firewall should be ready with overlay-routing configured.( See Here for Steps to enable Overlay-routing in AWS: https://docs.paloaltonetworks.com/vm-series/10-1/vm-series-deployment/set-up-the-vm-series-firewall-on-aws/vm-series-integration-with-gateway-load-balancer/integrate-the-vm-series-with-an-aws-gateway-load-balancer/enable-overlay-routing-for-the-vm-series-on-aws
)
5. Launch WebOnlyVPC via AWS Console, fill in parameters SecurityStackName with the stack name of SecurityVPC. This parameter will let the CFT to read the GWLBServiceID from the SecurityVPC and create GWLB endpoint in the WebOnlyVPC.
6. After the WebOnlyVPC is created successfully, you can now try to test traffic to and from the Webserver instance.
To test inbound traffic, simple locate the IP address in Output section in WebOnlyVPC and connect with http://<Web Server IP>. If connection is sucessful, you will see the page reposnse with "Hello World from ip-xxxx.ec2.internal". This mean HTTP Web service is up and responsive.
To test outbound traffic from the Webserver. The AWS Session Manager is already enabled during stack creation, therefore you can remote access the instance via aws console. Browse to EC2 page, then right-click the Webserver instance, Click "Connect" then Click "Connect" in the Session mananager tab. Then you can try to ping 8.8.8.8 from the instance to verify the instance is able to ping to internet via the Firewall. You should also see traffic log in the Firewall.