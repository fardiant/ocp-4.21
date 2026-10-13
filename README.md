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

# create the cluster ~2h
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
git add --all
git commit -m 'change sshPublicKey in 5g-deployment-lab/ztp-repository/site-configs/hub-1/ocp-sno-abi.yaml'
git push
```

# Apply the ZTP GitOps Pipeline configuration ~60min to finish
```
oc apply -k ~/5g-deployment-lab/ztp-pipeline/argocd/deployment/
```
<img width="1900" height="722" alt="image" src="https://github.com/user-attachments/assets/ee57592c-b8f8-4700-b2dd-1b96bf83a0af" />

# check the managedcluster
```
oc get agentclusterinstall,agent -A
oc get managedcluster -A
```

# check the MINIO bucket
```
s3cmd ls
```
<img width="1876" height="913" alt="image" src="https://github.com/user-attachments/assets/40369ee2-79ed-4282-80a2-aeb9bd16c17f" />

# check grafana
<img width="1902" height="1000" alt="image" src="https://github.com/user-attachments/assets/7f4eaa74-1248-4efd-be21-2f6dc5fbd51d" />
