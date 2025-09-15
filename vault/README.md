# Learn Terraform - Provision an EKS Cluster

This repo is a companion repo to the [Provision an EKS Cluster tutorial](https://developer.hashicorp.com/terraform/tutorials/kubernetes/eks), containing
Terraform configuration files to provision an EKS cluster on AWS.

This repo a fork from `https://github.com/hashicorp/learn-terraform-provision-eks-cluster`.

# Deploy EKS cluster using Terraform

`terraform init`
`terraform apply`

# Get k8s cluster info

`aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)`

`kubectl cluster-info`

# Deploy Vault 

1. Set license in k8s
   1. `export VAULT_LICENSE="<license_string>"`
   2. `secret=$VAULT_LICENSE`
   3. `kubectl create secret generic vault-ent-license --from-literal="license=${secret}"`
2. Deploy Vault Helm chart
   1. `helm install vault hashicorp/vault --values vault-values.yaml`
3. Confirm that pods are up
   1. `kubectl get pods`
4. Run init.sh to initialize, unseal, and configure clusters
   1. `./init.sh`
5. Check vault status on vault-0
   1. `kubectl exec vault-0 -- vault status`
6. Login to vault-0
   1. `kubectl exec vault-0 -- vault login $(jq -r ".root_token" cluster-a-keys.json)`
7. Check vault-0 Raft status
   1. `kubectl exec vault-0 -- vault operator raft list-peers`
8. Check vault status on vault-3
   1. `kubectl exec vault-3 -- vault status`
9. Login to vault-3
   1. `kubectl exec vault-3 -- vault login $(jq -r ".root_token" cluster-b-keys.json)`
10. Check vault-3 Raft status
    1.  `kubectl exec vault-3 -- vault operator raft list-peers`

# Cleanup

## Uninstall Vault Helm chart and delete PVCs
`./cleanup.sh`

## Cleanup EKS cluster resources
`terraform destroy`


# Notes

If PVCs are showing up as unbound, run `kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'` to patch the StorageClass so that gp2 is the default for the cluster.
