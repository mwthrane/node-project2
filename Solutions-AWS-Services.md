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

You can refer to this official document for the latest implementation of the aws cli commands, in case something has changed:
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnets-commands-example.html

**Create VPC with 1 Subnet**
```sh

# Create VPC and return VPC id
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text

# Create subnet in the VPC
aws ec2 create-subnet --vpc-id vpc-id --cidr-block 10.0.1.0/24

```

**Make our subnet public by attaching it internet gateway**
```sh
# Create internet gateway & return the gateway id
aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text

# Attach internet gateway to our VPC
aws ec2 attach-internet-gateway --vpc-id vpc-id --internet-gateway-id igw-id

# Create a custom Route table for our VPC & return route table ID
aws ec2 create-route-table --vpc-id vpc-id --query RouteTable.RouteTableId --output text

# Create Route rule for handling all traffic between internet & VPC
aws ec2 create-route --route-table-id rtb-id --destination-cidr-block 0.0.0.0/0 --gateway-id igw-id

# Valide your custom route table has correct configuraton, 1 local and 1 interent gateway routes
aws ec2 describe-route-tables --route-table-id rtb-id

# Associate subnet with the route table to allow internet traffic in the subnet as well
aws ec2 associate-route-table  --subnet-id subnet-id --route-table-id rtb-id

```

**Create security group in the VPC to allow access on port 22**
```sh
# Create security group - this will print security group id as output
aws ec2 create-security-group --group-name SSHAccess --description "Security group for SSH access" --vpc-id vpc-id

# Add incoming access on port 22 from all sources to security group
aws ec2 authorize-security-group-ingress --group-id sg-id --protocol tcp --port 22 --cidr 0.0.0.0/0
# You can also specify your IP address CIDR block instead of 0.0.0.0/0 for more security

```
</details>

******

<details>
<summary>Exercise 4: Create EC2 instance </summary>
 <br />

**Create EC2 instance into our subnet**
```sh
# Create key pair, save it locally in pem file and set stricter permission on it for later use
aws ec2 create-key-pair --key-name WebServerKeyPair --query "KeyMaterial" --output text > WebServerKeyPair.pem
chmod 400 MyKeyPair.pem

# Create 1 EC2 instance with the above key, in our subnet and using security group we created
aws ec2 run-instances --image-id ami-a4827dc9 --count 1 --instance-type t2.micro --key-name WebServerKeyPair --security-group-ids sg-id --subnet-id subnet-id

# Validate that EC2 instance is in a running state, and get its public ip address to connect via ssh
aws ec2 describe-instances --instance-id i-0146854b7443af453 --query "Reservations[*].Instances[*].{State:State.Name,Address:PublicIpAddress}"

```

</details>

******

<details>
<summary>Exercise 5: SSH into the server and install Docker on it </summary>
 <br />

**steps:**
```sh
# ssh into EC2 instance using th epublic IP address we got earlier
# Note: EC2 must be in a running state & relative path to WebServerKeyPair.pem must be correct
ssh -i "WebServerKeyPair.pem" ec2-user@public-ip-address

# Install Docker, start docker service and allow ec2-user to run docker commands without sudo by adding it to docker group
sudo yum update -y && sudo yum install -y docker
sudo systemctl start docker 
sudo usermod -aG docker ec2-user

```

</details>

******

<details>
<summary>EXERCISE 6: Add docker-compose for deployment </summary>
 <br />

**docker-compose.yaml:**

```sh
version: '3.8'
services:
    nodejs-app:
      image: ${IMAGE}
      ports:
        - 3000:3000

```

</details>

******

<details>
<summary>EXERCISE 7: Add "deploy to EC2" step to your pipeline </summary>
 <br />


**Create ssh keys credentials in Jenkins**
- name the keys credentials: `ec2-server-key` and add the private ssh key from `WebServerKeyPair.pem` as its content. You will use this to ssh into the EC2 server from Jenkins 

**Create server-cmds.sh file**
```sh
export IMAGE=$1
docker-compose -f docker-compose.yaml up --detach
echo "success"
```

Complete the Jenkinsfile from the previous exercise: 8 - Build Automation & CI/CD with Jenkins

**Jenkinsfile:**
```sh
pipeline {
    agent any
      tools {
        nodejs "node"
      }
      stages {
        stage('increment version') {
          ...
        }
        stage('Run tests') {
          ...
        }
        stage('Build and Push docker image') {
          ...
        }
        stage('deploy to EC2') {
            steps {
                script {
                   def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                   def ec2Instance = "ec2-user@public-ip-address"

                   sshagent(['ec2-server-key']) {
                       sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                       sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                       sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                   }     
                }
            }
        }
        stage('commit version update') {
          ...
        }
    }     
}

```

</details>

******


<details>
<summary>EXERCISE 8: Configure access from browser (EC2 Security Group) </summary>
 <br />

**Open application's port 3000 in security group to make app accessible from browser**

```sh
aws ec2 authorize-security-group-ingress --group-id sg-id --protocol tcp --port 3000 --cidr 0.0.0.0/0

```

</details>

******

<details>
<summary>EXERCISE 9: Configure automatic triggering of pipeline </summary>
 <br />

**Add branch based logic to Jenkinsfile**

```sh
# when the currently building branch is master, execute all steps. If it's not master, execute only the "run tests" step
pipeline {
    agent any
      tools {
        nodejs "node"
      }
      stages {
        stage('increment version') {
          when {
            expression {
              return env.GIT_BRANCH == "master"
            }
          }
          steps {
            script {
                ...  
            }
          }
        }
        stage('Run tests') {
          steps {
            script {
                ...  
            }
          }
        }
        stage('Build and Push docker image') {
          when {
            expression {
              return env.GIT_BRANCH == "master"
            }
          }
          steps {
            script {
                ...  
            }
          }
        }
        stage('deploy to EC2') {
          when {
            expression {
              return env.GIT_BRANCH == "master"
            }
          }
          steps {
            script {
                ...  
            }
          }
        }
        stage('commit version update') {
          when {
            expression {
              return env.GIT_BRANCH == "master"
            }
          }
          steps {
            script {
                ...  
            }
          }  
        }
    }     
}

```

**Add webhook to trigger pipeline automatically**
Refer to the demo video for this

</details>

******





