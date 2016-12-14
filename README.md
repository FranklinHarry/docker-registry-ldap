# Deploy Docker Registry
This repository contains the playbooks to deploy a Docker Registry with LDAP authentication.

First, make sure you have set up a [nginx reverse proxy](https://github.com/lawliet89/docker-nginx-tutorial).

Assuming that the files for nginx are in `~/nginx` and in particular, the certificats are stored at
`~/nginx/certs`, then you can do:

`ansible-playbook -i inventory site.yml --ask-vault-pass`

Otherwise, override the variables in the playbook as required.

# Reference

The LDAP setup is referenced from [this repository](https://github.com/kwk/docker-registry-setup)

# Manual Querying

```bash
# This is the operation we want to perform on the registry
REGISTRY_URL=https://registry.example.com/v2/_catalog

# Save the response headers of our first request to the registry to get the Www-Authenticate header
RESPONSE_HEADER=$(tempfile);
curl --dump-header $RESPONSE_HEADER $REGISTRY_URL

# Extract the realm, the service, and the scope from the Www-Authenticate header
WWWAUTH=$(cat $RESPONSE_HEADER | grep "Www-Authenticate")
REALM=$(echo $WWWAUTH | grep -o '\(realm\)="[^"]*"' | cut -d '"' -f 2)
SERVICE=$(echo $WWWAUTH | grep -o '\(service\)="[^"]*"' | cut -d '"' -f 2)
SCOPE=$(echo $WWWAUTH | grep -o '\(scope\)="[^"]*"' | cut -d '"' -f 2)

# Build the URL to query the auth server
AUTH_URL="${REALM}?service=${SERVICE}&scope=${SCOPE}"

# Query the auth server to get a token
TOKEN=$(curl -s --user "mozart:password" "$AUTH_URL")

# Get the bare token from the JSON string: {"token": "...."}
TOKEN=$(echo $TOKEN | jq .token | tr -d '"')

# Query the registry again, but this time with a bearer token
curl -v -H "Authorization: Bearer $TOKEN" $REGISTRY_URL
```

For more queries, refer to the [API documentation](https://docs.docker.com/registry/spec/api/)
