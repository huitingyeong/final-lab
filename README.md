# final-lab

$ lab destroy -i ac

$ lab deploy -i finallab

$ ansible all --list-hosts

$ mkdir $HOME/final-lab

$ cd $HOME/final-lab

==============================================
4.1.1. Ansible Automation Controller
Deploy Standalone Ansible Automation Controller with a database on the same node.

$ ansible localhost -m unarchive -a "src=/tmp/ansible-automation-platform-setup-2.1.0-1.tar.gz dest=/home/devops/final-lab"

$ cd ansible-automation-platform-setup-2.1.0-1/

$ cat > inventory <<EOF
[automationcontroller]
ac.example.com

[database]
ac.example.com

[all:vars]
admin_password='r3dh4t1!'
ansible_become=true
pg_host='ac.example.com'
pg_port='5432'
pg_database='automationcontroller'
pg_username='ac'
pg_password='r3dh4t1'
pg_sslmode='prefer'

# Execution Environment Configuration
# Credentials for container registry to pull execution environment images from,
# registry_username and registry_password are required for registry.redhat.io
registry_url='registry.redhat.io'
registry_username='outsideout1975'
registry_password='P@sw0rd123456'

EOF

$ ./setup.sh

https://bastion.lb39a2.dynamic.opentlc.com:49210/

==============================================
4.1.2. Private Automation Hub
Deploy Standalone Private Automation Hub with a database on the same node.

$ cat > inventory <<EOF
[automationcontroller]


[automationhub]
hub.example.com

[all:vars]
ansible_become=true
automationhub_admin_password= r3dh4t1!

automationhub_pg_host=''
automationhub_pg_port=''

automationhub_pg_database='automationhub'
automationhub_pg_username='automationhub'
automationhub_pg_password=r3dh4t1!
automationhub_pg_sslmode='prefer'

EOF

$ ./setup.sh

https://bastion.lb39a2.dynamic.opentlc.com:49211/


==============================================
4.2. Build Execution Environment

$ mkdir ee-finallab

$ cd ee-finallab/

Execution Environment Image   ee-mitzidotcom:1.0.0

$ cat > execution-environment.yml <<EOF
version: 1
dependencies:
  galaxy: requirements.yml
  python: requirements.txt
  system: bindep.txt
EOF

Python Library    openstacksdk==0.36.5

$ cat > requirements.txt <<EOF
openstacksdk==0.36.5
EOF

$ cat > bindep.txt <<EOF
python3 [platform:rpm]
EOF

$ cat > $HOME/final-lab/requirements.yml <<EOF
collections:
  - openstack.cloud
  - community.general
  - ansible.posix
  - community.mysql
  - name: http://bastion.example.com:5050/devops/mitzidotcom
    type: git
EOF

$ ansible-builder build -t ee-mitzidotcom:1.0.0

$ podman images

$ podman image prune

$ ansible-navigator images

Push ee-finallab:1.0.0 Execution Environment Image to Private Automation Hub (hub.example.com).

$ podman tag localhost/ee-mitzidotcom:1.0.0 hub.example.com/ee-mitzidotcom:1.0.0

$ podman login -u admin -p r3dh4t1! hub.example.com --tls-verify=false

$ podman push --tls-verify=false hub.example.com/ee-mitzidotcom:1.0.0 --remove-signatures