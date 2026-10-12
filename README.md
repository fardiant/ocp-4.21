Step-by-Step Execution
# increase SWAP and /
```
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
```
# install ansible 
```
dnf install ansible-core -y
```

# download the file
```
curl -L https://raw.githubusercontent.com/fardiant/ocp-4.21/refs/heads/main/create-cluster -o /root/create-cluster
curl -L https://github.com/fardiant/ocp-4.21/raw/refs/heads/main/run-command.yml -o /root/run-command.yml
curl -L https://raw.githubusercontent.com/fardiant/ocp-4.21/refs/heads/main/hub-4.21.8.yaml -o /root/hub.yml
```
# run the ansible
```
ansible-playbook -i localhost, -c local run-command.yml
```

# create the cluster
```
kcli create cluster openshift --pf hub.yml
```

# clone the ztp-repository and ztp-pipeline from Internet to local repo
```
curl -u 'student:student' -H 'Content-Type: application/json' -X POST --data '{"service":"2","clone_addr":"https://github.com/fardiant/ocp-4.21.git","uid":1,"repo_name":"5g-deployment-lab"}' http://infra.5g-deployment.lab:3000/api/v1/repos/migrate
```
<img width="1777" height="797" alt="image" src="https://github.com/user-attachments/assets/2e58468d-4105-46af-aa1b-b66fcc69854e" />


# Configure OpenShift Hub Cluster 
```
export KUBECONFIG=~/.kcli/clusters/hub/auth/kubeconfig
oc -n openshift-storage wait lvmcluster lvmcluster --for=jsonpath='{.status.state}'=Ready --timeout=900s
oc patch storageclass lvms-vg1 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
curl https://raw.githubusercontent.com/RHsyseng/5g-ran-deployments-on-ocp-lab/lab-4.21/lab-materials/lab-env-data/hub-cluster/argocd-patch.json -o /tmp/argopatch.json
oc patch argocd openshift-gitops -n openshift-gitops --type=merge --patch-file /tmp/argopatch.json
oc wait --for=condition=Ready pod -lapp.kubernetes.io/name=openshift-gitops-repo-server -n openshift-gitops
oc -n openshift-gitops adm policy add-cluster-role-to-user cluster-admin -z openshift-gitops-argocd-application-controller
```

# Deploy OpenShift HUB Cluster Operators anc check on ArgoCD
```
oc apply -f https://raw.githubusercontent.com/RHsyseng/5g-ran-deployments-on-ocp-lab/lab-4.21/lab-materials/lab-env-data/hub-cluster/hub-operators-argoapps.yaml
```
<img width="1491" height="701" alt="image" src="https://github.com/user-attachments/assets/6adb65b1-8958-4efc-8a1d-64b1ba3d7f8b" />


# edit sshPublicKey in "5g-deployment-lab/ztp-repository/site-configs/hub-1/ocp-sno-abi.yaml" and commit the changes
```
[root@dell-r730-051 5g-deployment-lab]# git add --all
[root@dell-r730-051 5g-deployment-lab]# git commit -m 'change sshPublicKey in 5g-deployment-lab/ztp-repository/site-configs/hub-1/ocp-sno-abi.yaml'
[main 1b4c447] change sshPublicKey in 5g-deployment-lab/ztp-repository/site-configs/hub-1/ocp-sno-abi.yaml
 Committer: root <root@dell-r730-051.bkr.lab.eng.rdu2.dc.redhat.com>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 1 file changed, 1 insertion(+), 1 deletion(-)
[root@dell-r730-051 5g-deployment-lab]# git push
Username for 'http://infra.5g-deployment.lab:3000': student
Password for 'http://student@infra.5g-deployment.lab:3000':
Enumerating objects: 11, done.
Counting objects: 100% (11/11), done.
Delta compression using up to 20 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 904 bytes | 904.00 KiB/s, done.
Total 6 (delta 5), reused 0 (delta 0), pack-reused 0
remote: . Processing 1 references
remote: Processed 1 references in total
To http://infra.5g-deployment.lab:3000/student/5g-deployment-lab.git
   fef790f..1b4c447  main -> main
```
