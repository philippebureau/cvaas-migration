# Custom README

## Before you start
  - make sure lines 9 and 11 have the proper targets
  - update lines 10 and 12 with proper service account tokens
  - verify credetials at lines 126, 127, 154 and 155
  - update lines 147 and 175 with correct terminattr config
  - update terminattr_default_vrf.cfg and terminattr_mgmt_vrf.cfg files (depends on optional step bellow)

## optional
  - You can change the TerminAttr config in the existing onprem CVP without executing the tasks so the migrated configuration will already have the proper TerminAttr configuration.  
  - if this method is used, you do not need to execute step5

## step1
run playbook with 'facts' tag

## step2
review cv_facts/cvp_containers.yaml
  - if you have images assigned to containers, make sure CVaaS has an existing bundle with the same name
review cv_facts/cvp_devices.yml (might have to clean up extra configlets)
  - if you have images assigned to containers, make sure CVaaS has an existing bundle with the same name

## step3
split the inventory manually
  - split with 2 headers [mgmt_vrf] and [default_vrf]
  - delete [onprem_devices]

## step4
run playbook with 'deploy' tag 
notes: 
  - make sure you point to the inventory folder, not idividual inventory file

## step5 (see optional step above)
Add The correct TerminAttr configlets to CVaaS appropriate containers

## step6
validate the generated tasks on CVaaS and execute.
