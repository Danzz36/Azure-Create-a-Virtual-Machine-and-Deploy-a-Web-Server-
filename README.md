# Azure: Create a Virtual Machine and Deploy a Web-Server

## Objective

The objective of this projcet is to learn how to securely deploy and manage resources in Azure. The work will be focused on learning the fundamental networking architecture in Azure by creating Virtual Networks, Subnets, Network Security Groups, and Virtual Machines. I'll also be learning how to use Azure Bastion to securely connect to a Linux machine via SSH, install a web server, and configure it to be accessible via a public IP and custom DNS label. 

### Skills Learned

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used

- Security Information and Event Management (SIEM) system for log ingestion and analysis.
- Network analysis tools (such as Wireshark) for capturing and examining network traffic.
- Telemetry generation tools to create realistic network traffic and attack scenarios.

## Steps

- Create a Resource Group
  - Log in to the Azure Portal and select "Create a Resource"
  - Search for Resource Group and once found, click create
  - Give it a name in order to find it later. I'll name it RG-USE-Nextcloud. RG-Resource Group, USE-US East Region, Nextcloud-for nextcloud server
  - Click on review+create and then create
  - Navigate to the resouce group
    

- Create a Virtual Network and a subnet
  - Once in the resource group, click on create and search for virtual network
  - Click on it and then click create
  - Make sure the correct resource group is selected under the project details and then give the virtual net a name under instance details. I'll name it VNET-USE-Nextcloud
  - When we create a virtual machine, it's important to remember that it needs to be in the same region as the virtual network
  - Now click on next and then click the IP addresses tab at the top
  - It will give you a default address space of 10.0.0.0/16, but we want to change that. Replace that address with the address 172.10.0.0/16
  - Next, click add subnet. Once the pane opens, give it a name of SNET-USE-Nextcloud, make sure the proper IPv4 address range is selected (172.10.0.0/16), set the starting address as 172.10.0.0 and then select a size of /24
  - Then click add
  - Then click the review + create button
  - Click create. Once the deployment is complete, click your resource group and make sure it is there. It may take a minute and/or you might need to refresh the page

- Protect a subnet using a Network Security Group
  - Go back to your resource group
  - Click create and then search for network security group
  - Click on it and then click create
  - Make sure the resource group is correct and then give it a name under the instance details. I'll use NSG-USE-Nextcloud. Then click review + create and then click create on the next page
  - Azure will have applied some inbound and outbound rules to it once its completely set up. Now we want to asign this network security group to our subnet
  - Navigate back to the virtual network and then to the subnet
  - Once there, click the subnet and navigate to the network security group drop down in the pane that opens and select the network security group we just created. Now click save.

- Deploy Bastion to connect to a Virtual Machine
  - Since creating a bastion instance can take a little while to set up, It's a good idea to create it now. In order to do this, we need to create a subnet for it first
  - To do this, navigate back to subnets
  - Click on add subnet
  - First, select Azure Bastion in the purpose dropdown menu. Then name it AzureBastionSubnet. The name is a requirement.
  - We'll use 172.10.1.0 for the starting address of the range and change the size to /24
  - We don't select any network security group. Click on save
  - Now its time to create the bastion resource. Navigate back to the resource group and click on create
  - Search for Bastion. Click on it and then click create.
  - Make sure the correct resouce group has already been selected and then name it. We'll name it BASTION-USE-Nextcloud
  - Select the virtual network in the virtual network dropdown
  - Name the Public IP Adress Name as BASTIONIP-USE-Nextcloud and then click on review + create and then create
  - It will take a while to deploy, so we can work on other things until Azure notifies us that it's done

- Create an Ubuntu Server Virtual Machine
  - Navigate back to the resource group and click on Create
  - Search for Ubuntu Server and use Ubuntu Server 18.04 LTS. Then click Create
  - Make sure the correct resource group has been selected
  - Give the virtual machine a name. I'll name it VM-USE-Nextcloud
  - We'll leave the default Availability zone as 1. This is becuase if we want to access the Nextcloud server, we'll need a public IP and the public IP needs to be deployed in the same availability zone as the virtual machine
  - For the size of the VM, click see all sizes. Something like the B1s option which has 1 cpu, 1 gb ram, 2 data disks etc. will work perfectly.
  - SSH Public Key should be selected on the VM configuration. You can use anything you want for the username. Modify the key pair name to be more descriptive. We'll call it VM-USE-Nextcloud_SSHkey
  - Select none for public inbound ports. We are going to be connecting to our virtual machine using bastion which is an internal resource
  - Click on Next: Disks. We can leave the defualt of 30 GiB on the Premium SSD as is.
  - Click on Next: Networking. Your virtual network and subnet should already be selected. If not, make sure to choose the appropriate networks.
  - Select None in the dropdown for Public IP. This is to allow use to learn more later in the process by doing it ourselves.
  - Click on review + create
  - Confirm all the settings are corect and then click create
  - Click download private key and create resource. This will download the key pair to your private machine.
  - Wait for the virtual machine to deploy. It will take a few minutes.

