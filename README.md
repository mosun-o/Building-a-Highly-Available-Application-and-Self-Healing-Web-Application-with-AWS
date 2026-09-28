
# Building a Highly Available Application and Self Healing Web Application with AWS

This project demonstrates a highly available and self-healing web application using AWS Auto Scaling Group (ASG) and Application Load Balancer (ALB). To test the setup, an instance was manually terminated to simulate a failure, the ALB health checks flagged it as unhealthy and the ASG automatically launched a replacement instance to restore the desired capacity

## Architecture Overview


                           [ Internet Users ]
                                   │
                                   │ HTTP (Port 80)
                                   ▼
                   ┌───────────────────────────────┐
                   │ Application Load Balancer (ALB)│
                   └───────────────┬───────────────┘
                                   │
                 ┌─────────────────┴─────────────────┐
                 │                                   │
                 ▼                                   ▼
      ┌─────────────────────┐             ┌─────────────────────┐
      │ Availability Zone A │             │ Availability Zone B │
      │                     │             │                     │
      │   ┌─────────────┐   │             │   ┌─────────────┐   │
      │   │ EC2 Instance│   │             │   │ EC2 Instance│   │
      │   │ (Subnet 1)  │   │             │   │ (Subnet 2)  │   │
      │   └─────────────┘   │             │   └─────────────┘   │
      └─────────────────────┘             └─────────────────────┘
                 ▲                                   ▲
                 └─────────────────┬─────────────────┘
                                   │
                    ┌──────────────────────────────┐
                    │ Auto Scaling Group (ASG)     │
                    │ Min: 2 | Desired: 2 | Max: 4 │
                    └──────────────────────────────┘

## Request Flow

- **The web application receives HTTP traffic from the internet through the ALB. The application instances themselves are not public facing**
 
- **The ALB distributes incoming traffic across instances provisioned across 2 Availability Zones for high availability**
 
- **The ASG monitors instance health using ALB health check, and automatically replaces any instance flagged as unhealthy to maintain the desired capacity**


# Deployment Steps (with Screenshots)

- ## Step 1: Create a Launch Template for ASG with User Data Script
  - A launch template defines the configuration for the EC2 instances, which includes AMI, instance type, security groups, and user data script. ASG uses it to automatically launch new EC2 instances with the same configuration when scaling or replacing instances.


<img width="1600" height="698" alt="WhatsApp Image 2026-09-28 at 23 07 40" src="https://github.com/user-attachments/assets/11ad1dc0-e6f0-478e-96c7-94566a4b0941" />


<img width="1600" height="689" alt="WhatsApp Image 2026-09-28 at 23 10 11" src="https://github.com/user-attachments/assets/542f9c0f-9b8b-4ece-ba18-b235ab8f2557" />


<img width="1600" height="690" alt="WhatsApp Image 2026-09-28 at 23 11 47" src="https://github.com/user-attachments/assets/23763067-1912-4ffc-b7e0-7aceac04a89c" />




<img width="1600" height="692" alt="WhatsApp Image 2026-09-28 at 23 13 08" src="https://github.com/user-attachments/assets/65d76ed5-0a98-453a-8d87-1d6be3ce8889" />



<img width="1600" height="680" alt="WhatsApp Image 2026-09-28 at 23 14 43" src="https://github.com/user-attachments/assets/a99bf69b-6d73-4d8b-9d48-46608f537b46" />



- ## Step 2: Configure Security Groups
  - Create a security group for the ALB to allow inbound HTTP traffic from the internet. Also, create a separate security group for the EC2 instances that allows inbound traffic only from ALB security group ID. This ensures the instances are not directly accessible from the public internet


<img width="1600" height="679" alt="WhatsApp Image 2026-09-28 at 23 19 43" src="https://github.com/user-attachments/assets/155a2b37-e8c0-4eaa-9cc4-0e6e11d3e9e6" />


<img width="1600" height="694" alt="WhatsApp Image 2026-09-28 at 23 22 15" src="https://github.com/user-attachments/assets/265d619d-4a2c-43f6-bd3d-62a48357d7d9" />



<img width="1600" height="692" alt="WhatsApp Image 2026-09-28 at 23 23 51" src="https://github.com/user-attachments/assets/2e42a9c7-7247-4a3c-ade3-cfbe215fa319" />



<img width="1600" height="698" alt="WhatsApp Image 2026-09-28 at 23 25 43" src="https://github.com/user-attachments/assets/b3c37ad0-01ca-4931-bd89-48024c40f5b4" />




**Attach the EC2 Security Group to the Launch Template**

 - Attaching the EC2 security group to the launch template creates a new version of the template, to avoid having multiple versions, you can instead create the EC2 security group before creating the launch template and attach it the initial creation of the launch template. This will keep the template at version 1, and if you have version 2, update the launch template to use version 2 as the default version




