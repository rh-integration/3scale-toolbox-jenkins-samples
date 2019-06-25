#!groovy

library identifier: '3scale-toolbox-jenkins@master',
        retriever: modernSCM([$class: 'GitSCMSource',
                              remote: 'https://github.com/rh-integration/3scale-toolbox-jenkins.git',
                              traits: [[$class: 'jenkins.plugins.git.traits.BranchDiscoveryTrait']]])

def service = null

node() {
  stage('Checkout Source') {
    checkout scm
  }

  stage("Deploy API in Dev") {
    // Prepare
    service = toolbox.prepareThreescaleService(
        openapi: [filename: "multi-environment-usecase/swagger.yaml"],
        environment: [ baseSystemName: "multi_environment_usecase",
                       publicBasePath: "/api/",
                       environmentName: "dev",
                       publicStagingWildcardDomain: params.PUBLIC_STAGING_WILDCARD_DOMAIN != "" ? params.PUBLIC_STAGING_WILDCARD_DOMAIN : null,
                       publicProductionWildcardDomain: params.PUBLIC_PRODUCTION_WILDCARD_DOMAIN != "" ? params.PUBLIC_PRODUCTION_WILDCARD_DOMAIN : null,
                       privateBaseUrl: params.PRIVATE_BASE_URL ],
        toolbox: [ openshiftProject: params.NAMESPACE,
                   destination: params.TARGET_INSTANCE,
                   image: "quay.io/redhat/3scale-toolbox:master", // TODO: remove me once the final image is released
                   insecure: params.DISABLE_TLS_VALIDATION == "yes",
                   secretName: params.SECRET_NAME],
        service: [:],
        applications: [
            [ name: "my-test-app", description: "This is used for tests", plan: "test", account: params.DEVELOPER_ACCOUNT_ID ]
        ],
        applicationPlans: [
          [ systemName: "test", name: "Test", defaultPlan: true, published: true ]
        ]
    )

    // Import OpenAPI
    service.importOpenAPI()
    echo "Service with system_name ${service.environment.targetSystemName} created !"

    // Create an Application Plan
    service.applyApplicationPlans()

    // Create an Application
    service.applyApplication()

    // Run integration tests
    // To run the integration tests when using APIcast SaaS instances, we need
    // to fetch the proxy definition to extract the staging public url
    def proxy = service.readProxy("sandbox")
    def userkey = service.applications[0].userkey
    sh """set -e
    echo "Public Staging Base URL is ${proxy.sandbox_endpoint}"
    echo "userkey is ${userkey}"
    curl -sfk -w "GetLocation: %{http_code}\n" -o /dev/null ${proxy.sandbox_endpoint}/api/location -H 'api-key: ${userkey}'
    curl -sfk -w "GetTimeframe: %{http_code}\n" -o /dev/null ${proxy.sandbox_endpoint}/api/timeframe -H 'api-key: ${userkey}'
    curl -sfk -w "GetParticipants: %{http_code}\n" -o /dev/null ${proxy.sandbox_endpoint}/api/participants -H 'api-key: ${userkey}'
    """

    // Promote to production
    service.promoteToProduction()
  }

  stage("Deploy API in Test") {
    // Prepare
    service = toolbox.prepareThreescaleService(
        openapi: [filename: "multi-environment-usecase/swagger.yaml"],
        environment: [ baseSystemName: "multi_environment_usecase",
                       publicBasePath: "/api/",
                       environmentName: "test",
                       publicStagingWildcardDomain: params.PUBLIC_STAGING_WILDCARD_DOMAIN != "" ? params.PUBLIC_STAGING_WILDCARD_DOMAIN : null,
                       publicProductionWildcardDomain: params.PUBLIC_PRODUCTION_WILDCARD_DOMAIN != "" ? params.PUBLIC_PRODUCTION_WILDCARD_DOMAIN : null,
                       privateBaseUrl: params.PRIVATE_BASE_URL ],
        toolbox: [ openshiftProject: params.NAMESPACE,
                   destination: params.TARGET_INSTANCE,
                   image: "quay.io/redhat/3scale-toolbox:master", // TODO: remove me once the final image is released
                   insecure: params.DISABLE_TLS_VALIDATION == "yes",
                   secretName: params.SECRET_NAME],
        service: [:],
        applications: [
            [ name: "my-test-app", description: "This is used for tests", plan: "test", account: params.DEVELOPER_ACCOUNT_ID ]
        ],
        applicationPlans: [
          [ systemName: "test", name: "Test", defaultPlan: true, published: true ]
        ]
    )

    // Import OpenAPI
    service.importOpenAPI()
    echo "Service with system_name ${service.environment.targetSystemName} created !"

    // Create an Application Plan
    service.applyApplicationPlans()

    // Create an Application
    service.applyApplication()

    // Run integration tests
    // To run the integration tests when using APIcast SaaS instances, we need
    // to fetch the proxy definition to extract the staging public url
    def proxy = service.readProxy("sandbox")
    def userkey = service.applications[0].userkey
    sh """set -e
    echo "Public Staging Base URL is ${proxy.sandbox_endpoint}"
    echo "userkey is ${userkey}"
    curl -sfk -w "GetLocation: %{http_code}\n" -o /dev/null ${proxy.sandbox_endpoint}/api/location -H 'api-key: ${userkey}'
    curl -sfk -w "GetTimeframe: %{http_code}\n" -o /dev/null ${proxy.sandbox_endpoint}/api/timeframe -H 'api-key: ${userkey}'
    curl -sfk -w "GetParticipants: %{http_code}\n" -o /dev/null ${proxy.sandbox_endpoint}/api/participants -H 'api-key: ${userkey}'
    """

    // Promote to production
    service.promoteToProduction()
  }

  stage("Deploy API in Prod") {
    // Prepare
    service = toolbox.prepareThreescaleService(
        openapi: [filename: "multi-environment-usecase/swagger.yaml"],
        environment: [ baseSystemName: "multi_environment_usecase",
                       publicBasePath: "/api/",
                       environmentName: "prod",
                       publicStagingWildcardDomain: params.PUBLIC_STAGING_WILDCARD_DOMAIN != "" ? params.PUBLIC_STAGING_WILDCARD_DOMAIN : null,
                       publicProductionWildcardDomain: params.PUBLIC_PRODUCTION_WILDCARD_DOMAIN != "" ? params.PUBLIC_PRODUCTION_WILDCARD_DOMAIN : null,
                       privateBaseUrl: params.PRIVATE_BASE_URL ],
        toolbox: [ openshiftProject: params.NAMESPACE,
                   destination: params.TARGET_INSTANCE,
                   image: "quay.io/redhat/3scale-toolbox:master", // TODO: remove me once the final image is released
                   insecure: params.DISABLE_TLS_VALIDATION == "yes",
                   secretName: params.SECRET_NAME],
        service: [:],
        applications: [
            [ name: "my-test-app", description: "This is used for tests", plan: "test", account: params.DEVELOPER_ACCOUNT_ID ]
        ],
        applicationPlans: [
          [ systemName: "test", name: "Test", defaultPlan: true, published: true ]
        ]
    )

    // Import OpenAPI
    service.importOpenAPI()
    echo "Service with system_name ${service.environment.targetSystemName} created !"

    // Create an Application Plan
    service.applyApplicationPlans()

    // Create an Application
    service.applyApplication()

    // Run integration tests
    // To run the integration tests when using APIcast SaaS instances, we need
    // to fetch the proxy definition to extract the staging public url
    def proxy = service.readProxy("sandbox")
    def userkey = service.applications[0].userkey
    sh """set -e
    echo "Public Staging Base URL is ${proxy.sandbox_endpoint}"
    echo "userkey is ${userkey}"
    curl -sfk -w "GetLocation: %{http_code}\n" -o /dev/null ${proxy.sandbox_endpoint}/api/location -H 'api-key: ${userkey}'
    curl -sfk -w "GetTimeframe: %{http_code}\n" -o /dev/null ${proxy.sandbox_endpoint}/api/timeframe -H 'api-key: ${userkey}'
    curl -sfk -w "GetParticipants: %{http_code}\n" -o /dev/null ${proxy.sandbox_endpoint}/api/participants -H 'api-key: ${userkey}'
    """

    // Promote to production
    service.promoteToProduction()
  }

}