- Install Nextcloud by connecting via SSH using Bastion
  - Once your VM has been deployed, you can check on it by going back to your resource group. Click on it and take note of it's Private IP. We will need that later.
  - You will want to start your virtual machine which can be done at the top of the machines details page. Note that if you want to pause the project or end it, make sure to hit the stop button to avoid incurring additional charges.
  - Now its time to connect to it. Click the connect buton in the top tool bar of the machines details page and then click on bastion. Now click on the Use Bastion button
  - Now enter the username that you chose earlier and then select SSH private key from local file, and then find and select the SSH key that was downloaded before on your local machine. Then click on connect.
  - A new tab will open and you will now be connected to your virtual machine using SSH via Bastion
  - Now let's install Nextcloud by typing 'sudo snap install nextcloud' and wait for it to download everything
  - Next we'll create a simple administrative account with a sample username and password, admin and daniel respectively. You can choose whatever you prefer.
  - Type 'sudo nextcloud.manual-install admin daniel' It will take a minute, so let it run and finish.
  - Next we'll create a simple self signed certificate. Type 'sudo nextcloud.enable-https self signed' and then wait until the certificate is generated.
  - Once done, we can now exit from the SSH connection. Type 'exit' and then close Bastion.

- Publish an IP
  - Now it's time to access our nextcloud instance on the web. In order to do this, we need to create a public IP and then allow only HTTPS connections
  - Navigate to the networking tab on the left side of the screen in the virtual machine details and click on it. Then click on the network interface card
  - Click on IP configurations in the left menu. We want to change IP config 1 to add a public address.
  - Click on the line item and then click on associate under Public IP Address, then click on create new. First give it a name. We'll use VMIP-USE-Nextcloud. Select standard SKU and then click OK
  - Now save the new configuration
  - Once it has finished setting it up, navigate back to your virtual machine and then click overview in the left side menu. You will now see a public IP address
  - If you copy that public IP address and try to open it in a new tab, it wil not work becuase we haven't set up a rule to allow inbound HTTPS traffic in our network security group. We want to set this up, but only allow traffic from our IP address. We can check and copy our IP address by using www.whatsmyip.com
  - Go back to azure and click on Networking in the left side menu of the virtual machine and click on add inbound port rule
  - Select IP addresses on the Source dropdown menu and then paste your IP adress in the source IP addresses/CIDR ranges field
  - Select IP addresses under the destination drop down menu
  - Add the virtual machines IP address in the destination IP addresses/CIDR ranges field
  - Select HTTPS on the service drop down menu
  - Make sure the Action is set to Allow and name the rule HTTPS_Nextcloud. Click add when done
  - If you try to navigate to the nextcloud servers IP address now, it will connect. Although your browser may tell you the connection is not secure because the certificate was self signed. You can ignore it by clicking on advanced and then proceed. It should now respond. However, it will tell you that you are accessing through an untrusted domain. What we need to do here is create a DNS entry for our public IP and then set it on Nextcloud

- Create a DNS label
  - Back in the Azure portal, navigate to your network diagram where you can see the visual representation of the resources in your network. You'll see your VMIP-USE-Nextcloud is connected to your virtual machines NIC. Go ahead and click on VMIP-USE-Nextcloud and then click on configuration in the left side menu.
  - In the DNS name label field, give it a name. You can use whatever you want. I'll use danielnextcloud. Once you find one that is available, click on save
  - Navigate back to your virtual machine by clicking on overview. Then click on your resource group. Then click on your virtual machine. You'll now see a publick IP address and your dns name.
  - Click on connect at the top of that screen and then click Bastion. Click use Bastion. Login using the same process as before. Use your username, select SSH private key from a local file and then find and select that SSH key file, then click connect
  - Now communicate your dns label to nextcloud by typing 'sudo nextcloud.occ config:system:set trusted_domains 1 --value=danielnextcloud.eastus.cloudapp.azure.com' Remember your DNS name will be different from mine.
  - You can now exit the SSH connection by typing exit and then closing Bastion.
  - Try navigating to the DNS name you just configured in a new tab. Accept the self signed certificate again and you will now be prompted with a nextcloud login page. It's now up and running. You can login with the credentials you set earlier. In my case it was admin, daniel.
 
- Once you are done, log out of the Nextcloud account in your browser and remember to go back into Azure and stop your virtual machine. Also stop your Bastion instnace by deleting it from the resource group to avoid being billed aymore for either unless you plan to continue using them. 


drag & drop screenshots here or use imgur and reference them using imgsrc

Every screenshot should have some text explaining what the screenshot is about.

Example below.

*Ref 1: Network Diagram*
