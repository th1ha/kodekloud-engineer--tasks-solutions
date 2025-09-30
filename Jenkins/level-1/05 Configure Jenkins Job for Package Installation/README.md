# Configure Jenkins Job for Package Installation

1. Access the `Jenkins` UI by clicking on the Jenkins button in the top bar. Log in using the credentials: username `admin` and password `Adm!n321`
2. Create a new Jenkins job named `install-packages` and configure it with the following specifications:
  - Add a string parameter named `PACKAGE`
  - Configure the job to install a package specified in the `$PACKAGE` parameter on the `storage server` within the `Stratos Datacenter`

**`Jenkins Version 2.492.1`**
---

### Create a new jobs `install-packages`
  ![create new item](./images/1.png)
  ![created](./images/2.png)
---

### Add parameter `PACKAGE`
  ![enable](./images/3.png)
  ![string parameter](./images/4.png)
  ![PACKAGE](./images/5.png)

### Configure build step
  ![execute shell](./images/6.png)
  ```bash
  sshpass -p 'PASSWORD' ssh -o StrictHostKeyChecking=no natasha@ststor01 "echo 'PASSWORD' | sudo -S yum install -y $PACKAGE"
  ```
  ![script](./images/7.png)

### Test job
  ![vim package](./images/8.png)
  ![Result](./images/9.png)

