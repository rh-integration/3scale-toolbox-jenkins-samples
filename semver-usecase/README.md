# Usecase "Semantic versioning": Deploy four versions of an API in three environments, all in one tenant

In this usecase, a [Jenkins pipeline](Jenkinsfile) will deploy **four versions of an API** on a 3scale SaaS instance, **in three environments**: `DEV`, `TEST` and `PROD`, using semantic versioning.

- The first version (**v0.9**) is not secured and used as a mock to be used by early adopters
- The second version (**v1.0**) is the first stable **major** version and secured with API Keys
- The third version (**v1.1**) is the first **minor** release of the 1.x branch and secured with API Keys
- The last version (**v2.0**) is the second **major** version and secured with OpenID Connect

## Pre-requisites

Make sure you completed the [SETUP guide](../SETUP.md).

## Installation

Use the [provided OpenShift template](setup.yaml) to install the Jenkins pipeline:

```sh
oc process -f semver-usecase/setup.yaml \
           -p DEVELOPER_ACCOUNT_ID="$SAAS_DEVELOPER_ACCOUNT_ID" \
           -p PRIVATE_BASE_URL="http://$EVENT_API_HOSTNAME" \
           -p PUBLIC_STAGING_WILDCARD_DOMAIN="$APICAST_SELF_MANAGED_STAGING_WILDCARD_DOMAIN" \
           -p PUBLIC_PRODUCTION_WILDCARD_DOMAIN="$APICAST_SELF_MANAGED_PRODUCTION_WILDCARD_DOMAIN" \
           -p OIDC_ISSUER_ENDPOINT="https://$CLIENT_ID:$CLIENT_SECRET@$SSO_HOSTNAME/auth/realms/$REALM" \
           -p NAMESPACE="$TOOLBOX_NAMESPACE" |oc create -f -
```

## Deployment

Deploy version 0.9:

```sh
oc start-build semver-usecase-v0.9
```

Deploy version 1.0:

```sh
oc start-build semver-usecase-v1.0
```

Deploy version 1.1:

```sh
oc start-build semver-usecase-v1.1
```

Deploy version 2.0:

```sh
oc start-build semver-usecase-v2.0
```
