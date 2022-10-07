# cvop2cvaas

Migrate all devices, containers and configlets from CV on-prem to CVaaS. 
The `cvaas_migration.yaml` playbook does the following:
- collects cvp facts (devices, containers and configlets) from the on-prem CVP system and stores it in memory
- fetches the devices in the inventory and builds a new inventory file to be used later
- generates TerminAttr onboarding token from CVaaS
- uploads the generated token to all devices using scp
- configures the devices with the TerminAttr config pointing to CVaaS
- pushes all the configlets from on-prem to CVaaS
- creates the container hierarchy (learned from the on-prem facts)
- moves the devices to their target container and applies the configlets
- creates a new TerminAttr configlet pointing to CVaaS and appends it to the list of configlets for each device

## Prerequisites

- python3 3.8+
- [ansible-cvp](https://cvp.avd.sh)
  - `$ ansible-galaxy collection install arista.cvp`
- [Minimum cvprac 1.0.7, recommended 1.2.0+](https://github.com/aristanetworks/cvprac/): `pip install cvprac>=1.2.0`
- scp (`pip install scp` or `pip3 install scp`) <-- Check with `pip --version` if it points to py2 or py3
- see more package and version requirements at [cvp.avd.sh](https://cvp.avd.sh/en/stable/docs/installation/requirements/)
- devices should run TerminAttr 1.15.3 (CVaaS requirement)
- devices should run EOS 4.23+ for non-prod clusters and 4.22+ for prod clusters (CVaaS requirement)
- all devices must have internet connectivity and be able to reach CVaaS, a quick ping test should be enough like below:

```shell
ping vrf MGMT apiserver.cv-staging.corp.arista.io
ping vrf MGMT www.cv-staging.corp.arista.io
```

## Steps

### Option 1 - Generate TerminAttr Config on the fly

This example is more of a faster approach for lab environments

1. Generate service account token on CVaaS ([steps](#how-to-generate-service-accounts))

> NOTE The token should be copied and saved to a file that can later be referred to, in this example it's in `/tokens/cvaas.tok`.

2. Generate service account token on CV on-prem ([steps](#how-to-generate-service-accounts)), save it to a file (e.g.: `/tokens/go178.tok`)

3. Export the tokens as env vars, e.g.:

```
export CVAAS_TOKEN=`cat /tokens/cvaas.tok`
export ON_PREM_TOKEN=`cat /tokens/go178.tok`
```

> Tip: Add them to your `.bashrc` or `.zshrc` to make them persistent and source them on the current terminal (`source ~/.zshrc`) or start a new session.

4. Go to CVaaS UI and generate the TerminAttr config and update the playbook under the `"Configuring TerminAttr on {{ inventory_hostname }}"` task
and inside the [terminattr.cfg](./terminattr.cfg) file

![terminattr_config_cvaas](./media/cvaas_ta_onboarding_config.png)

5. Update the `./inventory/inventory.yaml` file with the right credentials and IPs/FQDNs

6. Update `ansible.cfg` to point to the right folders for your `collections_paths` or just install ansible-cvp using ansible-galaxy and use that instead.

7. Generate the device inventory with `ansible-playbook cvaas_migrationV2.yaml -i inventory --tags devinv`
8. Finally run it as `ansible-playbook cvaas_migrationV2.yaml -i inventory --skip-tags=debug,devinv`


### Option 2 - Stream to both on-prem and CloudVision-As-a-Service

This example is recommended for production.

1. Update the `./inventory/inventory.yaml` file with the right credentials and IPs/FQDNs

2. Update `ansible.cfg` to point to the right folders for your `collections_paths` or just install ansible-cvp using ansible-galaxy and use that instead.

3. Generate service account token on CVaaS ([steps](#how-to-generate-service-accounts))

> NOTE The token should be copied and saved to a file that can later be referred to, in this example it's in `/tokens/cvaas.tok`.

4. Generate service account token on CV on-prem ([steps](#how-to-generate-service-accounts)), save it to a file (e.g.: `/tokens/go178.tok`)

5. Export the tokens as env vars, e.g.:

```
export CVAAS_TOKEN=`cat /tokens/cvaas.tok`
export ON_PREM_TOKEN=`cat /tokens/go178.tok`
```

> Tip: Add them to your `.bashrc` or `.zshrc` to make them persistent and source them on the current terminal (`source ~/.zshrc`) or start a new session.

6. Go to CVaaS UI and generate the TerminAttr config and build the new TerminAttr configuration to stream to both the existing on-prem cluster and CVaaS. An example can be found in the [TerminAttr most commonly used flags documentation](https://aristanetworks.force.com/AristaCommunity/s/article/terminattr-most-commonly-used-flags-and-sample-configurations) or in the [terminattr_multi_cluster.cfg](./terminattr_multi_cluster.cfg) file

![terminattr_config_cvaas](./media/cvaas_ta_onboarding_config.png)

7. Run `ansible-playbook option2_terminattr_multi_cluster.yaml --tags build` to generate the TerminAttr onboarding tokens for both on-prem and CVaaS and to generate the `onprem_devices.yaml` inventory file

8. Run `ansible-playbook option2_terminattr_multi_cluster.yaml --tags deploy -i inventory` to upload the tokens and reconfigure TerminAttr to stream to both clusters and wait for the devices to show up in the Undefined container on CVaaS UI.

>NOTE At this stage you should see the devices in the Undefined Container on your CVaaS tenant and on the on-prem cluster in their target container and out of configuration compliance.

9. Make sure the `terminattr_multi_cluster.cfg` file is updated as well and contains the multi-cluster streaming configuration.

10. Finally, run the playbook to migrate the containers, configlets, move the devices to their intended containers and assign the configlets to the containers and devices:
 `ansible-playbook option2_cvaas_migration.yaml -i inventory`

> NOTE At this change you should have the configlets uploaded, the containers created and `Add Device` tasks created for all the devices to move to their target containers and configlets to be attached.

11. Create a Change control from all the tasks, review and approve (there should be no config change).

1.  Later when the on-prem servers will be decommissioned the TerminAttr configuration can be changed to stream only to CVaaS.

## Disclaimer

- This is a proof-of-concept demo, highly recommended to take a backup before running the playbook.
- Tested with ansible-cvp `v3.3.1` and devel branch

## Caveats

- Due to [issue#508](https://github.com/aristanetworks/ansible-cvp/issues/508) in ansible-cvp `v3.4.0` due to schema differences between `cv_facts_v3` output  and `cv_container_v3` input the container topology creation will fail. Will be fixed in `v3.5.0`.

## TO-DO

- export Studio templates
- export Dashboards
- export tags
- figure out why `search_key: serialNumber` doesn't work

End to end example can be watched at [youtube](https://www.youtube.com/watch?v=rN6meAtXqss)

## Appendix

### How to generate service accounts

![serviceaccount1](./media/serviceaccount1.png)
![serviceaccount2](./media/serviceaccount2.png)
![serviceaccount3](./media/serviceaccount3.png)