# Jenkins Project Security

1. There is an existing Jenkins job named **Packages**, there are also two existing Jenkins users named **sam** with password **sam@pass12345** and **rohan** with password **rohan@pass12345**

2. Grant permissions to these users to access **Packages** job as per details mentioned below:
  - a. Make sure to select **Inherit permissions from parent ACL** under inheritance strategy for granting permissions to these users.
  - b. Grant mentioned permissions to **sam** user : **build**, **configure** and **read**.
  - c. Grant mentioned permissions to **rohan** user : **build**, **cancel**, **configure**, **read**, **update** and **tag**

> Jenkins Version 2.492.1
---

### Install Matrix Authorization Strategy Plugin
  ![install plugin](./images/1.png)
  ![download progress & restart jenkins](./images/2.png)
  ![restarting](./images/3.png)
  ![verify installed plugin](./images/4.png)
---

### Enable Project-based Matrix Authorization
  ![choose project-based matrix authorization strategy](./images/5.png)
  ![click save](./images/6.png)
  ![make sure the admin user has administer permission](./images/7.png)
  ![add users and grant overall read permission](./images/8.png)
---

### Configure the Packages job
  * Check the box to Enable project-based security and select **Inherit permissions from parent ACL**
  ![check the box to Enable project-based security](./images/9.png)
  * Add user and grant the required permissions
  ![sam & rohan](./images/10.png)
---

### Test login as sam
  ![login as sam](./images/11.png)
  ![package job status & run build now](./images/12.png)
  ![console output](./images/13.png)
  ![job status](./images/14.png)
---
