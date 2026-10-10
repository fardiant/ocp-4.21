Step-by-Step Execution

# install ansible 
dnf install ansible-core -y

# download the file
curl -L https://raw.githubusercontent.com/fardiant/ocp-4.21/refs/heads/main/create-cluster -o create-cluster

curl -L https://github.com/fardiant/ocp-4.21/raw/refs/heads/main/run-command.yml -o run-command.yml

# run the ansible
ansible-playbook -i localhost, -c local run-command.yml

# create the cluster
kcli create cluster openshift --pf hub.yml
