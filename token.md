# Vault use token for authentication
[reference](https://www.vaultproject.io/docs/concepts/tokens.html)

"token is like cookie created in your browser"

Token can used directly or use auth method to create one.

Token maps to policy and other information(metadata for audit etc)

# auth method and token store
token is the built-in auth method. "When any other auth method returns an identity, Vault core invokes the token method to create a new unique token for that identity."

the backend store of "token" auth method is called "token store". "The token store can also be used to bypass any other auth method: you can create tokens directly, as well as perform a variety of other operations on tokens such as renewal and revocation."

## login with token
### CLI
`vault login token=<token>`
### API
pass token to header `X-Vault-Token: <token>`

# Hierarchies and orphan tokens
when token holder(user) create new token, it become children of origical token, when parent token is revoded, all children token and their leases are revoked.

Orphan token is token without parent, there are 4 way to do it

*    Via the auth/token/create-orphan endpoint
*    By having sudo capability or root policy when accessing auth/token/create and setting the orphan parameter to true
*    Via token store roles
*    By logging in with any other (non-token) auth method

# Accessors
Accessor is a vault act as reference to token can only be used for
* lookup token peoperties
* lookup capability on a path
* revoke

*only* way to list tokens is to list `auth/token/accessors`

# TTL
every non-root token has TTL.

token can extend TTL is renew is allowed

see reference for use case.

# root token
Beside the "can do anything" root policy, root token is only token in Vault can be set to never expier without needs for renew, so leave root token in production Vault is not good idea. there's recommandation to revoke root token immediately when it not longer needed, *AND* verified by "multiple pair of eyes".

To create root token, follow [this example](https://learn.hashicorp.com/vault/operations/ops-generate-root)

# types:
## service token
"normal" token has all the features
## batch token
lightweight and scalable but lack flexiblity and features, see reference near bottom

