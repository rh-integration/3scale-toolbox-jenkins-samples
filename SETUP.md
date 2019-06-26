# Environment Setup

## Pre-requisites

- OpenShift Cluster
- Linux or Mac Workstation
- [3scale SaaS Tenant](https://www.3scale.net/signup)

## 3scale SaaS Environment

- Go to your 3scale SaaS Admin console
- [Generate a new Access Token](https://access.redhat.com/documentation/en-us/red_hat_3scale/2-saas/html/accounts/tokens) that has **write access** to the **Account Management API**
- Save the generated access token for later use:

```sh
export SAAS_ACCESS_TOKEN=123...456
```

- Save the name of your 3scale tenant (the string before `-admin.3scale.net` in your Admin Console) for later use

```sh
export SAAS_TENANT=nmasse-redhat
```

- Navigate to **Audience** > **Accounts** > **Listing**
- Click on **Developer**
- Saver the **Developer** Account ID that is the last part of the URL (after **/buyers/accounts/**)

```sh
export SAAS_DEVELOPER_ACCOUNT_ID=2445582535751
```

## 3scale on-prem environment

- [Deploy 3scale 2.5 on your OpenShift environment](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.5/html/installing_3scale/onpremises-installation)
- [Generate a new Access Token](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.5/html/admin_portal_guide/tokens) that has **write access** to the **Account Management API**
- Save the generated access token for later use:

```sh
export ONPREM_ACCESS_TOKEN=123...456
```

- Save the hostname of your 3scale Admin Portal for later use:

```sh
export ONPREM_ADMIN_PORTAL_HOSTNAME="$(oc get route system-provider-admin -o jsonpath='{.spec.host}')"
```

- Define your wildcard routes:

```sh
export OPENSHIFT_ROUTER_SUFFIX=app.openshift.test # Replace me !
export APICAST_ONPREM_STAGING_WILDCARD_DOMAIN=onprem-staging.$OPENSHIFT_ROUTER_SUFFIX
export APICAST_ONPREM_PRODUCTION_WILDCARD_DOMAIN=onprem-production.$OPENSHIFT_ROUTER_SUFFIX
```

**Note:** You will have to set the value of the `OPENSHIFT_ROUTER_SUFFIX` variable to the suffix of your OpenShift Router (usually something such as `app.openshift.test`).

- Add the wildcard routes to your existing 3scale on-prem instance

```sh
oc create route edge apicast-wildcard-staging --service=apicast-staging --hostname="wildcard.$APICAST_ONPREM_STAGING_WILDCARD_DOMAIN" --insecure-policy=Allow --wildcard-policy=Subdomain
oc create route edge apicast-wildcard-production --service=apicast-production --hostname="wildcard.$APICAST_ONPREM_PRODUCTION_WILDCARD_DOMAIN" --insecure-policy=Allow --wildcard-policy=Subdomain
```

- Navigate to **Audience** > **Accounts** > **Listing**
- Click on **Developer**
- Save the **Developer** Account ID that is the last part of the URL (after **/buyers/accounts/**)

```sh
export ONPREM_DEVELOPER_ACCOUNT_ID=5
```

## Deploy Red Hat SSO

Deploy Red Hat SSO 7.3 as explained in [the official documentation](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.3/html/red_hat_single_sign-on_for_openshift/get_started).

A short sumup is given for convenience:

```sh

oc replace -n openshift --force -f https://raw.githubusercontent.com/jboss-container-images/redhat-sso-7-openshift-image/sso73-dev/templates/sso73-image-stream.json
oc replace -n openshift --force -f https://raw.githubusercontent.com/jboss-container-images/redhat-sso-7-openshift-image/sso73-dev/templates/sso73-x509-postgresql-persistent.json
oc -n openshift import-image redhat-sso73-openshift:1.0
oc policy add-role-to-user view system:serviceaccount:$(oc project -q):default
oc new-app --template=sso73-x509-postgresql-persistent --name=sso -p DB_USERNAME=sso -p SSO_ADMIN_USERNAME=admin -p DB_DATABASE=sso
```

Save the hostname of your SSO installation for later use:

```sh
export SSO_HOSTNAME="$(oc get route sso -o jsonpath='{.spec.host}')"
```

## Configure Red Hat SSO

- Configure Red Hat SSO for 3scale as explained [in the documentation](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.5/html/using_the_developer_portal/openid-connect)
- Save the Realm name, client_id and client_secret for later use:

```sh
export CLIENT_ID=3scale-admin
export CLIENT_SECRET=123...456
export REALM=3scale
```

## Install Jenkins

Create an OpenShift project to hold all your artefacts:

```sh
oc project api-lifecycle
```

Save the name of the project for later use:

```sh
export TOOLBOX_NAMESPACE=api-lifecycle
```

Deploy a Jenkins master:

```sh
oc new-app -n "$TOOLBOX_NAMESPACE" --template=jenkins-ephemeral --name=jenkins -p MEMORY_LIMIT=2Gi
oc set env -n "$TOOLBOX_NAMESPACE" dc/jenkins JENKINS_OPTS=--sessionTimeout=86400
```

## Generate the 3scale toolbox secret

- First, [install the 3scale toolbox locally](https://github.com/3scale/3scale_toolbox#installation).
- Then, create a secret that contains all your [3scale remotes](https://github.com/3scale/3scale_toolbox/blob/master/docs/remotes.md):

```sh
3scale remote add 3scale-saas "https://$SAAS_ACCESS_TOKEN@$SAAS_TENANT-admin.3scale.net/"
3scale remote add 3scale-onprem "https://$ONPREM_ACCESS_TOKEN@$ONPREM_ADMIN_PORTAL_HOSTNAME/"
oc create secret generic 3scale-toolbox -n "$TOOLBOX_NAMESPACE" --from-file="$HOME/.3scalerc.yaml"
```

## Deploy the sample API backends

Deploy the sample Beer Catalog API Backend (used by the first three usecases):

```sh
oc new-app -n "$TOOLBOX_NAMESPACE" -i openshift/redhat-openjdk18-openshift:1.4 https://github.com/microcks/api-lifecycle.git --context-dir=/beer-catalog-demo/api-implementation --name=beer-catalog
oc expose -n "$TOOLBOX_NAMESPACE" svc/beer-catalog
```

Save the Beer Catalog API hostname for later use:

```sh
export BEER_CATALOG_HOSTNAME="$(oc get route -n "$TOOLBOX_NAMESPACE" beer-catalog -o jsonpath='{.spec.host}')"
```

Deploy the sample Red Hat Event API Backend (used by the subsequent usecases):

```sh
oc new-app -n "$TOOLBOX_NAMESPACE" -i openshift/nodejs:10 'https://github.com/nmasse-itix/rhte-api.git#085b015' --name=event-api
oc expose -n "$TOOLBOX_NAMESPACE" svc/event-api
```

Save the Event API hostname for later use:

```sh
export EVENT_API_HOSTNAME="$(oc get route -n "$TOOLBOX_NAMESPACE" event-api -o jsonpath='{.spec.host}')"
```

## Deploy APIcast instances

- Define your wildcard routes:

```sh
export APICAST_SELF_MANAGED_STAGING_WILDCARD_DOMAIN=saas-staging.$OPENSHIFT_ROUTER_SUFFIX
export APICAST_SELF_MANAGED_PRODUCTION_WILDCARD_DOMAIN=saas-production.$OPENSHIFT_ROUTER_SUFFIX
```

- Deploy APIcast instances (in the project of your choice) to be used with 3scale SaaS as self-managed instances:

```sh
oc create secret generic 3scale-tenant --from-literal=password=https://$SAAS_ACCESS_TOKEN@$SAAS_TENANT-admin.3scale.net
oc create -f https://raw.githubusercontent.com/3scale/apicast/v3.4.0/openshift/apicast-template.yml
oc new-app --template=3scale-gateway --name=apicast-staging -p CONFIGURATION_URL_SECRET=3scale-tenant -p CONFIGURATION_CACHE=0 -p RESPONSE_CODES=true -p LOG_LEVEL=info -p CONFIGURATION_LOADER=lazy -p APICAST_NAME=apicast-staging -p DEPLOYMENT_ENVIRONMENT=sandbox -p IMAGE_NAME=quay.io/3scale/apicast:v3.4.0
oc new-app --template=3scale-gateway --name=apicast-production -p CONFIGURATION_URL_SECRET=3scale-tenant -p CONFIGURATION_CACHE=60 -p RESPONSE_CODES=true -p LOG_LEVEL=info -p CONFIGURATION_LOADER=boot -p APICAST_NAME=apicast-production -p DEPLOYMENT_ENVIRONMENT=production -p IMAGE_NAME=quay.io/3scale/apicast:v3.4.0
oc scale dc/apicast-staging --replicas=1
oc scale dc/apicast-production --replicas=1
oc create route edge apicast-staging --service=apicast-staging --hostname="wildcard.$APICAST_SELF_MANAGED_STAGING_WILDCARD_DOMAIN" --insecure-policy=Allow --wildcard-policy=Subdomain
oc create route edge apicast-production --service=apicast-production --hostname="wildcard.$APICAST_SELF_MANAGED_PRODUCTION_WILDCARD_DOMAIN" --insecure-policy=Allow --wildcard-policy=Subdomain
```
