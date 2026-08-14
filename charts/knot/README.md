# knot

![Version: 0.2.0](https://img.shields.io/badge/Version-0.2.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v1.16.1-alpha](https://img.shields.io/badge/AppVersion-v1.16.1--alpha-informational?style=flat-square)

A knot is the git server of Tangled, the code collaboration platform built on ATProto.

**Homepage:** <https://docs.tangled.org/knots>

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| QJOLY | <github@une-tasse-de.cafe> |  |

## Source Code

* <https://tangled.org/tangled.org/knot-docker>
* <https://tangled.org/tangled.org/core>

## Requirements

Kubernetes: `>= 1.18`

| Repository | Name | Version |
|------------|------|---------|
| https://rubxkube.github.io/common-charts | common | v0.6.2 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| common.deployment.additionalContainers[0].command[0] | string | `"/bin/sh"` |  |
| common.deployment.additionalContainers[0].command[1] | string | `"-c"` |  |
| common.deployment.additionalContainers[0].command[2] | string | `"set -eu\napk add --no-cache sqlite >/dev/null\nmkdir -p /app/backup\n# the knot creates the database on its first boot, and this sidecar\n# can win that race\nwhile [ ! -f /app/knotserver.db ]; do\n  echo \"waiting for /app/knotserver.db\"\n  sleep 5\ndone\nwhile true; do\n  rm -f /app/backup/knotserver.db.tmp\n  sqlite3 file:/app/knotserver.db?mode=ro \\\n    \"VACUUM INTO '/app/backup/knotserver.db.tmp'\"\n  mv /app/backup/knotserver.db.tmp /app/backup/knotserver.db\n  tar cf /app/backup/ssh-host-keys.tar -C /etc/ssh/keys .\n  echo \"$(date -Iseconds) dumped $(stat -c %s /app/backup/knotserver.db) bytes\"\n  sleep \"${DUMP_INTERVAL_SECONDS}\"\ndone\n"` |  |
| common.deployment.additionalContainers[0].env[0].name | string | `"DUMP_INTERVAL_SECONDS"` |  |
| common.deployment.additionalContainers[0].env[0].value | string | `"21600"` |  |
| common.deployment.additionalContainers[0].image | string | `"alpine:3.22"` |  |
| common.deployment.additionalContainers[0].name | string | `"backup-dump"` |  |
| common.deployment.additionalContainers[0].resources.limits.memory | string | `"128Mi"` |  |
| common.deployment.additionalContainers[0].resources.requests.cpu | string | `"10m"` |  |
| common.deployment.additionalContainers[0].resources.requests.memory | string | `"32Mi"` |  |
| common.deployment.additionalContainers[0].volumeMounts[0].mountPath | string | `"/app"` |  |
| common.deployment.additionalContainers[0].volumeMounts[0].name | string | `"server"` |  |
| common.deployment.additionalContainers[0].volumeMounts[1].mountPath | string | `"/etc/ssh/keys"` |  |
| common.deployment.additionalContainers[0].volumeMounts[1].name | string | `"keys"` |  |
| common.deployment.additionalContainers[0].volumeMounts[1].readOnly | bool | `true` |  |
| common.deployment.cpuLimit | string | `nil` |  |
| common.deployment.cpuRequest | string | `"100m"` |  |
| common.deployment.memoryLimit | string | `"1Gi"` |  |
| common.deployment.memoryRequest | string | `"256Mi"` |  |
| common.deployment.podSecurityContext.fsGroup | int | `1000` |  |
| common.deployment.port | int | `5555` |  |
| common.deployment.strategy.type | string | `"Recreate"` |  |
| common.extraResources[0].apiVersion | string | `"v1"` |  |
| common.extraResources[0].kind | string | `"Service"` |  |
| common.extraResources[0].metadata.name | string | `"knot-ssh"` |  |
| common.extraResources[0].spec.ports[0].name | string | `"ssh"` |  |
| common.extraResources[0].spec.ports[0].port | int | `22` |  |
| common.extraResources[0].spec.ports[0].protocol | string | `"TCP"` |  |
| common.extraResources[0].spec.ports[0].targetPort | int | `22` |  |
| common.extraResources[0].spec.selector.app | string | `"knot"` |  |
| common.extraResources[0].spec.type | string | `"LoadBalancer"` |  |
| common.hpa.avgCpuUtilization | int | `50` |  |
| common.hpa.enabled | bool | `false` |  |
| common.hpa.maxReplicas | int | `2` |  |
| common.hpa.minReplicas | int | `1` |  |
| common.image.pullPolicy | string | `"IfNotPresent"` |  |
| common.image.repository | string | `"ghcr.io/qjoly/knot"` |  |
| common.image.repositorySettings.isPrivate | bool | `false` |  |
| common.image.repositorySettings.secretName | string | `nil` |  |
| common.image.tag | string | `"v1.16.1-alpha"` |  |
| common.ingress.enabled | bool | `false` |  |
| common.ingress.hostName | string | `"knot.example.com"` |  |
| common.ingress.ingressClassName | string | `nil` |  |
| common.ingress.tls.enabled | bool | `true` |  |
| common.ingress.tls.secretName | string | `""` |  |
| common.livenessProbe.failureThreshold | int | `3` |  |
| common.livenessProbe.httpGet.path | string | `"/"` |  |
| common.livenessProbe.httpGet.port | int | `5555` |  |
| common.livenessProbe.initialDelaySeconds | int | `30` |  |
| common.livenessProbe.periodSeconds | int | `60` |  |
| common.livenessProbe.timeoutSeconds | int | `5` |  |
| common.livenessProbeEnabled | bool | `true` |  |
| common.name | string | `"knot"` |  |
| common.persistence.enabled | bool | `true` |  |
| common.persistence.volumes[0].containerMount | string | `"/home/git/repositories"` |  |
| common.persistence.volumes[0].name | string | `"repositories"` |  |
| common.persistence.volumes[0].pvcClaim | string | `""` |  |
| common.persistence.volumes[0].size | string | `"20Gi"` |  |
| common.persistence.volumes[0].storageClassName | string | `""` |  |
| common.persistence.volumes[1].containerMount | string | `"/app"` |  |
| common.persistence.volumes[1].name | string | `"server"` |  |
| common.persistence.volumes[1].pvcClaim | string | `""` |  |
| common.persistence.volumes[1].size | string | `"2Gi"` |  |
| common.persistence.volumes[1].storageClassName | string | `""` |  |
| common.persistence.volumes[2].containerMount | string | `"/etc/ssh/keys"` |  |
| common.persistence.volumes[2].name | string | `"keys"` |  |
| common.persistence.volumes[2].pvcClaim | string | `""` |  |
| common.persistence.volumes[2].size | string | `"64Mi"` |  |
| common.persistence.volumes[2].storageClassName | string | `""` |  |
| common.readinessProbe.failureThreshold | int | `3` |  |
| common.readinessProbe.httpGet.path | string | `"/"` |  |
| common.readinessProbe.httpGet.port | int | `5555` |  |
| common.readinessProbe.initialDelaySeconds | int | `10` |  |
| common.readinessProbe.periodSeconds | int | `15` |  |
| common.readinessProbe.timeoutSeconds | int | `3` |  |
| common.readinessProbeEnabled | bool | `true` |  |
| common.service.containerPort | int | `5555` |  |
| common.service.enabled | bool | `true` |  |
| common.service.extraLabels | object | `{}` |  |
| common.service.servicePort | int | `5555` |  |
| common.service.type | string | `"ClusterIP"` |  |
| common.startupProbe | object | `{}` |  |
| common.startupProbeEnabled | bool | `false` |  |
| common.tests.classicHttp.enabled | bool | `true` |  |
| common.tests.curlHostHeader.enabled | bool | `true` |  |
| common.tests.curlHostHeader.path | string | `"/xrpc/sh.tangled.knot.version"` |  |
| common.variables.nonSecret.HOME | string | `"/home/git"` |  |
| common.variables.nonSecret.KNOT_REPO_SCAN_PATH | string | `"/home/git/repositories"` |  |
| common.variables.nonSecret.KNOT_SERVER_ADMIN_SECRET | string | `""` |  |
| common.variables.nonSecret.KNOT_SERVER_DB_PATH | string | `"/app/knotserver.db"` |  |
| common.variables.nonSecret.KNOT_SERVER_HOSTNAME | string | `"knot.example.com"` | REQUIRED. Public FQDN of the knot. It becomes the knot's atproto identity (`did:web:<hostname>`) and the audience of every service-auth JWT, so it must match the domain registered on the appview exactly. |
| common.variables.nonSecret.KNOT_SERVER_OWNER | string | `"did:plc:000000000000000000000000"` | REQUIRED. ATProto DID of the owner. Resolve yours with `curl "https://public.api.bsky.app/xrpc/com.atproto.identity.resolveHandle?handle=<handle>"` |
| common.variables.secret | object | `{}` |  |
| define | int | `5555` |  |

