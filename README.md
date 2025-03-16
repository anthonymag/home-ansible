#n home-ansible

# Configuring a k3s node

## configure the unique machine

**This needs to be repeated for each server and agent**

1. launch instance in Proxmox with the ubuntu-template

2. Confirm the authorized_keys are up to date

```
# FROM THE PROXMOX TTY
curl https://github.com/anthonymag.keys > /home/ant/.ssh/authorized_keys
```

3. change the desired hostname number in the ubuntu_bootstrap.yml playbook (e.g. agent00)

3. run the bootstrap playbook

```
ansible-playbook playbook/ubuntu_bootstrap.yml --ask-become-pass
```

4. Reboot the machine

## install k3s

log into the machine in the Proxmox console

get the new ip(s)

```
ip a
```

Update the Ips in inventory-new.yml

Get the roles from the k3s-ansible playbook (TODO: fix this)

```
cp -r roles/k3s/files/k3s-ansible/roles/* roles/
```

Run the playbook
```
ansible-playbook -i inventory-new.yml playbooks/install_k3s_argocd.yml
```

Note the new kube config

```
ls ~/.kube/config.new
```

Set context
```
kubectl config use-context ~/.kube/config.new k3s-ansible
``` 

Note nodes

```
kubectl get nodes --kubeconfig ~/.kube/config.new
```

## Install charts

Run the charts playbook
Make sure `helm` is installed

```
ansible-playbook playbooks/install_base_charts.yml
```

## Log into ArgoCD

Forward ports

```
kubectl -n argocd port-forward services/argocd-server 8443:https
```

Browse to https://localhost:8443

Log in with the secret from the chart (you'll have to decode with base64)

```
kubectl get secret -n argocd argocd-initial-admin-secret -o yaml
```
