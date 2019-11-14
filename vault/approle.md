# approle for applications

BTW, how you disable kv with version enabled?

```
vault auth enable approle

cat > approle_test1.hcl << EOF 
path "secret/apps/*" {
  capabilities = [ "read" ]
}
EOF

vault policy write approle_test1 approle_test1.hcl

vault write auth/approle/role/approle1 policies="approle_test1"

vault kv list auth/approle/role

vault kv get auth/approle/role/approle1

bash-4.4# vault read auth/approle/role/approle1/role-id
Key        Value
---        -----
role_id    517692da-ca2c-8c14-2879-4a12a047675b

```
generate a new secret ID

```
bash-4.4# vault write -f auth/approle/role/approle1/secret-id
Key                   Value
---                   -----
secret_id             e671a56a-e7cc-9e5e-fe8b-cd1e6a555753
secret_id_accessor    b1745958-d489-9829-030c-c688b812a2f9
bash-4.4# 

bash-4.4# vault kv list auth/approle/role/approle1/secret-id
Keys
----
b1745958-d489-9829-030c-c688b812a2f9
```

login
```
vault write auth/approle/login \
	role_id="517692da-ca2c-8c14-2879-4a12a047675b" \
	secret_id="e671a56a-e7cc-9e5e-fe8b-cd1e6a555753"

Key                     Value
---                     -----
token                   074ca47a-bdfa-a454-df3f-ee869d030c40
token_accessor          bc1bffbb-bdd2-f753-fd8d-86e41b5b8884
token_duration          768h
token_renewable         true
token_policies          ["approle_test1" "default"]
identity_policies       []
policies                ["approle_test1" "default"]
token_meta_role_name    approle1

bash-4.4# cat .vault-token 
bc06fb46-56cb-96e3-8e86-1314c285c2a8bash-4.4# 

bash-4.4# vault kv get secret/apps/test1
====== Metadata ======
Key              Value
---              -----
created_time     2019-11-06T03:54:45.975600468Z
deletion_time    n/a
destroyed        false
version          1

=== Data ===
Key    Value
---    -----
foo    bar


```
# test ansible
https://github.com/TerryHowe/ansible-modules-hashivault

ubuntu fix:
```
sudo ln -s /usr/local/lib/python2.7/dist-packages/ansible/modules/hashivault /usr/lib/python2.7/dist-packages/ansible/modules/hashivault
sudo ln -s /usr/local/lib/python2.7/dist-packages/ansible/module_utils/hashivault.py /usr/lib/python2.7/dist-packages/ansible/module_utils/hashivault.py
```
