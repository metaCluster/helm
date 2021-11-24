# Istio Operator

[Istio](https://istio.io/) is a service mesh that lets you connect, secure, control, and observe services. At a high level, _Istio_ helps reduce the complexity of these deployments, and eases the strain on your development teams. It is a completely open source service mesh that layers transparently onto existing distributed applications. It is also a platform, including APIs that let it integrate into any logging platform, or telemetry or policy system. _Istio's_ diverse feature set lets you successfully, and efficiently, run a distributed microservice architecture, and provides a uniform way to secure, connect, and monitor microservices.

## Important changes to consider when upgrading to Istio 1.12.0.

When you upgrade from Istio 1.10.0 or 1.11.0 to Istio 1.12.0, you need to consider the changes on this page. These notes detail the changes which purposefully break backwards compatibility with Istio 1.10.0 and 1.11.0. The notes also mention changes which preserve backwards compatibility while introducing new behavior. Changes are only included if the new behavior would be unexpected to a user of Istio 1.12.0.

### TCP probes now working as expected

When using TCP probes with older versions of Istio, the check was always successful. TCP probes simply check the port will accept a connection, and because all traffic is first redirected to the Istio sidecar, the sidecar will always accept the connection. In Istio 1.12, this issue is resolved by using the same mechanism used for HTTP probes. As a result, TCP probes in 1.12+ will start to properly check the health of the configured port. If your probes previously would have failed, they may now start failing unexpectedly. This change can be disabled temporarily by setting the REWRITE_TCP_PROBES=false environment variable in the Istiod deployment. The entire probe rewrite feature (HTTP and TCP) can also be disabled.

### Default revision must be switched when performing a revision-based upgrade

When installing a new Istio control plane revision the previous resource validator will remain unchanged to prevent unintended effects on the existing, stable revision. Once prepared to migrate over to the new control plane revision, cluster operators should switch the default revision. This can be done through istioctl tag set default --revision <new revision>, or if using a Helm-based flow, helm upgrade istio-base manifests/charts/base -n istio-system --set defaultRevision=<new revision>.

## Installing the Chart

Before you can install the chart you will need to add the `metakube` repo to [Helm](https://helm.sh/).

```shell
helm repo add metakube https://metacluster.github.io/helm
```

After you've installed the repo you can install the chart.

```shell
helm upgrade --install --namespace default --values ./my-values.yaml my-release metakube/istio-operator
```

## Configuration

The following table lists the configurable parameters of the _Istio Operator_ chart and their default values.

| Parameter                         | Description                                                                                                                                      | Default                    |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------- |
| `image.repository`                | Image repository.                                                                                                                                | `docker.io/istio/operator` |
| `image.tag`                       | Image tag, will override the default tag derived from the chart app version.                                                                     | `""`                       |
| `image.pullPolicy`                | Image pull policy.                                                                                                                               | `IfNotPresent`             |
| `image.pullSecrets`               | Image pull secrets.                                                                                                                              | `[]`                       |
| `nameOverride`                    | Override the `name` of the chart.                                                                                                                | `nil`                      |
| `fullnameOverride`                | Override the `fullname` of the chart.                                                                                                            | `nil`                      |
| `serviceAccount.create`           | If `true`, create a new service account.                                                                                                         | `true`                     |
| `serviceAccount.annotations`      | Annotations to add to the service account.                                                                                                       | `{}`                       |
| `serviceAccount.name`             | Service account to be used. If not set and `serviceAccount.create` is `true`, a name is generated using the full name template.                  | `nil`                      |
| `rbac.create`                     | If `true`, create a cluster role and a cluster role binding.                                                                                     | `true`                     |
| `podLabels`                       | Labels to add to the pod.                                                                                                                        | `{}`                       |
| `podAnnotations`                  | Annotations to add to the pod.                                                                                                                   | `{}`                       |
| `podSecurityContext`              | Security context for the pod.                                                                                                                    | `{}`                       |
| `securityContext`                 | Security context for the _istio-operator_ container.                                                                                             | `{}`                       |
| `priorityClassName`               | Priority class name to use.                                                                                                                      | `""`                       |
| `serviceMonitor.enabled`          | If `true`, create a _Prometheus_ service monitor.                                                                                                | `false`                    |
| `serviceMonitor.additionalLabels` | Additional labels to be set on the service monitor.                                                                                              | `{}`                       |
| `serviceMonitor.interval`         | _Prometheus_ scrape frequency.                                                                                                                   | `1m`                       |
| `dashboards.enabled`              | If _Grafana_ dashboards should be installed for the sidecar to pick up and apply.                                                                | `false`                    |
| `resources`                       | Resource requests and limits for the _istio-operator_ container.                                                                                 | `nil`                      |
| `nodeSelector`                    | Node labels for pod assignment.                                                                                                                  | `{}`                       |
| `tolerations`                     | Tolerations for pod assignment.                                                                                                                  | `[]`                       |
| `affinity`                        | Affinity for pod assignment.                                                                                                                     | `{}`                       |
| `istioNamespace`                  | Namespace to install _Istio_ into.                                                                                                               | `istio-system`             |
| `controlPlane.install`            | If `true`, install the _Istio_ control plane accoring to the `controlPlane.spec`.                                                                | `false`                    |
| `controlPlane.spec`               | The [Istio Operator Spec](https://istio.io/latest/docs/reference/config/istio.operator.v1alpha1/#IstioOperatorSpec) to deploy the control plane. | `{}`                       |