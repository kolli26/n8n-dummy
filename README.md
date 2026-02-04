# n8n Helm Chart (n8n-app)

This chart deploys n8n with PostgreSQL, Redis.

## Prerequisites

- k3s cluster (https://docs.k3s.io/quick-start)
- Helm 3 (https://helm.sh/docs/intro/install/)
- cert-manager (for TLS) (https://cert-manager.io/docs/installation/helm/#installing-from-the-oci-registry)

## Configure values

Update the values in [values.yaml](values.yaml) before installation.

### Required basics

- `subdomain` and `domainName` is the subdomain and domain name for your n8n instance.

### n8n settings

- `n8n.tag` controls the n8n image version.
- `n8n.executionsMode` is set to `queue`, which enables workers and external runners.
- `n8n.worker.replicas` control worker scaling.

### Database and Redis

- PostgreSQL and Redis service hostnames are set to `postgres-svc` and `redis-svc` by default.

### Secrets (generate your own)

The chart expects these secrets in [values.yaml](values.yaml):

- `secrets.n8nEncryptionKey` (generate with `openssl rand -base64 32`)
- `secrets.n8nRunnersAuthToken` (generate with `openssl rand -base64 32`)
- `secrets.dbPostgresPassword` (generate with `openssl rand -base64 16`)

### TLS with cert-manager (Cloudflare DNS01)

The chart includes a standard ACME solver configuration.

Tls can be disabled by setting `n8n.ingress.tls` to `false`.

Set these in `infra.issuer`:

- `name` and `privateKeyName` for the ClusterIssuer. Name should reflect the DNS provider and challenge type (e.g. `letsencrypt-dns01`).
- `dnsSecretsNamespace` where the Cloudflare token secret lives (default: `cert-manager`).
- `apiTokenSecret` is the **token value**, stored in the Cloudflare secret. this is only needed when using DNS01 challenge else set it to `null`.
- `email` for ACME registration.
- `solvers` uses DNS01 (Cloudflare) and HTTP01 (Traefik) defaults. Modify if needed. Read more at https://cert-manager.io/docs/configuration/acme/.

## Install

Example install into a namespace named `n8n`:

```bash
kubectl create namespace n8n
helm install n8n-app . -n n8n
```

## Upgrade after changing values

```bash
helm upgrade n8n-app . -n n8n
```

## Access

After installation, n8n is available at:

```
https://<subdomain>.<domainName>
```

## Uninstall

```bash
helm uninstall n8n-app -n n8n
kubectl delete namespace n8n
```