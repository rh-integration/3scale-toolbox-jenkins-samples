# Usecase "Multi-environment": Deploy an API in three environments, all in one tenant

In this usecase, a [Jenkins pipeline](Jenkinsfile) will deploy an API described by an [OpenAPI Specification file](swagger.yaml) on a 3scale SaaS instance, in three environments: `DEV`, `TEST` and `PROD`. The API is secured using API Keys as described in the OAS.

## Pre-requisites

Make sure you completed the [SETUP guide](../SETUP.md).

## Installation

Use the [provided OpenShift template](setup.yaml) to install the Jenkins pipeline:

```sh
oc process -f multi-environment-usecase/setup.yaml \
           -p DEVELOPER_ACCOUNT_ID="$SAAS_DEVELOPER_ACCOUNT_ID" \
           -p PRIVATE_BASE_URL="http://$EVENT_API_HOSTNAME" \
           -p PUBLIC_STAGING_WILDCARD_DOMAIN="$APICAST_SELF_MANAGED_STAGING_WILDCARD_DOMAIN" \
           -p PUBLIC_PRODUCTION_WILDCARD_DOMAIN="$APICAST_SELF_MANAGED_PRODUCTION_WILDCARD_DOMAIN" \
           -p NAMESPACE="$TOOLBOX_NAMESPACE" |oc create -f -
```

## Deployment

```sh
oc start-build multi-environment-usecase
```
