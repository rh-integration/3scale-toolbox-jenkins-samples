# Code samples using the 3scale toolbox Jenkins shared library

This repository holds code samples to showcase the use of the [3scale toolbox Jenkins shared library](https://github.com/rh-integration/3scale-toolbox-jenkins) to automate the delivery of APIs using CI/CD and more specifically Jenkins pipelines.

[Full API Lifecycle Management](https://developers.redhat.com/blog/2019/02/25/full-api-lifecycle-management-a-primer/) using the 3scale toolbox Jenkins shared library is showcased in this repository: [IntegrationApp-Automation](https://github.com/rh-integration/IntegrationApp-Automation).

## Usecases

Five usecases will be showcased, from the simpler one (API Key on 3scale SaaS), to the most complete one (multi-environment, semantic versioning).

| Usecase                                         | Security            | Target                           | Notes               |
|-------------------------------------------------|---------------------|----------------------------------|---------------------|
| [SaaS - API Key](saas-usecase-apikey/)          | API Key             | SaaS                             | -                   |
| [Hybrid - Open](hybrid-usecase-open/)           | Open                | Self-Managed + on-premises       | URL rewriting       |
| [Hybrid - OIDC](hybrid-usecase-oidc/)           | OpenID Connect      | Self-Managed + on-premises       | URL rewriting       |
| [Multi-environment](multi-environment-usecase/) | API Key             | 3 envs on 1 tenant, self-managed | -                   |
| [Semantic versioning](semver-usecase/)          | Open, API Key, OIDC | 3 envs on 1 tenant, self-managed | Semantic Versioning |

## Setup

Before you can deploy the provided pipelines, you will need to setup your environment accordingly.

**Follow the [SETUP guide](SETUP.md).**
