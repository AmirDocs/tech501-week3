# Two-tier Architecture deployed on AWS - Walkthrough

## Create your key pair on AWS

Create your key pair, associate it with your instance and download your (.pem) file. key on the AWS console in 'Launch instance'

    - Ensure AWS region is set to Ireland.

Set the correct permissions for the private key file (if needed):

```bash
chmod 400 /path/to/your-key-pair.pem
```

Replace /path/to/your-key-pair.pem with the actual path to your private key file (~/.ssh).

SSH into your instance using its Public DNS: your EC2 instance's public IP address:

```bash
ssh -i "tech501-amir-aws-key.pem" ubuntu@ec2-34-251-183-73.eu-west-1.compute.amazonaws.com
```
```bash
Accept the SSH key fingerprint:

The first time you connect, you'll be asked to confirm the host's fingerprint. Type yes to continue.
```

*or create the key pair on the console with:*

 ```bash
    ssh-keygen -t rsa -b 4096 -C <enter_personal_email_address>
```

## Create your Database Virtual Machine

- **name**: `tech501-amir-sparta-app-db-vm`
- **image**: `Ubuntu 22.04 LTS`
- **instance_type**: `t3.micro`

  - **key pair (login)**: `tech501-amir-aws-key`

    - **network settings**:
      - **VPC**: `default`
      - **Subnet**: `DevOpsStudent default 1c` (or `no preference`)
      - **auto-assign public IP**: `enable`
    
    - **firewall (security groups)**:
      - **Create security group**:
        - **name**: `tech501-amir-sparta-app-db-nsg`
        - **description**: `allow-inbound-anywhere-mongodb`
        
        - **ssh**:
          - **protocol**: `tcp`
          - **port range**: `22`
          - **source type**: `anywhere`
          - **description**: `AllowSSH`
        
        - **Custom TCP**:
          - **protocol**: `TCP`
          - **port range**: `27017`
          - **source type**: `custom`
          - **source**: `APP VM Private IP` *
          - **description**: `allow-mongo-db`

SSH into the DB virtual machine:

        ```bash
        ssh -i <tech501-amir-aws-key> ubuntu@<db_vm_public_ip>
        ```

############    - check mongodb status:

## Database Virtual Machine Script

```bash
#!/bin/bash
# This script will deploy a MongoDB database on an Ubuntu 22.04 server

# Step 2: Update package list and upgrade existing packages
sudo apt update && sudo apt upgrade -y

# Step 3: Install required dependencies (gnupg and curl)
sudo apt install gnupg curl -y

# Step 4: Import the MongoDB public GPG key
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Step 5: Add the MongoDB repository to sources list
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Step 6: Reload the package database
sudo apt update

# Step 7: Install MongoDB components (version 7.0.6)
sudo apt-get install -y mongodb-org=7.0.6 mongodb-org-database=7.0.6 mongodb-org-server=7.0.6 mongodb-mongosh mongodb-org-mongos=7.0.6 mongodb-org-tools=7.0.6

# Step 8: Enable and start the MongoDB service
sudo systemctl enable mongod
sudo systemctl start mongod

# Step 9: Modify MongoDB bindIp in the configuration file (for testing only!)
sudo sed -i 's/127.0.0.1/0.0.0.0/' /etc/mongod.conf

# Step 10: Restart the MongoDB service to apply changes
sudo systemctl restart mongod
```


## Create your Application Virtual Machine

- **name**: `tech501-amir-sparta-app-vm`
- **image**: `Ubuntu 22.04 LTS`
- **instance_type**: `t3.micro`
- **key pair (login)**: `tech501-amir-aws-key`

  - **network settings**:
    - **VPC**: `default`
    - **Subnet**: `DevOpsStudent default 1c` (or `no preference`)
    - **auto-assign public IP**: `enable`
    
    - **firewall (security groups)**:
      - **Create security group**:
        - **name**: `tech501-amir-sparta-app-nsg`
        - **description**: `allow-inbound-SSH`
        
        - **SSH**:
          - **protocol**: `tcp`
          - **port range**: `22`
          - **source type**: `anywhere`
          - **description**: `AllowSSH`
        
        - **HTTP**:
          - **protocol**: `tcp`
          - **port range**: `80`
          - **source type**: `anywhere`
          - **description**: `AllowHTTP`

SSH into your App virtual machine with:

        ```bash
        ssh -i <private_key_path> ubuntu@<app_vm_public_ip>
        ```

### Application Virtual Machine Script

The code below can be applied as User Data, a Script or pasted individually in the terminal.

```bash
#!/bin/bash

# Step 1: Update and upgrade package list
sudo apt update 
sudo apt upgrade -y

# Step 2: Install Nginx
sudo apt install nginx -y

# Step 3: Enable and start Nginx
sudo systemctl enable nginx
sudo systemctl start nginx

# Step 4: Install Node.js and npm
sudo DEBIAN_FRONTEND=noninteractive bash -c "curl -fsSL https://deb.nodesource.com/setup_20.x | bash -"
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y nodejs

# Step 5: Install PM2 process manager
sudo npm install -g pm2

# Step 6: Clone the Node.js application repository
git clone https://github.com/AmirDocs/tech501-sparta-app.git

# Step 7: Configure Nginx as a reverse proxy
sudo sed -i 's|try_files.*|proxy_pass http://127.0.0.1:3000;|' /etc/nginx/sites-available/default

# Step 8: Restart Nginx to apply changes
sudo systemctl reload nginx

# Step 9: Set up the database connection environment variable
export DB_HOST=mongodb://<db_private_ip>:27017/posts

# Step 10: Install application dependencies, check database connection, clear and reseed the database
npm install

# Step 11: Start the Node.js application in the background using PM2
pm2 start app.js
```

Your Application and /Post page should load.

![alt text](<../images-videos/Screenshot 2025-02-05 182852.png>)

## Troubleshooting

![alt text](<../images-videos/Screenshot 2025-02-05 173215.png>)
- /posts page does not load - something to do with the VM's connection to each other.