<img width="1600" height="679" alt="WhatsApp Image 2026-09-28 at 23 27 35 (1)" src="https://github.com/user-attachments/assets/0d74349f-5951-49c3-931e-d5ec2f17c094" />





## Step 3: Deploy Target Group and Application Load Balancer

- Create a target group for the ALB and configure health checks. The target group contains the EC2 instances that receives traffic from the ALB, while the ALB uses the health checks to route traffic only to healthy instances



<img width="1600" height="686" alt="WhatsApp Image 2026-09-28 at 23 30 47" src="https://github.com/user-attachments/assets/3a6cb55f-0acc-400c-9dea-7438ce713e7a" />



<img width="1600" height="680" alt="WhatsApp Image 2026-09-28 at 23 32 06" src="https://github.com/user-attachments/assets/9c6df292-ca25-4e90-b8de-2b393d615982" />



<img width="1600" height="689" alt="WhatsApp Image 2026-09-28 at 23 33 17" src="https://github.com/user-attachments/assets/6e22eb5d-5e44-4625-879d-23e668065ad0" />



<img width="1600" height="690" alt="WhatsApp Image 2026-09-28 at 23 34 20" src="https://github.com/user-attachments/assets/db0e6228-c512-468c-947e-fe6a70558a28" />





- **Provision the ALB across 2 Availability Zones, using a public subnet in each AZ, for high availability**




<img width="1600" height="699" alt="WhatsApp Image 2026-09-28 at 23 35 37" src="https://github.com/user-attachments/assets/f36f9e85-617f-405d-93bb-3bb1427dc983" />




<img width="1600" height="680" alt="WhatsApp Image 2026-09-28 at 23 36 50" src="https://github.com/user-attachments/assets/1e047815-9efe-41e5-821c-8a628e8e3817" />





- **Attach the previously created ALB security group to the ALB and configure the listener to forward incoming traffic to the target group created earlier**



<img width="1600" height="691" alt="WhatsApp Image 2026-09-28 at 23 37 48" src="https://github.com/user-attachments/assets/3c0d872f-d041-4645-b74d-e50ed8addf3a" />



<img width="1600" height="695" alt="WhatsApp Image 2026-09-28 at 23 39 11" src="https://github.com/user-attachments/assets/35227e6b-dbea-499e-82f3-1009cde9206e" />



<img width="1600" height="691" alt="WhatsApp Image 2026-09-28 at 23 40 15" src="https://github.com/user-attachments/assets/d32475df-f984-42e8-af80-0e3a8637728a" />





## Step 4: Configure Auto Scaling Group
 - Create the Auto Scaling Group using the launch template created in Step 1, selecting version 2 (in this case) of the launch template



<img width="1600" height="687" alt="WhatsApp Image 2026-09-28 at 23 43 36" src="https://github.com/user-attachments/assets/b0ecc1b3-2945-47a6-ae33-697ac66fab2c" />



<img width="1600" height="691" alt="WhatsApp Image 2026-09-28 at 23 46 06" src="https://github.com/user-attachments/assets/983a703c-05ac-485c-8a04-06b794445d5a" />




- **Attach the ASG to the ALB TG and enable ELB health checks**

<img width="1600" height="680" alt="WhatsApp Image 2026-09-28 at 23 47 13" src="https://github.com/user-attachments/assets/02b6d173-c45e-42d3-a07c-7eea619b0824" />



<img width="1600" height="692" alt="WhatsApp Image 2026-09-28 at 23 48 23" src="https://github.com/user-attachments/assets/3916ceeb-84c2-4d69-a7f3-351b6711a1c7" />





- **Configure the desired capacity for the ASG**



<img width="1600" height="686" alt="WhatsApp Image 2026-09-28 at 23 53 43" src="https://github.com/user-attachments/assets/ffdef16e-bffc-480c-9927-cea85f37d4b9" />


- **Review the configurations and create the ASG**


<img width="1600" height="689" alt="WhatsApp Image 2026-09-28 at 23 55 20" src="https://github.com/user-attachments/assets/8e5dd913-e84b-44c0-add8-261f8c5706d0" />



<img width="1600" height="692" alt="WhatsApp Image 2026-09-28 at 23 56 18" src="https://github.com/user-attachments/assets/4e8bf2f2-008a-450e-b565-8eeafe70f037" />



<img width="1600" height="689" alt="WhatsApp Image 2026-09-28 at 23 57 32" src="https://github.com/user-attachments/assets/6434d98a-47fb-439d-abbf-687903bac84f" />



<img width="1600" height="695" alt="WhatsApp Image 2026-09-28 at 23 58 40" src="https://github.com/user-attachments/assets/3c296407-a0f0-432a-81af-c1b8d42b350e" />



