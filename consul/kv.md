can run on any node in cluster

```
consul kv put redis/config/minconns 1
consul kv put redis/config/maxconns 25
consul kv put -flags=42 redis/config/users/admin abcd1234
consul kv get redis/config/minconns
consul kv get -detailed redis/config/users/admin
consul kv get -recurse
consul kv delete redis/config/minconns
consul kv delete -recurse redis
consul kv put foo bar
consul kv get foo
consul kv put foo zip
consul kv get foo
```
* what's flag for?
* need token if ACL enabled?
* `-http-addr` 
