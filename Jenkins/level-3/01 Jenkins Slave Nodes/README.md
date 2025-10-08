# Jenkins Slave Nodes

1. Add all app servers as SSH build agent/slave nodes in Jenkins. Slave node name for **app server 1**, **app server 2** and **app server 3** must be **App_server_1**, **App_server_2**, **App_server_3** respectively.

2. Add labels as below:
  - **App_server_1 : stapp01**
  - **App_server_2 : stapp02**
  - **App_server_3 : stapp03**

3. Remote root directory for **App_server_1** must be **/home/tony/jenkins**, for **App_server_2** must be **/home/steve/jenkins** and for **App_server_3** must be **/home/banner/jenkins**.

4. Make sure slave nodes are online and working properly.

> Jenkins Version 2.492.1
---

### Install SSH Build Agents Plugin
  ![ssh agent](./images/1.png)
  ![download progress](./images/2.png)
  ![verify](./images/3.png)
---

### Add Agents
  ![click new node](./images/4.png)
  ![App_server_1](./images/5.png)
  ![select launch agents via ssh](./images/6.png)
  ![create new credentials](./images/7.png)
  ![username with password](./images/8.png)
  ![credentials creation failed](./images/9.png)
  ![global credentials page](./images/10.png)
  ![create credentials for tony](./images/11.png)
  ![create credentials for steve](./images/12.png)
  ![create credentials for banner](./images/13.png)
  ![credentials list](./images/14.png)
  ![select credentials, usage and Host Key verification strategy](./images/15.0.png)
  ![java not found](./images/15.1.png)
  ![install java on the remote agent node](./images/15.2.png)
  ![verify java](./images/15.3.png)
  ![re-launch agent](./images/15.4.png)
  ![node app1 log](./images/15.5.png)
  ![node app1 status](./images/15.6.png)
---

### Install & Verify Java on the Remote Agent Nodes
```bash
sshpass -p "steve" ssh -o StrictHostKeyChecking=no steve@stapp02 "echo 'steve' | sudo -S dnf install -y java-17-openjdk"
sshpass -p "steve" ssh -o StrictHostKeyChecking=no steve@stapp02 "rpm -qa | grep java"

sshpass -p "banner" ssh -o StrictHostKeyChecking=no banner@stapp03 "echo 'banner' | sudo -S dnf install -y java-17-openjdk"
sshpass -p "banner" ssh -o StrictHostKeyChecking=no banner@stapp03 "rpm -qa | grep java"

```
---

### Repeat for the other two nodes
  ![App_server_2](./images/16.png)
  ![App_server_2_config](./images/17.png)
  ![App_server_3](./images/18.png)
  ![App_server_2_config](./images/19.png)
---

### Verification
  ![nodes list](./images/20.png)