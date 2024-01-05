# Cert-Manager ACME DNS01 Webhook Solver for Linode DNS Manager

[![Go Report Card](https://goreportcard.com/badge/github.com/codestation/cert-manager-webhook-linode)](https://goreportcard.com/report/github.com/codestation/cert-manager-webhook-linode)
[![Releases](https://img.shields.io/github/v/release/codestation/cert-manager-webhook-linode?include_prereleases)](https://github.com/codestation/cert-manager-webhook-linode/releases)
[![LICENSE](https://img.shields.io/github/license/codestation/cert-manager-webhook-linode)](https://github.com/codestation/cert-manager-webhook-linode/blob/master/LICENSE)

A webhook to use [Linode DNS
Manager](https://www.linode.com/docs/platform/manager/dns-manager) as a DNS01
ACME Issuer for [cert-manager](https://github.com/cert-manager/cert-manager).

## Installation

```bash
helm install cert-manager-webhook-linode \
  --namespace cert-manager \
  https://github.com/codestation/cert-manager-webhook-linode/releases/download/v0.3.0/cert-manager-webhook-linode-v0.3.0.tgz
```

## Usage

### Create Linode API Token Secret

```bash
kubectl create secret generic linode-credentials \
  --namespace=cert-manager \
  --from-literal=token=<LINODE TOKEN>
```

### Create Issuer

#### Cluster-wide Linode API Token

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: example@example.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - dns01:
      webhook:
        solverName: linode
        groupName: acme.megpoid.dev
```

By default, the Linode API token used will be obtained from the
linode-credentials Secret in the same namespace as the webhook.


#### Per Namespace Linode API Tokens

If you would prefer to use separate Linode API tokens for each namespace (e.g.
in a multi-tenant environment):

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-staging
  namespace: default
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: example@example.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - dns01:
      webhook:
        solverName: linode
        groupName: acme.megpoid.dev
        config:
          apiKeySecretRef:
            name: linode-credentials
            key: token
```

## Development

### Running the test suite

Conformance testing is achieved through Kubernetes emulation via the
kubebuilder-tools suite, in conjunction with real calls to the Linode API on an
test domain, using a valid API token.

The test configures a cert-manager-dns01-tests TXT entry, attempts to verify its
presence, and removes the entry, thereby verifying the Prepare and CleanUp
functions.

Run the test suite with:

```bash
./scripts/fetch-test-binaries.sh
export LINODE_TOKEN=$(echo -n "<your API token>" | base64 -w 0)
envsubst < testdata/linode/secret.yaml.example > testdata/linode/secret.yaml
TEST_ZONE_NAME=yourdomain.com. make verify
```
