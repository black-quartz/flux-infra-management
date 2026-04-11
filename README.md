# flux-infra-management

GitOps repository for managing Kubernetes operators and cluster add-ons with FluxCD.

## Overview

> [!NOTE]
> This repository follows the [ControlPlane Enterprise for Flux CD]("https://fluxcd.control-plane.io/") reference architecture.
> The `d2` reference architecture comprised of [d2-fleet](https://github.com/controlplaneio-fluxcd/d2-fleet), [d2-infra](https://github.com/controlplaneio-fluxcd/d2-infra) and [d2-apps](https://github.com/controlplaneio-fluxcd/d2-apps) is a set of best practices and production-ready examples for using Flux Operator and OCI Artifacts to manage the continuous delivery of Kubernetes infrastructure and applications on multi-cluster multi-tenant environments.

This repository contains the configurations for services the extend the functionality capabilities of Kubernetes. 

This includes services such as:
- `cert-manager` — Automates the distribution and rotation of TLS certificates in the cluster.
- `envoy-gateway` — Implements the Kubernetes Gateway API to allow ingress to Kubernetes services from outside the cluster.
- `external-secrets-operator` — Synchronizes secrets from external secrets stores to Kubernetes.


> [!NOTE]
> By itself, this repo does not have the ability to deploy resources to Kubernetes, as Flux is considered to watch the [`flux-fleet-management`]("https://github.com/black-quartz/flux-fleet-management") repository for its reconciliation. Operators in must first be onboarded in the fleet management repository using the tenant vending pattern to be deployed to a Kubernetes cluster.

## Deployment Pattern

Because configuring operators often requires CRDs that don't exist until the operator is installed, a two-phase deployment pattern is required to keep the Flux installation from failing:

```shell
cert-manager/
├── configs
│   ├── base
│   │   ├── bundles.yml
│   │   ├── cluster-issuers.yml
│   │   └── kustomization.yml
│   └── production
│       └── kustomization.yml
└── controllers
    ├── base
    │   ├── cert-manager.yml
    │   └── kustomization.yml
    └── production
        └── kustomization.yml
```

### Controllers

The `controllers/` directory for each cluster component includes installation resources , such as the `HelmRepository` and `HelmRelease`. The installation manifest is included in the `base/` directory, and environment-specific directories patch the base installation as needed.

### Configs

The `configs/` directory contains configurations that are applied after the component installation (e.g., `ClusterIssuers` to be created after `cert-manager` is installed). The Flux `Kustomization` for this directory is structured as a dependency for the base installation, ensuring that these resources are only created once the initial component installation has succeeded.