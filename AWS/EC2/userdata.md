## User Data Script (Amazon Linux 2 with Apache)

## When launching your EC2, paste this in Advanced → User Data:
```
#!/bin/bash
# Update and install Apache
yum update -y
yum install -y httpd

# Enable & start Apache
systemctl enable httpd
systemctl start httpd

# Get IMDSv2 token
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Get instance details
AZ=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/placement/availability-zone)

ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id)

# Write custom index.html
echo "<html>
<head><title>EC2 Demo</title></head>
<body style='font-family: Arial; text-align: center; margin-top: 100px;'>
  <h1>Hello from EC2 Instance</h1>
  <h2>Instance ID: $ID</h2>
  <h2>Availability Zone: $AZ</h2>
</body>
</html>" > /var/www/html/index.html
```

## For Ubuntu (Nginx instead of Apache)

```
#!/bin/bash
apt-get update -y
apt-get install -y nginx

systemctl enable nginx
systemctl start nginx

TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

AZ=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/placement/availability-zone)

ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id)

echo "<html>
<head><title>EC2 Demo</title></head>
<body style='font-family: Arial; text-align: center; margin-top: 100px;'>
  <h1>Hello from EC2 Instance</h1>
  <h2>Instance ID: $ID</h2>
  <h2>Availability Zone: $AZ</h2>
</body>
</html>" > /var/www/html/index.nginx-debian.html
```
