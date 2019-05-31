# IoT Application in VPC - CLI Steps

## Prerequisites

1. Install the [IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)
2. Have access to a public SSH key.
3. Install the vpc-infrastructure plugin.  
   `$ ibmcloud plugin install vpc-infrastructure`
4. Running IOT Platform and application setup
5. Mobile device configure and streaming the data to the platform

## Set Resource Group, Region and Zone

New VPC resources will be assigned the account's default Resource Group.  Use the ibmcloud target command to select the desired group and region for the VPC.  In our case we want to use group VPC1 instead of default, and locate the VPC in the us-south region.

```
$ ic resource groups
Retrieving all resource groups under account Phillip Trent's Account as pltrent@us.ibm.com...
OK
Name      ID                                 Default Group   State   
default   00d24065a2ec44efb9de172e6d19b919   true            ACTIVE   
VPC1      04620a177bad4baf999ad5704eaae2d2   false           ACTIVE
priyank   4fe35a9ad2b5438291bd88dae7aa208d   false           ACTIVE  
```

```
$ ic target  -g priyank -r us-south
Targeted resource group priyank

Switched to region us-south

API endpoint:      https://api.ng.bluemix.net   
Region:            us-south   
User:              priyankn@ca.ibm.com   
Account:           Phillip Trent's Account (843f59bad5553123f46652e9c43f9e89) <-> 1691265   
Resource group:    priyank   
CF API endpoint:      
Org:                  
Space:        
```
Now that we are in us-south, let's find out what zones are available using the zones command. We are going to use us-south-1 for this example.

```
$ ic is zones us-south
Listing zones in region us-south under account Phillip Trent's Account as user priyankn@ca.ibm.com...
Name         Region     Status   
us-south-3   us-south   available   
us-south-1   us-south   available   
us-south-2   us-south   available

$ export ZONE_NAME=us-south-1
```

## Create an SSH Key

An SSH key is required when creating a VPC instance. Copy the ssh public key you wish to use to vpc-key.pub and call the key-create command to load it to the VPC environment. Remember the key id for later use.

```
$ ic is key-create priyank-mac @/Users/priyanknarvekar/.ssh/id_rsa.pub
Creating key priyank-mac under account Phillip Trent's Account as user priyankn@ca.ibm.com...

ID            636f6d70-0000-0001-0000-000000140a55   
Name          priyank-mac   
Type          rsa   
Length        2048   
FingerPrint   SHA256:7LMi9Nn8SHDCnFovCaMSv+IIE5KxR7EXSGTA5aqW3RE   
Key           ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCylLp6CbMlvb8TQnAbmoUkvlq/90/bo4H1B7cjWIBdn44qrywvHYayVoHygZfMAjfcFdtvtJwGjGqg502z7eRg0yzh5uG4iIvF6CawL0N4QSQNkiJl5OW0RIswcQ8U+9wj1K0C3A2l7JkNi+Jc+A86FtQKFkXcjO/702DGR+bYFq2FdwajLxmCAd7zWrVGZXi8/5G1yrYDKcEdxVpgR/DAMNieRzcbZbuA62jDn7zf9Iab6CAM8Gd8AmdisoO4Pg0dByUkg6I6XQ2QhkUrmG9C6guNTM9xchmMPVFYY7QwLpgR/PLd54/jtNESNkn0MiYJW6uTGmGyZOgFsoaIBs6j   

Created       1 second ago

$ export KEY_ID=636f6d70-0000-0001-0000-000000140a55
```

## Create VPC
Create a VPC named iot-vpc.

```
$ ic is vpc-create iot-vpc
Creating vpc iot-vpc in resource group priyank under account Phillip Trent's Account as user priyankn@ca.ibm.com...

ID                       8b98e8a8-e442-404a-92d1-74f6eed5b664   
Name                     iot-vpc   
Default                  no   
Default Network ACL      allow-all-network-acl-8b98e8a8-e442-404a-92d1-74f6eed5b664(116815da-ea3f-424f-a897-25078fe8f0d9)   
Default Security Group   -   
Resource Group           (4fe35a9ad2b5438291bd88dae7aa208d)   
Created                  4 seconds ago   
Status                   available   

$ export VPC_ID=8b98e8a8-e442-404a-92d1-74f6eed5b664
```

## Create Address Prefix

Create address prefixes for 10.10.10.0/24

```
$ ic is vpc-address-prefix-create orange $VPC_ID $ZONE_NAME 10.10.10.0/24
Creating address prefix orange of vpc 8b98e8a8-e442-404a-92d1-74f6eed5b664 under account Phillip Trent's Account as user priyankn@ca.ibm.com...

ID            e38aab07-d6e1-48bf-8f29-22d412cf8b8a   
Name          orange   
CIDR Block    10.10.10.0/24   
Zone          us-south-1   
Has Subnets   no   
Created       1 second ago   

$ export PREFIX_ID=e38aab07-d6e1-48bf-8f29-22d412cf8b8a
```

## Create the VPC Subnet

Create a VPC subnet in us-south-1 for ipv4-cidr-blocks 10.10.10.0/24.  
The initial status of a newly created subnet is set to pending.  
You must wait until the subnet status is available before assiging any resources to it.

