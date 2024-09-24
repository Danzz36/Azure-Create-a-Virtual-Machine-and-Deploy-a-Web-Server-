# Azure: Create a Virtual Machine and Deploy a Web-Server

## Objective

The objective of this projcet is to learn how to securely deploy and manage resources in Azure. The work will be focused on learning the fundamental networking architecture in Azure by creating Virtual Networks, Subnets, Network Security Groups, and Virtual Machines. I'll also be learning how to use Azure Bastion to securely connect to a Linux machine via SSH, install a web server, and configure it to be accessible via a public IP and custom DNS label. This is a guided project.

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

## Create a Resource Group
- Log in to the Azure Portal and select "Create a Resource".
- Search for **Resource Group** and click **Create**.
- Name the resource group. Example: `RG-USE-Nextcloud` (RG = Resource Group, USE = US East Region, Nextcloud = for nextcloud server).
- Click **Review + Create**, then click **Create**.
- Navigate to the resource group.

## Create a Virtual Network and Subnet
- Inside the resource group, click **Create** and search for **Virtual Network**.
- Click on it and then click **Create**.
- Ensure the correct resource group is selected under project details and name the virtual network. Example: `VNET-USE-Nextcloud`.
- Remember that the virtual machine must be in the same region as the virtual network.
- Click **Next** and then select the **IP Addresses** tab.
- Replace the default address space (`10.0.0.0/16`) with `172.10.0.0/16`.
- Click **Add Subnet** and name it `SNET-USE-Nextcloud`. Set the IPv4 address range as `172.10.0.0/16` and the starting address as `172.10.0.0`. Choose `/24` for the size.
- Click **Add**, then **Review + Create**.
- Click **Create**. After deployment, verify it in the resource group.

## Protect a Subnet Using a Network Security Group
- Go back to your resource group.
- Click **Create** and search for **Network Security Group**.
- Click **Create** and ensure the correct resource group is selected. Name the network security group. Example: `NSG-USE-Nextcloud`.
- Click **Review + Create**, then **Create**.
- Azure will apply some default inbound and outbound rules. Next, assign this network security group to the subnet.
- Navigate to the virtual network and open the subnet settings.
- In the network security group dropdown, select the newly created network security group, then click **Save**.

## Deploy Bastion to Connect to a Virtual Machine
- Navigate to **Subnets** and click **Add Subnet**.
- Select **Azure Bastion** from the **Purpose** dropdown, then name it `AzureBastionSubnet` (this name is required).
- Use `172.10.1.0` as the starting address and choose `/24` for the size.
- Click **Save**.
- Go back to the resource group and click **Create**.
- Search for **Bastion**, then click **Create**.
- Ensure the correct resource group is selected and name the bastion resource. Example: `BASTION-USE-Nextcloud`.
- Select the virtual network and name the Public IP Address `BASTIONIP-USE-Nextcloud`.
- Click **Review + Create**, then **Create**. It will take a few minutes to deploy.

## Create an Ubuntu Server Virtual Machine
- Go to the resource group and click **Create**.
- Search for **Ubuntu Server 18.04 LTS**, then click **Create**.
- Ensure the correct resource group is selected and name the VM. Example: `VM-USE-Nextcloud`.
- Leave the default Availability zone as **1**. This is becuase if we want to access the Nextcloud server, we'll need a public IP and the public IP needs to be deployed in the same availability zone as the virtual machine
- Choose a size, such as **B1s** (1 CPU, 1 GB RAM).
- Set **SSH Public Key** as the authentication type and modify the key pair name. Example: `VM-USE-Nextcloud_SSHkey`. Use anything you want for the username.
- Select **None** for public inbound ports since we will use bastion.
- Leave the default disk settings and proceed to **Networking**.
- Ensure the correct virtual network and subnet are selected.
- Choose **None** for Public IP. This is to allow use to learn more later in the process by doing it ourselves.
- Click **Review + Create** and verify the settings.
- Click **Create** and download the private key when prompted.
- Wait for the VM to deploy.

## Install Nextcloud via SSH Using Bastion
- After deployment, navigate to the VM and take note of the private IP.
- Start the virtual machine from the VM details page. Note that if you want to pause the project or end it, make sure to hit the stop button to avoid incurring additional charges.
- Click **Connect** in the toolbar, then choose **Bastion** and click **Use Bastion**.
- Enter the username, choose **SSH private key from local file**, and select the previously downloaded key. Click **Connect**.
- A new tab will open with SSH access to the VM.
- Run the following commands to install Nextcloud and set up an admin account:

    ```bash
    sudo snap install nextcloud
    ```

     ```bash
    sudo nextcloud.manual-install admin daniel
    ```

- Create a self-signed certificate:

    ```bash
    sudo nextcloud.enable-https self-signed
    ```

- Exit the SSH connection by typing `exit` and closing Bastion.

## Publish an IP
- In the VM's **Networking** tab, click the network interface card.
- Select **IP Configurations**, and under **IP Config 1**, associate a new public IP.
- Name the public IP. Example: `VMIP-USE-Nextcloud`, then select **Standard** SKU. Save the new congifuration.
- After setup, navigate to the VM's **Overview** to view the public IP.
- If you copy that public IP address and try to open it in a new tab, it will not work becuase a rule to allow inbound HTTPS traffic in our network security group hasn't been set yet. This needs to be set up, but only to allow traffic from our IP address. We can check and copy our IP address by using `www.whatsmyip.com`.
- To allow HTTPS traffic, add an inbound port rule in the **Networking** tab:

    - Source: **IP addresses**, and add your IP from `www.whatsmyip.com`.
    - Destination: **IP addresses**, and add the VM's IP.
    - Service: **HTTPS**.
    - Action: **Allow** and name the rule `HTTPS_Nextcloud`.
 
- If you try to navigate to the nextcloud servers IP address now, it will connect. Although your browser may tell you the connection is not secure because the certificate was self signed. You can ignore it by clicking on advanced and then proceed. It should now respond. However, it will tell you that you are accessing through an untrusted domain. What's needed is to create a DNS entry for the VM's public IP and then set it on Nextcloud.

## Create a DNS Label
- Navigate to the public IP resource (e.g., `VMIP-USE-Nextcloud`) and select **Configuration**.
- In the **DNS Name Label** field, enter a DNS label (e.g., `danielnextcloud`).
- Save the configuration.
- SSH into the VM using Bastion and communicate the DNS label to Nextcloud:

    ```bash
    sudo nextcloud.occ config:system:set trusted_domains 1 --value=danielnextcloud.eastus.cloudapp.azure.com
    ```

- Exit the SSH session and try accessing the Nextcloud instance via the DNS name in a browser. It will now be up and running. You can login with the credentials you set earlier.

## Final Steps
- After finishing, log out of Nextcloud and stop both the virtual machine and the Bastion instance to avoid additional charges.


drag & drop screenshots here or use imgur and reference them using imgsrc

Every screenshot should have some text explaining what the screenshot is about.

Example below.

*Ref 1: Network Diagram*
