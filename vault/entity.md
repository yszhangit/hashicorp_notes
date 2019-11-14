"Vault clients can be mapped as entities and their corresponding accounts with authentication providers can be mapped as aliases. In essence, each entity is made up of zero or more aliases. Identity secrets engine internally maintains the clients who are recognized by Vault."
say if I have account in "userpass" and LDAP, I can have 2 alias associated with my account, if I login using LDAP, or userpass, different entity identifiers can be tied to authenticated token.

