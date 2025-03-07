# RBAC Manager

[RBAC Manager](https://fairwindsops.github.io/rbac-manager/) was designed to simplify authorization in Kubernetes. This is an operator that supports declarative configuration for RBAC with new custom resources. Instead of managing role bindings or service accounts directly, you can specify a desired state and RBAC Manager will make the necessary changes to achieve that state.

This project has three main goals:

1. Provide a declarative approach to RBAC that is more approachable and scalable.
2. Reduce the amount of configuration required for great auth.
3. Enable automation of RBAC configuration updates with CI/CD.

More information about RBAC Manager is available on [GitHub](https://github.com/FairwindsOps/rbac-manager) as well as from the [official documentation](https://fairwindsops.github.io/rbac-manager/).

## Installation

We recommend installing rbac-manager in its own namespace and a simple release name:

```
helm repo add fairwinds-stable https://charts.fairwinds.com/stable
helm install rbac-manager fairwinds-stable/rbac-manager --namespace rbac-manager
```

## Prerequisites

As of chart version 1.6.0 Kubernetes 1.16+, Helm 2.10+

Helm 3 will be made mandatory in the future.

## Upgrading to Chart Version 1.0.0

The upgrade to version 1.0.0 of this chart removes support for installing RBAC Definitions as part of the chart values. This change was made to simplify CRD installation with Helm. We recommend installing RBAC Definitions separately from the chart.

For backwards compatibility with the chart originally included in the rbac-manager repository, we've removed the Helm `install-crd` hook from this chart. Unfortunately as part of improving backwards compatibility with the chart in the rbac-manager repository, we have made it more difficult to upgrade from the inital versions of the charts here.

Some quirks in Helm make the upgrade process from 0.x of this chart to 1.x challenging due to the potential of the RBAC Definition CRD getting deleted. In most cases, reinstalling the chart will be the best path forward.

If either of the following apply and you are upgrading from an earlier version of the chart found in this repository, keep on reading:

1. A momentary lapse in access granted by RBAC Definitions is unacceptable
2. You're using auth tokens from Service Accounts created by RBAC Manager

The following process has worked repeatedly for us to upgrade from an older version of this chart to 1.0.0. These steps worked with Helm and Tiller 2.12.3 for us, but due to the absurdity of this process, we can't guarantee it will work for you.

1. Install rbac-manager with chart that uses install-crd hook (fairwinds-stable/rbac-manager@0.2.1)
2. Upgrade to rbac-manager chart that doesn't use install-crd hook (fairwinds-stable/rbac-manager@1.0.0) - this upgrade fails but is important later
3. Upgrade to original rbac-manager chart that uses install-crd hook (fairwinds-stable/rbac-manager@0.2.1) - this works
4. Rollback to revision 2 - this fails
5. Rollback to revision 2 - this works

In the above workflow, an RBAC Definition installed between revision 1 and 2 should persist through to revision 5. This process is admittedly quite strange, and in our testing the second rollback (step 5) is indeed required for this process to work.

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| image.repository | string | `"quay.io/reactiveops/rbac-manager"` | The image to run for rbac manager |
| image.tag | string | `"v1.4.2"` | The tag of the image to run |
| image.digest | string | `""` | The digest of the image to run |
| image.pullPolicy | string | `"Always"` | The image pullPolicy. Recommend not changing this |
| image.imagePullSecrets | list | `[]` |  |
| extraArgs | object | `{}` | A map of flag=value to pass to rbac-manager |
| installCRDs | bool | `true` | If true, install and upgrade CRDs. See the Helm documentation for [best practices regarding CRDs](https://helm.sh/docs/chart_best_practices/custom_resource_definitions/#install-a-crd-declaration-before-using-the-resource). |
| crds.additionalLabels | object | `{}` | add additional labels to the installed CRDs |
| rbac.additionalLabels | object | `{}` | add additional labels to the installed RBAC resources |
| resources | object | `{"limits":{"cpu":"100m","memory":"128Mi"},"requests":{"cpu":"100m","memory":"128Mi"}}` | A resources block for the rbac-manager pods |
| priorityClassName | string | `""` | The name of a priorityClass to use |
| nodeSelector | object | `{}` | Deployment nodeSelector |
| tolerations | list | `[]` | Deployment tolerations |
| affinity | object | `{}` | Deployment affinity |
| podAnnotations | object | `{}` | Annotations to apply to the pods |
| podLabels | object | `{}` | Labels to apply to the pod |
| podSecurityContext | object | `{}` | securityContext to apply to the whole pod |
| securityContext | object | `{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]},"privileged":false,"readOnlyRootFilesystem":true,"runAsNonRoot":true}` | securityContext to apply to the rbac-manager container |
| deploymentLabels | object | `{}` | Labels to apply to the Deployment resource |
| serviceMonitor.enabled | bool | `false` | If true, a ServiceMonitor will be created for Prometheus |
| serviceMonitor.additionalLabels | list | `[]` | Additional labels to ServiceMonitor |
| serviceMonitor.annotations | object | `{}` | Annotations to apply to the serviceMonitor and headless service |
| serviceMonitor.namespace | string | `""` | The namespace to deploy the serviceMonitor into |
| serviceMonitor.interval | string | `"60s"` | How often to scrape the metrics endpoint |
