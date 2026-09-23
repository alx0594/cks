# AppArmor

1. On the controlplane node, load the AppArmor profile:
   `root@controlplane:~# apparmor_parser -q /etc/apparmor.d/frontend`

2. The profile name used by this file is restricted-frontend (open the /etc/apparmor.d/frontend file to check).

   To verify that the profile was successfully loaded, use the aa-status command:

   ```bash
   root@controlplane:~# aa-status | grep restricted-frontend
   restricted-frontend
   ```

   Pod configuration:

   ```yaml
   spec:
     securityContext:
       appArmorProfile:
       type: Localhost
       localhostProfile: restricted-frontend ## Profile added
   ```

3. Use the service account with least privileges

   `serviceAccountName: frontend-default #Use the service account with least privileges`

# seccompProfile

Create a new pod named audit-nginx in the default namespace using the nginx:alpine image. Secure the syscalls that this pod can use by using the audit.json

1. Copy the audit.json seccomp profile to /var/lib/kubelet/seccomp/profiles on the controlplane node:
   **Copy the audit.json seccomp profile to:** `/var/lib/kubelet/seccomp/profiles on the controlplane node:`

Pod configuration:

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    labels:
    run: nginx
    name: audit-nginx
    spec:
    securityContext:
    seccompProfile:
        type: Localhost
        localhostProfile: profiles/audit.json # <<<
    containers:
    - image: nginx:alpine
        name: nginx
    ```

Result:

1 -> 19%

# Exam Mock 1 - 2 attempt

- **How do I find default seccomp directory?**

  On Kubernetes/Linux, the default directory for custom seccomp profiles is usually:
  `/var/lib/kubelet/seccomp/`

  To confirm the kubelet root directory:

  `ps aux | grep kubelet | grep -- --root-dir`

  Or:

  `systemctl cat kubelet | grep root-dir`

## Questions

- We have identified a few issues with our kubernetes setup and need your help in fixing them.

  - **Fix the following issues on kubelet:**

    For kubelet issues, run:

    `kube-bench --benchmark cis-1.10 --config-dir /opt/kube-bench/cfg run --targets node`

    Update the kubelet service file /usr/lib/systemd/system/kubelet.service and kubelet
    config YAML /var/lib/kubelet/config.yaml with the correct permissions:

  ```bash
   chmod 600 /usr/lib/systemd/system/kubelet.service
   chmod 600 /var/lib/kubelet/config.yaml
  ```

  - **Fix the following issues on etcd:**

    To fix the etcd issue, change the file ownership:

    sudo chown -R etcd:etcd /var/lib/etcd

    If the user and group are not present, add them:

  ```bash
   sudo groupadd --system etcd
   sudo useradd -s /sbin/nologin --system -g etcd etcd
  ```

  - **Fix the following issues on the controlplane node:**

    For controlplane and etcd issues, run:

    `kube-bench --benchmark cis-1.10 --config-dir /opt/kube-bench/cfg run --targets master`

    For controlplane fix, update the Controller Manager and Scheduler static pod definition file to make sure that the --profiling=false parameter is set. For this use the vi editor to edit both files:

    ```bash
    vi /etc/kubernetes/manifests/kube-controller-manager.yaml
    vi /etc/kubernetes/manifests/kube-scheduler.yaml
    ```

  - Kube-bench is installed, and its config files are available under /opt/kube-bench. Use the cis-1.10 benchmark with the current Kubernetes version.

Result:

1 -> 25%

# Exam Mock 1 - 3 attempt

### A pod has been created in the omni namespace, but it has a few issues that need to be addressed.

1. `aa-status | grep restrict` -> **_profile: restricted-frontend_**

2. To configure the pod with seccomp -> restricted-frontend

3. Check the AppArmor profile name

- `cat /etc/apparmor.d/frontend`

4. To verify Service Account with the minimum privileges

   ```bash
   kubectl auth can-i --list \
   --as=system:serviceaccount:omni:<service-account-name> \
   -n omni
   ```

### kube-bench

- I need to do kube-bench labs again.
- I should understand how I check only fails.

- check FAIL: `kube-bench --benchmark cis-1.10 --config-dir /opt/kube-bench/cfg run --targets master | grep '\[FAIL\]'`
- check issues using --check: `kube-bench --benchmark cis-1.10 --config-dir /opt/kube-bench/cfg run --targets master --check 1.1.12,1.2.5,1.3.2,1.4.1`

### FALCO

- Enable file_output in /etc/falco/falco.yaml on the controlplane node:

```yaml
file_output:
  enabled: true
  keep_alive: false
  filename: /opt/security_incidents/alerts.log
```

- Next, add the updated rule under the /etc/falco/falco_rules.local.yaml and hot reload the Falco service:

```yaml
- rule: Write below binary dir
  desc: an attempt to write to any file below a set of binary directories
  condition: >
    bin_dir and evt.dir = < and open_write
    and not package_mgmt_procs
    and not exe_running_docker_save
    and not python_running_get_pip
    and not python_running_ms_oms
    and not user_known_write_below_binary_dir_activities
  output: >
    File below a known binary directory opened for writing (user_id=%user.uid file_updated=%fd.name command=%proc.cmdline)
  priority: CRITICAL
  tags: [filesystem, mitre_persistence]
```

- To perform hot-reload falco use

`systemctl restart falco`

### FALCO IA STEP BY STEP

1. Localize a regra

`grep -Rni "File below a known binary directory opened for writing" /etc/falco`

2. Para visualizar o bloco completo:

```bash
grep -nA15 -B2 \
  "File below a known binary directory opened for writing" \
  /etc/falco/falco_rules.yaml
```

**This command is better because get complete block.**

```bash
 grep -A15 -B15 \
  "File below a known binary directory opened for writing"
  /etc/falco/falco_rules.yaml
```

3. Sobrescreva a regra no arquivo local

`vi /etc/falco/falco_rules.local.yaml`

No final do arquivo, adicione:

```yaml
- rule: Write below binary dir
  output: File below a known binary directory opened for writing (user_id=%user.uid file_updated=%fd.name command=%proc.cmdline)
  priority: CRITICAL
  override:
    output: replace
    priority: replace
```

4. Valide antes de reiniciar

`falco --validate /etc/falco/falco.yaml`

`falco -V /etc/falco/falco_rules.local.yaml`

5. Reinicie o Falco

`systemctl restart falco`

# Exam Mock 1 - 4 attempt

### Falco

1. Find field: `grep -nir "file_output" /etc/falco/falco.yaml`
2. In vi/vim, press Esc, type: `:277`

Result: 40%

# Exam Mock 1 - 5 attempt

## SBOM SPDX - Using bom

`bom generate --image-archive /root/ImageTarballs/<image_name>.tar --format json --output ~/bugged-fruit.spdx`

Result: 47%

# Exam Mock 1 - 6 attempt

## Admission Control

```yaml

  volumeMounts:
  - mountPath: /etc/admission-controllers
      name: admission-controllers
      readOnly: true

  volumes:
  - hostPath:
      path: /root/CKS/ImagePolicy/
      type: DirectoryOrCreate
    name: admission-controllers
```

# Exam Mock 1 - 7 attempt

- Verify question 12. What is IP used to hosts;
