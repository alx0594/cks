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
