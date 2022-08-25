# Postgres Operator for Kubernetes

- [Postgres Operator for Kubernetes](#postgres-operator-for-kubernetes)
  - [Prepare the Private Registry](#prepare-the-private-registry)
  - [Install the Helm Chart](#install-the-helm-chart)
  - [Deploy a Postgres Cluster](#deploy-a-postgres-cluster)
  - [Reference](#reference)

## Prepare the Private Registry

Make sure the machine you will be working from trusts the certificate of your private registry by specifying its issuing the CA certificate in the following command.

```bash
REG_CA_CERT=$(cat <<EOF
-----BEGIN CERTIFICATE-----
MIIFljCCBH6gAwIBAgITUwAAAUqnrxwJJKZXtQABAAABSjANBgkqhkiG9w0BAQsF
ADBPMRQwEgYKCZImiZPyLGQBGRYEZGVtbzEXMBUGCgmSJomT8ixkARkWB3RlcmFz
a3kxHjAcBgNVBAMTFXRlcmFza3ktREVNTy1EQy0wMS1DQTAeFw0yMjA3MjUwNjUy
NDNaFw0yNDA3MjUwNzAyNDNaMDIxEDAOBgNVBAoTB1RlcmFTa3kxHjAcBgNVBAMM
FSouaXQtdGFwLnRlcmFza3kuZGVtbzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC
AQoCggEBALnCw9gAvA/UnT/gYnVaV2BGRv76x4etNq030dEP4dklly64LXjiC9yo
iH9UKKqnLzhdrfESr+SKUC8J0kESiCFnZZbhGF188LnN/PONseVNOU2ALtzzDgrB
KyWV3bhknbpAc9LTQ7KXrvL1kCBWM0xkyKG2A4DMl+afmjFHNI8bMRdtt3fPRNIl
H4SJO9D2kxRsEgPkMxT9I5SKavufvdxpEL38Fh3slU3rsbuhLeFcytuMgg08/A5c
UEMvxwuyV/rPaWTKtlHBDN/uRoNuZZYMUL8DtH288T00vlPj92Z6ex8wDKT/lN5g
WDDXivcEZRmwwAtSUlipFhoTebKLmlECAwEAAaOCAoYwggKCMCAGA1UdEQQZMBeC
FSouaXQtdGFwLnRlcmFza3kuZGVtbzAdBgNVHQ4EFgQUHWBxgZypnHH5T166NMdl
7rsKOI8wHwYDVR0jBBgwFoAUcK6vN0MEoaHrpD82NgJ5ktWpMOUwgdoGA1UdHwSB
0jCBzzCBzKCByaCBxoaBw2xkYXA6Ly8vQ049dGVyYXNreS1ERU1PLURDLTAxLUNB
KDEpLENOPWRlbW8tZGMtMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZp
Y2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9dGVyYXNreSxEQz1k
ZW1vP2NlcnRpZmljYXRlUmV2b2NhdGlvbkxpc3Q/YmFzZT9vYmplY3RDbGFzcz1j
UkxEaXN0cmlidXRpb25Qb2ludDCByAYIKwYBBQUHAQEEgbswgbgwgbUGCCsGAQUF
BzAChoGobGRhcDovLy9DTj10ZXJhc2t5LURFTU8tREMtMDEtQ0EsQ049QUlBLENO
PVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3Vy
YXRpb24sREM9dGVyYXNreSxEQz1kZW1vP2NBQ2VydGlmaWNhdGU/YmFzZT9vYmpl
Y3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MA4GA1UdDwEB/wQEAwIGwDA8
BgkrBgEEAYI3FQcELzAtBiUrBgEEAYI3FQiB8dcEhoYrgpmVKYa9j3yBhJUoRYeG
u0eCmfEPAgFkAgEGMA8GA1UdJQQIMAYGBFUdJQAwFwYJKwYBBAGCNxUKBAowCDAG
BgRVHSUAMA0GCSqGSIb3DQEBCwUAA4IBAQBBk+GXWvJlPX+UWib4Mef7ur4S/p6d
DrCbMyRn1vOzcRxW9K0gct7auLnTkpOBJjykNxz2JqoUp47/FGvT1cS25BUtD6NQ
7RXdhI3CPdmTWrvmtLn/3fvk84IckohGkxZgUkrjr7HwA7chFc4QCF2ASaPlxGdr
rLLiTekr7yJay5oCC7d6kFG4KdZRvVPhnP3uGaTXpUPormspvGcAAgYgYgXYrp8O
xiw3tPIfBfqCNk7W+b8mtUl7xb5et8RDXcO9TQgtSDV7LGgbhDcsyhM4UdCuqpQ7
w8gaqL9FvHjP8tmhrTcpodo2imw+yOIdv35sf0pQHGpuD6XUjMHTZ88H
-----END CERTIFICATE-----
EOF
)
```

```bash
export REGISTRY_HOSTNAME=it-tkg-harbor.terasky.demo
sudo mkdir -p "/etc/docker/certs.d/${REGISTRY_HOSTNAME}"

sudo bash -c 'echo '"'$REG_CA_CERT'"' > /etc/docker/certs.d/'${REGISTRY_HOSTNAME}'/ca.crt'
sudo bash -c 'echo '"'$REG_CA_CERT'"' > /usr/local/share/ca-certificates/ca-cert.crt'

sudo update-ca-certificates
```

<!-- Create a public repository on the private registry.

For example, when using Harbor, you can run the following commands to create a project:

```bash
REGISTRY_USERNAME=admin
REGISTRY_PASSWORD=VMware1!
REGISTRY_PROJECT=postgres-operator

curl -k -H "Content-Type: application/json" \
-u "$REGISTRY_USERNAME:$REGISTRY_PASSWORD" \
-X POST "https://$REGISTRY_HOSTNAME/api/v2.0/projects" \
-d '{"project_name": "'"$REGISTRY_PROJECT"'", "public": false}'
``` -->

## Retrieve the Helm Chart for an Offline Deployment Using Relok8s

Add Helm repositories.

```bash
helm repo add postgres-operator https://opensource.zalando.com/postgres-operator/charts/postgres-operator
helm repo update
```

Cleanup any old copies of the chart in this directory.

```bash
rm -rf postgres-operator postgres-operator-*.tgz /tmp/postgres-operator
```

Rretrieve the Helm Chart version and pull it from the repository to a temporary location.

```bash
# Version pinning
CHART_VERSION="1.8.2"

# To get the latest chart version (not recommended - please watch out for breaking changes), run:
# CHART_VERSION=$(helm search repo postgres-operator/postgres-operator -o json | jq -r '.[0].version')

helm pull postgres-operator/postgres-operator --untar --version "$CHART_VERSION" --destination /tmp

# Hack - add additional image references for relok8s to process
VALUES_FILE=/tmp/postgres-operator/values.yaml
yq -i '.relok8s_custom.busybox.image = "busybox:latest"' "$VALUES_FILE"
yq -i '.relok8s_custom.postgres.image = "postgres:14"' "$VALUES_FILE"
```

Using the `relok8s` CLI, push the required container images to your container registry and get a modified copy of the Helm Chart.

```bash
relok8s chart move /tmp/postgres-operator \
-i relok8s-image-hints.yaml \
--repo-prefix "$TAP_REPO/postgres-operator" \
--registry "$PRIVATE_REGISTRY_HOSTNAME" \
--out *.tgz -y --force-push

tar -zxf postgres-operator-*.tgz
```

Cleanup temporary files.

```bash
rm -rf /tmp/postgres-operator
rm -f postgres-operator-*.tgz
```

## Install the Helm Chart

Install the Helm Chart.

```bash
helm upgrade -i postgres-operator postgres-operator -n postgres-system --create-namespace
```

Make sure the pods are running.

```bash
kubectl get pods -n postgres-system
```

## Deploy a Postgres Cluster

Create the `tap-gui-db` namespace.

```bash
kubectl apply -f tap-gui-db-namespace.yaml
```

Deploy the Postgres cluster.

```bash
kubectl apply -f tap-gui-postgres-db-cls.yaml
```

You can list the Postgres clusters.

```bash
kubectl get postgresql -n tap-gui-db
```

Example output:

```text
NAME             TEAM         VERSION   PODS   VOLUME   CPU-REQUEST   MEMORY-REQUEST   AGE     STATUS
tap-gui-db-cls   tap-gui-db   14        1      50Gi     10m           100Mi            7m12s   Running
```

You can also check out the deployed resources.

```bash
kubectl get pod,svc,sts,secret -n tap-gui-db
```

Get the Postgres cluster password.

```bash
export POSTGRES_PASSWORD=$(kubectl get secret postgres.tap-gui-db-cls.credentials.postgresql.acid.zalan.do -n tap-gui-db -o 'jsonpath={.data.password}' | base64 -d)
echo "$POSTGRES_PASSWORD"
```

Make a note of the password.

Use the `postgres-psql-test.yaml` manifest to test connectivity to the Postgres cluster.

```bash
envsubst < postgres-psql-test.yaml | kubectl apply -f -
```

Then inspect the log of the container. If a table of databases is returned, the connection to the database is successful.

```bash
kubectl logs -l app=postgres-psql-test -n tap-gui-db
```

Example output:

```text
                                     List of databases
   Name    |     Owner      | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------------+----------+-------------+-------------+-----------------------
 postgres  | postgres_owner | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres       | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |                |          |             |             | postgres=CTc/postgres
 template1 | postgres       | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |                |          |             |             | postgres=CTc/postgres
(3 rows)
```

Delete the test pod.

```bash
kubectl delete -f postgres-psql-test.yaml
```

## Reference

- [Postgres Operator](https://postgres-operator.readthedocs.io/en/latest/quickstart)
- [Postgres Operator on GitHub - Complete Postgres Manifest Example](https://github.com/zalando/postgres-operator/blob/master/manifests/complete-postgres-manifest.yaml)
- [Postgres Operator on GitHub - Minimal Postgres Manifest Example](https://github.com/zalando/postgres-operator/blob/master/manifests/minimal-postgres-manifest.yaml)
- [Postgres Environment Variables](https://www.postgresql.org/docs/current/libpq-envars.html)
- [Postgres Repository on Docker Hub](https://hub.docker.com/_/postgres)
- [VMware Tanzu - Asset Relocation Tool for Kubernetes (Relok8s)](https://github.com/vmware-tanzu/asset-relocation-tool-for-kubernetes)
