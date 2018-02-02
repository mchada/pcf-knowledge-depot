# Notes on PCF Availability Zones resources utilization

**Work in progress**

Recently, a Pivotal Cloud Foundry (PCF) customer had questions about why resources consumption for one of their PCF availability zones (AZ) cluster was substantially higher (~30%) than their other two allocated AZs.

That made me realize that there seems to be a misunderstanding about AZs resources allocation in PCF and which requires clarification in order to set the right expectations among customers. The goal of this article is to clarify that topic.

## PCF Reference Architecture and High Availability features

The [PCF Reference Architecture](https://docs.pivotal.io/pivotalcf/2-0/refarch/) recommends 3 availability zones to be setup for a PCF deployment. The reason is to enable [High Availability](https://docs.pivotal.io/pivotalcf/2-0/concepts/high-availability.html) of the platform by creating jobs/VMs across the resources allocated for those 3 AZs at the IaaS level.

High availability of the PCF jobs and running applications is accomplished by the redundancy of the platform VMs deployed across the multiple AZs. For example, if the entirety of AZ2 resources go down at the IaaS level, the other PCF jobs and running application instances in AZs 1 and 3 would still be functional, avoiding applications or services downtime.

However, that does not mean that IaaS resources will be equally consumed from the three AZs at the same quantity. If you check the corresponding IaaS resources consumption, you will probably notice that one AZ might have a higher percentage of usage than the other two AZs. The reason: **PCF singleton jobs and the BOSH CPI's job allocation algorithms**.


### Singleton Jobs

For PCF jobs deployed as singletons (i.e. only one instance), PCF Ops Manager requires that you select, on a per tile basis, the assigned AZ to host them. The result of that is, if your tile deployment contains several singleton jobs, all of the corresponding VMs will be instantiated on the selected AZ only.

*TBD: add list of PCF jobs and if they are singletons, e.g. File Storage then it will consume a lot of disk*

### BOSH Jobs allocation algorithm

For non-singleton PCF jobs, the corresponding VMs will be instantiated across all AZs by BOSH, typically in a round-robin fashion controlled by the order of AZs defined in Ops Manager.

For example, if the three AZs were defined in Ops Manager in order `AZ1, AZ2 and AZ3`, then for a job that was set to have 3 instances, e.g. Diego Brain, BOSH will allocated those VMs one in each AZ starting with `AZ1`.

Having that said, you probably already realized that for a job that was set to have `2` instances, in that order or AZs, BOSH will allocated VMs in `AZ1` and `AZ2`.

 Therefore, if `AZ1` is selected as the singletons AZ for most of the PCF tiles and if the order of AZs in Ops Mgr is set as `AZ1`, `AZ2` and `AZ3`, then it is very likely that `AZ1` will get a higher number of VMs assigned and consequently will have more of its resources consumed compared to the other two AZs.


## Alternatives to better balance PCF jobs allocation across AZs

If you want to stick with the configuration of `AZ1` for singletons and with the list of AZs being `AZ1, AZ2,AZ3` in Ops Manager, then make sure that `AZ1` has enough resources to accommodate all VMs.

However, if you want to spread the VMs allocation more evenly across all AZs, then here are a couple of alternatives:

- For each tile configuration in PCF Ops Manager, diversify the selected singletons AZ instead of selecting the same singletons AZ for all tiles.

- Change that order on how AZs are defined in OpsMg by having the chosen singleton AZ in the last position of that list. e.g. `AZ2, AZ3, AZ1` for when `AZ1` is the selected singleton AZ.
