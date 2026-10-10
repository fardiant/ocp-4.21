Step-by-Step Execution

# install ansible 
dnf install ansible-core -y

# download the file
curl -L https://raw.githubusercontent.com/fardiant/ocp-4.21/refs/heads/main/create-cluster -o create-cluster

curl -L 

# run the ansible
ansible-playbook -i localhost, -c local run_commands.yml

# create the cluster
kcli create cluster openshift --pf hub.yml
