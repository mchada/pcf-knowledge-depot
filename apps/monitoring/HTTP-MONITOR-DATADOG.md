<img src="https://docs.pivotal.io/images/cloud_rings.png" alt="PCF" height="60"/>&nbsp;
<img src="https://pbs.twimg.com/profile_images/654405223855824896/5_NBf5xl.png" alt="DataDog" height="70"/>


## Monitoring applications status with a DataDog HTTP monitor

This article provides a sample of an application monitoring mechanism that uses
a DataDog HTTP Monitor agent to check if a specific application URL endpoint is up or if the returned
content indicates that a service may be down.

Even though this monitoring technique is not specific to applications running
on PCF, the fact that the DataDog agent can be
deployed and configured with BOSH relates it to the platform.

### Sample of deployment architecture

For DataDog to monitor an application's state, it requires an [agent](https://docs.datadoghq.com/agent/) to be configured to monitor the application.
More specifically for this example, we will have to configure such agent with [HTTP Check](https://docs.datadoghq.com/integrations/http_check).

Let's assume that the application URL is not accessible from outside of your
firewall. In that case, the agent VM would have to be deployed in a subnet that
is able to access the application HTTP endpoint and also reach out to DataDog's
server on the internet.

```
                                              │  
┌────────────────────────┐    ┌────────┐             ┌─────────────────┐
│               ┌──────┐ │    │        │      │      │                 │
│               │ App1 │◀┼───▶│        │ ◀─────────▶ │                 │
│               └──────┘ │    │DataDog │      │      │                 │
│                        │    │ Agent  │             │ DataDog Server  │
│               ┌──────┐ │    │   VM   │      │      │                 │
│               │ AppN │ │    │        │             │                 │
│               └──────┘ │    │        │      │      │                 │
│                        │    └────────┘             └─────────────────┘
│          PCF           │    ┌────────┐      │                         
│                        │    │  Bosh  │                                
│                        │    │Director│      │                         
└────────────────────────┘    └────────┘                                
                                              │   

```



### Deploy the DataDog Agent with BOSH

DataDog maintains a [Bosh release of its agent](https://github.com/z4ce/datadog-agent-boshrelease), which can be used to easily deploy an agent VM in your infrastructure.

These instructions assume that you already have (1) a [Bosh Director](https://bosh.io/docs/init.html) available, (2) the [Bosh CLI](https://bosh.io/docs/cli-v2.html) installed on a machine that can reach that Bosh Director endpoint, and (3) a [DataDog account](https://app.datadoghq.com/signup).

**Note:** *even though you can use PCF's Bosh Director to deploy the agent VM, that is not recommend for production environments. Use a specific Bosh Director for tools or non-PCF related deployments in those environments.*

#### Prepare the DataDog agent deployment manifest

File [agent-with-http-check.yml](./samples/agent-with-http-check.yml) provides an example of a deployment manifest for a DataDog agent using the bosh release.

The [dd-agent job's documentation](https://bosh.io/jobs/dd-agent?source=github.com/DataDog/datadog-agent-boshrelease) provides information about all of its available configuration properties.

One section to highlight in that sample YML file is `integrations`, which contains an entry that enables an `HTTP Check` monitor in the agent:

```
  ...
  integrations:
    http_check:
      init_config:
      instances:
      - name: fortune-teller
        url: http://fortunes-ui.apps.company/random
        timeout: 5
        content_match: "Your future is bright!"
        reverse_content_match: true
        skip_event: true
  ...      
```

The sample [HTTP Check configuration](https://github.com/DataDog/integrations-core/blob/master/http_check/conf.yaml.example) instructs the agent to monitor both (1) if the application [fortune-teller application](https://github.com/spring-cloud-services-samples/fortune-teller) is down after a timeout of 5 seconds and (2) if the content returned matches a default message for when the application service is down.

DataDog documentation provides a list of all of the [available integrations](https://docs.datadoghq.com/integrations/) supported by its agent.


#### Deploy the agent VM

Three variables must be provided to the BOSH deploy command:
- `network_name` and `vm_type`, which can be obtained from the `cloud-config` of the Bosh Director instance.
- `api_key`, which is your [DataDog's account API key](https://app.datadoghq.com/account/settings#api).

BOSH deploy command:

`bosh -e <target> -d dd-agent deploy agent-with-http_check.yml -v network_name=<network-name> -v azs=<azs-list> -v vm_type=<vm-type-name> -v api_key=<datadog-api-key>`

Example:
`bosh -e toolsbosh -d dd-agent deploy agent-with-http_check.yml -v network_name=default -v azs=az1 -v vm_type=large -v api_key=<my-datadog-api-key-goes-here>`



### Configure your DataDog monitors

Once the DataDog agent VM is deployed and is successfully running, then login to your DataDog account and check your [list of monitors](https://app.datadoghq.com/monitors/manage). You should see the new HTTP Check monitor listed there.

Edit the monitor information and update the appropriate fields for the monitor to pickup the HTTP check results from the application's endpoint and send the appropriate notifications to the interested parties.

You could quickly test the new DataDog monitor instance by temporarily stopping the monitored application and then checking the reported monitor status and the `application down` notification functionality.

---

**Credits:** [@z4ce](https://github.com/z4ce) and [@lsilvapvt](https://github.com/lsilvapvt)

**Note:** [Pull request submitted to DataDog bosh release repository](https://github.com/DataDog/datadog-agent-boshrelease/pull/16) for the inclusion of the deployment manifest sample above as part of their `templates` folder.
