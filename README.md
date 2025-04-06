# webapp-asg-alb
Webapp Deployment with an AWS AutoScaling Group, an Application Load Balancer, and a Dynamic Scaling policy (metric: CPU Utilization) configured using a CloudWatch Alarm, SNS and IAM.

YouTube Video URL: https://youtu.be/dMUQTQS1l3g

YouTube Channel: https://www.youtube.com/@techapricate

Please follow the below steps to configure a WebApp running on an AWS EC2 instance (part of AutoScaling Group attached to an Application LB).
1.	Create an IAM user (example: adminjohn with Administrator access).
2.	Create a Security Group for LT (Launch Template) and ALB (Application Load balancer).
3.	Create a LT and an ASG (AutoScaling Group).
4.	Launch an EC2 instance from LT.
5.	SSH to the EC2 Instance and register a custom AMI (Amazon Machine Image) with WebApp Configuration.

   WebApp configuration steps:
   
   sudo su -
   
   yum install httpd -y

   git clone https://github.com/bhavukm/webapp-asg-alb.git

   cp -r webapp-asg-alb/* /var/www/html

   systemctl start httpd

   systemctl enable httpd
   
7.	Update the LT with new AMI.
8.	Perform ASG Instance Refresh.
9.	Create a TG (Target Group)) and an ALB.
10.	Attach the ALB with ASG.
11.	Create a Dynamic Scaling Policy with a CloudWatch Alarm and SNS (Simple Notification Service).
12.	Run CPU stress test on the ASG and check if scaling event triggers.   

Steps to install and configure stress utility on Amazon Linux:-

amazon-linux-extras install epel –y

yum install stress -y

stress --cpu 1 --timeout 800 &

Architectural Diagram:-

![image](https://github.com/user-attachments/assets/c88d21bd-6d6e-4f78-81cc-c5543c19745e)

AWS SNS:

![image](https://github.com/user-attachments/assets/262f3c8f-93b0-4838-a63a-4e3d41b0510f)

AWS CloudWatch:

![image](https://github.com/user-attachments/assets/eca2fdc7-7b04-4387-988b-28d46eef9df2)



