# Custom README

## Before you start
  - make sure lines 9 and 11 have the proper targets
  - update lines 10 and 12 with proper service account tokens
  - verify credetials at lines 126, 127, 154 and 155
  - update lines 147 and 175 with correct terminattr config
  - update terminattr_default_vrf.cfg and terminattr_mgmt_vrf.cfg files

## step1
run playbook with 'facts' tag

## step2
update terminattr_default_vrf.yml and terminattr_mgmt_vrf.yml files
review cv_facts/cvp_containers.yaml
review cv_facts/cvp_devices.yml (might have to clean up extra configlets)

## step3
split the inventory manually
  - split with 2 headers [mgmt_vrf] and [default_vrf]
  - delete [onprem_devices]

## step4
run playbook with 'deploy' tag 
notes: 
  - make sure you point to the inventory folder, not idividual inventory file

## step5
validate the generated tasks on CVaaS
  - ADD terminattr configlet to the correct containers ?!?