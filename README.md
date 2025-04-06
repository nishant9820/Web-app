# 🔐 EC2, ASG & ALB Deployment with Mixed Console & Terraform Setup

This guide explains how to deploy a high-availability web application using a combination of manual AWS Console steps and automated Terraform configurations. Key components include EC2 Bastion Host, Launch Template, Auto Scaling Group (ASG), Application Load Balancer (ALB), IAM setup, and CloudWatch monitoring with email alerts.

---

## 🏗️ Overview

We deploy a static web application hosted on EC2 instances, managed via an Auto Scaling Group (ASG), behind an Application Load Balancer (ALB). Some resources are created manually via the AWS Console, while others are automated using Terraform.

---

## 📌 Part 1: Manual Setup via AWS Console

### 🔑 IAM User

1. Create an IAM user with the following permissions:
   - AmazonEC2FullAccess
   - AutoScalingFullAccess
   - AmazonSNSFullAccess
   - CloudWatchFullAccess
2. Generate Access Key ID and Secret Key for Terraform use.

### 🛡️ Security Groups

#### ALB-SG (for ALB)

- **Inbound:** HTTP (80) from `0.0.0.0/0`
- **Outbound:** All traffic

#### LT-SG (for EC2 instances)

- **Inbound Rules:**
  - SSH (22) from `###.###.###.###/32` (your IP)
  - HTTP (80) from **ALB-SG**
- **Outbound:** All traffic

### 💻 Bastion Host (Manual EC2)

1. Launch a new EC2 instance (Amazon Linux 2) in a **public subnet**.
2. Enable auto-assign public IP.
3. Attach the LT-SG (with SSH allowed from your IP).
4. Use this as your bastion host to access instances in private subnets.

### 📦 Launch Template (LT)

1. Create from Console
2. Set user-data as:

```bash
#!/bin/bash
sudo su -
yum install httpd -y
systemctl start httpd
systemctl enable httpd
yum install git -y
cd /home/ec2-user
git clone https://github.com/nishant9820/Web-app.git
cp -r Web-app/webapp-asg-alb/* /var/www/html/
```

3. Use LT-SG and your existing key pair.

### 🔁 Auto Scaling Group (ASG)

1. Create ASG from Console using above Launch Template.
2. Set **Min**, **Max**, and **Desired** capacity all to `1`.
3. Attach the ALB Target Group (created via Terraform).
4. Enable default health checks.
5. Enable notifications via CloudWatch alarms and SNS email.

---

## 📌 Part 2: Terraform Setup

### 📂 IAM Setup

```hcl
resource "aws_iam_user" "terraform_user" {
  name = "terraform-user"
}

resource "aws_iam_user_policy_attachment" "attach_policies" {
  for_each = toset([
    "arn:aws:iam::aws:policy/AmazonEC2FullAccess",
    "arn:aws:iam::aws:policy/AutoScalingFullAccess",
    "arn:aws:iam::aws:policy/AmazonSNSFullAccess",
    "arn:aws:iam::aws:policy/CloudWatchFullAccess",
  ])
  user       = aws_iam_user.terraform_user.name
  policy_arn = each.value
}
```

### 🌐 ALB & Target Group

```hcl
resource "aws_lb" "web_alb" {
  name               = "web-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = ["sg-########"]
  subnets            = var.public_subnets
}

resource "aws_lb_target_group" "web_tg" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = var.vpc_id
  health_check {
    path     = "/"
    interval = 30
  }
}

resource "aws_lb_listener" "web_listener" {
  load_balancer_arn = aws_lb.web_alb.arn
  port              = 80
  protocol          = "HTTP"
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web_tg.arn
  }
}
```

### 📈 CloudWatch & SNS Email Notification

```hcl
resource "aws_sns_topic" "alert_topic" {
  name = "web-alerts"
}

resource "aws_sns_topic_subscription" "email_sub" {
  topic_arn = aws_sns_topic.alert_topic.arn
  protocol  = "email"
  endpoint  = "your-email@example.com"
}

resource "aws_cloudwatch_metric_alarm" "high_cpu_alarm" {
  alarm_name          = "cpu-util-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 60
  statistic           = "Average"
  threshold           = 40
  alarm_actions       = [aws_sns_topic.alert_topic.arn]
  dimensions = {
    AutoScalingGroupName = "your-asg-name"
  }
}
```

---

## ✅ Validation Steps

### 🔗 Access the Web App

1. Go to AWS Console → EC2 → Load Balancers → Select your ALB.
2. Copy the **DNS name** of your ALB.
3. Paste it into your browser. You should see the static web application.

### 🔁 Test Auto Scaling

1. Go to ASG → Edit scaling policy.
2. Temporarily reduce the threshold (e.g., set CPU threshold to 10%).
3. Simulate load using stress script or manually stop instances.
4. Observe if new EC2 instances are launched and attached to the ALB.

### 📧 Test Alerts

1. Trigger an alarm condition (e.g., high CPU load).
2. Confirm that an alert email is received via SNS subscription.

---

## 📊 Architecture Diagram

```text
                [ Your Local Machine ]
                          |
                          v
       [ Bastion Host (Public EC2 in Console) ]
                          |
                          v
         [ ALB (via Terraform - Public Facing) ]
                          |
                          v
   [ Target Group ] <----> [ ASG (from Console using LT) ]
                          |
                          v
         [ Private EC2 (with user-data script) ]
```

---

## 📬 Result Summary

- ✅ Bastion Host, Security Groups, Launch Template, and ASG created from Console
- ✅ ALB, Target Group, Listener, and CloudWatch alerts setup using Terraform
- ✅ DNS-based access to ALB verified
- ✅ SNS Email Alerts triggered on high CPU

🔗 **Visit your ALB DNS in browser to access the web app**  
💻 **SSH into EC2 using Bastion Host from Console**  
📩 **Check email for CloudWatch alerts when CPU crosses threshold**

