</details>

******

<details>
<summary>Exercise 1: Create IAM user </summary>
 <br />

**permissions needed for the AWS user**
- Create VPC and Subnet
- Create EC2 instance 
- Create Security Group for EC2

**Create new user and a group**

```sh
# Check that you have configured AWS admin user locally
aws configure list
cat ~/.aws/credentials

# Create a new IAM user "your name" with UI and CLI access
aws iam create-user --user-name nana

# Create a group "devops"
aws iam create-group --group-name devops

# Add your user to "devops" group
aws iam add-user-to-group --user-name nana --group-name devops

# Verify that devops group contains your user
aws iam get-group --group-name devops

````

**Give user UI and CLI access**

```sh
# Generate user keys for CLI access & save key.txt file in safe location
aws iam create-access-key -user-name nana > key.txt

# Generate user login credentials for UI & save password in safe location
aws iam create-login-profile --user-name nana --password MyTestPassword123

# Give user permission to change password
aws iam list-policies | grep ChangePassword
aws iam attach-user-policy --user-name devops --policy-arn "arn:aws:iam::aws:policy/IAMUserChangePassword"

```

**Assign permissions to the user through group**

```sh
# Check which policies are available for managing EC2 & VPC services, including subnet and security groups
aws iam list-policies | grep EC2FullAccess
aws iam list-policies | grep VPCFullAccess

# Give devops group needed permissions
aws iam attach-group-policy --group-name devops --policy-arn "arn:aws:iam::aws:policy/AmazonEC2FullAccess"
aws iam attach-group-policy --group-name devops --policy-arn "arn:aws:iam::aws:policy/AmazonVPCFullAccess"

# Check policies for group
aws iam list-attached-group-policies --group-name devops

```


</details>

******

<details>
<summary>Exercise 2: Configure AWS CLI </summary>
 <br />

**steps**

```sh
# Save your current admin user keys from ~/.aws/credentials in a safe location

# Set credentials for the new user in AWS CLI from key.txt file
$ aws configure
AWS Access Key ID [****]: new-access-key-id
AWS Secret Access Key [****]: new-secret-access-key
Default region name [us-west-1]: new-region
Default output format [None]:

# You can try to validate that ~/.aws/credentials contains the keys of the new user 

```

</details>

******

<details>
<summary>Exercise 3: Create VPC </summary>
 <br />

**steps:**
```sh
# Create VPC with subnet


# Create security group in the VPC to allow access on port 22
apt install -y nodejs npm

```

</details>

******

<details>
<summary>Exercise 4: Copy App and package.json </summary>
 <br />

**steps:**
```sh
# secure copy files from local machine to server. Execute from project's root folder.
scp -i ~/id_rsa app/bootcamp-node-project-1.0.0.tgz root@{server-ip-address}:/root
scp -i ~/id_rsa app/package.json root@{server-ip-address}:/root

```

</details>

******

<details>
<summary>Exercise 5: Run Node App </summary>
 <br />

**steps:**
```sh
# ssh into droplet
ssh -i ~/id_rsa root@{server-ip-address}

# unpack the node-project file
tar zxvf bootcamp-node-project-1.0.0.tgz

# change into unpacked directory called "package"
cd package

# install dependencies
npm install

# run the application
node server.js

```

</details>

******
