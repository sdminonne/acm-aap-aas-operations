ansible-playbook playbooks/lncl-enable-odap-operator.yml -e"@default-for-backup.yml"

ansible-playbook playbooks/lncl-enable-odap-object-storage.yml -e"@default-for-backup.yml"


ansible-playbook playbooks/lncl-enable-hub-backup -e"@default-for-backup.yml"


----------------------------


ansible-playbook playbooks/lncl-enable-odap-operator.yml -e"@default-for-restore.yml"

ansible-playbook playbooks/lncl-enable-odap-object-storage.yml -e"@default-for-restore.yml"

ansible-playbook playbooks/lncl-restore-hub.yml -e"@default-for-restore.yml"
