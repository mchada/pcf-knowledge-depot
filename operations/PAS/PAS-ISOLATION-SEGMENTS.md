<img src="https://docs.pivotal.io/images/cloud_rings.png" alt="PCF Knowledge Depot" height="60"/>&nbsp;<img src="http://docs.pivotal.io/images/icon-isolation_segment@2x.png" alt="PAS " height="70"/>

# Creating and testing PAS Isolation Segments

Complementary notes on the creation and testing of PAS Isolation Segments

---

### Post Isolation Segment tile install tasks

After [installing](https://docs.pivotal.io/pivotalcf/customizing/installing-pcf-is.html) the Isolation Segment tile:

- If GoRouters were created for the Isolation Segment, make sure that your DNS and Load Balancers handle correctly the corresponding domain assigned to the Isolation Segment.

- [Create an isolation segment](https://docs.pivotal.io/pivotalcf/adminguide/isolation-segments.html) instance with the same name entered in the tile configuration.

- [Enable](https://docs.pivotal.io/pivotalcf/2-0/adminguide/isolation-segments.html#relationships) the isolation segment for the desired Orgs/spaces and make it the default one if applicable

- If a specific domain will be assigned to apps running in the isolation segment, [create the corresponding domain instance](https://docs.pivotal.io/pivotalcf/2-0/devguide/deploy-apps/routes-domains.html#private-domains) for the corresponding orgs.


### Testing Application deployment and execution for Isolation Segments

- CF PUSH the [Spring Music app](https://github.com/cloudfoundry-samples/spring-music) to the Org/Space assigned to the Isolation segment

- If you deployed multiple Diego Cells for the Isolation Segment, then [scale](https://docs.pivotal.io/pivotalcf/2-0/devguide/deploy-apps/cf-scale.html#horizontal) the Spring Music app instances to the same number of cells available (1 instance per cell)

- Access the `env` endpoint URL of the deployed app to inspect its running information (e.g. `http://<spring-music-app-url>/env`)   
  Look for the content of variable `CF_INSTANCE_ADDR`. It should contain the IP address of a Diego Cell belonging to the Isolation Segment.  
  Refresh the page and you should see the content of that variable change to the IP address of another Diego Cell (only if there is more than one Diego Cell in the Isolation Segment and more than one Spring Music app instance deployed). Example:

  ```
  {
  "PATH": "/usr/local/bin:/usr/bin:/bin",
  ...
  "INSTANCE_INDEX": "0",
  ...
  "CF_INSTANCE_GUID": "0629a545-1c81-4782-690c-9dcb",
  ...
  "CF_INSTANCE_IP": "192.168.32.11",
  ...
  "CF_INSTANCE_ADDR": "192.168.32.11:61000",
  ...
  }
  ```

---

[Back to PAS Knowledge depot page](.)
