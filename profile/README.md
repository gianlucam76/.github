# Sveltos: A Kubernetes Add-on Controller that Simplifies Add-on Management

[![Twitter URL](https://img.shields.io/twitter/url/https/twitter.com/projectsveltos.svg?style=social&label=Follow%20%40projectsveltos)](https://twitter.com/projectsveltos)
[![Slack](https://img.shields.io/badge/join%20slack-%23projectsveltos-brighteen)](https://join.slack.com/t/projectsveltos/shared_invite/zt-1hraownbr-W8NTs6LTimxLPB8Erj8Q6Q)

👋 Welcome to our project! Our [documentation](https://projectsveltos.github.io/sveltos/) can help you get started and provides lots of in-depth information.

## ✨ What is Project Sveltos?

Sveltos is a Kubernetes add-on controller that simplifies the deployment and management of add-ons and applications across multiple clusters. It runs in the management cluster and can programmatically deploy and manage add-ons and applications on any cluster in the fleet, including the management cluster itself. Sveltos supports a variety of add-on formats, including Helm charts (support for OCI registries), raw YAML, Kustomize, Carvel ytt, and Jsonnet.

<p align="center">
  <img alt="Sveltos Kubernetes add-ons management across clusters" src="https://projectsveltos.github.io/sveltos/assets/multi-clusters.png" width="600"/>
</p>

Sveltos allows you to represent add-ons and applications as templates. Before deploying to managed clusters, Sveltos instantiates these templates. Sveltos can gather the information required to instantiate the templates from either the management cluster or the managed clusters themselves. This enables you to use the same add-on configuration across all of your clusters, while still allowing for some variation, such as different add-on configuration values. In other words, Sveltos lets you define add-ons and applications in a reusable way. You can then deploy these definitions to multiple clusters, with minor adjustments as needed. This can save you a lot of time and effort, especially if you manage a large number of clusters.

Sveltos provides precise control over add-on deployment order. Add-ons within a Profile/ClusterProfile are deployed in the exact order they appear, ensuring a predictable and controlled rollout. Furthermore, ClusterProfiles can depend on others, guaranteeing that dependent add-ons only deploy after their dependencies are fully operational. Finally Sveltos' event-driven framework offers additional flexibility. This framework allows for deploying add-ons and applications in response to specific events, enabling dynamic and adaptable deployments based on your needs.

👉 If you like Sveltos or to get updates, [⭐️ star](https://github.com/projectsveltos/addon-controller/stargazers) Sveltos.

## Cluster Management: Profiles vs. ClusterProfiles

Projectsveltos offers two powerful tools for managing cluster configurations: **Profiles** and **ClusterProfiles**:

1. ClusterProfiles: Apply across all clusters in any namespace. Ideal for platform admins maintaining global consistency and managing settings like networking, security, and resource allocation.
2. Profiles: Limited to a specific namespace, granting granular control to tenant admins. This isolation ensures teams manage, from the management cluster, their managed clusters independently without impacting others.

### Use Cases:

1. ClusterProfiles:
    - Enforce standardized configurations across all clusters.
    - Define global policies for networking, security, and resource allocation.

2. Profiles:
    - Tailor configurations for specific applications, services, or teams.
    - Grant tenant admins granular control over their clusters.

<p align="center">
  <img alt="Sveltos Profile vs ClusterProfile" src="https://projectsveltos.github.io/sveltos/assets/profile_vs_clusterprofile.png" width="600"/>
</p>

## Add-ons deployment

1. from the management cluster, selects one or more `clusters` with a Kubernetes [label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors);
2. lists which Kubernetes add-ons need to be deployed on such clusters;
3. add-ons can be expressed as templates and instantiated by Sveltos at deployment time using resources from the management cluster.

<p align="center">
  <img alt="Kubernetes add-on deployment" src="https://github.com/projectsveltos/sveltos/blob/main/docs/assets/addons_deployment.gif" width="600"/>
</p>

## Different SyncMode

1️⃣ *OneTime*: This mode is designed for bootstrapping critical components during the initial cluster setup. Think of it as a one-shot configuration injection:
    1. Deploying essential infrastructure components like CNI plugins, cloud controllers, or the workload cluster's package manager itself;
    2. Simplifies initial cluster setup;
    3. Hands over management to the workload cluster's own tools, promoting modularity and potentially simplifying ongoing maintenance.
    
2️⃣ *Continuous*: This mode continuously monitors ClusterProfiles or Profiles for changes and automatically applies them to matching clusters. It ensures ongoing consistency between your desired configuration and the actual cluster state: 
    1. Centralized control over deployments across multiple clusters for consistency and compliance;
    2. Simplifies management of configurations across multiple clusters.
    
3️⃣ *ContinuousWithDriftDetection*: Detects and automatically corrects configuration drifts in managed clusters, ensuring they remain aligned with the desired state defined in the management cluster.

## Add-on rollout strategy
With the rollout strategy defined in the ClusterProfile/Profile, users can control the upgrade behavior of the addon when there are changes in the supported configurations.

For example, the add-on user updates the “kyverno” ClusterProfile and wants to apply the change to a “canary” decision group of clusters first. If all the add-on upgrade successfully, then upgrade the rest of clusters progressively per cluster at a rate of 30% (*__ maxUpdate: 30%__). The rollout strategy can be defined as follows:

```yaml
apiVersion: config.projectsveltos.io/v1alpha1
kind: ClusterProfile
metadata:
  name: kyverno
spec:
  clusterSelector: env=fv
  syncMode: Continuous
  maxUpdate: 30%
  helmCharts:
  - repositoryURL:    https://kyverno.github.io/kyverno/
    repositoryName:   kyverno
    chartName:        kyverno/kyverno
    chartVersion:     v3.0.1
    releaseName:      kyverno-latest
    releaseNamespace: kyverno
    helmChartAction:  Install
```

## Configuration Drift Detection

Sveltos can automatically detect drift between the desired state, defined in the management cluster, and actual state of your clusters and recover from it.

<p align="center">
  <img alt="Configuration drift recovery" src="https://github.com/projectsveltos/demos/blob/main/configuration_drift/reconcile_configuration_drift.gif" width="600"/>
</p>

## Automatic Rolling Upgrades

Sveltos has the capability to monitor changes within ConfigMap and Secret resources and facilitate rolling upgrades for Deployments, StatefulSets, and DaemonSets. This functionality can be activated by simply setting the reloader field to true in the ClusterProfile.

<p align="center">
  <img alt="Projectsveltos: Rolling Upgrades" src="https://github.com/projectsveltos/sveltos/blob/main/docs/assets/rolling_upgrades.gif" width="600"/>
</p>

## Coordinate with Crossplane and other open source projects

Sveltos can also create resources in the management cluster itself. This allows Sveltos to coordinate with other open source projects
before deploying add-ons in the managed cluster.

<p align="center">
  <img alt="ClusterAPI, Sveltos and Crossplane" src="https://github.com/projectsveltos/sveltos/raw/main/docs/assets/sveltos_clusterapi_crossplane.gif" width="600"/>
</p>

## External Secret Management

The integration of External Secret Operator and Sveltos provides a powerful solution for secret management. External Secret Operator fetches secrets from external APIs and creates Kubernetes secrets, while Sveltos efficiently distributes these fetched secrets to the managed clusters. In case of any changes to the secrets in the external API, External Secret Operator updates the secrets in the management cluster, and Sveltos ensures the reconciliation of state in each managed cluster where the secret was distributed.

<p align="center">
  <img alt="External Secrets Operator and Sveltos integration" src="https://github.com/projectsveltos/sveltos/blob/main/docs/assets/external_secret.gif" width="600"/>
</p>

## Event driven framework

Sveltos supports defining an event using Lua. An event is a notification that is sent when a certain condition is met. For example, you could create an event that is sent when the PostgreSQL deployment becomes healthy. Events can then be used to trigger the deployment of other resources. For example, you could configure Sveltos to deploy the Job that creates the table in the database when it receives an event that the PostgreSQL deployment is healthy.
In this example Sveltos has been instructed to:

1️⃣ Deploy postgresql deployment and service\
2️⃣ Wait for postgresql deployment to be ready\
3️⃣ Deploy a Job that creates a table in the DB\
4️⃣ Wait for Job to be completed\
5️⃣ Deploy todo-app which can access PostgreSQL deployment\
6️⃣ Wait for todo-app to be healthy\
7️⃣ Deploy a Job that adds an entry to database via todo-app

<p align="center">
  <img alt="Event driven framework" src="https://github.com/projectsveltos/sveltos/raw/main/docs/assets/sveltos_resource_order.gif" width="600"/>
</p>

## Cluster classification

Sveltos Classifier is an optional component used to dynamically classify a cluster based on its runtime configuration (Kubernetes version, deployed resources, and more).

Classifier currently supports the following criteria:

1. Kubernetes version
2. Kubernetes resources

<p align="center">
  <img alt="Kubernetes cluster classification" src="https://github.com/projectsveltos/sveltos/blob/main/docs/assets/classifier.gif" width="600"/>
</p>

## Observability
Sveltos can monitor the healths of resources in managed clusters and send notifications when something happens. For instance detect Pod instances in crashloopbackoff and send a Slack notification.

<p align="center">
  <img alt="Detect Pods in crashloopbackoff" src="https://github.com/projectsveltos/sveltos/blob/main/docs/assets/notification.gif" width="600"/>
</p>

## Visualize managed cluster resources from central location

Sveltos now offers the ability to gather information from all or subsets of the clusters it manages. This information can then be accessed and displayed using Sveltos' CLI in the management cluster.

<p align="center">
  <img alt="Sveltosctl show resources" src="https://github.com/projectsveltos/sveltos/blob/main/docs/assets/show_resources.png" width="600"/>
</p>

## Horizontal Scaling

With its sharding strategy, Sveltos can manage hundreds of managed clusters and applications by distributing the load across multiple instances of Sveltos controllers. To achieve this, add the annotation sharding.projectsveltos.io/key to managed clusters.

<p align="center">
  <img alt="Sveltos sharding" src="https://github.com/projectsveltos/sveltos/blob/main/docs/assets/sharding.gif" width="600"/>
</p>


## Getting Started

* [Install Sveltos](https://projectsveltos.github.io/sveltos/install/install/)
* [Quickstart](https://projectsveltos.github.io/sveltos/install/quick_start/) for trying out Projectsveltos with a test cluster

## Documentation

* [Complete documentation](https://projectsveltos.github.io/sveltos/)

## Branching model

We use the git-flow branching model. The base branch is dev. If you are looking for a stable version, please use the main branch or tags labeled as v0.x.x.

## 🤗 Contributing to Sveltos

We love to hear from our community!

* Report bugs and suggest features
* Write documentation
* Submit code

## Contact

* <img src="https://github.com/projectsveltos/.github/blob/main/docs/slack_logo.png" alt="Slack" width="25" /> [Slack](https://projectsveltos.slack.com/)
* <img src="https://github.com/projectsveltos/.github/blob/main/docs/email_logo.png" alt="Email" width="25" /> [Email](mailto:hello@projectsveltos.io)
* <img src="https://github.com/projectsveltos/.github/blob/main/docs/twitter_logo.png" alt="Twitter" width="25" /> [Twitter](https://twitter.com/projectsveltos)

## License

Sveltos is licensed under the Apache License, Version 2.0.

If you like Sveltos, please [star](https://github.com/projectsveltos/addon-controller) [:star:](https://github.com/projectsveltos/addon-controller) the project on GitHub! This will help other people find it and learn more about it.


