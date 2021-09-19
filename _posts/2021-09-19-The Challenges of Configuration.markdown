# The challenges of configuration - featuring Azure App Configuration

## Intro

The configuration subject is never easy in a complex multi-domain application architecture. Here is a brief list of attempts to address this problem I have encountered so far:

-   (The most often default) local configuration file per project - like ARM template parameters
    -   Pros: it's obvious what is the configuration for a given application
    -   Cons: the full configuration picture is ambiguous and prone to contain redundant elements
-   One central configuration file with all possible project parameters crammed together on the deployment server
    -   Pros: all in one place
    -   Cons: unsecure and a hell to maintain
-   Shared code libraries with the configuration values only, pulled in build-time for the given environment
    -   Pros: supports advanced structure and clear versioning
    -   Cons: requires new build to apply changes
-   Shared database with the added feature to drop all updates in a JMS topic
    -   Pros: push updates, freedom of added attributes
    -   Cons: the schema and backup needs careful planning and hard to reach high security
-   A mixture of the above

These solutions intend to address different sides of the challenge to find the ideal configuration setup but they all have their shortcomings.

## The challenges of configuration

The following is a not comprehensive list of what we need to take into consideration when we design our configuration store.

The scope: the given configuration entry is only used in a single application/project or system-wide by multiple applications - like a shared database account on service bus connection string?

The update process: what does it take for our application to use an updated value - it matters especially when there are no additional (ex. code) changes. Can we update the value in the memory while the application is running, or do we have to restart/redeploy or even re-build our application?

The support and troubleshoot process: in case of an urgent incident, how long it takes and how complicated effort it is to have an accurate picture of the current state of the configuration of the application that we have a problem with.

Security: what is the access level of secure tokens or connection strings to resources that store sensitive information or mission critical on our platform?

Resilience: what happens when the configuration gets corrupted? Is falling back to a previous working state ensured? (Fun story: one of my previous companies' complete production environment broke because a dev accidentally committed an empty attribute in the routing record of the configuration database and it took days to even find the problem.)

Maintainability: is it user-proof - think of preventing duplicated values? Is it easy to find the section/value when we need to make an update?

## The ideal configuration

The following list is a condensed set of features of the ideal configuration store that take the previous section's points in account.

-   Single source of truth: all configuration for the entire platform is in one place.
-   Organized structure: support for multiple attributes for grouping and organizing the configuration entries.
-   Instant updates: provide a method to push configuration changes to the applications to update in runtime without service disruption.
-   Security: hide the values of sensitive values behind elevated permission levels.
-   Fault resilience: support version control and automated backups.
-   Accessibility: intuitive interface for humans to support easy search and update of values and straightforward API to integrate with various applications.

## Azure App Configuration

It's certainly not impossible to implement a custom configuration store that fulfills the requirements listed in the previous section but it can be a lengthy and complex project as almost each points would require a dedicated component tailored together to fit the purpose.

The App Configuration service is Azure's solution for the configuration challenge in the convenient package of a fully managed cloud service with added extras for increased fit for the use-case.

In the following sections we will briefly go through the features and see how they fulfill our requirements.

### Overview

Azure App Configuration is a fully managed service to centralize the settings of your applications separately from your code.

### Features

-   Structure:
    -   Namespaces for logical grouping
    -   Labels to create multiple versions of the same key with different values per label - think of different environments or staging/productions slots used in blue-green deployments
    -   Tags
-   Updates:
    -   Config cache with expiration - poll for changes
    -   Event grid - with some extra effort we can achieve instantaneous configuration update - sand it also helps to keep the transaction count down to avoid running high cost
-   Security:
    -   Data encryption at rest
    -   Key-vault integration - store Key-vault references instead of bare values and resolve the always latest version of the secret with each query
    -   Application access via managed identity
-   Fault resilience:
    -   Automated backups
    -   Replay/compare config state in a past point in time
-   Accessibility
    -   Via all Azure management options (Portal GUI, REST API, SDK, ARM, PS, CLI, ...)

The combined features of configuration labels and in-memory updates enables the use of blue-green deployments

Additional to configuration management, App Configuration also provides feature flag management to enable/disable features in running applications with the switch of a toggle and no downtime.

### Enabling factors for Blue-Green Deployments

Blue-green deployment is a technique that reduces downtime and risk by running two identical production environments called Blue and Green. At any time, only one of the environments is live, with the live environment serving all production traffic. Zero downtime deployment (and fall-back) can be achieved by deploying the new application version in the inactive instance then switch the traffic between instances. There is a great added benefit using this practice but the CI/CD automation is not trivial, especially if configuration is stored in a static way like environment variables.

Note: Azure has this option built into its App Services supporting to add up-to 5 slots per App Service.

Many of the Azure App Configuration features make it much easier to set-up Blue-Green Deployments:

-   Using App Configuration service, the configuration is owned and handled by the application code, therefore there is no external configuration dependency that we need to keep in sync with the code version during slot swaps. In fact all configuration synchronization can be completely kept out of the CI/CD process!
-   Configuration Labels allow to use the same config keys and filter the version we need in the initialization process
-   The Labels also allow to have "default' values by fetching the null label values first and overwrite them with the labeled values. This reduces redundancy when it's not necessary

## Closing thoughts

Based on my experience, the challenge of configuration is often underestimated. Moreover, due to the integral nature of the configuration, making changes in the practices or components later can be a huge effort in a production system.

Azure App Configuration is a comprehensive package of configuration storage features that fulfills the use-case of distributed architectures. It works well with containerized and serverless applications as well as CI/CD pipelines.

The benefits of a featureful configuration store as such can greatly contribute to the stability of an application system and smooth development and support process. It can completely eliminate the hunt for configuration values during urgent support incidents, with synchronized configuration there will be no more differences between config files and manually updated environment variables, and the option for instantaneous configuration changes across the whole system with is hard to overestimate.

## Author
Gergely Vajda 

Senior Integration Engineer
