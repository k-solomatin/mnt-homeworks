filebeat-role
=========


Requirements
------------

For debian and EL only.

Role Variables
--------------


| Variable name | Default | Description |
|-----------------------|----------|-------------------------|
| filebeat_version | "8.0.0" |

Example Playbook
----------------

    - hosts: all
      roles:
         - { role: filebeat-role }

License
-------

BSD
