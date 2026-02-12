# Personal Portfolio Website with External Image Storage


## A. EC2 Infrastructure Setup

## Objective

The objective of this lab is to launch and configure an Amazon EC2 instance using Amazon Linux 2. The instance is secured using a security group that allows SSH and HTTP access, and an SSH key pair is created to enable secure remote login.

---

## 1. Launch an Amazon Linux 2 EC2 Instance

1. Log in to the AWS Management Console.
2. Navigate to **EC2** from the AWS services menu.
3. Click **Launch instance**.
4. Configure the instance with the following settings:
   - **Name**: `WebServer-AL2`
   - **Amazon Machine Image (AMI)**: Amazon Linux 2 AMI (HVM)
   - **Architecture**: 64-bit (x86)

---

## 2. Select Instance Type

1. Under **Instance type**, select:
   - `t2.micro`
2. Confirm that the instance type is Free Tier eligible.

---

## 3. Configure Security Group

### 3.1 Create a New Security Group

1. Select **Create new security group**.
2. Provide the following details:
   - **Security Group Name**: `WebServer-SG`
   - **Description**: Allows SSH and HTTP access

### 3.2 Configure Inbound Rules

Add the following inbound rules:

| Type | Protocol | Port Range | Source    |
|------|----------|------------|-----------|
| SSH  | TCP      | 22         | My IP     |
| HTTP | TCP      | 80         | 0.0.0.0/0 |


- SSH (port 22) allows secure remote access to the EC2 instance.
- HTTP (port 80) allows public access to the web server.

Outbound rules are left as default (allow all traffic).

---

## 4. Create and Configure an SSH Key Pair

1. In the **Key pair (login)** section, click **Create new key pair**.
2. Configure the key pair as follows:
   - **Key pair name**: `WebServerKey`
   - **Key pair type**: RSA
   - **Private key format**: `.pem`
3. Download the key pair file and store it securely.

> **Note:** The private key file cannot be downloaded again once lost.

---

## 5. Launch the EC2 Instance

1. Review the instance configuration.
2. Click **Launch instance**.
3. Wait until the instance state shows **Running**.

---

## 6. Connect to the EC2 Instance Using SSH

### 6.1 Accessing the Instance via Terminal

1. Open a terminal (Git Bash on my Windows).
2. Navigate to the directory containing the key pair file:

```bash
cd Downloads
```

3. Change the file permissions:

```bash
chmod 400 WebServerKey.pem
```

4. Connect to the EC2 instance:

```bash
ssh -i WebServerKey.pem ec2-user@<PUBLIC_IP>
```


Replace <PUBLIC_IP> with the public IPv4 address of the EC2 instance.

### Expected Outcome

- Amazon Linux 2 EC2 instance is successfully launched.

- Security group rules allow SSH and HTTP traffic.

- Secure SSH connection to the EC2 instance is established.


### Conclusion

This lab demonstrated how to provision and secure an Amazon EC2 instance using Amazon Linux 2. Security groups and SSH key pairs were properly configured to ensure controlled and secure access.




## B. Web Server & Website Deployment


## Objective
The objective of this lab is to install and configure the Apache web server on an Amazon Linux 2 EC2 instance, deploy a Tooplate HTML template, and confirm the website is accessible via the EC2 public IP address.

---

## 1. Install and Start Apache (httpd)

1. Connect to the EC2 instance via SSH.
2. Update the package repository:

```bash
sudo yum update -y
```

3. Install Apache:

```bash
sudo yum install httpd -y
```

4. Start the Apache service:

```bash
sudo systemctl start httpd
```

5. Enable Apache to start automatically at system boot:

```bash
sudo systemctl enable httpd
```

6. Check the status of Apache:

```bash
sudo systemctl status httpd
```

Expected Result: Apache should show as active (running).

## 2. Download a Tooplate HTML Template
1. Navigate to a temporary directory:

```bash
cd /tmp
```
2. Install the unzip und wget utility if not already installed:

```bash
sudo yum install unzip wget -y
```

3. Download a Tooplate template (replace <TEMPLATE_URL> with the actual template link):

```bash
wget <TEMPLATE_URL> -O website.zip
```

4. Install the unzip und wget utility if not already installed:

```bash
sudo yum install unzip wget -y
```

5. Extract the template files:

```bash
unzip website.zip -d website
```

## 3. Deploy the Website

1. Copy the website files to Apache’s default root directory:

```bash
sudo cp -r website/* /var/www/html/
```

2. Set proper permissions:

```bash
sudo chown -R apache:apache /var/www/html/
sudo chmod -R 755 /var/www/html/
```
This ensures Apache can read and serve the website files correctly.

## 4. Confirm Website Access

1. Open a web browser.

2. Enter the EC2 instance’s public IPv4 address:

```
http://<EC2_PUBLIC_IP>
```

This loads the Tooplate HTML template indicating successful deployment.

### Summary of Commands

```bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
cd /tmp
wget <TEMPLATE_URL> -O website.zip
sudo yum install unzip -y
unzip website.zip -d website
sudo cp -r website/* /var/www/html/
sudo chown -R apache:apache /var/www/html/
sudo chmod -R 755 /var/www/html/
```

### Conclusion
This lab demonstrated the installation and configuration of the Apache web server on an EC2 instance. A Tooplate HTML template was successfully deployed, and the website was confirmed to be accessible via the EC2 public IP, demonstrating proper web server deployment and file permission management.



# Elastic Block Store (EBS) Configuration

## Objective
The objective of this lab is to create and attach an Amazon Elastic Block Store (EBS) volume to an EC2 instance, partition and format the volume, and mount it so it can be used to store data persistently.

---


## 1. Create a New EBS Volume

1. Navigate to **AWS Management Console → EC2 → Elastic Block Store → Volumes**.
2. Click **Create volume**.
3. Configure the volume:
   - **Volume type**: General Purpose SSD (gp3)
   - **Size**: Minimum 5 GB
   - **Availability Zone**: Must be the same as that of the EC2 instance
4. Click **Create volume**.

---

## 2. Attach the Volume to the EC2 Instance

1. Select the newly created volume.
2. Click **Actions → Attach volume**.
3. Configure:
   - **Instance**: Select your EC2 instance
   - **Device name**: `/dev/sdf` (maps to `/dev/nvme1n1` on Amazon Linux 2)
4. Click **Attach volume**.

---

## 3. Partition the Disk

1. SSH into your EC2 instance.
2. List all disks:

```bash
lsblk
```

3. Partition the new disk:

```bash
sudo fdisk /dev/nvme1n1
```

**Inside fdisk:**

- Type n → new partition

- Partition type: p → primary

- Partition number: 1

- First sector: press Enter (default)

- Last sector: press Enter (default, use full disk)

- Type w:  write changes and exit


## 4. Mount the Volume

1. Create a mount point:

```bash
sudo mkdir /mnt/ebs_volume
```

2. Mount the new partition:

```bash
sudo mount /dev/nvme1n1p1 /mnt/ebs_volume
```

2. Verify the mount:
```bash
df -h
```

### Summary of Commands

```bash
lsblk
sudo fdisk /dev/nvme1n1
sudo mkfs.ext4 /dev/nvme1n1p1
sudo mkdir /mnt/ebs_volume
sudo mount /dev/nvme1n1p1 /mnt/ebs_volume
df -h
```


# D. Website Image Migration

## Objective
The objective of this lab is to migrate website images to an attached EBS volume to ensure data persistence, create a symbolic link to maintain the website structure, and verify that images are served correctly by the Apache web server.

---

## 1. Move Website Images to the EBS Volume

1. SSH into the EC2 instance.
2. Move the website images folder to the EBS volume. Assuming images are located at `/var/www/html/images` and the EBS volume is mounted at `/mnt/ebs_volume`:

```bash
sudo mv /var/www/html/images /mnt/ebs_volume/
```
3. Verify the images are now on the EBS volume:

```bash
ls /mnt/ebs_volume/images
```

All website image files should be present in the EBS volume directory.


## 2. Replace Original Images Folder with a Symbolic Link

1. Remove the old images folder (if it still exists):

```bash
sudo rm -rf /var/www/html/images
```

2. Create a symbolic link pointing from the original website path to the EBS volume:

```bash
sudo ln -s /mnt/ebs_volume/images /var/www/html/images
```

3. Verify the symbolic link:

```bash
ls -l /var/www/html
```

The images folder should now appear as a symbolic link pointing to /mnt/ebs_volume/images.

# 3. Restart Apache and Confirm Images Display

1. Restart the Apache web server:

```bash
sudo systemctl restart httpd
```

2. Open a web browser and access the website using the EC2 public IP:

```
http://<EC2_PUBLIC_IP>
```

3. Confirm that all images load correctly.

Successful image display indicates that the migration and symbolic link configuration were successful.

### Summary of Commands

```bash
sudo mv /var/www/html/images /mnt/ebs_volume/
sudo rm -rf /var/www/html/images
sudo ln -s /mnt/ebs_volume/images /var/www/html/images
sudo systemctl restart httpd
```

### Conclusion

This lab demonstrated how to migrate website images to an EBS volume for persistent storage, create a symbolic link to maintain website structure, and verify functionality by restarting Apache and testing image display. Using an EBS volume ensures that website assets are preserved even if the EC2 instance is terminated or replaced.


# Lab Report: Persistence Configuration for EBS Volume

## Objective
The objective of this lab is to configure the attached EBS volume to automatically mount on system reboot using `/etc/fstab`, ensuring persistent storage for website files and images.

---

## E. Persistence Configuration

---

## 17. Configure the Volume to Auto-Mount

1. SSH into the EC2 instance.
2. Find the UUID of the EBS partition:

```bash
sudo blkid /dev/nvme1n1p1
```

Copy the UUID value from the output (e.g., abcd-1234).

3. Backup the current fstab file:

```bash
sudo cp /etc/fstab /etc/fstab.bak
```

4. Edit the fstab file:

```bash
sudo vi /etc/fstab
```

5. Add a new line at the end of the file:

```bash
UUID=<UUID_FROM_BLKID> /mnt/ebs_volume ext4 defaults,nofail 0 2
```

Replace <UUID_FROM_BLKID> with the actual UUID you copied.

### Explanation of options:

- **defaults** → standard mount options

- **nofail** → allows boot even if the volume is missing

- **0 2** → filesystem check order (2 means check after root filesystem)


6. Save and exit the file (:x or :wq).


7. Test the fstab configuration without rebooting:

```bash
sudo mount -a
```

- If no errors appear, the configuration is correct.

**Verify the mount:**

```bash
df -h
```

**/mnt/ebs_volume should appear as mounted from the EBS volume.**

### Summary of Commands

```bash
sudo blkid /dev/nvme1n1p1
sudo cp /etc/fstab /etc/fstab.bak
sudo nano /etc/fstab   # add UUID line
sudo mount -a
df -h
```

### Conclusion

This lab demonstrated how to configure an EBS volume to automatically mount at system startup using /etc/fstab. This ensures that website files, images, and other data stored on the EBS volume persist across EC2 instance reboots, maintaining web server functionality.



## Why EBS Was Used for Website Images

Amazon Elastic Block Store (EBS) was used to store website images to ensure **data persistence**. Unlike the default EC2 instance storage, which is **ephemeral** and is deleted when the instance is stopped or terminated, EBS volumes are **independent, persistent storage** that remain intact even if the EC2 instance is replaced.

By storing images on an EBS volume and using a symbolic link in the website folder, the website can access the images normally while ensuring they **survive instance reboots, stops, or replacements**, making the deployment more reliable and maintainable.

