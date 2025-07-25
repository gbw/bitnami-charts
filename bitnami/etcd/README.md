<!--- app-name: Etcd -->

# Bitnami package for Etcd

etcd is a distributed key-value store designed to securely store data across a cluster. etcd is widely used in production on account of its reliability, fault-tolerance and ease of use.

[Overview of Etcd](https://etcd.io/)

Trademarks: This software listing is packaged by Bitnami. The respective trademarks mentioned in the offering are owned by the respective companies, and use of them does not imply any affiliation or endorsement.

## TL;DR

```console
helm install my-release oci://registry-1.docker.io/bitnamicharts/etcd
```

Looking to use Etcd in production? Try [VMware Tanzu Application Catalog](https://bitnami.com/enterprise), the commercial edition of the Bitnami catalog.

## ⚠️ Important Notice: Upcoming changes to the Bitnami Catalog

Beginning August 28th, 2025, Bitnami will evolve its public catalog to offer a curated set of hardened, security-focused images under the new [Bitnami Secure Images initiative](https://news.broadcom.com/app-dev/broadcom-introduces-bitnami-secure-images-for-production-ready-containerized-applications). As part of this transition:

- Granting community users access for the first time to security-optimized versions of popular container images.
- Bitnami will begin deprecating support for non-hardened, Debian-based software images in its free tier and will gradually remove non-latest tags from the public catalog. As a result, community users will have access to a reduced number of hardened images. These images are published only under the “latest” tag and are intended for development purposes
- Starting August 28th, over two weeks, all existing container images, including older or versioned tags (e.g., 2.50.0, 10.6), will be migrated from the public catalog (docker.io/bitnami) to the “Bitnami Legacy” repository (docker.io/bitnamilegacy), where they will no longer receive updates.
- For production workloads and long-term support, users are encouraged to adopt Bitnami Secure Images, which include hardened containers, smaller attack surfaces, CVE transparency (via VEX/KEV), SBOMs, and enterprise support.

These changes aim to improve the security posture of all Bitnami users by promoting best practices for software supply chain integrity and up-to-date deployments. For more details, visit the [Bitnami Secure Images announcement](https://github.com/bitnami/containers/issues/83267).

## Introduction

This chart bootstraps a [etcd](https://github.com/bitnami/containers/tree/main/bitnami/etcd) deployment on a [Kubernetes](https://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.23+
- Helm 3.8.0+
- PV provisioner support in the underlying infrastructure

## Installing the Chart

To install the chart with the release name `my-release`:

```console
helm install my-release oci://REGISTRY_NAME/REPOSITORY_NAME/etcd
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.

These commands deploy etcd on the Kubernetes cluster in the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Configuration and installation details

### Resource requests and limits

Bitnami charts allow setting resource requests and limits for all containers inside the chart deployment. These are inside the `resources` value (check parameter table). Setting requests is essential for production workloads and these should be adapted to your specific use case.

To make this process easier, the chart contains the `resourcesPreset` values, which automatically sets the `resources` section according to different presets. Check these presets in [the bitnami/common chart](https://github.com/bitnami/charts/blob/main/bitnami/common/templates/_resources.tpl#L15). However, in production workloads using `resourcesPreset` is discouraged as it may not fully adapt to your specific needs. Find more information on container resource management in the [official Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/).

### [Rolling VS Immutable tags](https://techdocs.broadcom.com/us/en/vmware-tanzu/application-catalog/tanzu-application-catalog/services/tac-doc/apps-tutorials-understand-rolling-tags-containers-index.html)

It is strongly recommended to use immutable tags in a production environment. This ensures your deployment does not change automatically if the same tag is updated with a different image.

Bitnami will release a new chart updating its containers if a new version of the main container, significant changes, or critical vulnerabilities exist.

### Prometheus metrics

This chart can be integrated with Prometheus by setting `metrics.enabled` to true. This will expose the etcd native Prometheus port in the container and service (if `metrics.useSeparateEndpoint=true`). It will all have the necessary annotations to be automatically scraped by Prometheus.

#### Prometheus requirements

It is necessary to have a working installation of Prometheus or Prometheus Operator for the integration to work. Install the [Bitnami Prometheus helm chart](https://github.com/bitnami/charts/tree/main/bitnami/prometheus) or the [Bitnami Kube Prometheus helm chart](https://github.com/bitnami/charts/tree/main/bitnami/kube-prometheus) to easily have a working Prometheus in your cluster.

#### Integration with Prometheus Operator

The chart can deploy `PodMonitor` objects for integration with Prometheus Operator installations. To do so, set the value `*.metrics.podMonitor.enabled=true`. Ensure that the Prometheus Operator `CustomResourceDefinitions` are installed in the cluster or it will fail with the following error:

```text
no matches for kind "PodMonitor" in version "monitoring.coreos.com/v1"
```

Install the [Bitnami Kube Prometheus helm chart](https://github.com/bitnami/charts/tree/main/bitnami/kube-prometheus) for having the necessary CRDs and the Prometheus Operator.

### Update credentials

Bitnami charts configure credentials at first boot. Any further change in the secrets or credentials require manual intervention. Follow these instructions:

- Update the user password following [the upstream documentation](https://etcd.io/docs/latest/op-guide/authentication/)
- Update the password secret with the new values (replace the SECRET_NAME and PASSWORD placeholders)

```shell
kubectl create secret generic SECRET_NAME --from-literal=etcd-root-password=PASSWORD --dry-run -o yaml | kubectl apply -f -
```

### Cluster configuration

The Bitnami etcd chart can be used to bootstrap an etcd cluster, easy to scale and with available features to implement disaster recovery. It uses static discovery configured via environment variables to bootstrap the etcd cluster. Based on the number of initial replicas, and using the A records added to the DNS configuration by the headless service, the chart can calculate every advertised peer URL.

The chart makes use of some extra elements offered by Kubernetes to ensure the bootstrapping is successful:

- It sets a "Parallel" Pod Management Policy. This is critical, since all the etcd replicas should be created simultaneously to guarantee they can find each other.
- It records "not ready" pods in the DNS, so etcd replicas are reachable using their associated FQDN before they're actually ready.

Learn more about [etcd discovery](https://etcd.io/docs/current/op-guide/clustering/#discovery), [Pod Management Policies](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies) and [recording "not ready" pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pod-s-hostname-and-subdomain-fields).

Here is an example of the environment configuration bootstrapping an etcd cluster with 3 replicas:

| Member  | Variable                         | Value                                                                                                                                                                                                 |
|---------|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0       | ETCD_NAME                        | etcd-0                                                                                                                                                                                                |
| 0       | ETCD_INITIAL_ADVERTISE_PEER_URLS | <http://etcd-0.etcd-headless.default.svc.cluster.local:2380>                                                                                                                                            |
|---------|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1       | ETCD_NAME                        | etcd-1                                                                                                                                                                                                |
| 1       | ETCD_INITIAL_ADVERTISE_PEER_URLS | <http://etcd-1.etcd-headless.default.svc.cluster.local:2380>                                                                                                                                            |
|---------|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2       | ETCD_NAME                        | etcd-2                                                                                                                                                                                                |
| 2       | ETCD_INITIAL_ADVERTISE_PEER_URLS | <http://etcd-2.etcd-headless.default.svc.cluster.local:2380>                                                                                                                                            |
|---------|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| *       | ETCD_INITIAL_CLUSTER_TOKEN       | etcd-cluster-k8s                                                                                                                                                                                      |
| *       | ETCD_INITIAL_CLUSTER             | etcd-0=<http://etcd-0.etcd-headless.default.svc.cluster.local:2380>,etcd-1=<http://etcd-1.etcd-headless.default.svc.cluster.local:2380>,etcd-2=<http://etcd-2.etcd-headless.default.svc.cluster.local:2380> |

The probes (readiness & liveness) are delayed 60 seconds by default, to give the etcd replicas time to start and find each other. After that period, the *etcdctl endpoint health* command is used to periodically perform health checks on every replica.

#### Scalability

The Bitnami etcd chart uses etcd reconfiguration operations to add/remove members of the cluster during scaling.

When scaling down, a "pre-stop" lifecycle hook is used to ensure that the `etcdctl member remove` command is executed. The hook stores the output of this command in the persistent volume attached to the etcd pod. This hook is also executed when the pod is manually removed using the `kubectl delete pod` command or rescheduled by Kubernetes for any reason. This implies that the cluster can be scaled up/down without human intervention.

Here is an example to explain how this works:

1. An etcd cluster with three members running on a three-nodes Kubernetes cluster is bootstrapped.
2. After a few days, the cluster administrator decides to upgrade the kernel on one of the cluster nodes. To do so, the administrator drains the node. Pods running on that node are rescheduled to a different one.
3. During the pod eviction process, the "pre-stop" hook removes the etcd member from the cluster. Thus, the etcd cluster is scaled down to only two members.
4. Once the pod is scheduled on another node and initialized, the etcd member is added again to the cluster using the *etcdctl member add* command. Thus, the etcd cluster is scaled up to three replicas.

If, for whatever reason, the "pre-stop" hook fails at removing the member, the initialization logic is able to detect that something went wrong by checking the `etcdctl member remove` command output that was stored in the persistent volume. It then uses the `etcdctl member update` command to add back the member. In this case, the cluster isn't automatically scaled down/up while the pod is recovered. Therefore, when other members attempt to connect to the pod, it may cause warnings or errors like the one below:

```text
E | rafthttp: failed to dial XXXXXXXX on stream Message (peer XXXXXXXX failed to find local node YYYYYYYYY)
I | rafthttp: peer XXXXXXXX became inactive (message send to peer failed)
W | rafthttp: health check for peer XXXXXXXX could not connect: dial tcp A.B.C.D:2380: i/o timeout
```

Learn more about [etcd runtime configuration](https://etcd.io/docs/current/op-guide/runtime-configuration/) and how to safely [drain a Kubernetes node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/).

#### Cluster updates

When updating the etcd StatefulSet (such as when upgrading the chart version via the *helm upgrade* command), every pod must be replaced following the StatefulSet update strategy.

The chart uses a "RollingUpdate" strategy by default and with default Kubernetes values. In other words, it updates each Pod, one at a time, in the same order as Pod termination (from the largest ordinal to the smallest). It will wait until an updated Pod is "Running" and "Ready" prior to updating its predecessor.

Learn more about [StatefulSet update strategies](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#update-strategies).

#### Disaster recovery

If, for whatever reason, (N-1)/2 members of the cluster fail and the "pre-stop" hooks also fail at removing them from the cluster, the cluster disastrously fails, irrevocably losing quorum. Once quorum is lost, the cluster cannot reach consensus and therefore cannot continue accepting updates. Under this circumstance, the only possible solution is usually to restore the cluster from a snapshot.

> IMPORTANT: All members should restore using the same snapshot.

The Bitnami etcd chart solves this problem by optionally offering a Kubernetes cron job that periodically snapshots the keyspace and stores it in a RWX volume. In case the cluster disastrously fails, the pods will automatically try to restore it using the last avalable snapshot.

[Learn how to enable this disaster recovery feature](#enable-disaster-recovery-features).

The chart also sets by default a "soft" Pod AntiAffinity to reduce the risk of the cluster failing disastrously.

Learn more about [etcd recovery](https://etcd.io/docs/current/op-guide/recovery), [Kubernetes cron jobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) and [pod affinity and anti-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)

### Enable security for etcd

The etcd chart can be configured with Role-based access control and TLS encryption to improve its security.

#### Configure RBAC

In order to enable Role-Based Access Control for etcd, set the following parameters:

```text
auth.rbac.create=true
auth.rbac.rootPassword=ETCD_ROOT_PASSWORD
```

These parameters create a `root` user with an associate `root` role with access to everything. The remaining users will use the `guest` role and won't have permissions to do anything.

#### Configure TLS for server-to-server communications

In order to enable secure transport between peer nodes deploy the helm chart with these options:

```text
auth.peer.secureTransport=true
auth.peer.useAutoTLS=true
```

#### Configure certificates for client communication

In order to enable secure transport between client and server, create a secret containing the certificate and key files and the CA used to sign the client certificates. In this case, create the secret and then deploy the chart with these options:

```text
auth.client.secureTransport=true
auth.client.enableAuthentication=true
auth.client.existingSecret=etcd-client-certs
```

Learn more about the [etcd security model](https://etcd.io/) and how to [generate self-signed certificates for etcd](https://coreos.com/os/docs/latest/generate-self-signed-certificates.html).

### Enable disaster recovery features

The Bitnami etcd Helm chart supports automatic disaster recovery by periodically snapshotting the keyspace. If the cluster permanently loses more than (N-1)/2 members, it tries to recover the cluster from a previous snapshot.

Enable this feature with the following parameters:

```text
persistence.enabled=true
disasterRecovery.enabled=true
disasterRecovery.pvc.size=2Gi
disasterRecovery.pvc.storageClassName=nfs
```

If the `startFromSnapshot.*` parameters are used at the same time as the `disasterRecovery.*` parameters, the PVC provided via the `startFromSnapshot.existingClaim` parameter will be used to store the periodical snapshots.

> NOTE: The disaster recovery feature requires volumes with ReadWriteMany access mode.

### Backup and restore

Two different approaches are available to back up and restore this Helm Chart:

- Back up the data from the source deployment and restore it in a new deployment using etcd's built-in backup/restore tools.
- Back up the persistent volumes from the source deployment and attach them to a new deployment using Velero, a Kubernetes backup/restore tool.

#### Method 1: Backup and restore data using etcd's built-in tools

This method involves the following steps:

- Use the *etcdctl* tool to create a snapshot of the data in the source cluster.
- Make the snapshot available in a Kubernetes PersistentVolumeClaim (PVC) that supports ReadWriteMany access (for example, a PVC created with the NFS storage class)
- Restore the data snapshot in a new cluster using the <%= variable :catalog_name, :platform %> etcd Helm chart's *startFromSnapshot.existingClaim* and *startFromSnapshot.snapshotFilename* parameters to define the source PVC and source filename for the snapshot.

> NOTE: Under this approach, it is important to create the new deployment on the destination cluster using the same credentials as the original deployment on the source cluster.

#### Method 2: Back up and restore persistent data volumes

This method involves copying the persistent data volumes for the etcd nodes and reusing them in a new deployment with [Velero](https://velero.io/), an open source Kubernetes backup/restore tool. This method is only suitable when:

- The Kubernetes provider is [supported by Velero](https://velero.io/docs/latest/supported-providers/).
- Both clusters are on the same Kubernetes provider, as this is a requirement of [Velero's native support for migrating persistent volumes](https://velero.io/docs/latest/migration-case/).
- The restored deployment on the destination cluster will have the same name, namespace, topology and credentials as the original deployment on the source cluster.

This method involves the following steps:

- Install Velero on the source and destination clusters.
- Use Velero to back up the PersistentVolumes (PVs) used by the etcd deployment on the source cluster.
- Use Velero to restore the backed-up PVs on the destination cluster.
- Create a new etcd deployment on the destination cluster with the same deployment name, credentials and other parameters as the original. This new deployment will use the restored PVs and hence the original data.

### Exposing etcd metrics

The metrics exposed by etcd can be exposed to be scraped by Prometheus. Metrics can be scraped from within the cluster using any of the following approaches:

- Adding the required annotations for Prometheus to discover the metrics endpoints, as in the example below:

```yaml
podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/path: "/metrics/cluster"
  prometheus.io/port: "9000"
```

- Creating a ServiceMonitor or PodMonitor entry (when the Prometheus Operator is available in the cluster)
- Using something similar to the [example Prometheus scrape configuration](https://github.com/prometheus/prometheus/blob/master/documentation/examples/prometheus-kubernetes.yml).

If metrics are to be scraped from outside the cluster, the Kubernetes API proxy can be utilized to access the endpoint.

### Using custom configuration

In order to use custom configuration parameters, two options are available:

- Using environment variables: etcd allows setting environment variables that map to configuration settings. In order to set extra environment variables, you can use the `extraEnvVars` property. Alternatively, you can use a ConfigMap or a Secret with the environment variables using the `extraEnvVarsCM` or the `extraEnvVarsSecret` properties.

```yaml
extraEnvVars:
  - name: ETCD_AUTO_COMPACTION_RETENTION
    value: "0"
  - name: ETCD_HEARTBEAT_INTERVAL
    value: "150"
```

- Using a custom `etcd.conf.yml`: The etcd chart allows mounting a custom `etcd.conf.yml` file as ConfigMap. In order to so, you can use the `configuration` property. Alternatively, you can use an existing ConfigMap using the `existingConfigmap` parameter.

### Auto Compaction

Since etcd keeps an exact history of its keyspace, this history should be periodically compacted to avoid performance degradation and eventual storage space exhaustion. Compacting the keyspace history drops all information about keys superseded prior to a given keyspace revision. The space used by these keys then becomes available for additional writes to the keyspace.

`autoCompactionMode`, by default periodic. Valid values: "periodic", "revision".

- 'periodic' for duration based retention, defaulting to hours if no time unit is provided (e.g. "5m").
- 'revision' for revision number based retention.
`autoCompactionRetention` for mvcc key value store in hour, by default 0, means disabled.

You can enable auto compaction by using following parameters:

```console
autoCompactionMode=periodic
autoCompactionRetention=10m
```

### Sidecars and Init Containers

If you have a need for additional containers to run within the same pod as the etcd app (e.g. an additional metrics or logging exporter), you can do so via the `sidecars` config parameter. Simply define your container according to the Kubernetes container spec.

```yaml
sidecars:
  - name: your-image-name
    image: your-image
    imagePullPolicy: Always
    ports:
      - name: portname
       containerPort: 1234
```

Similarly, you can add extra init containers using the `initContainers` parameter.

```yaml
initContainers:
  - name: your-image-name
    image: your-image
    imagePullPolicy: Always
    ports:
      - name: portname
        containerPort: 1234
```

### Deploying extra resources

There are cases where you may want to deploy extra objects, such a ConfigMap containing your app's configuration or some extra deployment with a micro service used by your app. For covering this case, the chart allows adding the full specification of other objects using the `extraDeploy` parameter.

### Setting Pod's affinity

This chart allows you to set your custom affinity using the `affinity` parameter. Find more information about Pod's affinity in the [kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).

As an alternative, you can use of the preset configurations for pod affinity, pod anti-affinity, and node affinity available at the [bitnami/common](https://github.com/bitnami/charts/tree/main/bitnami/common#affinities) chart. To do so, set the `podAffinityPreset`, `podAntiAffinityPreset`, or `nodeAffinityPreset` parameters.

## Persistence

The [Bitnami etcd](https://github.com/bitnami/containers/tree/main/bitnami/etcd) image stores the etcd data at the `/bitnami/etcd` path of the container. Persistent Volume Claims are used to keep the data across statefulsets.

The chart mounts a [Persistent Volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) volume at this location. The volume is created using dynamic volume provisioning by default. An existing PersistentVolumeClaim can also be defined for this purpose.

If you encounter errors when working with persistent volumes, refer to our [troubleshooting guide for persistent volumes](https://docs.bitnami.com/kubernetes/faq/troubleshooting/troubleshooting-persistence-volumes/).

## Parameters

### Global parameters

| Name                                                  | Description                                                                                                                                                                                                                                                                                                                                                         | Value   |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| `global.imageRegistry`                                | Global Docker image registry                                                                                                                                                                                                                                                                                                                                        | `""`    |
| `global.imagePullSecrets`                             | Global Docker registry secret names as an array                                                                                                                                                                                                                                                                                                                     | `[]`    |
| `global.defaultStorageClass`                          | Global default StorageClass for Persistent Volume(s)                                                                                                                                                                                                                                                                                                                | `""`    |
| `global.security.allowInsecureImages`                 | Allows skipping image verification                                                                                                                                                                                                                                                                                                                                  | `false` |
| `global.compatibility.openshift.adaptSecurityContext` | Adapt the securityContext sections of the deployment to make them compatible with Openshift restricted-v2 SCC: remove runAsUser, runAsGroup and fsGroup and let the platform use their allowed default IDs. Possible values: auto (apply if the detected running cluster is Openshift), force (perform the adaptation always), disabled (do not perform adaptation) | `auto`  |

### Common parameters

| Name                     | Description                                                                                  | Value           |
| ------------------------ | -------------------------------------------------------------------------------------------- | --------------- |
| `kubeVersion`            | Force target Kubernetes version (using Helm capabilities if not set)                         | `""`            |
| `nameOverride`           | String to partially override common.names.fullname template (will maintain the release name) | `""`            |
| `fullnameOverride`       | String to fully override common.names.fullname template                                      | `""`            |
| `namespaceOverride`      | String to fully override common.names.namespace template                                     | `""`            |
| `commonLabels`           | Labels to add to all deployed objects                                                        | `{}`            |
| `commonAnnotations`      | Annotations to add to all deployed objects                                                   | `{}`            |
| `clusterDomain`          | Default Kubernetes cluster domain                                                            | `cluster.local` |
| `extraDeploy`            | Array of extra objects to deploy with the release                                            | `[]`            |
| `usePasswordFiles`       | Mount credentials as files instead of using environment variables                            | `true`          |
| `diagnosticMode.enabled` | Enable diagnostic mode (all probes will be disabled and the command will be overridden)      | `false`         |
| `diagnosticMode.command` | Command to override all containers in the deployment                                         | `["sleep"]`     |
| `diagnosticMode.args`    | Args to override all containers in the deployment                                            | `["infinity"]`  |

### etcd parameters

| Name                                   | Description                                                                                                          | Value                  |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------- |
| `image.registry`                       | etcd image registry                                                                                                  | `REGISTRY_NAME`        |
| `image.repository`                     | etcd image name                                                                                                      | `REPOSITORY_NAME/etcd` |
| `image.digest`                         | etcd image digest in the way sha256:aa.... Please note this parameter, if set, will override the tag                 | `""`                   |
| `image.pullPolicy`                     | etcd image pull policy                                                                                               | `IfNotPresent`         |
| `image.pullSecrets`                    | etcd image pull secrets                                                                                              | `[]`                   |
| `image.debug`                          | Enable image debug mode                                                                                              | `false`                |
| `auth.rbac.create`                     | Switch to enable RBAC authentication                                                                                 | `true`                 |
| `auth.rbac.allowNoneAuthentication`    | Allow to use etcd without configuring RBAC authentication                                                            | `true`                 |
| `auth.rbac.rootPassword`               | Root user password. The root user is always `root`                                                                   | `""`                   |
| `auth.rbac.existingSecret`             | Name of the existing secret containing credentials for the root user                                                 | `""`                   |
| `auth.rbac.existingSecretPasswordKey`  | Name of key containing password to be retrieved from the existing secret                                             | `""`                   |
| `auth.token.enabled`                   | Enables token authentication                                                                                         | `true`                 |
| `auth.token.type`                      | Authentication token type. Allowed values: 'simple' or 'jwt'                                                         | `jwt`                  |
| `auth.token.privateKey.filename`       | Name of the file containing the private key for signing the JWT token                                                | `jwt-token.pem`        |
| `auth.token.privateKey.existingSecret` | Name of the existing secret containing the private key for signing the JWT token                                     | `""`                   |
| `auth.token.signMethod`                | JWT token sign method                                                                                                | `RS256`                |
| `auth.token.ttl`                       | JWT token TTL                                                                                                        | `10m`                  |
| `auth.client.secureTransport`          | Switch to encrypt client-to-server communications using TLS certificates                                             | `false`                |
| `auth.client.useAutoTLS`               | Switch to automatically create the TLS certificates                                                                  | `false`                |
| `auth.client.existingSecret`           | Name of the existing secret containing the TLS certificates for client-to-server communications                      | `""`                   |
| `auth.client.enableAuthentication`     | Switch to enable host authentication using TLS certificates. Requires existing secret                                | `false`                |
| `auth.client.certFilename`             | Name of the file containing the client certificate                                                                   | `cert.pem`             |
| `auth.client.certKeyFilename`          | Name of the file containing the client certificate private key                                                       | `key.pem`              |
| `auth.client.caFilename`               | Name of the file containing the client CA certificate                                                                | `""`                   |
| `auth.peer.secureTransport`            | Switch to encrypt server-to-server communications using TLS certificates                                             | `false`                |
| `auth.peer.useAutoTLS`                 | Switch to automatically create the TLS certificates                                                                  | `false`                |
| `auth.peer.existingSecret`             | Name of the existing secret containing the TLS certificates for server-to-server communications                      | `""`                   |
| `auth.peer.enableAuthentication`       | Switch to enable host authentication using TLS certificates. Requires existing secret                                | `false`                |
| `auth.peer.certFilename`               | Name of the file containing the peer certificate                                                                     | `cert.pem`             |
| `auth.peer.certKeyFilename`            | Name of the file containing the peer certificate private key                                                         | `key.pem`              |
| `auth.peer.caFilename`                 | Name of the file containing the peer CA certificate                                                                  | `""`                   |
| `autoCompactionMode`                   | Auto compaction mode, by default periodic. Valid values: "periodic", "revision".                                     | `""`                   |
| `autoCompactionRetention`              | Auto compaction retention for mvcc key value store in hour, by default 0, means disabled                             | `""`                   |
| `initialClusterToken`                  | Initial cluster token. Can be used to protect etcd from cross-cluster-interaction, which might corrupt the clusters. | `etcd-cluster-k8s`     |
| `logLevel`                             | Sets the log level for the etcd process. Allowed values: 'debug', 'info', 'warn', 'error', 'panic', 'fatal'          | `info`                 |
| `maxProcs`                             | Limits the number of operating system threads that can execute user-level                                            | `""`                   |
| `configuration`                        | etcd configuration. Specify content for etcd.conf.yml                                                                | `""`                   |
| `existingConfigmap`                    | Existing ConfigMap with etcd configuration                                                                           | `""`                   |
| `extraEnvVars`                         | Extra environment variables to be set on etcd container                                                              | `[]`                   |
| `extraEnvVarsCM`                       | Name of existing ConfigMap containing extra env vars                                                                 | `""`                   |
| `extraEnvVarsSecret`                   | Name of existing Secret containing extra env vars                                                                    | `""`                   |
| `command`                              | Default container command (useful when using custom images)                                                          | `[]`                   |
| `args`                                 | Default container args (useful when using custom images)                                                             | `[]`                   |

### etcd statefulset parameters

| Name                                                | Description                                                                                                                                                                                                       | Value            |
| --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| `replicaCount`                                      | Number of etcd replicas to deploy                                                                                                                                                                                 | `1`              |
| `updateStrategy.type`                               | Update strategy type, can be set to RollingUpdate or OnDelete.                                                                                                                                                    | `RollingUpdate`  |
| `podManagementPolicy`                               | Pod management policy for the etcd statefulset                                                                                                                                                                    | `Parallel`       |
| `automountServiceAccountToken`                      | Mount Service Account token in pod                                                                                                                                                                                | `false`          |
| `hostAliases`                                       | etcd pod host aliases                                                                                                                                                                                             | `[]`             |
| `lifecycleHooks`                                    | Override default etcd container hooks                                                                                                                                                                             | `{}`             |
| `containerPorts.client`                             | Client port to expose at container level                                                                                                                                                                          | `2379`           |
| `containerPorts.peer`                               | Peer port to expose at container level                                                                                                                                                                            | `2380`           |
| `containerPorts.metrics`                            | Metrics port to expose at container level when metrics.useSeparateEndpoint is true                                                                                                                                | `9090`           |
| `podSecurityContext.enabled`                        | Enabled etcd pods' Security Context                                                                                                                                                                               | `true`           |
| `podSecurityContext.fsGroupChangePolicy`            | Set filesystem group change policy                                                                                                                                                                                | `Always`         |
| `podSecurityContext.sysctls`                        | Set kernel settings using the sysctl interface                                                                                                                                                                    | `[]`             |
| `podSecurityContext.supplementalGroups`             | Set filesystem extra groups                                                                                                                                                                                       | `[]`             |
| `podSecurityContext.fsGroup`                        | Set etcd pod's Security Context fsGroup                                                                                                                                                                           | `1001`           |
| `containerSecurityContext.enabled`                  | Enabled etcd containers' Security Context                                                                                                                                                                         | `true`           |
| `containerSecurityContext.seLinuxOptions`           | Set SELinux options in container                                                                                                                                                                                  | `{}`             |
| `containerSecurityContext.runAsUser`                | Set etcd containers' Security Context runAsUser                                                                                                                                                                   | `1001`           |
| `containerSecurityContext.runAsGroup`               | Set etcd containers' Security Context runAsUser                                                                                                                                                                   | `1001`           |
| `containerSecurityContext.runAsNonRoot`             | Set Controller container's Security Context runAsNonRoot                                                                                                                                                          | `true`           |
| `containerSecurityContext.privileged`               | Set primary container's Security Context privileged                                                                                                                                                               | `false`          |
| `containerSecurityContext.allowPrivilegeEscalation` | Set primary container's Security Context allowPrivilegeEscalation                                                                                                                                                 | `false`          |
| `containerSecurityContext.readOnlyRootFilesystem`   | Set container's Security Context readOnlyRootFilesystem                                                                                                                                                           | `true`           |
| `containerSecurityContext.capabilities.drop`        | List of capabilities to be dropped                                                                                                                                                                                | `["ALL"]`        |
| `containerSecurityContext.seccompProfile.type`      | Set container's Security Context seccomp profile                                                                                                                                                                  | `RuntimeDefault` |
| `resourcesPreset`                                   | Set container resources according to one common preset (allowed values: none, nano, micro, small, medium, large, xlarge, 2xlarge). This is ignored if resources is set (resources is recommended for production). | `micro`          |
| `resources`                                         | Set container requests and limits for different resources like CPU or memory (essential for production workloads)                                                                                                 | `{}`             |
| `livenessProbe.enabled`                             | Enable livenessProbe                                                                                                                                                                                              | `true`           |
| `livenessProbe.initialDelaySeconds`                 | Initial delay seconds for livenessProbe                                                                                                                                                                           | `60`             |
| `livenessProbe.periodSeconds`                       | Period seconds for livenessProbe                                                                                                                                                                                  | `30`             |
| `livenessProbe.timeoutSeconds`                      | Timeout seconds for livenessProbe                                                                                                                                                                                 | `5`              |
| `livenessProbe.failureThreshold`                    | Failure threshold for livenessProbe                                                                                                                                                                               | `5`              |
| `livenessProbe.successThreshold`                    | Success threshold for livenessProbe                                                                                                                                                                               | `1`              |
| `readinessProbe.enabled`                            | Enable readinessProbe                                                                                                                                                                                             | `true`           |
| `readinessProbe.initialDelaySeconds`                | Initial delay seconds for readinessProbe                                                                                                                                                                          | `60`             |
| `readinessProbe.periodSeconds`                      | Period seconds for readinessProbe                                                                                                                                                                                 | `10`             |
| `readinessProbe.timeoutSeconds`                     | Timeout seconds for readinessProbe                                                                                                                                                                                | `5`              |
| `readinessProbe.failureThreshold`                   | Failure threshold for readinessProbe                                                                                                                                                                              | `5`              |
| `readinessProbe.successThreshold`                   | Success threshold for readinessProbe                                                                                                                                                                              | `1`              |
| `startupProbe.enabled`                              | Enable startupProbe                                                                                                                                                                                               | `false`          |
| `startupProbe.initialDelaySeconds`                  | Initial delay seconds for startupProbe                                                                                                                                                                            | `0`              |
| `startupProbe.periodSeconds`                        | Period seconds for startupProbe                                                                                                                                                                                   | `10`             |
| `startupProbe.timeoutSeconds`                       | Timeout seconds for startupProbe                                                                                                                                                                                  | `5`              |
| `startupProbe.failureThreshold`                     | Failure threshold for startupProbe                                                                                                                                                                                | `60`             |
| `startupProbe.successThreshold`                     | Success threshold for startupProbe                                                                                                                                                                                | `1`              |
| `customLivenessProbe`                               | Override default liveness probe                                                                                                                                                                                   | `{}`             |
| `customReadinessProbe`                              | Override default readiness probe                                                                                                                                                                                  | `{}`             |
| `customStartupProbe`                                | Override default startup probe                                                                                                                                                                                    | `{}`             |
| `extraVolumes`                                      | Optionally specify extra list of additional volumes for etcd pods                                                                                                                                                 | `[]`             |
| `extraVolumeMounts`                                 | Optionally specify extra list of additional volumeMounts for etcd container(s)                                                                                                                                    | `[]`             |
| `extraVolumeClaimTemplates`                         | Optionally specify extra list of additional volumeClaimTemplates for etcd container(s)                                                                                                                            | `[]`             |
| `initContainers`                                    | Add additional init containers to the etcd pods                                                                                                                                                                   | `[]`             |
| `sidecars`                                          | Add additional sidecar containers to the etcd pods                                                                                                                                                                | `[]`             |
| `podAnnotations`                                    | Annotations for etcd pods                                                                                                                                                                                         | `{}`             |
| `podLabels`                                         | Extra labels for etcd pods                                                                                                                                                                                        | `{}`             |
| `podAffinityPreset`                                 | Pod affinity preset. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                               | `""`             |
| `podAntiAffinityPreset`                             | Pod anti-affinity preset. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                          | `soft`           |
| `nodeAffinityPreset.type`                           | Node affinity preset type. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                         | `""`             |
| `nodeAffinityPreset.key`                            | Node label key to match. Ignored if `affinity` is set.                                                                                                                                                            | `""`             |
| `nodeAffinityPreset.values`                         | Node label values to match. Ignored if `affinity` is set.                                                                                                                                                         | `[]`             |
| `affinity`                                          | Affinity for pod assignment                                                                                                                                                                                       | `{}`             |
| `nodeSelector`                                      | Node labels for pod assignment                                                                                                                                                                                    | `{}`             |
| `tolerations`                                       | Tolerations for pod assignment                                                                                                                                                                                    | `[]`             |
| `terminationGracePeriodSeconds`                     | Seconds the pod needs to gracefully terminate                                                                                                                                                                     | `""`             |
| `schedulerName`                                     | Name of the k8s scheduler (other than default)                                                                                                                                                                    | `""`             |
| `priorityClassName`                                 | Name of the priority class to be used by etcd pods                                                                                                                                                                | `""`             |
| `runtimeClassName`                                  | Name of the runtime class to be used by pod(s)                                                                                                                                                                    | `""`             |
| `shareProcessNamespace`                             | Enable shared process namespace in a pod.                                                                                                                                                                         | `false`          |
| `topologySpreadConstraints`                         | Topology Spread Constraints for pod assignment                                                                                                                                                                    | `[]`             |
| `persistentVolumeClaimRetentionPolicy.enabled`      | Controls if and how PVCs are deleted during the lifecycle of a StatefulSet                                                                                                                                        | `false`          |
| `persistentVolumeClaimRetentionPolicy.whenScaled`   | Volume retention behavior when the replica count of the StatefulSet is reduced                                                                                                                                    | `Retain`         |
| `persistentVolumeClaimRetentionPolicy.whenDeleted`  | Volume retention behavior that applies when the StatefulSet is deleted                                                                                                                                            | `Retain`         |

### Traffic exposure parameters

| Name                               | Description                                                                                                                                                    | Value       |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| `service.type`                     | Kubernetes Service type                                                                                                                                        | `ClusterIP` |
| `service.enabled`                  | create second service if equal true                                                                                                                            | `true`      |
| `service.clusterIP`                | Kubernetes service Cluster IP                                                                                                                                  | `""`        |
| `service.ports.client`             | etcd client port                                                                                                                                               | `2379`      |
| `service.ports.peer`               | etcd peer port                                                                                                                                                 | `2380`      |
| `service.ports.metrics`            | etcd metrics port when metrics.useSeparateEndpoint is true                                                                                                     | `9090`      |
| `service.nodePorts.client`         | Specify the nodePort client value for the LoadBalancer and NodePort service types.                                                                             | `""`        |
| `service.nodePorts.peer`           | Specify the nodePort peer value for the LoadBalancer and NodePort service types.                                                                               | `""`        |
| `service.nodePorts.metrics`        | Specify the nodePort metrics value for the LoadBalancer and NodePort service types. The metrics port is only exposed when metrics.useSeparateEndpoint is true. | `""`        |
| `service.clientPortNameOverride`   | etcd client port name override                                                                                                                                 | `""`        |
| `service.peerPortNameOverride`     | etcd peer port name override                                                                                                                                   | `""`        |
| `service.metricsPortNameOverride`  | etcd metrics port name override. The metrics port is only exposed when metrics.useSeparateEndpoint is true.                                                    | `""`        |
| `service.loadBalancerIP`           | loadBalancerIP for the etcd service (optional, cloud specific)                                                                                                 | `""`        |
| `service.loadBalancerClass`        | loadBalancerClass for the etcd service (optional, cloud specific)                                                                                              | `""`        |
| `service.loadBalancerSourceRanges` | Load Balancer source ranges                                                                                                                                    | `[]`        |
| `service.externalIPs`              | External IPs                                                                                                                                                   | `[]`        |
| `service.externalTrafficPolicy`    | %%MAIN_CONTAINER_NAME%% service external traffic policy                                                                                                        | `Cluster`   |
| `service.extraPorts`               | Extra ports to expose (normally used with the `sidecar` value)                                                                                                 | `[]`        |
| `service.annotations`              | Additional annotations for the etcd service                                                                                                                    | `{}`        |
| `service.sessionAffinity`          | Session Affinity for Kubernetes service, can be "None" or "ClientIP"                                                                                           | `None`      |
| `service.sessionAffinityConfig`    | Additional settings for the sessionAffinity                                                                                                                    | `{}`        |
| `service.headless.annotations`     | Annotations for the headless service.                                                                                                                          | `{}`        |

### Persistence parameters

| Name                       | Description                                                     | Value               |
| -------------------------- | --------------------------------------------------------------- | ------------------- |
| `persistence.enabled`      | If true, use a Persistent Volume Claim. If false, use emptyDir. | `true`              |
| `persistence.storageClass` | Persistent Volume Storage Class                                 | `""`                |
| `persistence.annotations`  | Annotations for the PVC                                         | `{}`                |
| `persistence.labels`       | Labels for the PVC                                              | `{}`                |
| `persistence.accessModes`  | Persistent Volume Access Modes                                  | `["ReadWriteOnce"]` |
| `persistence.size`         | PVC Storage Request for etcd data volume                        | `8Gi`               |
| `persistence.selector`     | Selector to match an existing Persistent Volume                 | `{}`                |

### Volume Permissions parameters

| Name                                  | Description                                                                                                                                                                                                                                           | Value                      |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| `volumePermissions.enabled`           | Enable init container that changes the owner and group of the persistent volume(s) mountpoint to `runAsUser:fsGroup`                                                                                                                                  | `false`                    |
| `volumePermissions.image.registry`    | Init container volume-permissions image registry                                                                                                                                                                                                      | `REGISTRY_NAME`            |
| `volumePermissions.image.repository`  | Init container volume-permissions image name                                                                                                                                                                                                          | `REPOSITORY_NAME/os-shell` |
| `volumePermissions.image.digest`      | Init container volume-permissions image digest in the way sha256:aa.... Please note this parameter, if set, will override the tag                                                                                                                     | `""`                       |
| `volumePermissions.image.pullPolicy`  | Init container volume-permissions image pull policy                                                                                                                                                                                                   | `IfNotPresent`             |
| `volumePermissions.image.pullSecrets` | Specify docker-registry secret names as an array                                                                                                                                                                                                      | `[]`                       |
| `volumePermissions.resourcesPreset`   | Set container resources according to one common preset (allowed values: none, nano, micro, small, medium, large, xlarge, 2xlarge). This is ignored if volumePermissions.resources is set (volumePermissions.resources is recommended for production). | `nano`                     |
| `volumePermissions.resources`         | Set container requests and limits for different resources like CPU or memory (essential for production workloads)                                                                                                                                     | `{}`                       |

### Network Policy parameters

| Name                                    | Description                                                     | Value  |
| --------------------------------------- | --------------------------------------------------------------- | ------ |
| `networkPolicy.enabled`                 | Enable creation of NetworkPolicy resources                      | `true` |
| `networkPolicy.allowExternal`           | Don't require client label for connections                      | `true` |
| `networkPolicy.allowExternalEgress`     | Allow the pod to access any range of port and all destinations. | `true` |
| `networkPolicy.extraIngress`            | Add extra ingress rules to the NetworkPolicy                    | `[]`   |
| `networkPolicy.extraEgress`             | Add extra ingress rules to the NetworkPolicy                    | `[]`   |
| `networkPolicy.ingressNSMatchLabels`    | Labels to match to allow traffic from other namespaces          | `{}`   |
| `networkPolicy.ingressNSPodMatchLabels` | Pod labels to match to allow traffic from other namespaces      | `{}`   |

### Metrics parameters

| Name                                      | Description                                                                                                                   | Value        |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------ |
| `metrics.enabled`                         | Expose etcd metrics                                                                                                           | `false`      |
| `metrics.useSeparateEndpoint`             | Use a separate endpoint for exposing metrics                                                                                  | `false`      |
| `metrics.podAnnotations`                  | Annotations for the Prometheus metrics on etcd pods                                                                           | `{}`         |
| `metrics.podMonitor.enabled`              | Create PodMonitor Resource for scraping metrics using PrometheusOperator                                                      | `false`      |
| `metrics.podMonitor.namespace`            | Namespace in which Prometheus is running                                                                                      | `monitoring` |
| `metrics.podMonitor.interval`             | Specify the interval at which metrics should be scraped                                                                       | `30s`        |
| `metrics.podMonitor.scrapeTimeout`        | Specify the timeout after which the scrape is ended                                                                           | `30s`        |
| `metrics.podMonitor.additionalLabels`     | Additional labels that can be used so PodMonitors will be discovered by Prometheus                                            | `{}`         |
| `metrics.podMonitor.scheme`               | Scheme to use for scraping                                                                                                    | `http`       |
| `metrics.podMonitor.tlsConfig`            | TLS configuration used for scrape endpoints used by Prometheus                                                                | `{}`         |
| `metrics.podMonitor.relabelings`          | Prometheus relabeling rules                                                                                                   | `[]`         |
| `metrics.prometheusRule.enabled`          | Create a Prometheus Operator PrometheusRule (also requires `metrics.enabled` to be `true` and `metrics.prometheusRule.rules`) | `false`      |
| `metrics.prometheusRule.namespace`        | Namespace for the PrometheusRule Resource (defaults to the Release Namespace)                                                 | `""`         |
| `metrics.prometheusRule.additionalLabels` | Additional labels that can be used so PrometheusRule will be discovered by Prometheus                                         | `{}`         |
| `metrics.prometheusRule.rules`            | Prometheus Rule definitions                                                                                                   | `[]`         |

### Snapshotting parameters

| Name                                                                         | Description                                                                                                                                                                                                                                                         | Value            |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| `startFromSnapshot.enabled`                                                  | Initialize new cluster recovering an existing snapshot                                                                                                                                                                                                              | `false`          |
| `startFromSnapshot.existingClaim`                                            | Existing PVC containing the etcd snapshot                                                                                                                                                                                                                           | `""`             |
| `startFromSnapshot.snapshotFilename`                                         | Snapshot filename                                                                                                                                                                                                                                                   | `""`             |
| `disasterRecovery.enabled`                                                   | Enable auto disaster recovery by periodically snapshotting the keyspace                                                                                                                                                                                             | `false`          |
| `disasterRecovery.cronjob.schedule`                                          | Schedule in Cron format to save snapshots                                                                                                                                                                                                                           | `*/30 * * * *`   |
| `disasterRecovery.cronjob.historyLimit`                                      | Number of successful finished jobs to retain                                                                                                                                                                                                                        | `1`              |
| `disasterRecovery.cronjob.snapshotHistoryLimit`                              | Number of etcd snapshots to retain, tagged by date                                                                                                                                                                                                                  | `1`              |
| `disasterRecovery.cronjob.snapshotsDir`                                      | Directory to store snapshots                                                                                                                                                                                                                                        | `/snapshots`     |
| `disasterRecovery.cronjob.podAnnotations`                                    | Pod annotations for cronjob pods                                                                                                                                                                                                                                    | `{}`             |
| `disasterRecovery.cronjob.podSecurityContext.enabled`                        | Enable security context for Snapshotter pods                                                                                                                                                                                                                        | `true`           |
| `disasterRecovery.cronjob.podSecurityContext.fsGroupChangePolicy`            | Set filesystem group change policy                                                                                                                                                                                                                                  | `Always`         |
| `disasterRecovery.cronjob.podSecurityContext.sysctls`                        | Set kernel settings using the sysctl interface                                                                                                                                                                                                                      | `[]`             |
| `disasterRecovery.cronjob.podSecurityContext.supplementalGroups`             | Set filesystem extra groups                                                                                                                                                                                                                                         | `[]`             |
| `disasterRecovery.cronjob.podSecurityContext.fsGroup`                        | Group ID for the Snapshotter filesystem                                                                                                                                                                                                                             | `1001`           |
| `disasterRecovery.cronjob.containerSecurityContext.enabled`                  | Enabled containers' Security Context                                                                                                                                                                                                                                | `true`           |
| `disasterRecovery.cronjob.containerSecurityContext.seLinuxOptions`           | Set SELinux options in container                                                                                                                                                                                                                                    | `{}`             |
| `disasterRecovery.cronjob.containerSecurityContext.runAsUser`                | Set containers' Security Context runAsUser                                                                                                                                                                                                                          | `1001`           |
| `disasterRecovery.cronjob.containerSecurityContext.runAsGroup`               | Set containers' Security Context runAsGroup                                                                                                                                                                                                                         | `1001`           |
| `disasterRecovery.cronjob.containerSecurityContext.runAsNonRoot`             | Set container's Security Context runAsNonRoot                                                                                                                                                                                                                       | `true`           |
| `disasterRecovery.cronjob.containerSecurityContext.privileged`               | Set container's Security Context privileged                                                                                                                                                                                                                         | `false`          |
| `disasterRecovery.cronjob.containerSecurityContext.readOnlyRootFilesystem`   | Set container's Security Context readOnlyRootFilesystem                                                                                                                                                                                                             | `true`           |
| `disasterRecovery.cronjob.containerSecurityContext.allowPrivilegeEscalation` | Set container's Security Context allowPrivilegeEscalation                                                                                                                                                                                                           | `false`          |
| `disasterRecovery.cronjob.containerSecurityContext.capabilities.drop`        | List of capabilities to be dropped                                                                                                                                                                                                                                  | `["ALL"]`        |
| `disasterRecovery.cronjob.containerSecurityContext.seccompProfile.type`      | Set container's Security Context seccomp profile                                                                                                                                                                                                                    | `RuntimeDefault` |
| `disasterRecovery.cronjob.resourcesPreset`                                   | Set container resources according to one common preset (allowed values: none, nano, micro, small, medium, large, xlarge, 2xlarge). This is ignored if disasterRecovery.cronjob.resources is set (disasterRecovery.cronjob.resources is recommended for production). | `nano`           |
| `disasterRecovery.cronjob.resources`                                         | Set container requests and limits for different resources like CPU or memory (essential for production workloads)                                                                                                                                                   | `{}`             |
| `disasterRecovery.cronjob.nodeSelector`                                      | Node labels for cronjob pods assignment                                                                                                                                                                                                                             | `{}`             |
| `disasterRecovery.cronjob.tolerations`                                       | Tolerations for cronjob pods assignment                                                                                                                                                                                                                             | `[]`             |
| `disasterRecovery.cronjob.podLabels`                                         | Labels that will be added to pods created by cronjob                                                                                                                                                                                                                | `{}`             |
| `disasterRecovery.cronjob.serviceAccountName`                                | Specifies the service account to use for disaster recovery cronjob                                                                                                                                                                                                  | `""`             |
| `disasterRecovery.cronjob.command`                                           | Override default snapshot container command (useful when you want to customize the snapshot logic)                                                                                                                                                                  | `[]`             |
| `disasterRecovery.pvc.existingClaim`                                         | A manually managed Persistent Volume and Claim                                                                                                                                                                                                                      | `""`             |
| `disasterRecovery.pvc.size`                                                  | PVC Storage Request                                                                                                                                                                                                                                                 | `2Gi`            |
| `disasterRecovery.pvc.storageClassName`                                      | Storage Class for snapshots volume                                                                                                                                                                                                                                  | `nfs`            |
| `disasterRecovery.pvc.subPath`                                               | Path within the volume from which to mount                                                                                                                                                                                                                          | `""`             |

### Service account parameters

| Name                                          | Description                                                  | Value   |
| --------------------------------------------- | ------------------------------------------------------------ | ------- |
| `serviceAccount.create`                       | Enable/disable service account creation                      | `true`  |
| `serviceAccount.name`                         | Name of the service account to create or use                 | `""`    |
| `serviceAccount.automountServiceAccountToken` | Enable/disable auto mounting of service account token        | `false` |
| `serviceAccount.annotations`                  | Additional annotations to be included on the service account | `{}`    |
| `serviceAccount.labels`                       | Additional labels to be included on the service account      | `{}`    |

### etcd "pre-upgrade" K8s Job parameters

| Name                                                              | Description                                                                                                                                                                                                                                                     | Value            |
| ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| `preUpgradeJob.enabled`                                           | Enable running a pre-upgrade job on Helm upgrades that removes obsolete members                                                                                                                                                                                 | `true`           |
| `preUpgradeJob.annotations`                                       | Add annotations to the etcd "pre-upgrade" job                                                                                                                                                                                                                   | `{}`             |
| `preUpgradeJob.podLabels`                                         | Additional pod labels for etcd "pre-upgrade" job                                                                                                                                                                                                                | `{}`             |
| `preUpgradeJob.podAnnotations`                                    | Additional pod annotations for etcd "pre-upgrade" job                                                                                                                                                                                                           | `{}`             |
| `preUpgradeJob.podAffinityPreset`                                 | Pod affinity preset. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                                                                             | `""`             |
| `preUpgradeJob.podAntiAffinityPreset`                             | Pod anti-affinity preset. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                                                                        | `soft`           |
| `preUpgradeJob.nodeAffinityPreset.type`                           | Node affinity preset type. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                                                                       | `""`             |
| `preUpgradeJob.nodeAffinityPreset.key`                            | Node label key to match. Ignored if `affinity` is set.                                                                                                                                                                                                          | `""`             |
| `preUpgradeJob.nodeAffinityPreset.values`                         | Node label values to match. Ignored if `affinity` is set.                                                                                                                                                                                                       | `[]`             |
| `preUpgradeJob.affinity`                                          | Affinity for pod assignment                                                                                                                                                                                                                                     | `{}`             |
| `preUpgradeJob.nodeSelector`                                      | Node labels for pod assignment                                                                                                                                                                                                                                  | `{}`             |
| `preUpgradeJob.tolerations`                                       | Tolerations for pod assignment                                                                                                                                                                                                                                  | `[]`             |
| `preUpgradeJob.containerSecurityContext.enabled`                  | Enabled "pre-upgrade" job's containers' Security Context                                                                                                                                                                                                        | `true`           |
| `preUpgradeJob.containerSecurityContext.seLinuxOptions`           | Set SELinux options in "pre-upgrade" job's containers                                                                                                                                                                                                           | `{}`             |
| `preUpgradeJob.containerSecurityContext.runAsUser`                | Set runAsUser in "pre-upgrade" job's containers' Security Context                                                                                                                                                                                               | `1001`           |
| `preUpgradeJob.containerSecurityContext.runAsGroup`               | Set runAsUser in "pre-upgrade" job's containers' Security Context                                                                                                                                                                                               | `1001`           |
| `preUpgradeJob.containerSecurityContext.runAsNonRoot`             | Set runAsNonRoot in "pre-upgrade" job's containers' Security Context                                                                                                                                                                                            | `true`           |
| `preUpgradeJob.containerSecurityContext.readOnlyRootFilesystem`   | Set readOnlyRootFilesystem in "pre-upgrade" job's containers' Security Context                                                                                                                                                                                  | `true`           |
| `preUpgradeJob.containerSecurityContext.privileged`               | Set privileged in "pre-upgrade" job's containers' Security Context                                                                                                                                                                                              | `false`          |
| `preUpgradeJob.containerSecurityContext.allowPrivilegeEscalation` | Set allowPrivilegeEscalation in "pre-upgrade" job's containers' Security Context                                                                                                                                                                                | `false`          |
| `preUpgradeJob.containerSecurityContext.capabilities.add`         | List of capabilities to be added in "pre-upgrade" job's containers                                                                                                                                                                                              | `[]`             |
| `preUpgradeJob.containerSecurityContext.capabilities.drop`        | List of capabilities to be dropped in "pre-upgrade" job's containers                                                                                                                                                                                            | `["ALL"]`        |
| `preUpgradeJob.containerSecurityContext.seccompProfile.type`      | Set seccomp profile in "pre-upgrade" job's containers                                                                                                                                                                                                           | `RuntimeDefault` |
| `preUpgradeJob.podSecurityContext.enabled`                        | Enabled "pre-upgrade" job's pods' Security Context                                                                                                                                                                                                              | `true`           |
| `preUpgradeJob.podSecurityContext.fsGroupChangePolicy`            | Set fsGroupChangePolicy in "pre-upgrade" job's pods' Security Context                                                                                                                                                                                           | `Always`         |
| `preUpgradeJob.podSecurityContext.sysctls`                        | List of sysctls to allow in "pre-upgrade" job's pods' Security Context                                                                                                                                                                                          | `[]`             |
| `preUpgradeJob.podSecurityContext.supplementalGroups`             | List of supplemental groups to add to "pre-upgrade" job's pods' Security Context                                                                                                                                                                                | `[]`             |
| `preUpgradeJob.podSecurityContext.fsGroup`                        | Set fsGroup in "pre-upgrade" job's pods' Security Context                                                                                                                                                                                                       | `1001`           |
| `preUpgradeJob.resourcesPreset`                                   | Set etcd "pre-upgrade" job's container resources according to one common preset (allowed values: none, nano, small, medium, large, xlarge, 2xlarge). This is ignored if preUpgradeJob.resources is set (preUpgradeJob.resources is recommended for production). | `micro`          |
| `preUpgradeJob.resources`                                         | Set etcd "pre-upgrade" job's container requests and limits for different resources like CPU or memory (essential for production workloads)                                                                                                                      | `{}`             |
| `preUpgradeJob.startDelay`                                        | Optional delay before starting the pre-upgrade hook (in seconds).                                                                                                                                                                                               | `""`             |

### Defragmentation parameters

| Name                                                               | Description                                                                                                                                | Value            |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ | ---------------- |
| `defrag.enabled`                                                   | Enable automatic defragmentation. This is most effective when paired with auto compaction: consider setting "autoCompactionRetention > 0". | `false`          |
| `defrag.cronjob.startingDeadlineSeconds`                           | Number of seconds representing the deadline for starting the job if it misses scheduled time for any reason                                | `""`             |
| `defrag.cronjob.schedule`                                          | Schedule in Cron format to defrag (daily at midnight by default)                                                                           | `0 0 * * *`      |
| `defrag.cronjob.concurrencyPolicy`                                 | Set the cronjob parameter concurrencyPolicy                                                                                                | `Forbid`         |
| `defrag.cronjob.suspend`                                           | Boolean that indicates if the controller must suspend subsequent executions (not applied to already started executions)                    | `false`          |
| `defrag.cronjob.successfulJobsHistoryLimit`                        | Number of successful finished jobs to retain                                                                                               | `1`              |
| `defrag.cronjob.failedJobsHistoryLimit`                            | Number of failed finished jobs to retain                                                                                                   | `1`              |
| `defrag.cronjob.labels`                                            | Additional labels to be added to the Defrag cronjob                                                                                        | `{}`             |
| `defrag.cronjob.annotations`                                       | Annotations to be added to the Defrag cronjob                                                                                              | `{}`             |
| `defrag.cronjob.activeDeadlineSeconds`                             | Number of seconds relative to the startTime that the job may be continuously active before the system tries to terminate it                | `""`             |
| `defrag.cronjob.restartPolicy`                                     | Set the cronjob parameter restartPolicy                                                                                                    | `OnFailure`      |
| `defrag.cronjob.podLabels`                                         | Labels that will be added to pods created by Defrag cronjob                                                                                | `{}`             |
| `defrag.cronjob.podAnnotations`                                    | Pod annotations for Defrag cronjob pods                                                                                                    | `{}`             |
| `defrag.cronjob.podSecurityContext.enabled`                        | Enable security context for Defrag pods                                                                                                    | `true`           |
| `defrag.cronjob.podSecurityContext.fsGroupChangePolicy`            | Set filesystem group change policy                                                                                                         | `Always`         |
| `defrag.cronjob.podSecurityContext.sysctls`                        | Set kernel settings using the sysctl interface                                                                                             | `[]`             |
| `defrag.cronjob.podSecurityContext.supplementalGroups`             | Set filesystem extra groups                                                                                                                | `[]`             |
| `defrag.cronjob.podSecurityContext.fsGroup`                        | Group ID for the Defrag filesystem                                                                                                         | `1001`           |
| `defrag.cronjob.containerSecurityContext.enabled`                  | Enabled containers' Security Context                                                                                                       | `true`           |
| `defrag.cronjob.containerSecurityContext.seLinuxOptions`           | Set SELinux options in container                                                                                                           | `{}`             |
| `defrag.cronjob.containerSecurityContext.runAsUser`                | Set containers' Security Context runAsUser                                                                                                 | `1001`           |
| `defrag.cronjob.containerSecurityContext.runAsGroup`               | Set containers' Security Context runAsGroup                                                                                                | `1001`           |
| `defrag.cronjob.containerSecurityContext.runAsNonRoot`             | Set container's Security Context runAsNonRoot                                                                                              | `true`           |
| `defrag.cronjob.containerSecurityContext.privileged`               | Set container's Security Context privileged                                                                                                | `false`          |
| `defrag.cronjob.containerSecurityContext.readOnlyRootFilesystem`   | Set container's Security Context readOnlyRootFilesystem                                                                                    | `true`           |
| `defrag.cronjob.containerSecurityContext.allowPrivilegeEscalation` | Set container's Security Context allowPrivilegeEscalation                                                                                  | `false`          |
| `defrag.cronjob.containerSecurityContext.capabilities.drop`        | List of capabilities to be dropped                                                                                                         | `["ALL"]`        |
| `defrag.cronjob.containerSecurityContext.seccompProfile.type`      | Set container's Security Context seccomp profile                                                                                           | `RuntimeDefault` |
| `defrag.cronjob.nodeSelector`                                      | Node labels for pod assignment in Defrag cronjob                                                                                           | `{}`             |
| `defrag.cronjob.tolerations`                                       | Tolerations for pod assignment in Defrag cronjob                                                                                           | `[]`             |
| `defrag.cronjob.serviceAccountName`                                | Specifies the service account to use for Defrag cronjob                                                                                    | `""`             |
| `defrag.cronjob.command`                                           | Override default container command for defragmentation (useful when using custom images)                                                   | `[]`             |
| `defrag.cronjob.args`                                              | Override default container args (useful when using custom images)                                                                          | `[]`             |
| `defrag.cronjob.resourcesPreset`                                   | Set container resources according to one common preset                                                                                     | `nano`           |
| `defrag.cronjob.resources`                                         | Set container requests and limits for different resources like CPU or                                                                      | `{}`             |
| `defrag.cronjob.extraEnvVars`                                      | Extra environment variables to be set on defrag cronjob container                                                                          | `[]`             |
| `defrag.cronjob.extraEnvVarsCM`                                    | Name of existing ConfigMap containing extra env vars                                                                                       | `""`             |
| `defrag.cronjob.extraEnvVarsSecret`                                | Name of existing Secret containing extra env vars                                                                                          | `""`             |

### Other parameters

| Name                 | Description                                                    | Value  |
| -------------------- | -------------------------------------------------------------- | ------ |
| `pdb.create`         | Enable/disable a Pod Disruption Budget creation                | `true` |
| `pdb.minAvailable`   | Minimum number/percentage of pods that should remain scheduled | `51%`  |
| `pdb.maxUnavailable` | Maximum number/percentage of pods that may be made unavailable | `""`   |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```console
helm install my-release \
  --set auth.rbac.rootPassword=secretpassword oci://REGISTRY_NAME/REPOSITORY_NAME/etcd
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.

The above command sets the etcd `root` account password to `secretpassword`.

> NOTE: Once this chart is deployed, it is not possible to change the application's access credentials, such as usernames or passwords, using Helm. To change these application credentials after deployment, delete any persistent volumes (PVs) used by the chart and re-deploy it, or use the application's built-in administrative tools if available.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```console
helm install my-release -f values.yaml oci://REGISTRY_NAME/REPOSITORY_NAME/etcd
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.
> **Tip**: You can use the default [values.yaml](https://github.com/bitnami/charts/tree/main/bitnami/etcd/values.yaml)

## Troubleshooting

Find more information about how to deal with common errors related to Bitnami's Helm charts in [this troubleshooting guide](https://docs.bitnami.com/general/how-to/troubleshoot-helm-chart-issues).

## Upgrading

### To 11.0.0

This version introduces the following breaking changes:

- Remove `initialClusterState` which was unreliable at detecting cluster state. From now on, each node will contact other members to determine cluster state. If no members are available and the data dir is empty, then it bootstraps a new cluster.
- Remove `removeMemberOnContainerTermination` which was unreliable at removing stale members during replica count updates. Instead, a pre-upgrade hook is added to check and remove stale members.
- Remove support for manual scaling with `kubectl` or autoscaler. Upgrading of any kind including increasing replica count must be done with `helm upgrade` exclusively. CD automation tools that respect Helm hooks such as ArgoCD can also be used.

### To 10.7.0

This version introduces image verification for security purposes. To disable it, set `global.security.allowInsecureImages` to `true`. More details at [GitHub issue](https://github.com/bitnami/charts/issues/30850).

### To 10.0.0

This major bump changes the following security defaults:

- `runAsGroup` is changed from `0` to `1001`
- `readOnlyRootFilesystem` is set to `true`
- `resourcesPreset` is changed from `none` to the minimum size working in our test suites (NOTE: `resourcesPreset` is not meant for production usage, but `resources` adapted to your use case).
- `global.compatibility.openshift.adaptSecurityContext` is changed from `disabled` to `auto`.

This could potentially break any customization or init scripts used in your deployment. If this is the case, change the default values to the previous ones.

### To 9.0.0

This version adds a new label `app.kubernetes.io/component=etcd` to the StatefulSet and pods. Due to this change, the StatefulSet will be replaced (as it's not possible to add additional `spec.selector.matchLabels` to an existing StatefulSet) and the pods will be recreated. To upgrade to this version from a previous version, you need to run the following steps:

1. Add new label to your pods

    ```console
    kubectl label pod my-release-0 app.kubernetes.io/component=etcd
    # Repeat for all etcd pods, based on configured .replicaCount (excluding the etcd snappshoter pod, if .disasterRecovery.enabled is set to true)
    ````

2. Remove the StatefulSet keeping the pods:

    ```console
    kubectl delete statefulset my-release --cascade=orphan
    ```

3. Upgrade your cluster:

    ```console
    helm upgrade my-release oci://REGISTRY_NAME/REPOSITORY_NAME/etcd --set auth.rbac.rootPassword=$ETCD_ROOT_PASSWORD
    ```

    > Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.

### To 8.0.0

This version reverts the change in the previous major bump ([7.0.0](https://github.com/bitnami/charts/tree/main/bitnami/etcd#to-700)). Now the default `etcd` branch is `3.5` again once confirmed by the [etcd developers](https://github.com/etcd-io/etcd/tree/main/CHANGELOG#production-recommendation) that this version is production-ready once solved the data corruption issue.

### To 7.0.0

This version changes the default `etcd` branch to `3.4` as suggested by [etcd developers](https://github.com/etcd-io/etcd/tree/main/CHANGELOG#production-recommendation). In order to migrate the data follow the official etcd instructions.

### To 6.0.0

This version introduces several features and performance improvements:

- The statefulset can now be scaled using `kubectl scale` command. Using `helm upgrade` to recalculate available endpoints is no longer needed.
- The scripts used for bootstrapping, runtime reconfiguration, and disaster recovery have been refactored and moved to the etcd container with two purposes: removing technical debt & improving the stability.
- Several parameters were reorganized to simplify the structure and follow the same standard used on other Bitnami charts:
  - `etcd.initialClusterState` is renamed to `initialClusterState`.
  - `statefulset.replicaCount` is renamed to `replicaCount`.
  - `statefulset.podManagementPolicy` is renamed to `podManagementPolicy`.
  - `statefulset.updateStrategy` and `statefulset.rollingUpdatePartition` are merged into `updateStrategy`.
  - `securityContext.*` is deprecated in favor of `podSecurityContext` and `containerSecurityContext`.
  - `configFileConfigMap` is deprecated in favor of `configuration` and `existingConfigmap`.
  - `envVarsConfigMap` is deprecated in favor of `extraEnvVars`, `extraEnvVarsCM` and `extraEnvVarsSecret`.
  - `allowNoneAuthentication` is renamed to `auth.rbac.allowNoneAuthentication`.
- New parameters/features were added:
  - `extraDeploy` to deploy any extra desired object.
  - `initContainers` and `sidecars` to define custom init containers and sidecars.
  - `extraVolumes`, `extraVolumeMounts` and `extraVolumeClaimTemplates` to define custom volumes, mount points and volume claim templates.
  - Probes can be now customized, and support to startup probes is added.
  - LifecycleHooks can be customized using `lifecycleHooks` parameter.
  - The default command/args can be customized using `command` and `args` parameters.
- Metrics integration with Prometheus Operator does no longer use a ServiceMonitor object, but a PodMonitor instead.

Consequences:

- Backwards compatibility is not guaranteed unless you adapt you **values.yaml** according to the changes described above.

### To 5.2.0

This version introduces `bitnami/common`, a [library chart](https://helm.sh/docs/topics/library_charts/#helm) as a dependency. More documentation about this new utility could be found [here](https://github.com/bitnami/charts/tree/main/bitnami/common#bitnami-common-library-chart). Please, make sure that you have updated the chart dependencies before executing any upgrade.

### To 5.0.0

[On November 13, 2020, Helm v2 support formally ended](https://github.com/helm/charts#status-of-the-project). This major version is the result of the required changes applied to the Helm Chart to be able to incorporate the different features added in Helm v3 and to be consistent with the Helm project itself regarding the Helm v2 EOL.

### To 4.4.14

In this release we addressed a vulnerability that showed the `ETCD_ROOT_PASSWORD` environment variable in the application logs. Users are advised to update immediately. More information in [this issue](https://github.com/bitnami/charts/issues/1901).

### To 3.0.0

Backwards compatibility is not guaranteed. The following notables changes were included:

- **etcdctl** uses v3 API.
- Adds support for auto disaster recovery.
- Labels are adapted to follow the Helm charts best practices.

To upgrade from previous charts versions, create a snapshot of the keyspace and restore it in a new etcd cluster. Only v3 API data can be restored.
You can use the command below to upgrade your chart by starting a new cluster using an existing snapshot, available in an existing PVC, to initialize the members:

```console
helm install new-release oci://REGISTRY_NAME/REPOSITORY_NAME/etcd \
  --set statefulset.replicaCount=3 \
  --set persistence.enabled=true \
  --set persistence.size=8Gi \
  --set startFromSnapshot.enabled=true \
  --set startFromSnapshot.existingClaim=my-claim \
  --set startFromSnapshot.snapshotFilename=my-snapshot.db
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.

### To 1.0.0

Backwards compatibility is not guaranteed unless you modify the labels used on the chart's deployments.
Use the workaround below to upgrade from versions previous to 1.0.0. The following example assumes that the release name is etcd:

```console
kubectl delete statefulset etcd --cascade=false
```

## License

Copyright &copy; 2025 Broadcom. The term "Broadcom" refers to Broadcom Inc. and/or its subsidiaries.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

<http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.