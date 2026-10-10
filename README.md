Step-by-Step Execution

# install ansible 
dnf install ansible-core -y

# run the ansible
ansible-playbook -i localhost, -c local run_commands.yml

# create the cluster
kcli create cluster openshift --pf hub.yml
