# Haneen Alyhya -- SDA 1001

# Blog App MERN Deployment

## Part 1: MongoDB Atlas Configuration (OPTIONAL)

- **MongoDB Atlas Setup**:
  - You can use the connection string from the `.env` file in the backend or create a new one by following these steps:
    1. Create an account on MongoDB Atlas.
    2. Create a free Cluster.
    3. Allow EC2 IP in the Network Access settings.
    4. Create a Database User and get your connection string.

---

## Part 2: S3 Bucket for Frontend

1. **Create a Bucket** named `yourname-blogapp-frontend` in `eu-north-1`.
2. **Disable Block all public access**: Uncheck the option to allow public access.
3. **Enable Static website hosting**.
4. **Add a Bucket Policy** to allow public access as shown below:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::yourname-blogapp-frontend/*"
    }
  ]
}

Deliverables:

Screenshot of the public URL for S3 after uploading index.html.

Output of curl -I showing 200 OK.

Example:
curl -I https://yourname-blogapp-frontend.s3.eu-north-1.amazonaws.com/index.html
# 200 OK


## Part 3: S3 for Media Uploads
1. Create a second Bucket named yourname-blogapp-media.

2. Disable Block all public access.

3. Configure CORS to support file uploads via the browser:
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": ["ETag"]
  }
]
4. Test file upload and retrieval.

Deliverables:

Screenshot proving file upload success.

## Part 4: IAM user and policy for S3 media bucket access

1. Go to IAM Console > Users > Add users.

2. User name: blog-app-user.

3. Permissions: Attach existing policies directly.

4. Create a custom policy with the following JSON (replace yourname-blogapp-media with your actual bucket name):

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::yourname-blogapp-media",
        "arn:aws:s3:::yourname-blogapp-media/*"
      ]
    }
  ]
}

4. Attach this policy to the user.

5. Save the Access Key ID and Secret Access Key (do not share these).

## Part 5: EC2 Backend Setup
1. Create an EC2 instance of type t3.micro in eu-north-1 using Ubuntu 22.04 LTS.

2. Allow only necessary traffic: SSH (22), HTTP (80), HTTPS (443), Custom TCP (5000).

3. SSH into the EC2 instance and run the following User Data script:

#!/bin/bash 
apt update -y 
apt install -y git curl unzip tar gcc g++ make unzip 
su - ubuntu << 'EOF'
export NVM_DIR="$HOME/.nvm" 
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash 
source "$NVM_DIR/nvm.sh" 
nvm install --lts 
nvm use --lts 
npm install -g pm2 
EOF 

# MongoDB Shell 
curl -L https://downloads.mongodb.com/compass/mongosh-2.1.1-linux-x64.tgz -o mongosh.tgz 
tar -xvzf mongosh.tgz 
mv mongosh-*/bin/mongosh /usr/local/bin/ 
chmod +x /usr/local/bin/mongosh 
rm -rf mongosh* 

# AWS CLI 
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
unzip awscliv2.zip 
./aws/install 
rm -rf aws awscliv2.zip 
4. Clone the MERN App:
git clone https://github.com/cw-barry/blog-app-MERN.git /home/ubuntu/blog-app 
cd /home/ubuntu/blog-app 
5. Configure the .env file for the backend with MongoDB, JWT, and AWS S3 settings:
# Create a .env file in the backend
cat > .env << EOF 
PORT=5000 
HOST=0.0.0.0 
MODE=production 

# MongoDB settings 
MONGODB=mongodb+srv://Hav-mongodb-connection-string

# JWT settings
JWT_SECRET=$(openssl rand -hex 32) 
JWT_EXPIRE=30min 
JWT_REFRESH=$(openssl rand -hex 32) 
JWT_REFRESH_EXPIRE=3d 

# AWS S3 settings
AWS_ACCESS_KEY_ID=<access-key-from-step-6> 
AWS_SECRET_ACCESS_KEY=<secret-key-from-step-6> 
AWS_REGION=eu-north-1 
S3_BUCKET=yourname-blogapp-media
MEDIA_BASE_URL=https://haneen-blogapp-media.s3.eu-north-1.amazonaws.com

# Miscellaneous
DEFAULT_PAGINATION=20 
EOF
6. Configure the frontend .env file:
cd ../frontend 

# Create .env file in frontend
cat > .env << EOF 
VITE_BASE_URL=http://<your-ec2-dns>:5000/api 
VITE_MEDIA_BASE_URL=https://haneen-blogapp-media.s3.eu-north-1.amazonaws.com 
EOF
7. Configure AWS CLI:
aws configure 
# Enter Access Key ID when prompted 
# Enter Secret Access Key when prompted 
# Select default region `eu-north-1` 
# Select default output format `json`
8. Deploy the Backend:
cd /home/ubuntu/blog-app/backend 
npm install 
mkdir -p logs 
pm2 start design.png --name "blog-backend" 
pm2 startup 
sudo pm2 startup systemd -u ubuntu 
sudo env PATH=$PATH:/home/ubuntu/.nvm/versions/node/v22.15.0/bin /home/ubuntu/.nvm/versions/node/v22.15.0/lib/node_modules/pm2/bin/pm2 
9.Build and Deploy the Frontend:
cd /home/ubuntu/blog-app/frontend 
npm install -g pnpm@latest-10 
pnpm install 
pnpm run build 
aws s3 sync dist/ s3://yourname-blogapp-frontend/ 

Deliverables:

Screenshot of the backend server running via pm2.

Screenshot of the frontend on S3.

Now everything is clearly detailed in one message for easier copying.
