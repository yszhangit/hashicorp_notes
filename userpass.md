# auth
* auth method

Vault support multiple authentication methods, username/password, LDAP, Github etc.

* token

Vault verify your identity and then generating a token to associate with that identity, token then used for subsequent access.

* auth leases

Just like secrets, identities have leases associated with them. This means that you must reauthenticate after the given lease period to continue accessing Vault. Identities can be renewed without having to completely reauthenticate: `vault token renew <token>`


## Create user

### enable "userpass" auth method
`vault auth enable userpass`

### create user
`vault write auth/userpass/users/yzhang policies=default password=something`

there's no UI option

### list users
`vault  kv list  auth/userpass/users`
* query user

```
vault  kv get auth/userpass/users/yzhang
======= Data =======
Key            Value
---            -----
bound_cidrs    []
max_ttl        0
policies       [admins]
ttl            0
```


## user login
### CLI

[doc](https://www.vaultproject.io/docs/commands/login.html)


`vault login -method=userpass username=yzhang` will prompt password

or

`vault login -method=userpass username=yzhang password=something`

`.vault_token` create in $HOME, dont need to specify token afterwards



### API
[doc](https://www.vaultproject.io/api/auth/userpass/index.html)

```
curl --request POST --data '{"password":"something"} \
    http://127.0.0.1:8200/v1/auth/userpass/login/yzhang \
| jq '.'

{
  "request_id": "8d0cede7-5516-0171-d363-99d2bc982661",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": null,
  "wrap_info": null,
  "warnings": null,
  "auth": {
    "client_token": "a9c323ef-f1c0-2ff4-a4cd-ae8c0f249aed",
    "accessor": "f67cbc13-dbe7-5dfb-90e8-a38318d72de3",
    "policies": [
      "admins",
      "default"
    ],
    "token_policies": [
      "admins",
      "default"
    ],
    "metadata": {
      "username": "yzhang"
    },
    "lease_duration": 2764800,
    "renewable": true,
    "entity_id": "e53e910d-b1df-15cd-e2c2-cb9c56d99cd4"
  }
}

```

* use payload
```
cat yzhang_pwd.json 
{
  "password": "yinshu"
}

$ curl --request POST --data @yzhang_pwd.json http://127.0.0.1:8200/v1/auth/userpass/login/yzhang | jq '.'
```


