


# pocket-id




![Version](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fapi.github.com%2Frepos%2FRobSlgm%2Fpocket-id-helm%2Freleases%2Flatest&query=%24.name&label=Version)![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) 
![AppVersion](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fapi.github.com%2Frepos%2Fpocket-id%2Fpocket-id%2Freleases%2Flatest&query=%24.name&label=Version)

Pocket ID Helm chart for Kubernetes

### Requirements

A postgresql database is required but **not** installed by this helm chart. Install and configure with your prefered toolset (e.g. CloudnativePg)

Certain artifacts, such as user profile pictures and OIDC client logos, require blob storage. If you choose filesystem storage (as opposed to database or S3), persistent disk storage must be provisioned. If you opt for S3 storage, the relevant environment variables (e.g., S3_* settings) must be configured.

> [!WARNING]
> Changes to the `blobBackend` (`FILE_BACKEND` environment setting) require a data migration.

If you enable the **GeoLite2 Database**, filesystem storage is mandatory, regardless of the blob storage backend you have selected for other assets.

### Installing the Chart
To install the chart with the release name my-release:

```
helm install my-release oci://ghcr.io/robslgm/charts/pocket-id
```

### Uninstalling the Chart
To uninstall/delete the my-release deployment:

```
helm delete my-release
```
The command removes all the Kubernetes components associated with the chart and deletes the release.

## Advanced configuration

Use the config section of your `values.yaml` to setup Pocket ID. The full configuration can be supplied. Disable the config UI with the `uiConfigDisabled` setting.

