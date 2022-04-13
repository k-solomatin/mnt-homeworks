kibana-role
=========

Requirements
------------

for debian and EL omly.

Role Variables
--------------


| Variable name | Default | Description |
|-----------------------|----------|-------------------------|
| kibana_version | "8.0.0" |

Example Playbook
----------------

    - hosts: all
      roles:
         - { role: kibana-role }
