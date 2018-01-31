## Monitoring application status with a DataDog HTTP monitor

This article provides a sample of a application monitoring mechanism that uses
a DataDog HTTP Monitor agent to check if a specific URL is up or returning
content that may indicate that a certain service is down.

Even though this monitoring technique is not specific to applications running
on PCF, what makes it related to the platform is that the DataDog agent can be
deployed and configured with BOSH.

### Deployment architecture

For DataDog to monitor an application's state, it requires an [agent](https://docs.datadoghq.com/agent/) to be configured to monitor the application.
More specifically for this example, we will have to configure such agent with [HTTP Check](https://docs.datadoghq.com/integrations/http_check).

Let's assume that the application URL is not accessible from outside of your
firewall. In that case, the agent VM would have to be deployed in a subnet that
is able to access both the application HTTP endpoint and reach out to DataDog's
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

### Deploying the DataDog Agent with BOSH

DataDog maintains a [Bosh release of its agent](https://github.com/z4ce/datadog-agent-boshrelease), which can be used to easily deploy an agent VM in your infrastructure.

These instructions assume that you already have (1) a [Bosh Director](https://bosh.io/docs/init.html) available, (2) the [Bosh CLI](https://bosh.io/docs/cli-v2.html) installed on a box that can reach that Bosh Director endpoint, and (3) a [DataDog account](https://app.datadoghq.com/signup).

(Note: even though you can use PCF's Bosh Director to deploy the agent VM, I would not recommend it for production environments. Use a specific Bosh Director for tools and non-PCF related deployments in those environments).

#### Prepare the DataDog agent deployment manifest

File [agent-with-http-check.yml](./samples/agent-with-http-check.yml) provides an example of a deployment manifest for a DataDog agent using its bosh release.

The [configuration document for the dd-agent job](https://bosh.io/jobs/dd-agent?source=github.com/DataDog/datadog-agent-boshrelease) provides information about all of its available properties.

The section to highlight in that sample YML is the one that enables an `HTTP Check` monitor in the agent is `integrations`.

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

The provided sample defines an [HTTP Check configuration](https://github.com/DataDog/integrations-core/blob/master/http_check/conf.yaml.example) to monitor the service endpoint of a [fortune-teller application](https://github.com/spring-cloud-services-samples/fortune-teller) deployed on PCF. That configuration instructs the agent to monitor both (1) if the application is down after a timeout of 5 seconds and (2) if the content returned matches a default message for when the application service is down.


#### Deploy command

Three variables must be provided to the BOSH deploy command:
- `network_name` and `vm_type` can be obtained from the `cloud-config` of the Bosh Director instance.
- `api_key` is you [DataDog's account API key](https://app.datadoghq.com/account/settings#api).

BOSH deploy command:

`bosh -e <target> -d dd-agent deploy agent-with-http_check.yml -v network_name=<network-name> -v azs=<azs-list> -v vm_type=<vm-type-name> -v api_key=<datadog-api-key>`

Example:
`bosh -e toolsbosh -d dd-agent deploy agent-with-http_check.yml -v network_name=default -v azs=az1 -v vm_type=large -v api_key=<my-datadog-api-key-goes-here>`

#### Configuring DataDog Dashboards
