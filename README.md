# k0s cluster
Guide to deploy an 1 controller + 1 worker kubernetes cluster using k0s
### prerequisites:
- A homelab:
    - 2 hosts:
        - ubuntu server 22.04
        - fixed IP address
    - 1 router with access to internet
    > Note: the homelab can be replaced by 2 server on the cloud
- 1 main PC:
    - golang installed
    - connectivity to homelab network

## how to setup
1. On main PC install k0sctl and kubectl:
```bash
    go install github.com/k0sproject/k0sctl@latest
    # see more in https://github.com/k0sproject/k0sctl
    sudo apt install kubectl # for debian family OS
    sudo dnf install kubectl # for redhat family OS
```
2. Generate configuration file template
```bash
    k0sctl init > k0sctl.yaml
```
3. Edit configuration file
```bash
    nano k0sctl.yaml
```
```yaml
    apiVersion: k0sctl.k0sproject.io/v1beta1
    kind: Cluster
    metadata:
      name: k0s-cluster
      user: admin
    spec:
      hosts:
      - ssh:
          address: <ip-1> #server1 ip address
          user: root
          port: 22
          keyPath: ~/.ssh/id_rsa
        role: controller
      - ssh:
          address: <ip-2> #server2 ip address
          user: root
          port: 22
          keyPath: ~/.ssh/id_rsa
        role: worker
```
4. Create ssh keys and safe
```bash
    ssh-keygen -t rsa -b 4096
```
5. copy public key's content (*~/.ssh/id_rsa.pub*) to each server's root directory ".ssh/authorized_keys"

6. Apply the configuration file
```bash
    k0sctl apply --config k0sctl.yaml
```
7. Generate the configuration file for cluster management
```bash
    k0sctl kubeconfig > kubeconfig
    export KUBECONFIG=/path/to/kubeconfig
```
8. Test kubectl
```bash
    kubectl get pods -A
    # should display something similar to the next output:
    # NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
    # kube-system   calico-kube-controllers-5f6546844f-w8x27   1/1     Running   0          3m50s
    # kube-system   calico-node-vd7lx                          1/1     Running   0          3m44s
    # kube-system   coredns-5c98d7d4d8-tmrwv                   1/1     Running   0          4m10s
    # kube-system   konnectivity-agent-d9xv2                   1/1     Running   0          3m31s
    # kube-system   kube-proxy-xp9r9                           1/1     Running   0          4m4s
    # kube-system   metrics-server-6fbcd86f7b-5frtn            1/1     Running   0          3m51s
```
9. Done! Now you have a lightweight kubernetes cluster configured