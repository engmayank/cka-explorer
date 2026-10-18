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

1. Pick one scenario in **Trace a request** and say each step out loud before clicking Next.
2. Use **Kill a component** to practise the troubleshooting reflex: symptom, component, first command.
3. Run the **Quiz** for 10 minutes a day.
4. Reproduce each break-fix on a real kubeadm cluster (Lima or Multipass VMs on an Apple Silicon Mac work well). The explorer is a memory aid, not a replacement for hands-on practice.

## Scope and accuracy

- Models a **kubeadm-style cluster with containerd**, stacked etcd, and iptables-mode kube-proxy.
- Ports, paths and defaults reflect common kubeadm defaults. They can differ by Kubernetes version and distribution, so **verify against the [official docs](https://kubernetes.io/docs/) and the CKA curriculum for your exam version**.
- Not yet covered: storage (PV/PVC/CSI), RBAC, Ingress/Gateway, scheduling policies in depth.

## Contributing

Found a wrong port, path or flag? Have a better quiz question? Open an issue or pull request. Everything lives in one file (`index.html`): the data is near the top (`COMPS`, `EDGES`, `SCN`, `RAW`), the rest is the UI.

Ideas welcome: storage and RBAC maps, more break-fix scenarios, a CKA domain tracker.

## License

[MIT](LICENSE)
