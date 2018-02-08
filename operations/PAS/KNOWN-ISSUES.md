# Pivotal Application Service - Undocumented Known ISSUES

Issues found with PCF PAS not yet documented.
Once an issue is published to product documentation or to Knowledge Base articles, it should be removed from the list below.

---

### PAS 2.0

- **PAS Fresh Install - CredHub post-start scripts fails**

    if your PAS 2.x deployment fails while doing a fresh install on

    `updating credhub.... result: 1 of 2 post-start scripts failed. Failed Jobs: credhub. `

    then run the following commands.

    ```bosh -e alias -d cf-guid ssh mysql
    sudo su
    mysql -u root -p
    Password: https://OPS-MANAGER_URL/api/v0/deployed/products/cf-guid/variables?name=mysql-admin-credentials - <copy the password>
    > show databases;
    > drop database credhub;
    > create database credhub;
    > exit

    bosh -e alias -d cf-guid ssh credhub
    sudo su
    > monit stop credhub
    > monit start credhub
    ```
    Safe to drop the database as this is a fresh install and nothing has been stored into credhub yet.

---

[Back to PAS Knowledge depot page](.)
