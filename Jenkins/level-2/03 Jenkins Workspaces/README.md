# Jenkins Workspaces

1. There is a Git repository named **web_app** on Gitea where developers are pushing their changes. It has three branches **version1**, **version2** and **version3** (excluding the master branch). You need not to make any changes in the repository.

2. Create a Jenkins job named **app-job**.

3. Configure this job to have a choice parameter named **Branch** with choices as given below:
  - **version1**
  - **version2**
  - **version3**

4. Configure the job to fetch changes from above mentioned Git repository and make sure it should fetches the changes from the respective branch which you are passing as a choice in the choice parameter while building the job. For example if you choose **version1** then it must fetch and deploy the changes from branch **version1**.

5. Configure this job to use custom workspace rather than a default workspace and custom workspace directory should be created under **/var/lib/jenkins** (for example **/var/lib/jenkins/version1**) location rather than under any sub-directory etc. The job should use a workspace as per the value you will pass for Branch parameter while building the job. For example if you choose **version1** while building the job then it should create a workspace directory called **version1** and should fetch Git repository etc within that directory only.

6. Configure the job to deploy code (fetched from Git repository) on storage server (in Stratos DC) under **/var/www/html** directory. Since its a shared volume.

7. You can access the website by clicking on **App** button.

> Jenkins Version 2.492.1
---

### Create a new jobs **app-job**
  ![create new item](./images/1.png)
---

### Add parameters
  ![check the box for this project is parameterized](./images/2.png)
  ![click add parameter and select choice parameter](./images/3.png)
  ![add choice parameter](./images/4.png)
---

### Install Git plugin
  ![no git](./images/5.png)
  ![install git plugin](./images/6.png)
  ![restart jenkins](./images/7.png)
  ![restarting](./images/8.png)
  ![verify git plugin](./images/9.png)
  ![check git plugin](./images/10.png)
---

### Create Gitea credentials
  ![add credentials](./images/11.png)
  ![add username, password and ID](./images/12.png)
  ![verify user](./images/13.png)
---

### Configure Source Code Management (Gitea)
  ![add repo cred and branch ](./images/14.png)
---

### Configure Dynamic Custom Workspace
  ![check the box for use custom workspace](./images/15.png)
  ```bash
  /var/lib/jenkins/${Branch}
  ```
  ![paste the commands](./images/16.png)
---

### Configure Build Step
  ![click add build step and select execute shell](./images/17.png)
  ```bash
  #!/bin/bash
  set -e

  echo "Deploying branch ${Branch} from workspace ${WORKSPACE}"

  REMOTE_USER="natasha"
  REMOTE_HOST="ststor01"
  REMOTE_PASS="password"
  REMOTE="${REMOTE_USER}@${REMOTE_HOST}"


  sshpass -p "${REMOTE_PASS}" ssh -o StrictHostKeyChecking=no "${REMOTE}" \
    "echo '${REMOTE_PASS}' | sudo -S rm -rf /var/www/html/*"

  sshpass -p "${REMOTE_PASS}" scp -o StrictHostKeyChecking=no -r ${WORKSPACE}/* "${REMOTE}:/var/www/html/"
  ```
  ![past the script](./images/18.png)
---

### Test the Job
  ![click build with parameters and select version2](./images/19.png)
  ![console output](./images/20.png)
  ![click on app button](./images/21.png)