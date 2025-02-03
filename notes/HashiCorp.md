
**Secrets** - Anything that grants authentication or authorization to a system
- Username password
- DB credentials
- API tokens
- TLS certificates

Secret sprawl -> Secrets spread all over the infrastructure
To centralize this we use HashiCorp
To control access to these secrets
To encrypt at rest as well as in transit
To get an Audit trail 

Applications when provided with credentials exposes them to logs, diagnostics, monitoring. 
So we use Dynamic secrets. These are Ephemeral. Valid for a set time.
Unique credentials for 50 machines to get the exact point of leak



# Installation
```bash
# Add hashicorp gpg key
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# Add hashicorp Repo
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# Update & install
sudo apt update && sudo apt install vault
```

# Start Stop
### 2 modes 
1. Dev mode 
2. Server mode

`vault server -dev`

**Vault Address**
**Port** - 8200
**Storage** - inmem
	All storage done in memory 

But for prod server or prod mode creds store in disk or db

**Unseal key**

**Root token**

```bash 
export VAULT_ADDR=""
export VAULT_TOKEN=""
vault status
```


---------------------------------------
# Read | Write | Delete Secrets

#### Enable secret engine
`vault secrets enable -path=my kv`

### Write
`vault kv put my/path myKey1=value1

### Read
`vault kv get my/path`
`vault kv get -format=json my/path`

### Delete
`vault kv delete my/path`

#### Secret engine
Additional plugins to store different types of secrets

List all enabled secrets engine
`vault secrets list`

Enable secret engine
`vault secrets enable -path=aws aws`

Disable secret engine
`vault secrets disable aws`

# AWS Secret engine 

Enable AWS secret engine
`vault secrets enable -path=aws aws`
### 1. 
```bash 
vault write aws/config/root \
access_key=fljkadf \
secret_key=fasfda \
region=eu-north-1
```


### 2. Set up a role
```bash
vault write aws/roles/my-ec2-role \
    credential_type=iam_user \
    policy_document='{
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "Stmt1426528957000",
          "Effect": "Allow",
          "Action": [
            "ec2:*"
          ],
          "Resource": [
            "*"
          ]
        }
      ]
    }'

		
```

### 3. Generate dynamic ACCESS_KEY & SECRET_KEY

`vault read aws/creds/my-ec2-role`

### 4. Destroy or revoke this dynamically generated secret

```bash
vault lease revoke aws/creds/my-ec2-role/<lease_id>
```

--------------------------

# Policies

Secrets are stored at different paths on the server
Policies are applied to these paths
Whenever we write a policy we tell vault what kind of operation to control with the policy (read, write, fetch, update)
We can reuse policies on different paths 



Commands
`vault policy list

Create a Policy
```bash
vault policy write my-policy - <<EOF
path "secret/data/*" {
  capabilities = ["create", "update"]
}

path "secret/data/foo" {
  capabilities = ["read"]
}
EOF

```

Or create a policy file (my-policy.hcl)and then write it 
```hcl
path "secret/data/*" {
  capabilities = ["create", "update"]
}

path "secret/data/foo" {
  capabilities = ["read"]
}

```
Write is using the file name
```hcl
vault policy write my-policy ./my-policy.hcl
```

Read a policy 
```bash
vault policy read my-policy
```

Delete a policy
```bash
vault policy delete my-policy
```

----------------------------------
# Vault Tokens

Root token is shown when we start dev server
With root token you can access everything in the vault

### How to attach Token to a Policy

`export VAULT_TOKEN="$(vault token create -field token -policy=my-policy)"

----------------------------------
# Write some secrets with policy

```
vault kv put -mount=secrets creds password="myPassword"
```

	kv -> key value
	-mount=secret   ->      (to mention path "secret/data/creds" for some reason you dont need to specify data)
	password="myPassword"  -> is just a key-value pair



-----------------------------------------------
# Auth Methods & Policies 

Auth methods is just using external services like github, aws, etc. to provide external authentication 

List Auth methods
`vault auth list`

Enable custom auth method
`vault auth enable approle`

```BASH
vault write auth/approle/role/my-role \
    secret_id_ttl=10m \
    token_num_uses=10 \
    token_ttl=20m \
    token_max_ttl=30m \
    secret_id_num_uses=40 \
    token_policies=my-policy

```

```
export ROLE_ID="$(vault read -field=role_id auth/approle/role/my-role/role-id)"

export SECRET_ID="$(vault write -f -field=secret_id auth/approle/role/my-role/secret-id)"
```


--------------------------
# Token Authentication

Root token needs to be provided to access the vault


`vault token create`  ensure the policies are root 

`vault login`  then enter token generated

`vault token revoke <token>`

--------------------------
# GitHub Authentication

Use GitHub account to generate access token and using that login to HashiCorp Vault

1. On GitHub create an organisation
2. On the same page create a team

On Vault CLI

```BASH
vault auth enable github

vault write auth/github/config organisation=zta-test-org-1

vault write auth/github/map/teams/ZTA-Team values=default,application
```

The people on that team will have access at default and application level

Generate a PAT on GitHub give is admin.org permissions

```
vault login -method=github         (enter then paste the PAT token here)
```

Now we have accessed the vault using GitHub credentials or PAT


-------------------------------
# Deploy HashiCorp Vault to Production 

Stop the server Ctrl+c
unset VAULT_TOKEN

Create a vault configuration file with .hcl extension
```
mkdir -p ./vault/data
nano config.hcl
```

```hcl 
storage "raft" {
  path    = "./vault/data"
  node_id = "node1"
}

listener "tcp" {
  address     = "127.0.0.1:8200"
  tls_disable = true
}

api_addr = "http://127.0.0.1:8200"
cluster_addr = "https://127.0.0.1:8201"
ui = true
```

storage
	path where everything will be stored by hashicorp vault 
	node_id -> can be anything
listener
	ip of vault server
	disabled https (for demo only - not to be done in prod or in our project)

Now we need to start server in production  mode

```
vault server -config=config.hcl
```

Now in another terminal
`export VAULT_ADDR=http://127.0.0.1:8200

Initial the vault. You will get the unseal keys and the Root token 
`vault operator init`

Note the keys and token
```
Unseal Key 1: qXLWnximvUPzo87wGpMfMshvVUkJneD5kCTeOLBMomtr
Unseal Key 2: x+ZYq3vUzY1RQ40EHNc19/bUr6CotXHD4oGnfJZeZqo8
Unseal Key 3: p66EruyPGAcuGDPFRiKEGJmFAgLnSCm0SEMzSeqW+/kL
Unseal Key 4: ls7ONnRW2KnOkLg5M09QUm/wHzN1Q0vU5C6Jgw0IIJ4N
Unseal Key 5: uDn3AAyv3kEifviPvEKtI5XSY9T6hee/Kif1l3yGLEeZ

Initial Root Token: hvs.8PYQMs3NdW5760o1Z0ISVxxP
```

### What is seal unseal in Hashicorp Vault? 
When vault starts it starts in a sealed state
In sealed state we can access data but not perform any read/write operations
So we need to unseal the vault using the unseal keys


# Accessing Secrets via the REST APIs
