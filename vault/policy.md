# intro
[reference](https://www.vaultproject.io/docs/concepts/policies.html)

* Everything in Vault is path base, as for policies
* Deny by default, an empty policy grant nothing

# view/create/update/delete policy
## CLI
```
vault read sys/policy
vault policy write policy-name policy-file.hcl
vault write sys/policy/my-existing-policy policy=@updated-policy.json
vault delete sys/policy/policy-name
```

## API
```
curl \
  --header "X-Vault-Token: ..." \
  https://{ vault API URL }/v1/sys/policy
 curl \
  --request POST \
  --header "X-Vault-Token: ..." \
  --data '{"policy":"path \"...\" {...} "}' \
  https://{ vault API URL }/v1/sys/policy/policy-name
 curl \
  --request POST \
  --header "X-Vault-Token: ..." \
  --data '{"policy":"path \"...\" {...} "}' \
  https://{ vault API URL }/v1/sys/policy/my-existing-policy
curl \
  --request DELETE \
  --header "X-Vault-Token: ..." \
  https://{ vault API URL }/v1/sys/policy/policy-name

```

# HCL syntax
grant read permission to path "secret/foo"
```
path "secret/foo" {
  capabilities = ["read"]
}
```

## pattern 

### `*` it is not a regex, only works at last char of a path

```
# grant all premission to "secret" but "secret/super-secret"
path "secret/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}
path "secret/super-secret" {
  capabilities = ["deny"]
}

# match pw-foo pw-bar
path "secret/pw-*" {
  capabilities = ["read"]
}
```

### `+` match any number of char

```
# Permit reading the "teamb" path under any top-level path under secret/
path "secret/+/teamb" {
  capabilities = ["read"]
}

# Permit reading secret/foo/bar/teamb, secret/bar/foo/teamb, etc.
path "secret/+/+/teamb" {
  capabilities = ["read"]
}
```

### multi-match, need example
| Policy paths are matched using the most specific path match. This may be an exact match or the longest-prefix match of a glob. This means if you define a policy for "secret/foo", the policy would also match "secret/foobar".

## capabilities

| Each path must define one or more capabilities which provide fine-grained control over permitted (or denied) operations. As shown in the examples above, capabilities are always specified as a list of strings, even if there is only one capability.

common capabilities are create/read/update/delete/list/deny, not so common sudo is to allow access root-protected path.

## template
see reference for list of parameter names. use ID whenever possible, name can be changed but not ID.

```
path "secret/data/{{identity.entity.id}}/*" {
  capabilities = ["create", "update", "read", "delete"]
}
```

## fine-grained control
Policy can be further restricted, with you can do under a path, they are `required_parameters/allowed_parameters/denied_parameters`, see reference for details. 

## Wrapped response
`min_wrapping_ttl` and `max_wrapping_ttl` specify TTLs when response requests being wrapped, accept `s/m/h`. see wrapping.md for detail

# Builtin Policies
`default` and `root`, they cant be removed.

## default
attach to all token by default, provide basic functions. to get list of path and capabilities run `vault read sys/policy/default`. to disable `vault token create -no-default-policy`.

## root
can not modified. a user associated with this policy can do *ANYTHING* in vault. therefore revoke root token is recommended in production Vault after installation.

