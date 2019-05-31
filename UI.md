# IoT Application in VPC - UI Steps

## Prerequisites

1. Account must be enabled to access VPC Infrastructure.
2. Have access to a public SSH key.
3. Create a resource group for this scenario called `priyank`.
4. Running IOT Platform and application setup
5. Mobile device configure and streaming the data to the platform

## Login to IBM Cloud

Browse to https://cloud.ibm.com/login and login.

<kbd>![Log in](images/ui/00-login.png)</kbd>

Select VPC Infrastructure from the hamburger menu in the upper left corner.

<kbd>![VPC Infrastructure](images/ui/01-menu-vpc.png)</kbd>

## Deploy VPC Infrastructure

### Create an SSH Key

1. An SSH key is required when creating a VPC instance. From the Compute menu select "SSH keys"
<kbd>![SSH Key](images/ui/10-sshkey-overview.png)</kbd>
1. Click the "Add SSH key" button. In the modal dialog provide a name & public key and click the "Add SSH key" button.
<kbd>![Add SSH Key](images/ui/11-sshkey-create.png)</kbd>
3. Confirm the ssh key was added.
<kbd>![Confirm SSH Key](images/ui/12-sshkey-confirm.png)</kbd>

### Resource Group, Region and Zone

For this scenario use group `priyank` instead of default, and for location use Dallas with zone Dallas 1.

### Create VPC

1. From the `Network` menu in the navigation bar select `VPC and Subnets` & click the "New virtual private cloud" button.
<kbd>![VPC Overview](images/ui/20-vpc-overview.png)</kbd>
1. Click the "New virtual private cloud" button. Next fill out the form and hit the "Create virtual private cloud" button
<kbd>![VPC Create](images/ui/21-vpc-create.png)</kbd>
3. Confirm the newly created VPC is available
<kbd>![Confirm VPC Availability](images/ui/22-vpc-confirm.png)</kbd>   
4. Check the status of the subnet. Initially it will be set to pending.
<kbd>![Pending Subnet Status](images/ui/23-subnet-pending.png)</kbd>
5. Confirm the status of the subnet is `available`
<kbd>![Confirm Subnet Status](images/ui/24-subnet-available.png)</kbd>

### Create the instance to run IOT Application
1. From the `Compute` menu in the navigation bar select "Virtual Server Instances" & click the `New Instance` button.
<kbd>![Instance Overview](images/ui/30-instance-overview.png)</kbd>
2. Next fill out the form and hit the `Create virtual server instance` button
<kbd>![Instance create](images/ui/31-instance-create.png)</kbd>
3. Check the status of the newly created instance. Initially it will be set to pending.
<kbd>![VPC Pending](images/ui/32-instance-pending.png)</kbd>
4. Confirm the status of the stance is `available`
<kbd>![VPC Available](images/ui/33-instance-confirm.png)</kbd>

### Reserve and Assign a Floating IP to the instance
1. From the `Network` menu in the navigation bar select "Floating IPs" & click the `Reserve Floating IP` button.
<kbd>![FIP Overview](images/ui/40-floatingip-overview.png)</kbd>
2. Next fill out the form and hit the `Reserve IP` button
<kbd>![FIP Reserve](images/ui/41-floatingip-reserve.png)</kbd>
3. Confirm a Floating IP has been reserved and associated with the instance.
<kbd>![FIP Confirm](images/ui/42-floatingip-confirm.png)</kbd>

### Configure the Security Groups
1. From the `Network` menu in the navigation bar select "Security Groups" & select-click the security group associated with the vpc we created, in this scenario `vpc-iot`.
<kbd>![SG Overview](images/ui/50-sg-overview.png)</kbd>
2. Next click the `Add Rule` button
<kbd>![SG Rule overview](images/ui/51-sg-rule-overview.png)</kbd>
3. Next fill out the form for TCP port 7000 and hit the `Save` button
<kbd>![SG Rule add](images/ui/52-sg-rule-add.png)</kbd>
4. Confirm the rule for port 7000 has been added
<kbd>![SG Confirm](images/ui/53-sg-rule-confirm.png)</kbd>

### Accessing the application
The application should now be available on the FloatingIP at port 7000 - http://169.61.244.117:7000
<kbd>![IOT Rickshaw Application](images/ui/60-app-in-browser.png)</kbd>
