# Configure Jenkins User Access

1. Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login with username `admin` and password `Adm!n321`
2. Create a jenkins user named `john` with the password `GyQkFRVNr3`. Their full name should match `John`
3. Utilize the `Project-based Matrix Authorization Strategy` to assign `overall read` permission to the `john` user
4. Remove all permissions for `Anonymous` users (if any) ensuring that the `admin` user retains overall `Administer` permissions
5. For the existing job, grant `john` user only `read` permissions, disregarding other permissions such as Agent, SCM etc


[Matrix Authorization Strategy](https://plugins.jenkins.io/matrix-auth/)

**`Jenkins Version 2.492.1`**
---

### Install the `Matrix Authorization Strategy` plugin
  ![install plugins](./images/1.png)  
---

### Restart Jenkins
  ![restart jenkins](./images/2.png)
  ![restarting](./images/3.png)
---

### Verify installed plugins
  ![verify](./images/4.png)
---

### Create a new user
  ![user list](./images/5.png)
  ![user info](./images/6.png)
  ![verify](./images/7.png)
  * login as the `john` user
  ![john](./images/8.png)
---

### Configure Authorization
  ![matrix](./images/9.png)
  ![view](./images/10.png)
---

### Configure read permission for `john`
  ![add user](./images/11.png)
  ![check the read permission](./images/12.png)
---

### Configure administer permission for `admin`
  ![add user](./images/13.png)
  ![check the administer permission](./images/14.png)
---

### Login as john user
  ![verify](./images/15.png)
---