[Official documentation](https://pocket-id.org/docs/configuration/environment-variables) of the configuration variables. 

Change environment variables (e.g., **LDAP_BIND_PASSWORD**) to **lower camel case** (e.g., **ldapBindPassword**) for use in the config section.


Example:

~~~yaml
config:
  appName: My Pocket ID instance
  # encryptionKey: (see secret)
  # maxmindLicenseKey: (see secret)
  # geoliteDbPath: ""
  uiConfigDisabled: true
  emailsVerified: true
  smtpHost: mail.example.com
  smtpPort: "465"
  smtpFrom: auth@example.com
  # smtpUser:  (see secret)
  # smtpPassword:  (see secret)
  # smtpPasswordFile: (not used)
  smtpTls: tls
  # smtpSkpCertVerify:
  allowUserSignups: withToken
  emailLoginNotificationEnabled: "true"
  emailOneTimeAccessAsAdminEnabled: "true"
  emailApiKeyExpirationEnabled: "true"
  emailOneTimeAccessAsUnauthenticatedEnabled: "false"
  accentColor: "oklch(0.25 0.18 80)"
  # see comment
  # signupDefaultUserGroupIds: '["83473555-2343-42bb-b4ba-879a8a7a29ab"]'
  logJson: "true"
  # logLevel: info
~~~

### Networking & IP Handling 

When using GeoLite2 and/or the feature to send an email to the user when they log in from a new device, ensure your infrastructure (Load Balancers, Firewalls, etc.) is configured to pass the real client IP to Pocket ID. 

While Pocket ID provides built-in rate limiting and Geo-lookup, you may also consider offloading these tasks to an infrastructure-level WAF. Handling these at the edge can be beneficial for your broader environment, as it secures the entire network path before traffic even reaches your individual applications.

### Distroless

Enable Distroless images by setting `image.distroless=true`. When enabled, ensure you configure the appropriate permissions within the `securityContext`, as these images do not run as root by default.

If you are using as `blobBackend` the filesystem, start first without distroless to allow proper creation of the filesystem and enable in a second step distroless.

> [!WARNING]
> **Filesystem Backends**:
> If you are using the **filesystem** as your `blobBackend`, please perform the initial deployment with Distroless disabled (the default). Once the necessary storage directories are initialized, you can enable Distroless in a subsequent update.

> [!NOTE] 
> **Database or S3 backends**:
> Consider setting `readOnlyRootFilesystem` to `true` in your security context and to drop all capabilities.

### Backup
While a backup solution is not natively bundled with this Helm chart, we strongly recommend implementing a strategy that covers the following areas:

- Ensure you have a scheduled backup process for your PostgreSQL instance, as it contains the core application state and user data.
- If you have configured Pocket ID to use the filesystem for blob storage, these directories must be backed up separately from the database to avoid data loss of user assets.
- If you choose to use Filesystem or S3 storage, be mindful of recovery consistency. Because the database contains references to the files in your blob storage, these two components are strictly linked. To minimize data mismatches during a restore, you should optimally plan to trigger your database and filesystem/S3 backups at the same time.

> [!NOTE]
> Opting to store blobs directly within the database simplifies your backup routine, as a single database dump will include both your application data and your stored artifacts.

Pocket ID 2.2 has an **experimental** import and export function (works also with distroless).
Example to store the export on your local machine:

~~~shell
kubectl exec -i -n pocket-id pocket-id-0  --  /app/pocket-id export --path - >C:\temp\pocket-id.zip
~~~

## Source Code

* <https://github.com/RobSlgm/pocket-id-helm>
* <https://github.com/pocket-id/pocket-id>

Originally based on https://github.com/hobbit44/pocket-id-helm/commits/main/ with several fixes from https://github.com/matslarson/pocket-id-helm



## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` |  |
| appUrl | string | `"auth.example.com"` | Set Url for Pocket ID instance Set's Pocket ID `APP_URL` property with prepended https:// and adds hostname to HttpRoute |
| blobBackend | string | `"database"` | Choose backend for large binary objects (e.g. logo, background images, ...) Valid values are: "filesystem", "database" or "s3". Set's Pocket ID `FILE_BACKEND` property |
| config | object | `{}` | Pocket-id configuration variables For more information and full list of options see: https://pocket-id.org/docs/configuration/environment-variables |
| connectionString | object | `{}` | Secret for a database connection string  Supply name of secret and key containing a fully qualified uri for the database connection. *Using config to supply the database connection string is not recommended* |
| existingSecret | string | `""` | Name of an existing secret containing any environment variables Add secret containing **ENCRYPTION_KEY** and if applicable SMTP_USER, SMTP_PASSWORD, ... |
| fullnameOverride | string | `""` | The full resource name override |
| httpRoute | object | {} | Expose the service via gateway-api HTTPRoute Requires Gateway API resources and suitable controller installed within the cluster (see: https://gateway-api.sigs.k8s.io/guides/) |
| httpRoute.annotations | object | `{}` | HTTPRoute annotations. |
| httpRoute.enabled | bool | `false` | Enables a Gateway API HTTPRoute (use either ingress or httpRoute) |
| httpRoute.hostnames | list | `["chart-example.local"]` | Hostnames matching HTTP header. |
| httpRoute.parentRefs | list | {}   | Which Gateways this Route is attached to. |
| httpRoute.rules | list | {}   | List of rules and filters applied. |
| image.distroless | bool | `false` | Overrides the image tag whose default is the chart version. |
| image.pullPolicy | string | `"IfNotPresent"` | The pull policy for images |
| image.repository | string | `"ghcr.io/pocket-id/pocket-id"` | The container image to run |
| image.tag | string | `""` |  |
| imagePullSecrets | list | `[]` | This is for the secretes for pulling an image from a private repository more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/ |
| ingress.annotations | object | `{}` | Annotations for the ingress |
| ingress.className | string | `""` |  |
| ingress.enabled | bool | `false` | Enables an ingress for the application  (use either ingress or httpRoute) |
| ingress.hosts[0].host | string | `"chart-example.local"` |  |
| ingress.hosts[0].paths[0].path | string | `"/"` |  |
| ingress.hosts[0].paths[0].pathType | string | `"Prefix"` |  |
| ingress.tls | list | `[]` | Adds tls to the ingress |
| livenessProbe.httpGet.path | string | `"/health"` | Healthcheck endpoint |
| livenessProbe.httpGet.port | string | `"http"` |  |
| livenessProbe.initialDelaySeconds | int | `60` |  |
| nameOverride | string | `""` | The resource name suffix |
| nodeSelector | object | `{}` |  |
| persistence.accessMode | string | `"ReadWriteOnce"` |  |
| persistence.annotations | object | `{}` |  |
| persistence.enabled | bool | `true` | Persist data to a persistent volume |
| persistence.existingClaim | string | `""` | A manually managed Persistent Volume and Claim Requires persistence.enabled: true If defined, PVC must be created manually before volume will be bound |
| persistence.size | string | `"8Gi"` |  |
| persistence.storageClass | string | `""` | Persistent Volume Storage Class If defined, storageClassName: <storageClass> If set to "-", storageClassName: "", which disables dynamic provisioning If undefined (the default) or set to null, no storageClassName spec is   set, choosing the default provisioner.  (gp2 on AWS, standard on   GKE, AWS & OpenStack) |
| podAnnotations | object | `{}` |  |
| podLabels | object | `{}` |  |
| podSecurityContext | object | `{}` |  |
| readinessProbe.httpGet.path | string | `"/health"` | Healthcheck endpoint |
| readinessProbe.httpGet.port | string | `"http"` |  |
| readinessProbe.initialDelaySeconds | int | `15` |  |
| resources | object | `{}` | Specifiy resources for the pod |
| securityContext | object | `{}` | Security context.  If supplied runAsUser is used for Pocket ID `PUID`. If supplied runAsGroup is used for Pocket ID `PGID` |
| serviceAccount.annotations | object | `{}` | Annotations to add to the service account |
| serviceAccount.automount | bool | `true` | Automatically mount a ServiceAccount's API credentials? |
| serviceAccount.create | bool | `true` | Specifies whether a service account should be created |
| serviceAccount.name | string | `""` | The name of the service account to use. If not set and create is true, a name is generated using the fullname template |
| tolerations | list | `[]` |  |
| volumeMounts | list | `[]` | Additional volumeMounts on the pod definition. |
| volumes | list | `[]` | Additional volumes on the pod definition. |


----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)
