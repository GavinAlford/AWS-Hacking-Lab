# AWS-Hacking-Lab
AWS Hacking Lab

Before we get started on the AWS parts, lets download our virtual machine!

Make your way to https://www.vulnhub.com and search for the vulnerable machine labeled "breach" 

![2](https://user-images.githubusercontent.com/110936713/183985361-c45f0146-d017-4ebf-96e0-b99bfe018873.PNG)

Go ahead and download this machine and take note of the IP Adress it has listed we will need it for later

![3](https://user-images.githubusercontent.com/110936713/183985556-c52ff9eb-a6d5-414a-b8f9-0d11c4c5ad05.PNG)

Now since the image had its DHCP disabled that means we are going to have to break into it by creating our own VPC in Aws, to do this make your way back to AWS click on services > Network & Content Delivery > VPC 

![4](https://user-images.githubusercontent.com/110936713/183986108-6621e527-4d57-4db7-99fc-3272d3e8e1b3.PNG)

Next click create VPC 

![5](https://user-images.githubusercontent.com/110936713/183986152-99f70b8d-dfc7-4e4f-9fa8-d608b1cd8621.PNG)

Here you want to change the name of the machine to your liking, then change the IP adress to mach the vulnerable machine we downloaded earlier and then change the number of AZ's down to 1

![7](https://user-images.githubusercontent.com/110936713/183986331-2f288ca8-53cc-49d1-a172-264b83c4703b.PNG)

Then click Launch
Now we want to go back to services > compute > EC2 

![8](https://user-images.githubusercontent.com/110936713/183986511-d5bd0de7-0b6f-4d73-b0ec-2021066845bd.PNG)

You are going to click launch instance and name the lab, then search for the image kali linux 

![10](https://user-images.githubusercontent.com/110936713/183986648-a1c0166c-07bd-4b7f-8671-87a92788cc49.PNG)

Once you have found the image make sure you keep the image size at t2.medium or t2.small, then create a key pair so you can log into your machine 

![10](https://user-images.githubusercontent.com/110936713/183986907-6ab59d12-819e-4318-b79a-77d93605a34f.PNG)

Dont change any of the settings name the key and place it in a directory you will remember later 

![11](https://user-images.githubusercontent.com/110936713/183987039-03d0bcf4-c3ca-447f-b2b4-8a230da543f7.PNG)

Next change your VPC to the one you created, change the subnet to public and enable auto assign IP

![13](https://user-images.githubusercontent.com/110936713/183987360-a005c8e1-4119-4eb8-8e0c-019a088e71d4.PNG)

Next we are going to need to create some storage, so go to services > storage > s3

![13](https://user-images.githubusercontent.com/110936713/183987512-aae35e7f-bc8f-453d-b2cb-d258d060b0e4.PNG)

Then click on create bucket 

![14](https://user-images.githubusercontent.com/110936713/183987580-0f8bf993-45e7-4efe-bee6-4db792da789a.PNG)

Here you want to make sure that you have the bucket in the same region you set up your server in. Also make sure you disable the block all public acess button 

![15](https://user-images.githubusercontent.com/110936713/183987759-e7d50d4e-c538-4768-bf1d-8ee6b7b98c4f.PNG)

Next you want to unzip and upload your breach image you downloaded earlier to your bucket 

![17](https://user-images.githubusercontent.com/110936713/183987961-cc184da7-ccfc-46af-a704-19589376d55b.PNG)

Open up your breach file in another tab, because we will need their s3 url and ARN shortly 

![17](https://user-images.githubusercontent.com/110936713/183988220-cdd0f146-cd93-4a84-bb72-d321b6cffe55.PNG) 

In another tab locate and click on the AWS command console 

![18](https://user-images.githubusercontent.com/110936713/183988312-1618b7cb-f035-45f6-9727-2b1406cbc9e8.PNG)

Once your shell has finished loading, we can create a new file using this command:

vi containers.json

Now hit the letter "i" to enter insert mode 

Copy and paste the following code 

[
{
"Description": "My Server OVA",
"Format": "ova",
"Url": "s3://my-import-bucket/vms/my-server-vm.ova"
}
]

From here we want to edit this code by replacing "My Server OVA" with "<Your Server Name>" and the s3 url with your buckets s3 url 
  
![20](https://user-images.githubusercontent.com/110936713/183989034-4c8964c5-b492-468c-bed7-17a33b78f25a.PNG)

 Once done hit the Esc button followed by :wq to write and quit the file 
  
 Now we need to adjust our permissions to import the file. To do that we need to open another file using the command 
  
 vi trust-policy.json
  
 Press i again and paste the following command
  
 {
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Principal": { "Service": "vmie.amazonaws.com" },
"Action": "sts:AssumeRole",
"Condition": {
"StringEquals":{
"sts:Externalid": "vmimport"
}
}
}
]
}
  
Then we can press esc and :wq to write and exit the file 
  
Now to create a policy paste this command 
  
aws iam create-role --role-name vmimport --assume-role-policy-document "file://~/trust-policy.json"
  
Now we need to create a role 

vi role-policy.json
  
Press "i" again and paste this 
  
{
"Version":"2012-10-17",
"Statement":[
{
"Effect": "Allow",
"Action": [
"s3:GetBucketLocation",
"s3:GetObject",
"s3:ListBucket"
],
"Resource": [
"<your arn>",
"<your arn>/*"
]
},
{
"Effect": "Allow",
"Action": [
"s3:GetBucketLocation",
"s3:GetObject",
"s3:ListBucket",
"s3:PutObject",
"s3:GetBucketAcl"
],
"Resource": [
"<your arn>",
"your arn/*"
]
},
{
"Effect": "Allow",
"Action": [
"ec2:ModifySnapshotAttribute",
"ec2:CopySnapshot",
"ec2:RegisterImage",
"ec2:Describe*"
],
"Resource": "*"
}
]
}

Now we want to change the areas under the recorce tabs with our own arn's from our bucket 
  
![22](https://user-images.githubusercontent.com/110936713/183989982-edfbc49d-c575-4cc6-8f0e-cb114b507d42.PNG)
  
![23](https://user-images.githubusercontent.com/110936713/183990021-b54a2bd8-bd9d-49c9-b841-136b5792e660.PNG)
  
When you are finished press Esc and :wq (write and quit) the file

From here navigate your way back to EC2 and create another instance using the breach image we just created 
  
![24](https://user-images.githubusercontent.com/110936713/183990226-51422c95-e9ae-433f-b245-bbcedfc845aa.PNG)

It is very cruical that you proceed without a key pair 
  
![25](https://user-images.githubusercontent.com/110936713/183990334-b83d7b3f-2434-4dbb-bec3-2eb1715808b7.PNG)

Next you will need to modify the security group
  
![image](https://user-images.githubusercontent.com/110936713/183990463-16a6fcfc-233d-49d9-b33f-98252ef0ae9d.png)

Now we can launch our instance by navigating our way back to the AWS consoncle and clicking on instances 
  
Go back to the main instances page and select our Kali Box, then click the connect button. Click SSH and copy the example 
  
![image](https://user-images.githubusercontent.com/110936713/183990800-447904cf-5e4b-456d-aea4-6dc92400f0f4.png)
  
Launch the commad line of your choice and connect to you kali by navigating to the directory your key is in and inputing the following command 
  
ssh -i <your key.pem> kali@ec2-18-116-43-16.us-east-2.compute.amazonaws.com

Say yes to the fingerprint and you are in 
  
https://files.cdn.thinkific.com/file_uploads/337008/images/724/e04/f51/1654274516322.jpg
  
From here go back to the instances screen and copy the private IP of the vulnerable machine and attemopt to ping!







