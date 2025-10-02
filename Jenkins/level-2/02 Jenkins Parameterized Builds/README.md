# Jenkins Parameterized Builds

1. Create a **parameterized** job which should be named as **parameterized-job**

2. Add a **string** parameter named **Stage**; its default value should be **Build**.

3. Add a **choice** parameter named **env**; its choices should be **Development**, **Staging** and **Production**.

4. Configure job to execute a shell command, which should echo both parameter values (you are passing in the job).

5. Build the Jenkins job at least once with choice parameter value **Staging** to make sure it passes.

> Jenkins Version 2.492.1
---

### Create a new jobs **parameterized-job**
  ![create new item](./images/1.png)
---

### Add parameters
  ![check the box for this project is parameterized](./images/2.png)
  ![click add parameter and select string parameter](./images/3.png)
  ![add string parameter](./images/4.png)
  ![click add parameter and select choice parameter](./images/5.png)
  ![add env choice parameter](./images/6.png)
---

### Configure the Shell Command
  ![click add build step and select execute shell](./images/7.png)
  ```sh
  echo "Stage parameter: $Stage"
  echo "Environment parameter: $env"
  ```
  ![paste the commands](./images/8.png)

### Build the job
  ![click build with parameters](./images/9.png)

### Verify output
  ![build history](./images/10.png)
  ![console output](./images/11.png)


