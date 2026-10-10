Step-by-Step Execution
# increase SWAP and /

HOME_PART=$(mount | awk '/home/ {print $1}')

ROOT_PART=$(mount | awk '/root/ {print $1}')

SWAP_PART=$(awk '/swap/ {print $1}' /etc/fstab)

umount /home

mount ${HOME_PART} /mnt

mv /mnt/* /home/

umount /mnt

lvremove -y ${HOME_PART}

lvextend -L 32G ${SWAP_PART}

lvextend -l +100%FREE ${ROOT_PART}

xfs_growfs /

sed -i '/home/d' /etc/fstab

swapoff ${SWAP_PART}

mkswap ${SWAP_PART}

swapon -va


# install ansible 
dnf install ansible-core -y

# download the file
curl -L https://raw.githubusercontent.com/fardiant/ocp-4.21/refs/heads/main/create-cluster -o create-cluster

curl -L https://github.com/fardiant/ocp-4.21/raw/refs/heads/main/run-command.yml -o run-command.yml

# run the ansible
ansible-playbook -i localhost, -c local run-command.yml

# create the cluster
kcli create cluster openshift --pf hub.yml
