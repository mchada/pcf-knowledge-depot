# Pivotal Application Service - Undocumented Known ISSUES

Issues found with PCF PAS not yet documented.
Once an issue is published to product documentation or to Knowledge Base articles, it should be removed from the list below.


## PAS 2.0

---

### Isolation Segment fails on fresh install of PAS 2.0 with enabled route services

  [KB article published on 2/9/2018](https://discuss.pivotal.io/hc/en-us/articles/360000863274)

  While deploying *PAS Isolation Segment 2.0.3* tile on top of a *PAS 2.0.3* fresh deployment, the following error occurs for the isolation-segment deploy:

  ```
  Task 2002 | 16:57:12 | Preparing deployment: Preparing deployment (00:00:03)
  Task 2002 | 16:57:18 | Error: Unable to render instance groups for deployment. Errors are:
    - Unable to render jobs for instance group 'isolated_router'. Errors are:
      - Unable to render templates for job 'gorouter'. Errors are:
        - Failed to find variable '/p-bosh/p-isolation-segment-5ef3ccdfc6e1011f2868/router-route-services-secret' from config server: HTTP Code '404', Error: 'The request could not be completed because the credential does not exist or you do not have sufficient authorization.'
  ```      

  The PAS tile configuration had the default `Networking > Enable route services` option selected.
  Given the name of the missing variable in the error message, we suspected that configuration to be the cause of the problem.

  **Resolution**: indeed, after disabling route services in the PAS tile configuration (`Networking > Disable route services`), the Isolation Segment tile deployment finished successfully.

  This problem will be reported to support/releng for investigation.

  *Note: quick fact for root cause analysis, variable `router.route_services_secret` [got moved to CredHub in PAS 2.0](https://docs.pivotal.io/pivotalcf/2-0/pcf-release-notes/runtime-rn.html#bosh-credhub)*

---

### PAS 2.0 and above Fresh Install Fails while Updating Credhub Instance

  [KB article published on 2/8/2018](https://discuss.pivotal.io/hc/en-us/articles/360000825793)

---

[Back to PAS Knowledge depot page](.)
