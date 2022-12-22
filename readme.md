##Background
<p dir="auto">This CFT is modified from Palo Alto Network official GitHub repo (Credit: <a href="https://github.com/PaloAltoNetworks/AWS-GWLB-VMSeries/tree/main/CFT_2_Firewalls">https://github.com/PaloAltoNetworks/AWS-GWLB-VMSeries/tree/main/CFT_2_Firewalls</a>) which is a CFT that provide easy deployment to integrate with AWS GWLB, I used it as the base and make several modifications so it can be used to demonstrate an overlay-routing deployment.</p>
<p dir="auto">This Cloud Formation Template (CFT) only serve to demonstrate using Palo Alto Network VM-Series Firewall to control Ingress and Egress traffic in AWS Environment, with the integration support of AWS Gateway Load Balancer(GWLB) and Overlay-routing capabilities of Palo Alto VM-Series.</p>
<p dir="auto">SecurityVPC.yaml
will create a Security VPC with GWLB, NAT Gateway, VM-series Firewall for Security inspection to protect App VPC inbound and outbound traffic.</p>
<p dir="auto">WebOnlyVPC.yaml
will create a APP VPC with a simple HTTP Webserver instance to test inbound HTTP traffic and outbound traffic.</p>
<p dir="auto">Please note that this template only support <strong>AWS US-east-1</strong> region. If you want to launch this CFT in other AWS region, you need to change AMI ID of the VM-series according to the region. See detail at <a href="https://docs.paloaltonetworks.com/vm-series/11-0/vm-series-deployment/set-up-the-vm-series-firewall-on-aws/deploy-the-vm-series-firewall-on-aws/obtain-the-ami/get-amazon-machine-image-ids" rel="nofollow">Here</a> .</p>
<h2 dir="auto"><a id="user-content-diagram" class="anchor" aria-hidden="true" href="#diagram"><svg class="octicon octicon-link" viewBox="0 0 16 16" version="1.1" width="16" height="16" aria-hidden="true"><path fill-rule="evenodd" d="M7.775 3.275a.75.75 0 001.06 1.06l1.25-1.25a2 2 0 112.83 2.83l-2.5 2.5a2 2 0 01-2.83 0 .75.75 0 00-1.06 1.06 3.5 3.5 0 004.95 0l2.5-2.5a3.5 3.5 0 00-4.95-4.95l-1.25 1.25zm-4.69 9.64a2 2 0 010-2.83l2.5-2.5a2 2 0 012.83 0 .75.75 0 001.06-1.06 3.5 3.5 0 00-4.95 0l-2.5 2.5a3.5 3.5 0 004.95 4.95l1.25-1.25a.75.75 0 00-1.06-1.06l-1.25 1.25a2 2 0 01-2.83 0z"></path></svg></a>Diagram</h2>
<p dir="auto"><a target="_blank" rel="noopener noreferrer nofollow" href="https://user-images.githubusercontent.com/22727679/208853498-8d8966a7-b9b2-472b-a91a-5cc8a2e70d00.png"><img src="https://user-images.githubusercontent.com/22727679/208853498-8d8966a7-b9b2-472b-a91a-5cc8a2e70d00.png" alt="image" style="max-width: 100%;"></a></p>
<h2 dir="auto"><a id="user-content-prerequisites" class="anchor" aria-hidden="true" href="#prerequisites"><svg class="octicon octicon-link" viewBox="0 0 16 16" version="1.1" width="16" height="16" aria-hidden="true"><path fill-rule="evenodd" d="M7.775 3.275a.75.75 0 001.06 1.06l1.25-1.25a2 2 0 112.83 2.83l-2.5 2.5a2 2 0 01-2.83 0 .75.75 0 00-1.06 1.06 3.5 3.5 0 004.95 0l2.5-2.5a3.5 3.5 0 00-4.95-4.95l-1.25 1.25zm-4.69 9.64a2 2 0 010-2.83l2.5-2.5a2 2 0 012.83 0 .75.75 0 001.06-1.06 3.5 3.5 0 00-4.95 0l-2.5 2.5a3.5 3.5 0 004.95 4.95l1.25-1.25a.75.75 0 00-1.06-1.06l-1.25 1.25a2 2 0 01-2.83 0z"></path></svg></a>Prerequisites</h2>
<ul dir="auto">
<li>Bootstrap Configuration</li>
</ul>
<p dir="auto">Upload the provided bootstrap files to a S3 Bucket, then fill in the name of the bucket to  "AWS S3 Bucket Name containing the VM-Series Bootstrap Information" parameter when launching the CFT. (By default the bucket name is:aws-gwlb-vmseries-1az-bootstrap)
For The Bootstrap files and folder structure, you can find <a href="https://docs.paloaltonetworks.com/vm-series/11-0/vm-series-deployment/bootstrap-the-vm-series-firewall/bootstrap-the-vm-series-firewall-in-aws" rel="nofollow">here</a></p>
<ul dir="auto">
<li>EC2 Key pair</li>
</ul>
<p dir="auto">Prepare a EC2 Key pair for SSH connectivity to the VM-series Firewall via tools like Putty.
This Key pair will be used to authenticate with firewall to start SSH connection.</p>
<ul dir="auto">
<li>AMI ID</li>
</ul>
<p dir="auto">Since this CFT only aim to demonstrate the GWLB integration and overlay-routing capabilities,
By default, it will be using PANOS 10.2.3 BYOL AMI image to keep the cost as low as possible.
If you wanted to try out Palo Alto Firewall Security subscription during the test,
You can change the AMI ID for other licenses.
To get the AMI ID for other licenses such as Bundle 1, 2,
Please see detailed steps from <a href="https://docs.paloaltonetworks.com/vm-series/11-0/vm-series-deployment/set-up-the-vm-series-firewall-on-aws/deploy-the-vm-series-firewall-on-aws/obtain-the-ami/get-amazon-machine-image-ids" rel="nofollow">Palo Alto Network official Guide</a></p>
<h2 dir="auto"><a id="user-content-instruction" class="anchor" aria-hidden="true" href="#instruction"><svg class="octicon octicon-link" viewBox="0 0 16 16" version="1.1" width="16" height="16" aria-hidden="true"><path fill-rule="evenodd" d="M7.775 3.275a.75.75 0 001.06 1.06l1.25-1.25a2 2 0 112.83 2.83l-2.5 2.5a2 2 0 01-2.83 0 .75.75 0 00-1.06 1.06 3.5 3.5 0 004.95 0l2.5-2.5a3.5 3.5 0 00-4.95-4.95l-1.25 1.25zm-4.69 9.64a2 2 0 010-2.83l2.5-2.5a2 2 0 012.83 0 .75.75 0 001.06-1.06 3.5 3.5 0 00-4.95 0l-2.5 2.5a3.5 3.5 0 004.95 4.95l1.25-1.25a.75.75 0 00-1.06-1.06l-1.25 1.25a2 2 0 01-2.83 0z"></path></svg></a>Instruction</h2>
<ol dir="auto">
<li>
<p dir="auto">Launch SecurityVPC Template via AWS console, fill in parameters such as Stack Name and EC2 Keypair, S3 bucket name for VM-Series bootstrap files.</p>
</li>
<li>
<p dir="auto">Wait Until SecurityVPC Stack is Fully created, you should see VM-Series Management URL and GWLB serviceid information in the output section of the stack.</p>
</li>
<li>
<p dir="auto">Try connect to VM-Series Management URL and login with username (admin) and password (Pal0Alto!) , please note that the firewall might still in bootstrap process even if the CFT stack is already shown as completed. Wait for few minutes if firewall is not yet ready.</p>
</li>
<li>
<p dir="auto">Verify the Firewall is configured correctly. If bootstrap is successful, the firewall should be ready with overlay-routing configured. For reference, steps to enable Overlay-routing in AWS can be found <a href="https://docs.paloaltonetworks.com/vm-series/10-1/vm-series-deployment/set-up-the-vm-series-firewall-on-aws/vm-series-integration-with-gateway-load-balancer/integrate-the-vm-series-with-an-aws-gateway-load-balancer/enable-overlay-routing-for-the-vm-series-on-aws" rel="nofollow">Here</a>.</p>
</li>
<li>
<p dir="auto">Launch WebOnlyVPC Template  via AWS Console, fill in parameters Security Stack Name with the stack name of SecurityVPC. This parameter will let the CFT to read the GWLBServiceID from the SecurityVPC and create GWLB endpoint in the WebOnlyVPC.</p>
</li>
<li>
<p dir="auto">After the WebOnlyVPC is created successfully, you can now try to test traffic to and from the Webserver instance.</p>
</li>
</ol>
<p dir="auto">To test inbound traffic, simple locate the IP address in Output section in WebOnlyVPC and connect with http://. If connection is successful, you will     see the page response with "Hello World from ip-xxxx.ec2.internal". This mean HTTP Web service is up and responsive.</p>
<p dir="auto">To test outbound traffic from the Webserver. The AWS Session Manager is already enabled during stack creation, therefore you can remote access the instance via aws console. Browse to EC2 page, then right-click the Webserver instance, click "Connect" then Click "Connect" in the Session manager tab. Then you can try to ping       8.8.8.8 from the instance to verify the instance is able to ping to internet via the Firewall. You should also see traffic log in the Firewall.</p>
</article>
