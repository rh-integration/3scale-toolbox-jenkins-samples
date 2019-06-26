# Usecase "SaaS - API Key": Deploy a simple API on 3scale SaaS

In this usecase, a [Jenkins pipeline](Jenkinsfile) will deploy an API described by an [OpenAPI Specification file](swagger.yaml) on a 3scale SaaS instance. The API is secured using API Keys as described in the OAS.

## Pre-requisites

Make sure you completed the [SETUP guide](../SETUP.md).

## Installation

Use the [provided OpenShift template](setup.yaml) to install the Jenkins pipeline:

```sh
oc process -f saas-usecase-apikey/setup.yaml \
           -p DEVELOPER_ACCOUNT_ID="$SAAS_DEVELOPER_ACCOUNT_ID" \
           -p PRIVATE_BASE_URL="http://$BEER_CATALOG_HOSTNAME" \
           -p NAMESPACE="$TOOLBOX_NAMESPACE" |oc create -f -
```

## Deployment

```sh
oc start-build saas-usecase-apikey
```
