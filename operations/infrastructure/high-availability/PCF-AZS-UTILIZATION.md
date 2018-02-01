# Notes on PCF Availability Zones utilization

**Work in progress**

Recently, a PCF customer had questions about why resources consumption for one of their PCF availability zones (AZ) cluster was substantially higher (~30%) than their other two allocated AZs.

That made me realize that there seems to be a misunderstanding about AZs resources allocation in PCF and which requires clarification in order to set the right expectations among customers. The goal of this article is to clarify that topic.

## PCF Reference Architecture and High Availability features

The [PCF Reference Architecture](https://docs.pivotal.io/pivotalcf/2-0/refarch/) recommends 3 availability zones to be setup for a PCF deployment. The reason is to enable [High Availability](https://docs.pivotal.io/pivotalcf/2-0/concepts/high-availability.html) of the platform by creating jobs/VMs across the resources allocated for those 3 AZs at the IaaS level.

High availability of the PCF jobs and running applications is accomplished by the redundancy of the platform VMs deployed across the multiple AZs. For example, if the entirety of AZ2 resources go down at the IaaS level, the other PCF jobs and running application instances in AZs 1 and 3 would still be functional, not resulting in application or services downtime.

However, that does not mean that IaaS resources will be equally consumed from the three AZs at the same quantity. If you check the corresponding IaaS resources consumption, you will probably notice that one AZ might have a higher percentage of usage than the other two AZs. The reason: **PCF singleton jobs and the Bosh CPI's job allocation algorithm**.


## Singleton Jobs and Jobs allocation algorithm


*TBD: add list of PCF jobs and if they are singletons*
*All singletons will go to AZ1 if so selected.*
*Plus if they have File Storage then it will consume a lot of disk for both foundations in AZ1.*
*round robin allocation of VMs across all AZs*

## Alternatives to better balance PCF jobs allocation across AZs


*You can balance other tiles singleton to AZ2 or AZ3… or just make the first cluster higher resources
*The order on how you define the AZs in Ops Mgr: most of the time we define in the order of AZ1, AZ2, AZ3. When jobs have more than one instance (e.g. 2), apparently the CPI goes in a round-robin fashion from the top of that list, meaning, AZ1 and AZ2 would be used for 2 jobs allocation
Inverting that order of the AZs definition in OpsMgr, e.g. AZ3, AZ2, AZ1,   or   AZ2, AZ3, AZ1 ,   then the first two AZs in the list would be used for the allocation of a 2 instances job. ==> Validate this statement*

*Customer is observing this scenario in an environment (clusters) shared by two foundations, SANDBOX and NON-PROD, where both have AZs defined in the same way in OpsMgr and which probably makes the AZ unbalance even more noticeable
