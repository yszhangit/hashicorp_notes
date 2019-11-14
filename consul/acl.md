# consul ACL
is used to secure UI,API,CLI, and communication. At the core, ACLs operate by grouping rules into policies, then associating one or more policies with a token.

# acl bootstrapping, on exisiting consul
[ref](https://learn.hashicorp.com/consul/day-0/acl-guide?utm_source=consul.io&utm_medium=docs)

### enable on all servers
add below to config and restart all consul servers

```
{
  "acl": {
    "enabled": true,
    "default_policy": "deny",
    "down_policy": "extend-cache"
  }
}
```
if `global-mamagerment` creation message in log, ACL is not enabled, need to create bootstrap token

### create bootstrap token
```
root@consul1:~# consul acl bootstrap
AccessorID:       e0c7b566-e951-ddf0-f60f-a1f921c379d2
SecretID:         810f0a9a-b0bf-3c7c-4fd1-728afaa0c7ca
Description:      Bootstrap Token (Global Management)
Local:            false
Create Time:      2019-11-14 04:35:02.798995868 +0000 UTC
Policies:
   00000000-0000-0000-0000-000000000001 - global-management
```
use token

```
consul members
# not output

consul members -token "810f0a9a-b0bf-3c7c-4fd1-728afaa0c7ca"

Node     Address            Status  Type    Build  Protocol  DC    Segment
consul1  192.168.56.2:8301  alive   server  1.6.1  2         home  <all>
consul2  192.168.56.3:8301  alive   server  1.6.1  2         home  <all>

# on any node
export CONSUL_HTTP_TOKEN=810f0a9a-b0bf-3c7c-4fd1-728afaa0c7ca
consul members
# should see output
```

The bootstrap token can also be used in the server configuration file as the `master` token.

Note, the bootstrap token can only be created once, bootstrapping will be disabled after the master token was created

once ACL enabled, you can use CLI to manage ACL

`consul acl token list`

### create agent token policy, and agent token
```
# agent-policy.hcl contains the following:
node_prefix "" {
   policy = "write"
}
service_prefix "" {
   policy = "read"
}
```

This policy will allow all nodes to be registered and accessed and any service to be read. Note, this simple policy is not recommended in production.

```
consul acl policy create -name "agent-token" -description "Agent Token Policy" -rules @agent-policy.hcl

ID:           922a7f36-4d1f-5aa1-d97c-c8fcb83e70aa
Name:         agent-token
Description:  Agent Token Policy
Datacenters:  
Rules:
# agent-policy.hcl contains the following:
node_prefix "" {
   policy = "write"
}
service_prefix "" {
   policy = "read"
}

consul acl token create -description "Agent Token" -policy-name "agent-token"

AccessorID:       8a93e50a-f9b8-e7d3-492f-49be4d38a9f4
SecretID:         4547d404-a5dd-c674-5780-3945cad68ab1
Description:      Agent Token
Local:            false
Create Time:      2019-11-14 05:17:09.77305422 +0000 UTC
Policies:
   922a7f36-4d1f-5aa1-d97c-c8fcb83e70aa - agent-token


```

### add agent token to all server
```
  "acl": {
    "enabled": true,
    "default_policy": "deny",
    "down_policy": "extend-cache",
    "tokens": {
      "agent": "4547d404-a5dd-c674-5780-3945cad68ab1"
    }
  }
```
restart and check
```
export CONSUL_HTTP_TOKEN=4547d404-a5dd-c674-5780-3945cad68ab1
 consul members
Node     Address            Status  Type    Build  Protocol  DC    Segment
consul1  192.168.56.2:8301  alive   server  1.6.1  2         home  <all>
consul2  192.168.56.3:8301  alive   server  1.6.1  2         home  <all>
```

### enable ACL on Consul client
this can add to agent since there no restrict any node or service prefixes.

```
  "acl": {
    "enabled": true,
    "default_policy": "deny",
    "down_policy": "extend-cache",
    "tokens": {
      "agent": "4547d404-a5dd-c674-5780-3945cad68ab1"
    }
  }
```


### UI token
use master token

```
export CONSUL_HTTP_TOKEN=810f0a9a-b0bf-3c7c-4fd1-728afaa0c7ca
```
create policy
```
consul acl policy create -name "ui-policy" \
-description "Necessary permissions for UI functionality" \
-rules 'key_prefix"" { policy = "write" } node_prefix "" { policy = "read" } service_prefix "" { policy = "read" }'

ID:           49437c4b-863d-3853-133a-6c56f6788797
Name:         ui-policy
Description:  Necessary permissions for UI functionality
Datacenters:  
Rules:
key_prefix"" { policy = "write" } node_prefix "" { policy = "read" } service_prefix "" { policy = "read" }
```
create token
```
consul acl token create -description "UI Token" -policy-name "ui-policy"
AccessorID:       e5fd6011-f824-ec21-b923-4fce8d18f554
SecretID:         a0b3be9e-d5d6-53f7-fd96-870b903b0e25
Description:      UI Token
Local:            false
Create Time:      2019-11-14 05:42:55.464107883 +0000 UTC
Policies:
   49437c4b-863d-3853-133a-6c56f6788797 - ui-policy
```
use `a0b3be9e-d5d6-53f7-fd96-870b903b0e25` in web ACL, and click `save`, should get a message `you are logged in`
