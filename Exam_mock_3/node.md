## Question 1

- Change the ownership of the docker file:

`sudo chown root:root /var/run/docker.sock`

- Then add --group=root to the ExecStart of docker systemd file:

`sudo systemctl edit docker`

```bash
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --group=root
```

- Then reload the docker daemon:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart docker
```

- To remove the TCP external connections, modify the /etc/docker/daemon.json to remove the tcp section so that the file looks like this:

```json
{
  "hosts": ["unix:///var/run/docker.sock"]
}
```

- Then restart docker again.

## Question 5

**Task**

- Delete all pods from the alpha namespace that are not immutable.
  Note: A pod is considered non-immutable if it uses elevated privileges or can store state inside the container.

**Solution**

Pod solaris is immutable as it have `readOnlyRootFilesystem: true` so it should not be deleted.

Pod `sonata` is running with privileged: true and `triton` doesn't define `readOnlyRootFilesystem: true` so both break the concept of immutability and should be deleted.

## Question 8

```yaml
authentication:
  anonymous:
    enabled: false
```

```yaml
authorization:
  mode: Webhook
```

`sudo systemctl restart kubelet`

- To make the cluster info inaccessible without the kubeconfig flag:

```bash
mv ~/.kube/config ~/.kube/config.bak
unset KUBECONFIG
```

- Delete the custom-role using this kubeconfig file:
  `kubectl delete role custom-role -n delta --kubeconfig=/root/custom-config/admin.conf`

## Question 13

- First run the following command from the controlplane node:
  `kubectl get nodes`
  `ssh node02`

- Use any text editor you prefer to open the file that defines the Kubernetes apt repository.

  `vim /etc/apt/sources.list.d/kubernetes.list`
  `deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /`

- After making changes, save the file and exit from your text editor. Proceed with the next instruction.

  ```bash
    sudo apt-get update
    apt-cache madison kubeadm
  ```

- Momentarily go back to cluster1-controlplane node to drain the worker node:
  `kubectl drain node02 --ignore-daemonsets --delete-emptydir-data`

- Based on the version information displayed by `apt-cache madison`, it indicates that for Kubernetes version `1.35.0`, one of the available package version is `1.35.0-1.1`. Therefore, to install kubeadm for Kubernetes `v1.35.0`, use the following command:

  `sudo apt-get install -y kubeadm=1.35.0-1.1`

- Run the following command to upgrade the node:

  `sudo kubeadm upgrade node`

- Now, unhold and then upgrade the kubelet and kubectl versions:

```bash
sudo apt-mark unhold kubelet kubectl
sudo apt-get install --allow-change-held-packages -y kubelet=1.35.0-1.1 kubectl=1.35.0-1.1
```

- Run the following commands to refresh the systemd configuration and apply changes to the Kubelet service:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

- Go back to the controlplane node again and uncordon node02:
  `kubectl uncordon node02`

- Finally verify the version upgrade:

`kubectl get nodes`

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.35.0-1.1' && \
sudo apt-mark hold kubeadm
