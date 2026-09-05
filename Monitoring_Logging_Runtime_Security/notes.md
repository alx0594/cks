# 05/09/2026

## Falco Overview and Installation

- Falco Architeture

  ![Falco Architeture](images/falco-architeture.png)

- Install as a Package

![Falco as a Package](images/install-falco-as-a-package.png)

- Falco Install DaemonSet

![Falco As a DaemonSet](images/falco-as-a-daemonset.png)

## Use Falco to Detect Threats

- `systemctl status falco`

- `journalctl -fu falco`

While I’m running Kubernetes commands, I can see the events in the output of `journalctl -fu falco`.

![journalctl -fu falco](images/journalctl-falco-and-k8s-commands.png)

- Falco Rules Example

![Falco Rules](images/falco-rules.png)

- Real Falco Rules

![Real Falco Rules](images/real-falco-rules.png)

## Falco Configuration Files

![Falco File Config](images/falco-file-config.png)

- Falco Yaml

![Falco Yaml](images/falco-yaml.png)

- Falco Output

![Falco Output](images/falco-output.png)

## Mutable vs Immutable Infrastructure

- Mutable -> Update cluster regurlarly

- Immultable -> Doesnt update cluster

## Ensure Immutability of Containers at Runtime

- FileSystem ReadOnly: true
- userRunNonPrivilegies: true

## Kubernets Auditing

- Audit Policy

![Audit Policy](images/audit-policy.png)

- Kube-apiserver config audit log

![kube-apiserver audit log](images/kube-apiserver-config-auditlogs.png)
