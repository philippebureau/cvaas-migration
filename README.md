# Instructions

## Before you start
  - Update inventory/group_vars/all.yml variables with CVP and EOS variables for your infrastructure
  - update terminattr.cfg file with correct configuration if planned to be used (see terminattr tag description).  This file will be uploaded as a configlet on CVaaS and assigned to all devices

## step1
run playbook with 'facts' tag
```
ansible-playbook cvaas_migrationV2.yaml -i inventory --tags facts
```

## step2
review cv_facts/cvp_containers.yaml
  - if you have images assigned to containers, make sure CVaaS has an existing bundle with the same name
review cv_facts/cvp_devices.yml (might have to clean up extra configlets)
  - if you have images assigned to devices, make sure CVaaS has an existing bundle with the same name

## step3
split the inventory/onprem_devices.yaml inventory manually
  - split with headers [group1], [group2]...
  - delete [onprem_devices]
  example:
  ![](./media/inventory_split_example.png)

## step4
modify the inventory/group_vars/all.yml file deployement variables to point at the group you want (ex: group1) defined in step3
run playbook with 'deploy' tag or terminattr and deploy tags if you want to add a new terminattr configlet
notes: 
  - make sure you point to the inventory folder, not individual inventory file (-i inventory)

```
ansible-playbook cvaas_migrationV2.yaml -i inventory --tags deploy
```
or 

```
ansible-playbook cvaas_migrationV2.yaml -i inventory --tags deploy, terminattr
```

## step6
validate the generated tasks on CVaaS and execute.

## TAGS description
- **facts** -- will import data from on-prem CVP and save it to cv_facts folder 
- **deploy** -- will upload the configlets, configure the devices to stream to CVaaS, move devices to the proper containers and attach relevant configlets
- **terminattr** -- will create a new configlet on CVaaS for the new terminattr config and assign it to every node.  This is done in memory during the playbook and will not modify cv_facts/CVP_DEVICES.yaml file