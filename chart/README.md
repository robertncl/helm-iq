# Sonatype IQ Server High Availability Helm Chart (AKS)

This Helm chart deploys a highly available Sonatype IQ Server cluster on Azure Kubernetes Service (AKS) using Azure Files (managed-csi) for shared storage and PostgreSQL for the database.

## Prerequisites
- Azure Kubernetes Service (AKS) cluster
- [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) (1.23+)
- [helm](https://helm.sh/docs/helm/) (3.9.3+)
- A PostgreSQL (10.7 or newer) database
- A Sonatype IQ Server license that supports the High Availability (HA) feature
- Azure Files CSI driver (managed-csi) enabled (default in AKS)

## Quick Start

1. **Create a namespace (optional):**
   ```sh
   kubectl create namespace iq-server
   ```

2. **Configure your values:**
   Edit `values.yaml` and set:
   - `license`: Path to your Sonatype IQ Server license file
   - `database.hostname`, `database.username`, `database.password`, `database.name`: Your PostgreSQL connection details
   - (Optional) Adjust `resources` and `replicas` as needed

3. **Install the chart:**
   ```sh
   helm install iq-server ./chart -n iq-server
   ```
   Or to upgrade:
   ```sh
   helm upgrade iq-server ./chart -n iq-server
   ```

## Storage (Azure Files)
This chart is pre-configured to use the `managed-csi` storage class, which is the default for Azure Files in AKS. It will create a PersistentVolumeClaim with `ReadWriteMany` access for shared storage between pods.

If you need a custom storage class, update `persistence.storageClassName` in `values.yaml`.

## Example values.yaml
```yaml
license: "/path/to/your/license.lic"
database:
  hostname: "my-postgres.postgres.database.azure.com"
  port: 5432
  name: "iq"
  username: "postgres"
  password: "your-password"
persistence:
  persistentVolumeClaimName: "iq-server-pvc"
  size: "10Gi"
  storageClassName: "managed-csi"
  accessModes:
    - ReadWriteMany
  csi:
    driver: "file.csi.azure.com"
replicas: 2
```

## Accessing the Application
- By default, the service is `ClusterIP`. To expose externally, create an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) or change `serviceType` to `LoadBalancer` in `values.yaml`.
- Example Ingress (using NGINX):
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: iq-server-ingress
    namespace: iq-server
    annotations:
      kubernetes.io/ingress.class: nginx
  spec:
    rules:
      - host: iq.example.com
        http:
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: iq-server-iq-server-service
                  port:
                    number: 8070
  ```

## Notes
- The chart is stripped of all AWS, EFS, EKS, and fluentd configuration.
- Only AKS, Azure Files, and PostgreSQL are supported out of the box.
- For advanced customizations, edit the templates as needed.

## Support
For Sonatype IQ Server documentation, visit: https://help.sonatype.com/iqserver
