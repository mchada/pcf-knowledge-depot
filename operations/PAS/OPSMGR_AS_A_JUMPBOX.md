<img src="https://docs.pivotal.io/images/cloud_rings.png" alt="PCF" height="60"/>&nbsp;<img src="http://icons.iconarchive.com/icons/artua/mac/256/Terminal-icon.png" alt="SSH " height="70"/>

# Using Ops Manager VM as a jumpbox to a PCF deployment

The PCF Ops Manager VM can be used as a jumpbox to access and inspect a PCF deployment infrastructure, given that it is usually deployed inside of the PCF *infrastructure network* (see [PCF Reference Architecture](https://docs.pivotal.io/pivotalcf/refarch/index.html)) - which gives it access to the BOSH Director VM -, and that it contains key PCF and BOSH management tools pre-installed (e.g. `uaac` and `bosh 2.0` CLIs).

---
## Establish an ssh session with the Ops Manager VM

During its deployment, the Ops Manager VM is given an domain name/ip address and an `ssh` password for the default `ubuntu` user id.

Using such information, establish an `ssh` session with the Ops Manager VM:
```
   ssh ubuntu@<ops-mgr-domain-or-ip-address>
```
---
#### Connect and login to the BOSH Director

Once connected to the Ops Manager VM, you can use the BOSH 2 CLI to inspect the Bosh Director configuration.

1) Find the BOSH Director VM IP address  
   From the Ops Manager web interface, click on the Director tile > Status tab  and then copy the `IP address` of the `Ops Manager Director` row.  

2) Collect the Director credentials  
   From the Ops Manager web interface, click on the Director tile > Credentials tab > Director Credentials (url `https://<ops-mgr-domain>/api/v0/deployed/director/credentials/director_credentials`) and then copy the value of the `password` field.  

3) Set an environment alias for the director  
   `bosh alias-env director -e <ip_address_from_step1> --ca-cert=/var/tempest/workspaces/default/root_ca_certificate`  

4) Login to the BOSH Director  
   `bosh -e director login`  
   Enter `director` for `Email()`.
   Then, enter the password collected for step 2 above when prompted.  

Once logged in, you can issue BOSH CLI commands targeting the `director` alias/environment.  
Examples:  
`bosh -e diretor vms       # list all VMs of all deployments`  
`bosh -e director tasks    # list all running tasks for the BOSH Director`  
`bosh -e director -d <deployment_name> ssh    # ssh into one of the VMs of a deployment`  

---
#### Connect and login to the Ops Manager UAA server

Ops Manager runs its own UAA server. In certain cases, it is necessary to create client IDs or inspect the configuration of that server. Here is how you would connect to it using `uaac` CLI from the Ops Manager VM.

`uaac target https://127.0.0.1/uaa --skip-ssl-validation`

`uaac token owner get opsman <ops_manager_admin_userid> -s "" -p <ops_manager_admin_userid>`

Once authenticated, you can then issue any `uaac` command targetting that Ops Manager UAA.
Example:
`uaac users     # list all users`

---
#### Connect and login to the BOSH Director UAA server

The BOSH Director VM runs its own UAA server. In certain cases, it is necessary to [create client IDs](https://docs.pivotal.io/pivotalcf/customizing/opsmanager-create-bosh-client.html) or inspect the configuration of that server. Here is how you would connect to it using `uaac` CLI from the Ops Manager VM.

1) Find the BOSH Director VM IP address  
   From the Ops Manager web interface, click on the Director tile > Status tab  and then copy the `IP address` of the `Ops Manager Director` row.

2) Target the Bosh Director UAA server  
   `uaac target https://<director_ip_address_from_step1>:8443 --ca-cert /var/tempest/workspaces/default/root_ca_certificate`

3) Find the "Uaa Login Client Credentials" password for the Director UAA  
   From the Ops Manager web interface, click on the Director tile > Credentials tab > "Uaa Login Client Credentials", click on `Link to Credential` and then copy the value of the `password` field.

4) Find the "Uaa Admin User Credentials" password for the Director UAA  
   From the Ops Manager web interface, click on the Director tile > Credentials tab > "Uaa Admin User Credentials", click on `Link to Credential` and then copy the value of the `password` field.

5) Login and get a client token

   `uaac token owner get login -s <uaa_login_password_from_step3>`
   Once prompted for `User name`, enter `admin`
   Then, once prompted for `Password`, enter the password retrieved in step 4.

Once authenticated, then you can issue any `uaac` command targeting the BOSH Director UAA server.

---
#### Connect and login to the PAS UAA server

`uaac target uaa.<PAS-system-domain> --skip-ssl-validation`

`uaac token client get admin  # get secret from Ops Mngr>PAS tile>Credentials> UAA - Admin Client Credentials`

Once authenticated, you can then issue any `uaac` command targeting the PAS UAA server.

---
#### Connect and login to the PKS UAA server


`uaac target https://<PKS-uaa-url>:8443 --skip-ssl-validation`
Get the PKS UAA/API url from Ops Mgr web interface > PKS tile > Settings > UAA > UAA URL field.

`uaac token client get admin`
Once prompted, get the secret from Ops Mgr web interface > PKS tile > Credentials > Uaa Admin Secret (url `https://<ops-mgr-domain>/api/v0/deployed/products/<pks-deployment-id>/credentials/.properties.uaa_admin_secret`)

Once authenticated, you can then issue any `uaac` command targeting the PKS UAA server.

---

[Back to PAS Knowledge depot page](.)