```
$ ic is subnet-create orange $VPC_ID $ZONE_NAME --ipv4-cidr-block 10.10.10.0/24
Creating Subnet orange under account Phillip Trent's Account as user priyankn@ca.ibm.com...

ID               0a082713-2179-4777-8984-da991c784c1d   
Name             orange   
IPv*             ipv4   
IPv4 CIDR        10.10.10.0/24   
IPv6 CIDR        -   
Addr available   251   
Addr Total       256   
ACL              allow-all-network-acl-8b98e8a8-e442-404a-92d1-74f6eed5b664(116815da-ea3f-424f-a897-25078fe8f0d9)   
Gateway          -   
Created          1 second ago   
Status           pending   
Zone             us-south-1   
VPC              iot-vpc(8b98e8a8-e442-404a-92d1-74f6eed5b664)   

$ export SUBNET_ID=0a082713-2179-4777-8984-da991c784c1d
```

## Reserve a Floating IP for the instance

```
$ ic is floating-ip-reserve rickshaw-iot --zone $ZONE_NAME
Creating floating IP rickshaw-iot under account Phillip Trent's Account as user priyankn@ca.ibm.com...

ID            cef8df3f-e1d4-4620-8ef4-94901ce86f26   
Address       169.61.244.117   
Name          rickshaw-iot   
Target        -   
Target Type   -   
Target IP        
Created       now   
Status        pending   
Zone          us-south-1   

$ export FLOATING_IP_ID=cef8df3f-e1d4-4620-8ef4-94901ce86f26
```

## Create the instance to run IOT Application
For the purpose of this demo we will be deploying the [Rickshaw 4 IOT application](https://github.com/ibm-watson-iot/rickshaw4iot). To set it up we use a [bootstrap/cloud-init script](./bootstrap.sh) and pass it as user-data to the create instance command.

```
$ # Image id for ubuntu-18.04-amd64
$ export IMAGE_ID=cfdaf1a0-5350-4350-fcbc-97173b510843
$ ic is instance-create \
    iot-rickshaw \
    $VPC_ID \
    $ZONE_NAME \
    c-2x4 \
    $SUBNET_ID \
    100 \
    --image-id $IMAGE_ID \
    --key-ids $KEY_ID \
    --user-data @bootstrap.sh \
    --security-group-ids $SG_ID

Creating instance iot-rickshaw under account Phillip Trent's Account as user priyankn@ca.ibm.com...

ID                1daa7ff7-6c76-4d28-9c7a-adb375096600   
Name              iot-rickshaw   
Profile           c-2x4   
CPU Arch          amd64   
CPU Cores         2   
CPU Frequency     2000   
Memory            4   
Primary Intf      primary(ec5999db-3c2c-4e70-80e9-ac2181423d95)   
Primary Address   10.10.10.8   
Image             ubuntu-18.04-amd64(cfdaf1a0-5350-4350-fcbc-97173b510843)   
Status            pending   
Created           5 seconds ago   
VPC               iot-vpc(8b98e8a8-e442-404a-92d1-74f6eed5b664)   
Zone              us-south-1   

$ export INSTANCE_ID=1daa7ff7-6c76-4d28-9c7a-adb375096600
$ export NIC_ID=ec5999db-3c2c-4e70-80e9-ac2181423d95
```

## Configure the Security Groups

Next we will get the security group id associated with the instance nic, to configure it to allow tcp traffic for the port on which the Rickshaw Application listens(7000).

```
$ ic is in-nic $INSTANCE_ID $NIC_ID
Getting network interface ec5999db-3c2c-4e70-80e9-ac2181423d95 of instance 1daa7ff7-6c76-4d28-9c7a-adb375096600 under account Phillip Trent's Account as user priyankn@ca.ibm.com...

ID                ec5999db-3c2c-4e70-80e9-ac2181423d95   
Name              primary   
Type              primary   
Subnet            orange(0a082713-2179-4777-8984-da991c784c1d)   
Speed             100   
Primary V4        10.10.10.8   
Security Groups   roundness-demote-foothold-sputter-accent-preformed(2d364f0a-a870-42c3-a554-000001180185)   
Status               
Created           33 minutes ago   

$ export SG_ID=2d364f0a-a870-42c3-a554-000001180185
$ ic is security-group-rule-add $SG_ID inbound TCP --remote 0.0.0.0/0 --port-max 7000 --port-min 7000
Creating rule for security group 2d364f0a-a870-42c3-a554-000001180185 under account Phillip Trent's Account as user priyankn@ca.ibm.com...

ID                     b597cff2-38e8-4e6e-999d-000002302081   
Direction              inbound   
IPv*                   ipv4   
Protocol               tcp   
Min Destination Port   7000   
Max Destination Port   7000   
Remote                 0.0.0.0/0   

```

## Configure the Floating IP association
In order to allow access from internet next we will configure the floating IP association for instance

```
$ ic is instance-network-interface-floating-ip-add $INSTANCE_ID $NIC_ID $FLOATING_IP_ID
Creating floatingip cef8df3f-e1d4-4620-8ef4-94901ce86f26 for instance 1daa7ff7-6c76-4d28-9c7a-adb375096600 under account Phillip Trent's Account as user priyankn@ca.ibm.com...

ID            cef8df3f-e1d4-4620-8ef4-94901ce86f26   
Address       169.61.244.117   
Name          rickshaw-iot   
Target        primary(ec5999db-.)   
Target Type   intf   
Target IP     10.10.10.8   
Created       7 hours ago   
Status        available   
Zone          us-south-1  
```

## Accessing the application

The application should now be available on the FloatingIP at port 7000 - http://169.61.244.117:7000

## Error Scenarios

List error scenarios.

## Documentation Provided


[VPC documentation](hhttps://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-on-classic-getting-started)