<img width="1600" height="698" alt="WhatsApp Image 2026-09-28 at 23 59 30" src="https://github.com/user-attachments/assets/ee8d83df-5108-4a37-b181-96e07d993086" />





- **Register targets for the ASG**


<img width="1600" height="692" alt="WhatsApp Image 2026-09-29 at 00 00 56" src="https://github.com/user-attachments/assets/15ab4472-2eb0-49f7-94d2-f203cb3b8237" />


<img width="1600" height="689" alt="WhatsApp Image 2026-09-29 at 00 02 29" src="https://github.com/user-attachments/assets/3288178c-be55-4d87-96fd-89e278574bd1" />





## Verification and Testing: 

- Copy the ALB DNS name from the ALB page and paste it into a new incognito browser tab, prefixed with http://



<img width="1600" height="681" alt="WhatsApp Image 2026-09-29 at 00 03 46" src="https://github.com/user-attachments/assets/13a6418b-48e8-4fbb-a4f8-59fb019b9ed7" />




- **After a refresh; traffic is served from a different instance ID and AZ**



<img width="1600" height="695" alt="WhatsApp Image 2026-09-29 at 00 04 56" src="https://github.com/user-attachments/assets/83ad9b78-4232-4e26-a5f4-dd0190a0404c" />





- **Manually terminate one running instance to simulate a failure**




<img width="1600" height="698" alt="WhatsApp Image 2026-09-29 at 00 06 57" src="https://github.com/user-attachments/assets/db605225-03cf-4ea8-9f36-664949193135" />





- After manually terminating an instance, the ASG launches a replacement to restore the number of running instances to the configured desired capacity (in this case, set to 2). The minimum setting defines the lowest the ASG is allowed to scale down to, while desired capacity is the actual target the ASG continuously maintains


<img width="1600" height="677" alt="WhatsApp Image 2026-09-29 at 00 08 09" src="https://github.com/user-attachments/assets/3f8c4ea1-1fc5-4865-8e89-42464381de58" />




<img width="1600" height="703" alt="WhatsApp Image 2026-09-29 at 00 09 15" src="https://github.com/user-attachments/assets/13522fdd-5aa5-4a3b-9fee-b41b00337047" />







- **After another refresh of the server page, traffic is served by a different instance ID located in us-east-1a**




<img width="1600" height="675" alt="WhatsApp Image 2026-09-29 at 00 10 33" src="https://github.com/user-attachments/assets/f2e9fea6-f848-4f47-8f79-ac4139480018" />





The instance ID in **us-east-1b** remains the same



<img width="1600" height="692" alt="WhatsApp Image 2026-09-29 at 00 11 47" src="https://github.com/user-attachments/assets/5b3ea2ef-2c87-4c2c-b357-b1c67cd74776" />





  - **Result**:
 
 - The web application remained accessible despite an instance failure
 - The ALB detected the terminated instance as unhealthy and stopped routing traffic to it
 - The ASG automatically launched a replacement instance to restore the desired capacity of 2
 - The replacement instance passed ALB health checks and began serving traffic without any manual intervention
 - Traffic was successfully verified as being served by a new instance ID in a healthy state (us-east-1a)


## Outcome

The architecture successfully achieved high availability and self-healing confirming that a single instance failure does not cause application downtime, and that capacity is automatically restored without manual intervention 



## Issues I Encountered During This Setup and How I Resolved Them

- **At the first test, the browser returned a 'This site can't be reached' response**




<img width="1600" height="694" alt="WhatsApp Image 2026-09-29 at 00 12 54" src="https://github.com/user-attachments/assets/7305c972-dd30-4d42-a957-5158e9d9fd1d" />






**Steps I took to Resolve this Issue**

- EC2 instances not receiving traffic: The EC2 security group had not been attached to the launch template, so instances launched without the correct inbound rule allowing traffic from the ALB. I corrected this by attaching the EC2 security group to the launch template

- Existing instances still not receiving traffic after the fix; Attaching the security group created version 2 of the launch template, but the ASG default version was still set to version 1, and the already running instances had launched under the old configuration. I updated the ASG default version to version 2 and terminated existing instances, allowing the ASG to relaunch them using the corrected configuration

- Target group showing no healthy targets: the running EC2 instances had not been registered as targets in the target group, I corrected this by manually selecting the running instances and registering them as targets

- After the above steps, I tested the ALB DNS name using curl from the CLI and confirmed that the web server was responding correctly. This help verifies that the issue was not with  the web server configuration. The issue was I did not prefix the ALB DNS name with http:// in the browser, since the ALB listener was configured for HTTP on port 80 only. I accessed the ALB using http:// in the browser which resolves the issue


<img width="1600" height="695" alt="WhatsApp Image 2026-09-29 at 00 13 52" src="https://github.com/user-attachments/assets/b0905e47-6a37-447c-b310-025746dfca74" />





