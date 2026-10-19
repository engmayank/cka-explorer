# Day 1: Troubleshooting, nodes and control plane

- **Date:** 20 Oct 2026
- **Domain:** Troubleshooting (30% of the CKA)
- **Environment:** Killercoda Kubernetes playground (`controlplane` + `node01`)
- **Goal:** break each control plane and node component on purpose, then find and fix it using only the cluster's own tools.

## Drills

| # | I broke | Symptom | How I found it | Fix |
|---|---|---|---|---|
| 1 | Moved `kube-scheduler.yaml` out of `/etc/kubernetes/manifests` | New Pod `t1` stuck `Pending`, `describe` showed no events | `k get pods -n kube-system \| grep scheduler` returned nothing; `ls` on the manifests dir | Moved the file back; kubelet recreated the Pod |
| 2 | Bad value in the scheduler manifest | Scheduler Pod unhealthy, `t2` stuck | `k logs -n kube-system kube-scheduler-controlplane` | Edited the manifest back |
| 3 | Controller-manager config | `k create deploy web --replicas=3` made a Deployment but no ReplicaSet or Pods | `k get deploy,rs`, then looked at the manifest | Edited the manifest back |
| 4 | `kube-apiserver.yaml` flag | `kubectl` stopped working (connection refused) | `crictl ps -a \| grep apiserver`, then `crictl logs <id>` | Edited the manifest, kubelet restarted the Pod |
| 5 | `etcd.yaml` | `kubectl` failed, etcd container exited | `crictl ps -a \| grep etcd`, then `crictl logs <id>` | Edited the manifest back |
| 6 | Stopped kubelet on `node01` | Node went `NotReady` | `k describe node node01` (conditions), then `systemctl status kubelet` on the node | `systemctl start kubelet` |
| 7 | `server:` port in `/etc/kubernetes/kubelet.conf` on `node01` | Node `NotReady` | `journalctl -u kubelet -n 30 --no-pager` on the node | Restored the port, restarted kubelet |
| 8 | Stopped containerd on `node01` | Node `NotReady`, `crictl ps` could not connect | `crictl ps` errors, kubelet journal | `systemctl start containerd`, then `systemctl restart kubelet` |

## What went well

- For the API server and etcd I used the right order without prompting: `crictl ps -a`, then `crictl logs`, then fix the manifest.
- I restored files into `/etc/kubernetes/manifests` and let the kubelet restart the static Pods instead of touching containers directly.
- I kept a backup of the manifests in `/root/bk`, outside the manifests directory.

## Mistakes to fix (from my shell history)

1. **Ran `kubectl` on `node01` about 14 times.** Workers have no kubeconfig, so `kubectl` there tries `localhost:8080` and fails (`The connection to the server localhost:8080 was refused`). That error says nothing about cluster health.
   - *Habit:* check the prompt or `hostname` first. `k` commands run on the controlplane; `systemctl`, `journalctl` and `crictl` for a node run on that node.
2. **Wrong `kubectl logs` syntax** (`k logs controller-manager`, `k logs pod controller-manager`). Correct form:
   `k logs -n kube-system kube-controller-manager-controlplane`
3. **`crictl logs` with no container ID.** Get the ID first with `crictl ps -a`, then `crictl logs <id>`.
4. **`journalctl -u kubelet` with no limits.** It floods the screen. Use `journalctl -u kubelet -n 50 --no-pager`.
5. **Checked the kubelet on the controlplane when `node01` was the broken node.** Run `systemctl status kubelet` on the node that is `NotReady`.
6. **Skipped an explicit "verify" step** after most fixes. Fixed means: all nodes `Ready`, all kube-system Pods `Running`, and a fresh Pod runs.

## Cheat sheet (type these from memory)

```bash
k get nodes ; k get pods -n kube-system
k describe node <node>
crictl ps -a | grep <component>
crictl logs <id>
systemctl status kubelet --no-pager | head
journalctl -u kubelet -n 50 --no-pager
ss -ltnp | grep -E '6443|2379|10250'
ls /etc/kubernetes/manifests
```

| Symptom | First suspect |
|---|---|
| Pod `Pending`, no events | scheduler |
| Deployment exists, no ReplicaSet or Pods | controller-manager |
| `kubectl` connection refused on 6443 | API server (or etcd behind it) |
| Node `NotReady` | kubelet, container runtime, or CNI on that node |

## End-of-day verify

```bash
k get nodes                  # all Ready
k get pods -A                # nothing stuck
k get rs                     # web ReplicaSet exists
k run ok --image=nginx && k get pod ok
```

## Tomorrow (Day 2)

Troubleshooting: Pods, logs, events, `kubectl top`, Service and DNS faults.
