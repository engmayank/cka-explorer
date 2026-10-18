# CKA Architecture Explorer

An interactive, single-page study aid for the **Certified Kubernetes Administrator (CKA)** exam. It helps you remember what each Kubernetes component does, what it talks to, which ports and files matter, and what breaks when it fails.

**Live demo:** https://engmayank.github.io/cka-explorer/

> Unofficial study aid. Not affiliated with or endorsed by the CNCF or The Linux Foundation. Kubernetes and CKA are trademarks of The Linux Foundation.

<!-- Add a screenshot: save it as screenshot.png in this repo, then uncomment the next line -->
<!-- ![CKA Architecture Explorer](screenshot.png) -->

## Why

Most people struggle with CKA not because kubectl is hard, but because the components blur together: who talks to etcd, which port is the kubelet on, what does the scheduler actually write? This tool turns those isolated facts into a map you can click through, flows you can watch, and failures you can predict.

## Features

| Tab | What it does |
|---|---|
| **Explore** | Clickable map of 11 components (kubectl, kube-apiserver, etcd, scheduler, controller-manager, kubelet, kube-proxy, containerd, CNI, CoreDNS, Pod). Click a box for its role, ports, config files, connections, debug commands and exam tips. Click an arrow to see what that connection carries. |
| **Trace a request** | Step-by-step animated flows: `kubectl apply` a Deployment, Service + DNS, `kubectl logs/exec`, a node going NotReady, and the control plane booting from static Pods. Play, pause, or step with the arrow keys. |
| **Kill a component** | Fail any component and predict what breaks before revealing the symptoms, what keeps running, and how to diagnose and fix it. |
| **Quiz** | About 60 flashcards on ports, file paths, flows, troubleshooting and ops. Spaced repetition: missed cards return more often. Keys: space = show, 1/2/3 = rate. |
| **Commands** | Searchable, click-to-copy cheat sheet for `kubectl`, `crictl`, `etcdctl`, `kubeadm`, `journalctl` and more. |

## Run it

No install, no build, no network needed.

- **Online:** open the live demo link above.
- **Locally:** download `index.html` and double-click it.

```bash
git clone https://github.com/engmayank/cka-explorer.git
open cka-explorer/index.html      # macOS; use xdg-open on Linux
```

Quiz progress is saved in your browser's localStorage only. Nothing is collected or sent anywhere.

## How to study with it

The rule: **predict first, then click**. Recalling an answer (even a wrong one) sticks far better than reading it.

### Step 1: Learn the map (Explore, ~30 min)
1. Open **Explore** and go through the components in this order: `kubectl` → `kube-apiserver` → `etcd` → `kube-scheduler` → `controller-manager` → `kubelet` → `containerd` → `CNI` → `kube-proxy` → `CoreDNS`.
2. For each one, cover the side panel and say out loud: *what it does, which port it uses, which file or manifest configures it.* Then read the panel to check yourself.
3. Click every **arrow** and ask: *who opens this connection, and why?* Remember the rule: only the API server talks to etcd, and everything else talks to the API server.

### Step 2: Learn the flows (Trace a request, ~20 min per scenario)
1. Start with **kubectl apply a Deployment**. It touches nearly every component.
2. Before each click on **Next**, answer: *which component acts now, and what does it write or call?*
3. Do the scenarios in this order: Deployment → Service + DNS → logs/exec → node NotReady → control plane boot.
4. Read the **CKA angle** box at the end of each scenario. It maps the flow to a symptom you will see in the exam.

### Step 3: Learn to troubleshoot (Kill a component, ~15 min per component)
1. Start with the common ones: scheduler, controller-manager, API server, etcd, kubelet, CNI, CoreDNS.
2. Before clicking **Reveal impact**, write down three things: *what stops, what keeps running, what I would see in kubectl.*
3. Then compare with the answer and copy the debug commands. Practise typing them from memory.
4. Aim to be able to say instantly: *Pending with no events → scheduler. Deployment makes no Pods → controller-manager. Connection refused on 6443 → API server. Node NotReady → kubelet, runtime or CNI.*

### Step 4: Drill daily (Quiz, 10 min)
1. Begin with the **Ports** and **Files** decks, then move to **Flow**, **Troubleshoot** and **Ops**.
2. Rate honestly: *Missed it* brings the card back soon; *Got it* pushes it out. Only press 3 if you answered correctly before revealing.
3. Use **See on map** after a miss to revisit that component in Explore.
4. Stop when your known-card bar stops growing. Short daily sessions beat one long one.

### Step 5: Prove it on a real cluster (hands-on)
The explorer is a memory aid, not a replacement for practice. Build a multi-node kubeadm cluster (Lima or Multipass VMs work well on an Apple Silicon Mac), then repeat each Kill scenario for real:

| Explorer scenario | Real drill |
|---|---|
| Kill scheduler | Move `kube-scheduler.yaml` out of `/etc/kubernetes/manifests`, create a Pod, watch it stay Pending, restore the file |
| Kill kubelet | `systemctl stop kubelet` on a worker, watch the node go NotReady, then fix it using only `journalctl` |
| Kill API server | Put a typo in a flag in `kube-apiserver.yaml`, then diagnose with `crictl ps -a` and `crictl logs` |
| etcd | Take a snapshot, restore it to a new data dir, and update the hostPath in `etcd.yaml` |
| Kill CNI / CoreDNS | Remove the CNI config or scale CoreDNS to 0, then see what breaks and fix it |
| Certificates | Run `kubeadm certs check-expiration`, renew them, and restart the static Pods |

### Suggested 7-day loop
| Day | Focus |
|---|---|
| 1 | Explore: all components and arrows |
| 2 | Trace: Deployment + Service/DNS scenarios |
| 3 | Trace: logs/exec, NotReady, control plane boot |
| 4 | Kill mode: control plane components, then a real cluster drill |
| 5 | Kill mode: node and network components, then a real cluster drill |
| 6 | Quiz (all decks), plus draw the architecture from memory on paper |
| 7 | Timed practice on a real cluster; note what you missed and repeat it next week |

Each week, redo the loop and spend extra time on whatever the Quiz shows as "learning". Closer to the exam, add timed practice (the exam simulator included with your exam registration is worth using) and the kubectl speed habits: aliases, `--dry-run=client -o yaml`, and `kubectl explain`.

## Scope and accuracy

- Models a **kubeadm-style cluster with containerd**, stacked etcd, and iptables-mode kube-proxy.
- Ports, paths and defaults reflect common kubeadm defaults. They can differ by Kubernetes version and distribution, so **verify against the [official docs](https://kubernetes.io/docs/) and the CKA curriculum for your exam version**.
- Not yet covered: storage (PV/PVC/CSI), RBAC, Ingress/Gateway, scheduling policies in depth.

## Contributing

Found a wrong port, path or flag? Have a better quiz question? Open an issue or pull request. Everything lives in one file (`index.html`): the data is near the top (`COMPS`, `EDGES`, `SCN`, `RAW`), the rest is the UI.

Ideas welcome: storage and RBAC maps, more break-fix scenarios, a CKA domain tracker.

## License

[MIT](LICENSE)
