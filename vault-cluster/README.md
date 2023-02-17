# Prereqs
Run `terraform apply --auto-approve` to build cluster

# Add context
```
aws eks --region $(terraform output -raw region) update-kubeconfig \
    --name $(terraform output -raw cluster_name)
```

# Add license to k8s secret
```
secret=$VAULT_LICENSE
`kubectl create secret generic vault-ent-license --from-literal="license=${secret}"`
```

# Install Vault
`helm install vault hashicorp/vault --values vault-cluster.yaml`

# Initialize Vault on vault-0
`kubectl exec vault-0 -- vault operator init -key-shares=1 -key-threshold=1 -format=json > cluster-keys.json`

# Unseal vault-0
`VAULT_UNSEAL_KEY=$(jq -r ".unseal_keys_b64[]" cluster-keys.json)`
`kubectl exec vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY`

# Add nodes to cluster 
`kubectl exec -ti vault-1 -- vault operator raft join http://vault-0.vault-internal:8200`
`kubectl exec -ti vault-1 -- vault operator unseal $VAULT_UNSEAL_KEY`
`kubectl exec -ti vault-2 -- vault operator raft join http://vault-0.vault-internal:8200`
`kubectl exec -ti vault-2 -- vault operator unseal $VAULT_UNSEAL_KEY`

# Login to vault-0
`kubectl exec --stdin=true --tty=true vault-0 -- /bin/sh`