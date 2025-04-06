Sure Nishant! Here's the updated version of your instructions with all the URLs and image links **removed** — just plain text:

---

# webapp-asg-alb

Webapp Deployment with an AWS AutoScaling Group, an Application Load Balancer, and a Dynamic Scaling policy (metric: CPU Utilization) configured using a CloudWatch Alarm, SNS, and IAM.

## Steps to configure a WebApp running on an AWS EC2 instance (part of AutoScaling Group attached to an Application Load Balancer):

1. Create an IAM user (example: adminjohn) with Administrator access.

2. Create Security Groups:
   - One for the Launch Template (LT)
   - One for the Application Load Balancer (ALB)

3. Create a Launch Template and an Auto Scaling Group (ASG).

4. Launch an EC2 instance from the Launch Template.

5. SSH into the EC2 instance and register a custom AMI (Amazon Machine Image) with WebApp configuration.

### WebApp configuration steps:

```bash
sudo su -
yum install httpd -y
git clone <webapp repo>
cp -r webapp-asg-alb/* /var/www/html
systemctl start httpd
systemctl enable httpd
```

6. Update the Launch Template with the new AMI.

7. Perform ASG Instance Refresh.

8. Create a Target Group (TG) and an Application Load Balancer (ALB).

9. Attach the ALB with the Auto Scaling Group.

10. Create a Dynamic Scaling Policy with a CloudWatch Alarm and SNS (Simple Notification Service).

11. Run a CPU stress test on the ASG and check if the scaling event triggers.

---

### Steps to install and configure stress utility on Amazon Linux:

```bash
amazon-linux-extras install epel -y
yum install stress -y
stress --cpu 1 --timeout 800 &
```

---

Let me know if you want this exported as a PDF or markdown file too!
