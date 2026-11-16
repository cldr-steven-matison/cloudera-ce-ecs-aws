<!-- Copyright 2026 Cloudera, Inc.

     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at

         https://www.apache.org/licenses/LICENSE-2.0

     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License. -->

# Cloudera On Premise Community Edition

Ansible + Terraform automation for deploying Cloudera Private Cloud on AWS. Constructs a ring-fenced, 10-node Base cluster and 4-node ECS cluster accessible via SSH and reverse HTTPS proxies.

- Self-contained DNS, Kerberos, database, and TLS services
- ACME-managed TLS termination on the reverse proxy
- Cloudera Manager with Kerberos and Auto-TLS
- Ozone-enabled base and ECS cluster deployments
- Fully idempotent — run repeatedly with no unintended state changes

## This fork

This is a fork of [`cloudera-labs/cloudera-ce-ecs-aws`](https://github.com/cloudera-labs/cloudera-ce-ecs-aws).
`main` matches upstream. Branch `srm-561` is upstream plus three changes I needed to deploy it from a Mac:

- `ansible-navigator.yml` pins the execution environment by digest instead of `:latest`, so every run
  uses the same image.
- `patches/RedHat-pre.yml` is mounted over the `cloudera.exe` Caddy role's install tasks. The stock role
  runs `dnf copr enable @caddy/caddy`, which RHEL 9 EC2 hosts without a Red Hat subscription cannot
  reach. The patch builds a local Caddy RPM from the upstream release binary instead.
- `playbooks/ecs-cluster.yml` runs the two ECS-master kubernetes plays on `/usr/bin/python3`. Without the
  change they fail with `Failed to import the required Python library (kubernetes)`, because the module
  runs on `/usr/libexec/platform-python` while the library is installed for Python 3.11.

The launcher, preflight, status and teardown scripts for this branch are in
[`srm-runbook-cloudera-ce-ecs-aws`](https://github.com/cldr-steven-matison/srm-runbook-cloudera-ce-ecs-aws).
Everything else in this README is upstream's.

## Quick Start

```bash
cp config-template.yml config.yml    # Set name_prefix, infra_region, common_password, owner
ansible-navigator run playbooks/infrastructure.yml playbooks/services.yml playbooks/cms.yml playbooks/ozone-cluster.yml playbooks/ecs-cluster.yml -e @config.yml
```

## Documentation

> **[View the full documentation site](https://cloudera-labs.github.io/cloudera-ce-ecs-aws/)**

Setup instructions, configuration reference, and operational guides are also available in the **[docs/](docs/)** directory.

| Section | Description |
|---------|-------------|
| [Getting Started](docs/getting-started/) | Prerequisites, credentials, configuration |
| [Deployment](docs/deployment/) | Infrastructure, services, Cloudera Manager, clusters, and optional [data services](docs/deployment/data-services.md) (AI, Data Engineering, Data Warehouse) |
| [Operations](docs/operations/) | Accessing endpoints, SSH, tear down |
| [Reference](docs/reference/) | Architecture, execution environment |
