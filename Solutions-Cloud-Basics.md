</details>

******

<details>
<summary>Exercise 0: Clone Git Repository </summary>
 <br />

**steps:**

```sh
# clone repository & change into project dir
git clone git@gitlab.com:devops-bootcamp3/node-project.git
cd node-project

# remove remote repo reference and create your own local repository
rm -rf .git
git init 
git add .
git commit -m "initial commit"

# create git repository on Gitlab and push your newly created local repository to it
git remote add origin git@gitlab.com:{gitlab-user}/{gitlab-repo}.git
git push -u origin master

```

</details>

******

<details>
<summary>Exercise 1: Package NodeJS App </summary>
 <br />

**steps**

```sh
npm pack

```

</details>

******

<details>
<summary>Exercise 2, 3: Create and prepare server on Digital Ocean </summary>
 <br />

**steps:**
```sh
# ssh into your newly created server
ssh -i ~/id_rsa root@{server-ip-address}

# install nodejs and npm
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
