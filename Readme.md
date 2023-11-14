#
using in to folder have Vagrantfile
```
vagrant ssh -- -vvv
```

if you catch "Permission denied (publickey,gssapi-keyex,gssapi-with-mic)"

## Solution when catch Error "[preflight] Running pre-flight checks error execution phase preflight: [preflight] Some fatal errors occurred: [ERROR CRI]: container runtime is not running: output: time="2020-09-24T11:49:16Z" level=fatal msg="getting status of runtime failed: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService", error: exit status 1"

down version of kubernates
```
sudo yum downgrade kubeadm-1.16.9 kubernetes-cni-0.7.5 kubelet-1.16.9 kubectl-1.16.9
```

```
kubectl apply -f https://docs.projectcalico.org/archive/v3.12/manifests/calico.yaml

```

```
rm /etc/containerd/config.toml
systemctl restart containerd
systemctl enable containerd 
kubeadm init
```

## Error
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

fix

```
ssh-keygen -R 192.168.3.10
```