# CKS preparation guide

**Last researched: 17 November 2026.** A combined study plan for the [KodeKloud CKS notes][kk], [Killer Shell CKS scenarios][kc], and the [Linux Foundation CKS exam][lf-course]. Use the checkboxes to track practical competence: a topic is complete when you can configure it, verify it, and troubleshoot it without following a solution.

**Jump to:** [Tips](#1-tips-first) · [kubectl tricks](#2-normal-kubectl-shell-and-editor-tricks) · [Exam content](#3-exam-content-and-current-requirements) · [Topic checklist](#4-topic-list-and-how-the-two-resources-fit-together) · [Topic details](#5-minimize-microservice-vulnerabilities-mmv-20) · [Miscellaneous](#11-miscellaneous) · [Practice catalog](#12-killercoda-practice-catalog) · [Study plan](#13-recommended-study-order-and-readiness-checks) · [Sources](#14-sources-and-maintenance)

**Exam domains — highest to lowest weight:** [Microservice vulnerabilities (MMV) — 20%](#5-minimize-microservice-vulnerabilities-mmv-20) · [Supply chain (SCS) — 20%](#6-supply-chain-security-scs-20) · [Monitoring, logging and runtime (MLRS) — 20%](#7-monitoring-logging-and-runtime-security-mlrs-20) · [Cluster setup (CS) — 15%](#8-cluster-setup-cs-15) · [Cluster hardening (CH) — 15%](#9-cluster-hardening-ch-15) · [System hardening (SH) — 10%](#10-system-hardening-sh-10).

Sections 1–4 retain the general preparation and reference material. The weighted topic sections follow in the order above, with related Killercoda tasks linked directly inside each subsection. Miscellaneous contains setup exercises that do not belong to a single exam domain.

**KodeKloud links:** each topic includes direct links to the public KodeKloud lesson and lab notes, using the chapter/lesson numbers from section 4.2. The notes contain course video/lab links, which may require enrollment or redirect to a general dashboard. Some public notes use older Kubernetes examples; retain this guide's corrections and use current official documentation. Related background links are explicitly distinguished from dedicated coverage where a topic has no matching public lesson.

## 1. Tips first

**Related KodeKloud preparation:** [Course Introduction][kk01-01] · [Exam Information][kk01-02]. Use the Linux Foundation references below for current exam rules rather than relying on older course notes.

### How to study efficiently

1. **Study one topic, then immediately do its lab.** Read the KodeKloud explanation, solve the matching Killercoda scenario, and repeat it from a fresh environment without the solution.
2. **Spend most of your preparation time at a terminal.** A useful personal target is 70% practice, 20% targeted reading, and 10% reviewing mistakes. These are study recommendations, not exam rules.
3. **Learn verification alongside configuration.** For RBAC, prove both an allowed and a denied operation. For NetworkPolicy, prove required traffic works and unwanted traffic fails. For audit logging, generate a request and find the event.
4. **Practise recovery early.** Deliberately break a lab API-server manifest, recover with `crictl` and kubelet logs, then repeat. API-server edits appear in several security workflows.
5. **Cover all six domains.** Microservice vulnerabilities, supply-chain security, and runtime security together account for **60%** of the curriculum. Give these substantial practice time without skipping cluster and host hardening.
6. **Use the documentation version matching the lab or exam.** Old screenshots and commands can reference removed APIs, old AppArmor annotations, legacy service-account tokens, or Docker on nodes that now use containerd.
7. **Build a small mistake log.** Record the symptom, root cause, fix, verification command, and time lost. Revisit this before doing another mock exam.
8. **Practise documentation navigation.** Know the search terms `NetworkPolicy`, `security context`, `Pod Security Standards`, `encrypt data`, `auditing`, `AppArmor`, `seccomp`, `RuntimeClass`, and `kubeadm upgrade`.
9. **Use mixed, timed practice after individual labs.** Completing a familiar lab is different from identifying the right control in an unfamiliar task.
10. **Treat this file as preparation material.** During the exam, use the resources allowed by the Linux Foundation and the task's Quick Reference links; personal notes and course sites are not general exam resources.

### A repeatable workflow for every task

**Read → enter the designated host → inspect → change → verify → return to base.**

- Read the whole task, including its namespace, object names, host, required output paths, and constraints.
- Follow the task's SSH instructions. Current CKS instructions require work on a designated host and a return to `base` after each task; nested SSH is not supported.
- Confirm `hostname`, the Kubernetes context, namespace, and node before changing anything. Switch context when instructed; SSH and kubeconfig context are separate things.
- Inspect the existing configuration before choosing a fix. Preserve settings needed by the workload or cluster.
- Back up node configuration **outside** the static-Pod manifest directory. A `.bak` copy in `/etc/kubernetes/manifests` can be read as another manifest.
- Make the required change in the persistent source: Deployment template, static-Pod manifest, systemd configuration, or the appropriate policy file.
- Verify the final state and application health. A command returning successfully does not prove a policy works.
- Save requested results in the exact file and on the exact host named by the task. Reopen the file to check it.
- After `sudo -i`, exit the root shell, then exit SSH, and confirm you are back on `base`.

### Time management

Suggested allocation for a two-hour attempt:

| Time | Goal |
| --- | --- |
| Minutes 0–5 | Get comfortable with the environment and task navigation; inspect the provided tools. |
| Minutes 5–75 | First pass: complete tasks you understand and verify them immediately. |
| Minutes 75–105 | Return to skipped or partially completed tasks. |
| Minutes 105–120 | Verify outputs, namespaces, workload health, and any control-plane changes. |

If you make no progress for several minutes, mark the task and return later. Use task weight and your confidence to decide where to spend time. This is a strategy, not a promise about question order or difficulty.

### Exam-day practical tips

- Run the PSI system check on the computer and network you will actually use. Check your ID/name match, webcam, microphone, single-monitor setup, and remote-desktop readability.
- Practise the Linux remote desktop even if your own machine is a Mac. Terminal copy/paste is normally **Ctrl+Shift+C / Ctrl+Shift+V**; other desktop applications use **Ctrl+C / Ctrl+V**.
- Current instructions say to use **Ctrl+Alt+W instead of Ctrl+W** and to use Vim's `i` rather than the Insert key.
- `kubectl`, the `k` alias, Bash completion, `yq`, `curl`, `wget`, and man pages are currently documented as available on the designated SSH hosts. Check before spending time configuring them.
- Do not reboot `base`. Treat hostnames and the task infobox as authoritative.
- Use included simulator access deliberately: first a timed diagnostic, then targeted review, then another timed attempt. Check your purchase entitlement; **CKS-SINGLE does not include simulator access** according to the current FAQ.

Sources: [CKS instructions][lf-instructions], [FAQ][lf-faq], [allowed resources][lf-resources], and [candidate handbook][lf-handbook].

## 2. Normal kubectl, shell, and editor tricks

**Related KodeKloud lessons and labs:** [KubeConfig][kk03-19] · [KubeConfig lab][kk03-20] · [Kubectl Proxy and Port Forward][kk03-29] · [Proxy/Port Forward lab][kk03-30]. These supplement the command reference below; they are not a dedicated Vim course.

Commands below target the **Linux lab/exam host**, not your local macOS shell. Examples are independent: replace example names, image versions, ports, and file paths with the task values. A YAML fragment is labelled when it is only part of an object.

### 2.1 Minimal shell setup

Check existing setup first:

```bash
type k
kubectl version --client
```

If needed in a Bash lab shell:

```bash
alias k=kubectl
source <(kubectl completion bash)
complete -o default -F __start_kubectl k
export KUBE_EDITOR=vim
```

Avoid relying on a large alias collection. Keep the commands portable between hosts.

### 2.2 Context, namespace, and discovery

```bash
hostname
k config get-contexts
k config current-context
k config use-context CONTEXT_FROM_TASK
k config view --minify

k get ns
k -n team-a get pods -o wide
k get nodes -o wide
k api-resources
k explain pod.spec.securityContext
k explain pod.spec.containers.securityContext --recursive
k explain networkpolicy.spec --recursive
```

- Prefer explicit `-n team-a` in task commands. `k config set-context --current --namespace=team-a` is convenient, but its namespace persists until changed.
- `k get all` does **not** show all resource types. Query Secrets, NetworkPolicies, RBAC objects, RuntimeClasses, and admission resources explicitly.
- Short names worth knowing: `po`, `deploy`, `ds`, `sts`, `svc`, `ns`, `sa`, `cm`, `netpol`, `ing`, `csr`.

### 2.3 Generate YAML instead of typing everything

```bash
k -n team-a run test --image=busybox:1.37 --restart=Never \
  --dry-run=client -o yaml --command -- sleep 3600 > pod.yaml

k -n team-a create deployment web --image=nginx:stable \
  --dry-run=client -o yaml > deployment.yaml

k -n team-a create serviceaccount reader --dry-run=client -o yaml > sa.yaml

k -n team-a create role pod-reader --verb=get,list,watch --resource=pods \
  --dry-run=client -o yaml > role.yaml

k -n team-a create rolebinding pod-reader \
  --role=pod-reader --serviceaccount=team-a:reader \
  --dry-run=client -o yaml > binding.yaml
```

For objects without an imperative generator, copy a small example from the appropriate documentation and adapt it. The image tags above demonstrate syntax; use the image required by your lab.

### 2.4 Inspect, change, and validate

```bash
k -n team-a get deploy web -o yaml > web.before.yaml
k -n team-a edit deployment web

k apply --dry-run=server -f desired.yaml
k diff -f desired.yaml
k apply -f desired.yaml
k -n team-a rollout status deployment/web --timeout=90s
k -n team-a get pods -o wide
```

- A server-side dry run exercises API validation and applicable admission checks, but does not prove runtime health.
- `kubectl diff` returns exit code `1` when differences exist; this is not automatically an error.
- Modify `spec.template.spec` for a Deployment, DaemonSet, or StatefulSet. Editing a managed Pod does not update its controller template.
- Most Pod security fields cannot be changed in place. Update the controller or recreate a standalone Pod as required, using a cleaned manifest rather than replaying all live metadata and status.
- `replace --force` deletes and recreates an object. Use it only when the task calls for that lifecycle, not as the default response to an edit failure.

Example targeted patch:

```bash
k -n team-a patch deployment web --type=strategic -p \
  '{"spec":{"template":{"spec":{"automountServiceAccountToken":false}}}}'
```

### 2.5 Labels, selectors, JSONPath, and jq

```bash
k get ns --show-labels
k -n team-a get pods --show-labels
k -n team-a get pods -l app=web -o wide
k get pods -A --field-selector spec.nodeName=node01 -o wide

k get pods -A -o custom-columns=\
'NS:.metadata.namespace,NAME:.metadata.name,NODE:.spec.nodeName,SA:.spec.serviceAccountName'

k -n team-a get pod web -o jsonpath='{.metadata.ownerReferences}{"\n"}'
k -n team-a get pod web -o jsonpath='{.status.containerStatuses[*].imageID}{"\n"}'

# Include regular, init, and ephemeral containers when inventorying image references.
k get pods -A -o json | jq -r '
  .items[] | .metadata as $m |
  (.spec.initContainers[]?, .spec.containers[]?, .spec.ephemeralContainers[]?) |
  [$m.namespace, $m.name, .name, .image] | @tsv'
```

`jq` is useful when available; JSONPath and custom columns cover many tasks without it. Check `yq --version` before using `yq` examples: different implementations have different syntax.

### 2.6 Fast troubleshooting

```bash
k -n team-a describe pod web
k -n team-a get events --sort-by=.metadata.creationTimestamp
k -n team-a logs web -c app --tail=100
k -n team-a logs web -c app --previous
k -n team-a logs deployment/web --all-containers=true --tail=100
k -n team-a exec web -c app -- id
k -n team-a get svc,endpointslices
k -n team-a wait --for=condition=Ready pod/web --timeout=60s
```

Useful symptom distinctions:

| Symptom | First checks |
| --- | --- |
| `Pending` | Scheduling events, node selectors, taints, RuntimeClass, resources. |
| `CreateContainerConfigError` | Missing Secret/ConfigMap, invalid configuration. |
| `CreateContainerError` | Runtime errors, profile availability, mounts, security settings. |
| `CrashLoopBackOff` | Current and previous logs; non-root permissions; missing writable directories. |
| `Forbidden` | Actual identity, RBAC, admission rejection details. |
| Connection timeout | Listener, Service endpoints, source egress, destination ingress, host firewall. |
| API connection refused | Host/context, API-server container state, static manifest, etcd connection. |

### 2.7 Permissions and network checks

```bash
k auth can-i get pods -n team-a --as=system:serviceaccount:team-a:reader
k auth can-i delete pods -n team-a --as=system:serviceaccount:team-a:reader
k auth can-i list secrets -n team-a --as=system:serviceaccount:team-a:reader
k auth can-i --list -n team-a --as=system:serviceaccount:team-a:reader

k -n team-a get networkpolicy
k -n team-a describe networkpolicy allow-web
k -n team-a exec client -- curl --connect-timeout 2 --max-time 5 http://web:8080
k -n team-a exec client -- nslookup kubernetes.default.svc.cluster.local
```

Impersonation requires permission to impersonate. For policies that grant permissions through groups, include the appropriate `--as-group` values or test using the real credentials. `can-i` checks authorization, not whether an admission policy or network rule will allow the request to complete.

Use a test Pod with the correct labels, namespace, tools, and security settings. `kubectl port-forward` is not a valid test of ordinary Pod-to-Pod NetworkPolicy enforcement.

### 2.8 Node-side commands you should know

```bash
sudo systemctl status kubelet --no-pager
sudo journalctl -u kubelet -n 100 --no-pager
sudo systemctl cat kubelet
sudo crictl ps -a
sudo crictl pods
sudo crictl logs CONTAINER_ID
sudo crictl inspect CONTAINER_ID
sudo ss -lntup
ps -ef
sudo stat /etc/kubernetes/manifests/kube-apiserver.yaml
openssl x509 -in server.crt -noout -subject -issuer -dates -ext subjectAltName
sha256sum ./kubelet
```

`crictl` needs the node's CRI endpoint. Check `/etc/crictl.yaml` and the installed runtime rather than assuming Docker is the Kubernetes runtime.

Common kubeadm-cluster locations:

| Location | Typical purpose |
| --- | --- |
| `/etc/kubernetes/manifests/` | Static control-plane Pod manifests. |
| `/etc/kubernetes/pki/` | Kubernetes and etcd certificates/keys. |
| `/etc/kubernetes/admin.conf` | Administrative kubeconfig. |
| `/var/lib/kubelet/config.yaml` | Kubelet configuration; confirm the actual `--config` path. |
| `/var/lib/kubelet/seccomp/` | Local seccomp profiles with the default kubelet root directory. |
| `/etc/apparmor.d/` | AppArmor profile files on the node. |
| `/etc/containerd/config.toml` | Common containerd configuration location. |
| `/etc/falco/falco.yaml` | Common Falco configuration; inspect the active rules-file list. |

Audit and encryption-policy file paths are configurable. Read the API-server arguments rather than memorizing one path.

### 2.9 Vim and shell speed

In Vim:

```vim
:set number expandtab tabstop=2 shiftwidth=2
:set paste
```

Use `:set paste` only when terminal paste needs it, then `:set nopaste` afterward.

| Action | Shortcut |
| --- | --- |
| Insert / return to normal mode | `i` / `Esc` |
| Save and quit / quit without saving | `:wq` / `:q!` |
| Search / next result | `/text` / `n` |
| Go to start / end | `gg` / `G` |
| Go to line 42 | `:42` |
| Delete / copy / paste line | `dd` / `yy` / `p` |
| Undo / redo | `u` / `Ctrl+r` |
| Indent / unindent line | `>>` / `<<` |
| Select lines | `V`, move, then indent/copy/delete |

Shell habits:

- `Ctrl+r`: search command history; `Ctrl+a` / `Ctrl+e`: start/end of line.
- Use single-quoted heredocs, `cat <<'EOF'`, to preserve `$variables` and YAML literally.
- `grep -n` gives line numbers; `grep -A`/`-B` gives context; `less` is useful for large logs.
- `sudo command > file` does not elevate the redirection. Use `command | sudo tee file` when needed.
- Use `--timeout` or `curl --max-time` for checks that might otherwise consume exam time.

## 3. Exam content and current requirements

**Related KodeKloud notes:** [Exam Information][kk01-02]. This is background reading only; the official Linux Foundation sources in this section take precedence for current requirements.

### 3.1 Format and logistics

| Item | Published information checked on 17 November 2026 |
| --- | --- |
| Format | Online, proctored, performance-based Linux/Kubernetes tasks. |
| Duration | **2 hours**. |
| Task count | Current CKS instructions describe **15–20 tasks**. |
| Passing score | **67%**. |
| Prerequisite | Have passed the **CKA** before attempting CKS; CNCF says the CKA does not have to remain active. |
| Certification validity | **2 years**. |
| Environment version | LF's exam page, FAQ, and CKS instructions currently say **Kubernetes v1.37**. Recheck before your appointment. |
| Simulator | Standard eligible registrations include two attempts with 36 hours of access per activation; CKS-SINGLE is excluded. Confirm in your portal. |

The software version and purchase entitlements can change. The task environment, candidate portal, and current official instructions take precedence over a static study file.

### 3.2 Official domain weights and coverage

The weights below are taken from the **[CKS curriculum v1.37 PDF][curriculum-pdf]**, and agree with the Linux Foundation exam page checked for this guide.

| Domain | Weight | What you must cover |
| --- | ---: | --- |
| Minimize Microservice Vulnerabilities | **20%** | Pod Security Standards; Secrets; isolation/multi-tenancy/sandboxed containers; Pod-to-Pod encryption with Cilium/Istio. |
| Supply Chain Security | **20%** | Minimal images; SBOM/CI/CD/artifact repositories; permitted registries and signed/validated artifacts; static analysis of workloads/images. |
| Monitoring, Logging and Runtime Security | **20%** | Behavioral analytics; threat detection; attack investigation; immutable containers; Kubernetes audit logging. |
| Cluster Setup | **15%** | NetworkPolicies; CIS review of Kubernetes components; TLS Ingress; node metadata/endpoints; platform binary verification. |
| Cluster Hardening | **15%** | RBAC; service-account precautions and minimal privileges; restricted API access; Kubernetes upgrades. |
| System Hardening | **10%** | Minimal host footprint; least-privilege identity/access; reduced external network access; AppArmor and seccomp. |

**Source discrepancy:** the CNCF certification summary page retrieved during research showed Cluster Setup at 10% and System Hardening at 15%. The current curriculum PDF and LF exam page show **15% and 10%**, respectively; this guide follows the latter two. Recheck the [curriculum repository][curriculum] if the outline is updated.

### 3.3 Documentation available during the exam

The current [CKS allowed-resources list][lf-resources] includes:

- [Kubernetes documentation](https://kubernetes.io/docs/) and [Kubernetes blog](https://kubernetes.io/blog/).
- [Falco documentation](https://falco.org/docs/).
- [BOM CLI documentation](https://kubernetes-sigs.github.io/bom/cli-reference/).
- [etcd documentation](https://etcd.io/docs/).
- [NGINX Ingress Controller configuration documentation](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/).
- [Cilium documentation](https://docs.cilium.io/).
- [Istio documentation](https://istio.io/latest/docs/).
- Additional documentation explicitly provided in a task's **Quick Reference** box.
- Installed distribution documentation and permitted distribution packages.

Use the browser inside the exam environment. The Kubernetes documentation search is allowed, but opening external search results is not. A study link in this file does not automatically make that site an allowed exam resource. Practise navigating directly; personal browser bookmarks are not permitted under LF's current certification FAQ.

## 4. Topic list and how the two resources fit together

### 4.1 Topic IDs and shorthand

These are **shorthand topic IDs used in this guide**, not official CKS exam codes.

| Prefix | Meaning | Example |
| --- | --- | --- |
| **MMV** | Minimize Microservice Vulnerabilities | **MMV1** = Security contexts and capabilities; **MMV2** = Pod Security Standards and Admission. |
| **SCS** | Supply Chain Security | **SCS1** = Image footprint and Dockerfile hardening. |
| **MLRS** | Monitoring, Logging and Runtime Security | **MLRS1** = Falco and behavioral threat detection. |
| **CS** | Cluster Setup | **CS1** = NetworkPolicy. |
| **CH** | Cluster Hardening | **CH2** = Service accounts and tokens. |
| **SH** | System Hardening (host operating system) | **SH1** = Host hardening, SSH, permissions, and firewall. |
| **F** | Foundations | **F0** = Attack surface and container foundations. |
| **KK** | KodeKloud chapter reference | **KK03** = KodeKloud's “Cluster Setup and Hardening” chapter. |

Domain topic IDs use the **domain acronym followed by the topic number**, for example **MMV1**, **SCS1**, and **CH2**. The **number identifies a topic, not its exam weight or priority**. For example, **MMV6** appears beside MMV1 and MMV2 because admission controls relate closely to Pod Security. Topic numbers are retained, and the checklist, topic links, study-plan references, and practice catalog all use the same acronym convention. **F0** remains the cross-domain foundations topic; **KK** references identify KodeKloud chapters rather than exam domains.

Lowercase **`k`** in command examples is simply an alias for **`kubectl`**; it is not a topic ID or a KodeKloud chapter reference.

### 4.2 KodeKloud chapter key

The shorthand below refers to directories in the public GitHub notes. Lesson numbers are the filename prefixes visible when this guide was prepared.

The chapter links provide an overview; the **KodeKloud lessons and labs** blocks inside each topic link directly to the relevant files.

| Key | Chapter |
| --- | --- |
| KK02 | [Understanding the Kubernetes Attack Surface][kk02] |
| KK03 | [Cluster Setup and Hardening][kk03] |
| KK04 | [System Hardening][kk04] |
| KK05 | [Minimize Microservice Vulnerabilities][kk05] |
| KK06 | [Supply Chain Security][kk06] |
| KK07 | [Monitoring, Logging and Runtime Security][kk07] |
| KK08 | [Mock Exams][kk08] |
| KK09 | [CKS Challenges][kk09] |

### 4.3 Master checklist

Every topic row has a corresponding detail section below. The topic IDs also make it easy to record weak areas in your mistake log. Groups follow descending exam weight; topics within each group are placed beside related material rather than ranked by individual weight, because the outline assigns percentages only to domains.

| Done | ID | Topic | KodeKloud starting point | Killercoda practice / supplement |
| --- | --- | --- | --- | --- |
| | | **Minimize Microservice Vulnerabilities — 20%** | | |
| [ ] | [MMV1](#mmv1-security-contexts-capabilities-and-privilege-escalation) | Security contexts and capabilities | [KK05 02–03][kk05-02]; [KK04 25][kk04-25] | Privileged Containers; Privilege Escalation Containers. |
| [ ] | [MMV2](#mmv2-pod-security-standards-and-pod-security-admission) | Pod Security Standards and Admission | [KK05 09][kk05-09], [11][kk05-11] | Official PSS/PSA namespace-label exercises. |
| [ ] | [MMV6](#mmv6-admission-control-and-policy-engines) | Admission control and policy engines | [KK05 04–07][kk05-04], [12–15][kk05-12] | ImagePolicyWebhook; supplemental admission exercises. |
| [ ] | [MMV3](#mmv3-secrets-and-encryption-at-rest) | Secrets and encryption at rest | [KK05 16–17][kk05-16] | Four Secret scenarios, especially ETCD Encryption. |
| [ ] | [F0](#f0-attack-surface-and-container-foundations) | Attack surface and container foundations | [KK02][kk02]; [KK03 security primitives][kk03-07] | Container namespaces: Docker/Podman. |
| [ ] | [MMV4](#mmv4-isolation-multi-tenancy-gvisorkata-and-runtimeclass) | Multi-tenancy, RuntimeClass, gVisor/Kata | [KK05 18–23][kk05-18] | Sandbox gVisor; container namespace labs. |
| [ ] | [MMV5](#mmv5-pod-to-pod-encryption-cilium-and-istio) | Cilium encryption and Istio mTLS | [KK05 24–25][kk05-24] for mTLS basics | Official Cilium/Istio encryption exercises. |
| | | **Supply Chain Security — 20%** | | |
| [ ] | [SCS1](#scs1-minimize-image-footprint-and-harden-dockerfiles) | Image footprint and Dockerfile hardening | [KK06 02–04][kk06-02] | Image Footprint User; Container Hardening. |
| [ ] | [SCS2](#scs2-sbom-cicd-artifact-repositories-and-provenance) | SBOM, CI/CD, artifact provenance | Supplement the public [KK06 notes][kk06] | BOM CLI exercises and your own build pipeline lab. |
| [ ] | [SCS3](#scs3-registries-digests-signatures-and-image-admission) | Registries, digests, signatures, image admission | [KK06 03–06][kk06-03] | Image Use Digest; ImagePolicyWebhook Setup; signature exercise. |
| [ ] | [SCS4](#scs4-static-analysis-of-workloads-and-images) | Static workload/image analysis | [KK06 07–08][kk06-07] | Static Manual Analysis K8s; Docker. |
| [ ] | [SCS5](#scs5-image-vulnerability-scanning-with-trivy) | Vulnerability scanning and remediation | [KK06 09–10][kk06-09] | Image Vulnerability Scanning Trivy. |
| | | **Monitoring, Logging and Runtime Security — 20%** | | |
| [ ] | [MLRS1](#mlrs1-falco-and-behavioral-threat-detection) | Falco and behavioral detection | [KK07 02–06][kk07-02] | Falco Change Rule. |
| [ ] | [MLRS2](#mlrs2-runtime-investigation-and-attack-timeline) | Runtime investigation and attack timeline | [KK04 17–18][kk04-17]; [KK07 02–06][kk07-02] | Syscall Activity Strace; Falco investigation. |
| [ ] | [MLRS3](#mlrs3-immutability-of-containers-at-runtime) | Container immutability | [KK07 07–09][kk07-07] | Immutability Readonly Filesystem. |
| [ ] | [MLRS4](#mlrs4-kubernetes-audit-policy-and-logs) | Kubernetes audit policy and logs | [KK07 10–11][kk07-10] | Auditing Enable Audit Logging. |
| | | **Cluster Setup — 15%** | | |
| [ ] | [CS1](#cs1-networkpolicy) | NetworkPolicy | [KK03 40–42][kk03-40] | Default deny; namespace communication; Cilium policy playground. |
| [ ] | [CS2](#cs2-cis-benchmarks-and-kube-bench) | CIS benchmarks and kube-bench | [KK03 02–06][kk03-02] | CIS control-plane remediation. |
| [ ] | [CS3](#cs3-ingress-with-tls) | Ingress with TLS | [KK03 11–16][kk03-11], [43–46][kk03-43] | Ingress Create; Ingress Secure. |
| [ ] | [CS4](#cs4-node-metadata-and-endpoint-protection) | Metadata and endpoint protection | [KK03 kubelet/networking][kk03-27]; [KK04 network access][kk04-14] | NetworkPolicy Metadata Protection. |
| [ ] | [CS5](#cs5-verify-platform-binaries) | Platform binary verification | [KK03 34–35][kk03-34] | Verify Platform Binaries. |
| | | **Cluster Hardening — 15%** | | |
| [ ] | [CH1](#ch1-rbac-and-least-privilege) | RBAC and least privilege | [KK03 22–26][kk03-22] | RBAC users; RBAC service accounts. |
| [ ] | [CH2](#ch2-service-accounts-and-tokens) | Service accounts and tokens | [KK03 09–10][kk03-09] | Token Mounting; Secret ServiceAccount Pod. |
| [ ] | [CH3](#ch3-authentication-tls-csr-and-kubeconfig) | Authentication, certificates, CSRs, kubeconfig | [KK03 08][kk03-08], [11–20][kk03-11] | CSR manual signing; signing through API. |
| [ ] | [CH4](#ch4-api-server-and-kubelet-hardening-recovery) | API server, kubelet, NodeRestriction, recovery | [KK03 07–08][kk03-07], [22][kk03-22], [27–33][kk03-27] | API-server Crash; Misconfigured; NodeRestriction. |
| [ ] | [CH5](#ch5-upgrade-kubernetes-to-address-vulnerabilities) | Kubernetes upgrades | [KK03 36–39][kk03-36] | Repeat an upgrade in a suitable kubeadm lab. |
| | | **System Hardening — 10%** | | |
| [ ] | [SH1](#sh1-host-hardening-least-privilege-ssh-and-firewall) | Host footprint, SSH, sudo, IAM, firewall | [KK04 02–16][kk04-02] | Close Open Ports; Manage Packages. |
| [ ] | [SH2](#sh2-seccomp-and-syscall-restrictions) | seccomp and syscalls | [KK04 17–21][kk04-17] | Syscall Activity Strace; official seccomp tutorial. |
| [ ] | [SH3](#sh3-apparmor) | AppArmor | [KK04 22–26][kk04-22] | AppArmor. |

### 4.4 Material to supplement or update

- **PSS/PSA:** the public KodeKloud `09-PSA-and-PSS.md` file currently contains a video link and `Lecture: TODO`. Read the Kubernetes documentation and practise enforcement, even if you have read every repository file.
- **PodSecurityPolicy:** removed in Kubernetes **v1.25**. The repository retains PSP/retired material; use PSS/PSA for current exam preparation.
- **SBOM, signing, and provenance:** the public supply-chain chapter does not contain dedicated lessons covering all of the current SBOM/signing objectives. Add the exercises under SCS2 and SCS3.
- **Cilium/Istio:** the public mTLS notes give a short Istio-oriented introduction. Add hands-on Cilium transparent encryption and Istio policy verification.
- **Killercoda inventory:** the public profile returned **42 entries**, including playgrounds and editor setup, when checked. It did not expose separate scenarios named PSA, seccomp, SBOM, image signing, or Pod-to-Pod encryption. Use the supplemental exercises here rather than assuming the catalog alone completes those objectives.
- **AppArmor and service accounts:** prefer current structured AppArmor fields and projected, short-lived tokens. Recognize older examples when troubleshooting, but do not assume their syntax or token behavior is current.
- **Docker, Dashboard, OPA, Kata, and Tracee:** useful supporting material. Learn how they illustrate the official competencies and how to operate them when supplied; avoid letting a deep product-specific detour replace practising the listed objectives.

The mapping is based on public notes and scenario descriptions. Interactive labs were not executed as part of preparing this document.

## 5. Minimize Microservice Vulnerabilities (MMV, 20%)

**KodeKloud chapter:** [KK05 — Minimize Microservice Vulnerabilities][kk05], with supporting kernel-security material in [KK04 — System Hardening][kk04]. Topic-specific lesson and lab links follow.

**Topic details and hands-on completion criteria:** grouped by exam domain in descending weight order. Domains with the same weight retain the order in the supplied exam outline. Topic numbers are unchanged; prefixes now use the domain acronyms in section 4.1, consistently across the master checklist, topic links, and study plan.

### MMV1. Security contexts, capabilities, and privilege escalation

**KodeKloud lessons and labs**

- [KK05 02 — Security Contexts][kk05-02] · [03 — Security Contexts lab][kk05-03].
- [KK04 25 — Linux Capabilities][kk04-25].

**Related exam objective:** Use appropriate pod security standards; these workload controls underpin the PSS/PSA topic below.

**Killercoda exercises**

- [ ] [Privileged Containers](https://killercoda.com/killer-shell-cks/scenario/privileged-containers)
- [ ] [Privilege Escalation Containers](https://killercoda.com/killer-shell-cks/scenario/privilege-escalation-containers)

**Cover**

- Pod-level versus container-level security contexts and which fields are valid at each level.
- `runAsNonRoot`, numeric `runAsUser`/`runAsGroup`, volume access via `fsGroup`, and image file permissions.
- `privileged: false`, `allowPrivilegeEscalation: false`, dropping capabilities, and adding back only specifically needed capabilities.
- `readOnlyRootFilesystem`, writable volumes, seccomp, AppArmor, and host namespace settings.
- Inspect **all** regular, init, and applicable ephemeral containers, not only the first application container.

Runnable lab example, assuming namespace `team-a` and the image are available:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hardened
  namespace: team-a
spec:
  automountServiceAccountToken: false
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      image: busybox:1.37
      command: [sh, -c, "sleep 3600"]
      securityContext:
        privileged: false
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: [ALL]
      volumeMounts:
        - name: tmp
          mountPath: /tmp
  volumes:
    - name: tmp
      emptyDir: {}
```

```bash
k -n team-a exec hardened -- id
k -n team-a exec hardened -- sh -c 'grep -E "NoNewPrivs|CapEff|Seccomp" /proc/1/status'
k -n team-a exec hardened -- touch /should-fail
k -n team-a exec hardened -- touch /tmp/should-work
```

**Traps**

- `runAsNonRoot` is a constraint; it does not rewrite a root-only image to make it work as a non-root user.
- `allowPrivilegeEscalation: false` is separate from selecting a non-root UID and from dropping capabilities.
- `readOnlyRootFilesystem` does not make mounted volumes read-only.
- `fsGroup` affects supported volume ownership/access, not arbitrary image-layer file ownership.
- Adding a capability such as `NET_BIND_SERVICE` should be driven by the workload/task; do not restore a broad capability set just to make a Pod start.

**Reference:** [Security contexts](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).

### MMV2. Pod Security Standards and Pod Security Admission

**KodeKloud lessons and labs**

- [KK05 09 — Pod Security Admission and Pod Security Standards][kk05-09] · [11 — PSA lab][kk05-11].
- The lesson-notes file is a placeholder with a video link; use the PSA lab notes and the official references below for implementation practice. Skip the retired PSP lessons for current-cluster configuration.

**Exam objective:** Use appropriate pod security standards.

**Killercoda coverage:** no dedicated PSS/PSA scenario appears in the current Killer Shell CKS catalog. Use the related [Privileged Containers](https://killercoda.com/killer-shell-cks/scenario/privileged-containers) and [Privilege Escalation Containers](https://killercoda.com/killer-shell-cks/scenario/privilege-escalation-containers) labs to understand the controls, then complete the [official namespace-label exercise](https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/) and the drill below.

- [ ] **Supplemental exercise — PSA:** enforce Restricted in a test namespace; repair a rejected workload; verify compliant Pods run.

**Cover**

- Standards: **Privileged**, **Baseline**, **Restricted**.
- Admission modes: **enforce** rejects violations; **warn** returns warnings; **audit** adds audit annotations.
- Namespace labels and per-mode version labels; optional cluster-wide admission defaults and exemptions.
- Restricted-policy expectations for Linux workloads: non-root execution, no privilege escalation, dropping all capabilities with only the allowed add-back, and an allowed seccomp profile, alongside Baseline restrictions.
- PSS requirements evolve by Kubernetes version. Read the actual table for the policy version requested.

```bash
# Choose the policy version specified by the task / supported by the cluster.
POLICY_VERSION=v1.37
k label namespace team-a \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version="$POLICY_VERSION" \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/warn-version="$POLICY_VERSION" \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/audit-version="$POLICY_VERSION" \
  --overwrite

k get namespace team-a --show-labels
k -n team-a apply --dry-run=server -f disallowed-pod.yaml
k -n team-a apply --dry-run=server -f hardened-pod.yaml
```

**Practise:** begin with warn/audit, identify violations, repair the workload, and enable enforce. Test both a rejected Pod and an admitted, functioning Pod.

**Traps**

- Enabling enforcement does not automatically evict existing violating Pods.
- A Deployment object may be accepted while its controller's subsequent Pod creation is rejected; inspect ReplicaSet events and Pod creation.
- PSA enforces a standard; it does not automatically mutate workloads into compliance.
- Read-only root filesystems and disabled token automount are useful additional controls, but are not universal requirements of the Restricted standard.
- PodSecurityPolicy is obsolete. Do not spend final revision time learning to deploy PSP objects on a current cluster.

**References:** [PSS](https://kubernetes.io/docs/concepts/security/pod-security-standards/), [PSA](https://kubernetes.io/docs/concepts/security/pod-security-admission/), [namespace-label enforcement](https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/).

### MMV6. Admission control and policy engines

**KodeKloud lessons and labs**

- [KK05 04 — Admission Controllers][kk05-04] · [05 — Admission Controllers lab][kk05-05].
- [06 — Validating and Mutating Admission Controllers][kk05-06] · [07 — Validating/Mutating Admission lab][kk05-07].
- [12 — Open Policy Agent][kk05-12] · [13 — OPA lab][kk05-13].
- [14 — OPA Gatekeeper in Kubernetes][kk05-14] · [15 — OPA in Kubernetes lab][kk05-15].

**Related exam objectives:** Use appropriate pod security standards; secure your supply chain through admission enforcement. General admission controls stay with workload policy here; the image-specific workflow is under [SCS3](#scs3-registries-digests-signatures-and-image-admission).

**Related Killercoda exercise**

- [ ] [ImagePolicyWebhook Setup](https://killercoda.com/killer-shell-cks/scenario/image-policy-webhook-setup) — image admission practice; not a general OPA/Gatekeeper or PSA lab.

**Cover**

- Admission occurs after authentication and authorization. Mutating admission can change an object; validating admission accepts or rejects it.
- Built-in admission plugins, dynamic webhooks, matching rules/selectors, webhook TLS/CA trust, timeout, and failure policy.
- Pod Security Admission for built-in Pod standards.
- OPA/Gatekeeper concepts from the course: ConstraintTemplate, Constraint, matching scope, violations, and deny versus audit/dry-run behavior.
- Recognize native ValidatingAdmissionPolicy/CEL if provided, without assuming every policy-engine product is a separate required exam topic.

**Practise:** enforce a supplied rule, demonstrate a permitted object and a rejected object, then diagnose a webhook certificate or service-endpoint problem.

**Traps:** a policy existing does not prove its match criteria cover the target namespace or resource. A policy reporting violations in audit mode may still allow creation. A webhook that fails open can allow requests when the webhook is unavailable.

For registry restrictions and the specialized ImagePolicyWebhook workflow, see SCS3.

### MMV3. Secrets and encryption at rest

**KodeKloud lessons and labs**

- [KK05 16 — Manage Kubernetes Secrets][kk05-16] · [17 — Manage Kubernetes Secrets lab][kk05-17].
- These cover Secret fundamentals. Keep the distinction between base64 encoding and encryption, and complete the encryption-at-rest drill below separately rather than assuming Secret creation enables storage encryption.

**Exam objective:** Manage Kubernetes secrets.

**Killercoda exercises**

- [ ] [Secret Access in Pods](https://killercoda.com/killer-shell-cks/scenario/secret-pod-access)
- [ ] [Secret Read and Decode](https://killercoda.com/killer-shell-cks/scenario/secret-read-secrets)
- [ ] [Secret ServiceAccount Pod](https://killercoda.com/killer-shell-cks/scenario/secret-serviceaccount-pod)
- [ ] [Secret ETCD Encryption](https://killercoda.com/killer-shell-cks/scenario/secret-etcd-encryption)

**Cover**

- `data` versus `stringData`; **base64 is encoding, not encryption**.
- Create Secrets from files/literals; use `secretKeyRef`, `envFrom`, Secret volumes, key selection, and file permissions.
- Limit access through RBAC and workload-creation rights. Permission to create a Pod that consumes a Secret can expose it even without direct Secret-read permission.
- Environment variables are not refreshed in an already running container when a Secret changes. Mounted Secret volumes normally update eventually, but `subPath` mounts do not receive those automatic updates.
- API-server storage encryption is separate from TLS in transit, Kubernetes API authorization, and disk encryption.
- `EncryptionConfiguration`, resource selection, provider ordering, keys, API-server mounts/flags, rewriting old objects, and rotation.

```bash
k -n team-a create secret generic app-secret --from-file=password=./password.txt
k -n team-a get secret app-secret -o jsonpath='{.data.password}' | base64 -d
openssl rand -base64 32
```

Encryption configuration **template** for a lab requiring AES-CBC; replace the placeholder with a valid generated key:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources: [secrets]
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: REPLACE_WITH_BASE64_ENCODED_32_BYTE_KEY
      - identity: {}
```

**Encryption-at-rest drill**

1. Create a lab Secret before enabling encryption and inspect its raw etcd storage using the provided etcd credentials.
2. Save the encryption configuration on the control-plane host and restrict access to its key material.
3. Mount the configuration into the API-server container using the appropriate `hostPath` volume and `volumeMount`.
4. Add `--encryption-provider-config=CONTAINER_PATH` to the API server. Make consistent changes across all API servers if the cluster has multiple replicas.
5. Wait for healthy startup; create a new Secret and inspect its raw etcd representation.
6. Rewrite the required existing Secrets through the API. Example for **one named lab Secret**:

   ```bash
   k -n team-a get secret app-secret -o json | k replace -f -
   ```

7. Verify both old and new Secrets are encrypted in storage and still readable through the authorized API path.

**Key points:** the first provider is used for new writes. `identity` first means new writes are plaintext. Keeping identity later permits reading existing unencrypted data during migration. Enabling encryption does not retroactively rewrite old records. Retain old decryption keys until all relevant records have been migrated; practise the documented rotation sequence.

An encrypted etcd record has a provider-specific encryption prefix; API `get secret` still returns its normal decoded API representation. Therefore `kubectl get secret -o yaml` does not verify storage encryption.

**Reference:** [Encrypting confidential data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/).

### F0. Attack surface and container foundations

**KodeKloud lessons**

- [KK02 01 — The Attack][kk02-01] · [02 — The 4Cs of Cloud Native Security][kk02-02].
- [KK03 07 — Kubernetes Security Primitives][kk03-07].

**Related exam objective:** Understand and implement isolation techniques (multi-tenancy, sandboxed containers, etc.). These foundations also support the other security domains.

**Killercoda exercises**

- [ ] [Container Namespaces Docker](https://killercoda.com/killer-shell-cks/scenario/container-namespaces-docker)
- [ ] [Container Namespaces Podman](https://killercoda.com/killer-shell-cks/scenario/container-namespaces-podman)

**Cover**

- The four Cs: cloud/infrastructure, cluster, container, and code. Explain how a weak outer layer can undermine inner-layer controls.
- Kubernetes request flow: TLS → authentication → authorization → admission → persistence.
- Linux namespaces isolate views of processes, networking, mounts, users, and other resources; cgroups govern resource usage. Neither makes a container equivalent to a separate kernel.
- Understand host namespaces, privileged containers, hostPath mounts, runtime sockets, service-account credentials, and node metadata as possible paths from a workload to broader access.
- Distinguish prevention (RBAC, NetworkPolicy, admission, seccomp) from detection (audit logs, Falco).

**Practise:** compare a normal container with one sharing another container's PID namespace; inspect `ps`, `/proc`, and namespace links. Explain why mounting the container runtime socket is a high-privilege operation.

**Complete when:** you can name the control that addresses a given weakness and explain what that control does not enforce.

### MMV4. Isolation, multi-tenancy, gVisor/Kata, and RuntimeClass

**KodeKloud lessons and labs**

- [KK05 18 — Container Sandboxing][kk05-18] · [19 — gVisor][kk05-19] · [20 — Kata Containers][kk05-20].
- [21 — Runtime Classes][kk05-21] · [22 — Using Runtimes in Kubernetes][kk05-22] · [23 — Runtimes lab][kk05-23].
- For tenant boundaries, also use the KodeKloud RBAC, NetworkPolicy, and PSA lessons linked under CH1, CS1, and MMV2.

**Exam objective:** Understand and implement isolation techniques (multi-tenancy, sandboxed containers, etc.).

**Killercoda exercises**

- [ ] [Sandbox gVisor](https://killercoda.com/killer-shell-cks/scenario/sandbox-gvisor)
- [ ] [Container Namespaces Docker](https://killercoda.com/killer-shell-cks/scenario/container-namespaces-docker) — namespace foundations.
- [ ] [Container Namespaces Podman](https://killercoda.com/killer-shell-cks/scenario/container-namespaces-podman) — namespace foundations.

Related tenant-boundary practice: [NetworkPolicy](#cs1-networkpolicy), [RBAC](#ch1-rbac-and-least-privilege), and [PSS/PSA](#mmv2-pod-security-standards-and-pod-security-admission).

**Cover**

- Namespaces organize resources; meaningful tenant isolation combines RBAC, network policy, admission controls, quotas, and workload/node boundaries.
- ResourceQuota and LimitRange help constrain resource consumption but are not substitutes for security isolation.
- Ordinary containers share the host kernel. gVisor interposes a userspace kernel; Kata uses lightweight virtualization for a different isolation boundary.
- RuntimeClass selects a **configured runtime handler**; it does not install that runtime.
- The handler must exist on eligible nodes. Use RuntimeClass scheduling configuration or other appropriate scheduling constraints where support differs by node.

```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: sandboxed
handler: runsc
```

Pod-spec fragment:

```yaml
spec:
  runtimeClassName: sandboxed
```

`runsc` is an example handler; it must match the installed runtime configuration exactly. RuntimeClass is cluster-scoped.

**Practise:** configure the supplied runtime on a lab node, create a RuntimeClass, launch a workload using it, and diagnose an unknown/missing runtime handler.

**Verify:** inspect events, the Pod's `runtimeClassName`, and runtime-side evidence that the requested handler is actually being used. A RuntimeClass object existing in the API is insufficient.

**Reference:** [RuntimeClass](https://kubernetes.io/docs/concepts/containers/runtime-class/).

### MMV5. Pod-to-Pod encryption: Cilium and Istio

**KodeKloud lessons**

- [KK05 24 — One-way SSL vs Mutual SSL][kk05-24] · [25 — Implement Pod-to-Pod Encryption with mTLS][kk05-25].
- These provide TLS/mTLS background and a short Istio-oriented introduction. No dedicated Cilium encryption lesson appears in these public notes; complete the Cilium and Istio implementation/verification exercises below as supplements.

**Exam objective:** Implement Pod-to-Pod encryption (Cilium, Istio).

**Killercoda coverage:** no dedicated Pod-to-Pod encryption scenario appears in this catalog. The [Cilium Network Policy Playground](https://killercoda.com/killer-shell-cks/scenario/playground-cilium-network-policy) is related network-policy practice, not an encryption lab. Complete the linked official Cilium/Istio exercises and verification drills below in a suitable environment.

- [ ] **Supplemental exercise — Cilium encryption:** enable the requested encryption mode in a suitable lab; prove cross-node transport is encrypted.
- [ ] **Supplemental exercise — Istio mTLS:** enroll the test workloads, require STRICT mTLS, and verify authorized meshed traffic plus plaintext rejection.

This is an explicit current curriculum objective. Practise **configuration and verification**, not just definitions.

#### Cilium

- Understand transparent encryption with **WireGuard** and **IPsec**, their prerequisites, and which traffic is covered.
- Inspect the installed Cilium version/configuration before changing it.
- WireGuard Helm settings include `encryption.enabled=true` and `encryption.type=wireguard`. Apply them using the installation/upgrade method specified by the lab while preserving existing values and chart version.
- Current Cilium documentation also describes the ConfigMap setting `enable-wireguard: "true"` followed by agent restarts. Choose the supported method for the supplied installation.
- IPsec requires appropriate key configuration/distribution and rotation; use the version-specific documentation and any provided Secret/configuration.

Useful inspection commands in a typical installation:

```bash
k -n kube-system get configmap cilium-config -o yaml
k -n kube-system get pods -l k8s-app=cilium -o wide
k -n kube-system exec ds/cilium -- cilium-dbg status
k -n kube-system exec ds/cilium -- cilium-dbg encrypt status
```

**Verify:** inspect encryption status/peers, send traffic between Pods on **different nodes**, and use packet capture on the appropriate node interface to confirm encrypted transport. With WireGuard, same-node traffic is intentionally not tunneled; a capture on the decrypted tunnel interface can show plaintext inner packets. Choose the capture point carefully.

#### Istio

- Understand one-way TLS versus mutual TLS, workload identities, trust, and mesh enrollment.
- `PeerAuthentication` controls accepted inbound mTLS policy; `DestinationRule` can control client-side TLS behavior. Understand auto-mTLS and avoid introducing a conflicting explicit client policy.
- `PERMISSIVE` accepts both plaintext and mTLS; `STRICT` requires mTLS for the selected workloads.
- For a sidecar-mode lab, enable the provided injection label/revision and recreate workloads when needed. For an ambient-mode lab, follow that mode's enrollment and verification procedure.

Example namespace-level policy for an installed mesh:

```yaml
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: team-a
spec:
  mtls:
    mode: STRICT
```

Check the CRD versions served by the installed Istio release. **Creating this object without the mesh/data-plane setup does not encrypt traffic.**

**Verify:** authenticated meshed clients can connect, a suitable non-meshed plaintext client is rejected, and proxy/mesh telemetry confirms mTLS. A successful `curl` alone does not establish that the connection was encrypted.

**Remember:** NetworkPolicy restricts who can connect; encryption protects traffic in transit; authorization controls what an authenticated peer may do. These solve different problems.

**References:** [Cilium encryption](https://docs.cilium.io/en/stable/security/network/encryption/), [WireGuard](https://docs.cilium.io/en/stable/security/network/encryption-wireguard/), [Istio mTLS migration](https://istio.io/latest/docs/tasks/security/authentication/mtls-migration/).

## 6. Supply Chain Security (SCS, 20%)

**KodeKloud chapter:** [KK06 — Supply Chain Security][kk06]. Topic-specific links below identify both the available lessons and the current curriculum gaps.

### SCS1. Minimize image footprint and harden Dockerfiles

**KodeKloud lessons and labs**

- [KK06 02 — Minimize Base Image Footprint][kk06-02].
- [03 — Image Security][kk06-03] · [04 — Image Security lab][kk06-04].

**Exam objective:** Minimize base image footprint.

**Killercoda exercises**

- [ ] [Container Image Footprint User](https://killercoda.com/killer-shell-cks/scenario/container-image-footprint-user)
- [ ] [Container Hardening](https://killercoda.com/killer-shell-cks/scenario/container-hardening)

**Cover**

- Multi-stage builds: keep compilers/build dependencies out of the runtime image.
- Select a maintained minimal base appropriate to the application: minimal distribution, distroless, or `scratch` when suitable.
- Non-root execution, numeric UID/GID, file ownership, explicit working directory, and application compatibility.
- Exclude source/build caches, package managers, debugging tools, and secrets that are unnecessary at runtime.
- Use `.dockerignore`; understand image layers and why deleting a secret in a later layer does not remove it from an earlier layer.
- Use build-secret facilities rather than embedding credentials in `ARG`, `ENV`, or copied files.
- Tags versus digests; rebuild and redeploy to update a vulnerable base/dependency.

**Practise:** take a supplied oversized/root-running Dockerfile, separate build/runtime stages, reduce included files, run as non-root, build it, and demonstrate that the application still starts.

**Traps**

- `scratch` has no shell, package manager, CA trust store, or standard libraries unless you supply what the application needs.
- Distroless images may have no `/bin/sh`; use direct executable commands and appropriate debugging methods.
- A small image is not automatically vulnerability-free or signed.
- A digest pins content; it does not make that content safe.

### SCS2. SBOM, CI/CD, artifact repositories, and provenance

**Related KodeKloud background**

- [KK06 — Supply Chain Security chapter][kk06] · [03 — Image Security][kk06-03].
- The public chapter has no dedicated SBOM, CI/CD, or provenance lesson. These are contextual links, not complete coverage; use the BOM CLI reference, commands, and build-pipeline exercise below for the missing objectives.

**Exam objective:** Understand your supply chain (e.g. SBOM, CI/CD, artifact repositories).

**Killercoda coverage:** no dedicated SBOM, CI/CD, or artifact-provenance scenario appears in this catalog. Complete the [BOM CLI exercises](https://kubernetes-sigs.github.io/bom/cli-reference/) and your own build-pipeline lab using the commands and completion criteria below.

- [ ] **Supplemental exercise — SBOM:** generate an SPDX document, locate a package/version, and associate it with the correct artifact.

**Cover**

- A **Software Bill of Materials** inventories components, versions, identifiers, relationships, and often license information.
- Know **SPDX** and **CycloneDX** as common SBOM formats.
- An SBOM is not a vulnerability scan or a signature. A vulnerability scanner can use an SBOM to find known vulnerable components.
- Track source → dependencies → build environment → image → registry → deployment digest.
- Build provenance/attestations describe how and from what an artifact was produced. Signatures help bind assertions/artifacts to an expected identity.
- CI/CD controls: minimal credentials, isolated builders, pinned dependencies/actions/base images, protected publishing permissions, scanning, signing, and deployment verification.

Example BOM practice commands:

```bash
bom version
bom generate --help
bom generate --dirs ./source --output source.spdx
bom generate --image registry.example.com/team/app:1.0 --output image.spdx
bom document outline image.spdx
```

Use a real available source directory/image. Check the installed tool's help for supported options and analyzers. With Trivy, a typical image-to-SBOM workflow is:

```bash
trivy image --format spdx-json --output image.spdx.json IMAGE_FROM_LAB
trivy sbom image.spdx.json
```

**Practise:** generate an SBOM, find a package and its version, associate the document with the correct image digest, and scan it if the lab provides that workflow.

**Complete when:** you can distinguish inventory, vulnerabilities, signing, provenance, and admission enforcement—and produce the exact requested output format/path.

**Reference:** [BOM CLI](https://kubernetes-sigs.github.io/bom/cli-reference/).

### SCS3. Registries, digests, signatures, and image admission

**KodeKloud lessons and labs**

- [KK06 03 — Image Security][kk06-03] · [04 — Image Security lab][kk06-04].
- [05 — Whitelist Allowed Registries / ImagePolicyWebhook][kk06-05] · [06 — ImagePolicyWebhook lab][kk06-06].
- No dedicated artifact-signing/provenance lesson appears in this public chapter; complete the signing and verification drill below separately.

**Exam objective:** Secure your supply chain (permitted registries, sign and validate artifacts, etc.).

**Killercoda exercises**

- [ ] [Image Use Digest](https://killercoda.com/killer-shell-cks/scenario/image-use-digest)
- [ ] [ImagePolicyWebhook Setup](https://killercoda.com/killer-shell-cks/scenario/image-policy-webhook-setup)

**Killercoda coverage:** these scenarios cover digests and image admission, not artifact signing. Complete the signing and verification drill below separately.

- [ ] **Supplemental exercise — Signing:** sign a disposable practice artifact if tooling/registry are available; verify the expected signer and demonstrate rejection of an untrusted artifact in the supplied admission system.

**Cover**

- Restrict images to approved registries/repositories, including images in init containers and other relevant workload fields.
- An image reference such as `registry.example.com/team/app@sha256:...` pins content. A mutable tag can point to different content later.
- `imagePullPolicy: Always` is not a signature check or a registry allowlist.
- Registry authentication through `imagePullSecrets` does not enforce permitted image sources.
- Verify artifact signatures against the expected public key or trusted keyless identity and issuer. Apply admission enforcement where the task requires prevention of unverified deployments.

Illustrative key-based verification when Cosign is supplied:

```bash
cosign verify --key cosign.pub IMAGE_AT_DIGEST
```

For a complete signing drill using a disposable registry/repository you control:

```bash
cosign generate-key-pair
cosign sign --key cosign.key IMAGE_AT_DIGEST
cosign verify --key cosign.pub IMAGE_AT_DIGEST
```

The signing command publishes signature material to the configured registry workflow; use the exercise's own artifact and credentials. Practise a negative verification with a different public key as well.

For keyless verification, practise setting the expected certificate identity and OIDC issuer using the installed tool's supported flags. Verification against an arbitrary signer is not verification of the publisher you intended to trust. Use a disposable practice registry to rehearse signing and verification when you need the full workflow.

#### ImagePolicyWebhook drill

1. Inspect the provided webhook service, endpoint, certificate trust, and kubeconfig.
2. Configure the API server's admission configuration for `ImagePolicyWebhook` and enable the plugin.
3. Ensure the admission config, webhook kubeconfig, and referenced certificate files are visible inside the API-server container.
4. Configure the requested caching/retry behavior and failure behavior; `defaultAllow: false` is the fail-closed setting in the ImagePolicyWebhook configuration.
5. Restart/reload as appropriate, verify API health, then test approved and unapproved images.
6. In a lab, test what happens when the webhook is unreachable and account for cached decisions.

**Trap:** specialized `ImagePolicyWebhook` configuration differs from `ValidatingWebhookConfiguration`; the latter uses its own `failurePolicy`. Registry patterns must match the actual image reference, including its registry prefix.

**Reference:** [Admission controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/).

### SCS4. Static analysis of workloads and images

**KodeKloud lessons and labs**

- [KK06 07 — Static Analysis of User Workloads][kk06-07] · [08 — Kubesec lab][kk06-08].

**Exam objective:** Perform static analysis of user workloads and container images (e.g. Kubesec, KubeLinter).

**Killercoda exercises**

- [ ] [Static Manual Analysis K8s](https://killercoda.com/killer-shell-cks/scenario/static-manual-analysis-k8s)
- [ ] [Static Manual Analysis Docker](https://killercoda.com/killer-shell-cks/scenario/static-manual-analysis-docker)
- [ ] **Supplemental exercise — Static analysis:** use at least one actual analyzer in addition to the manual-analysis scenarios.

**Cover:** manual inspection plus tools such as Kubesec and KubeLinter. Understand the finding, its affected field, and a working remediation.

```bash
kubesec scan deployment.yaml
kube-linter lint manifests/
```

Look for:

- Privileged/root execution and unnecessary capabilities.
- Privilege escalation, missing seccomp, and writable root filesystems.
- Host namespaces, dangerous hostPath mounts, and runtime socket exposure.
- Overly powerful service accounts and unnecessary token mounts.
- Embedded credentials, unapproved images, mutable image tags, and risky Dockerfile instructions.
- Resource settings and workload reliability findings reported by the selected tool.

**Practise:** analyse a Pod, a Deployment, a StatefulSet, and a Dockerfile; correct the requested findings; rerun the analysis; confirm the workload still functions.

**Trap:** a clean linter report does not prove runtime security. Fix the issue described by the task rather than optimizing an arbitrary tool score at the expense of application behavior.

### SCS5. Image vulnerability scanning with Trivy

**KodeKloud lessons and labs**

- [KK06 09 — Scan Images for Known Vulnerabilities with Trivy][kk06-09] · [10 — Trivy lab][kk06-10].

**Related exam objectives:** Perform static analysis of user workloads and container images; understand and secure the supply chain. Known-vulnerability scanning complements, but does not replace, manifest/Dockerfile analysis.

**Killercoda exercises**

- [ ] [Image Vulnerability Scanning Trivy](https://killercoda.com/killer-shell-cks/scenario/image-vulnerability-scanning-trivy)

**Cover:** OS and language-package vulnerabilities; severity; CVE identifiers; installed/fixed versions; image digest; database availability/freshness; JSON/table reports.

```bash
trivy image --severity HIGH,CRITICAL IMAGE_FROM_LAB
trivy image --severity HIGH,CRITICAL --format json \
  --output report.json IMAGE_FROM_LAB

jq -r '.Results[]? | .Vulnerabilities[]? |
  [.VulnerabilityID, .PkgName, .InstalledVersion, .FixedVersion, .Severity] |
  @tsv' report.json
```

**Practise:** inventory all relevant workload images, scan the requested set, identify images matching the task's severity/CVE conditions, update or remove only the specified workloads, and rescan/recheck deployment health.

**Traps**

- `--ignore-unfixed` changes the result set; use it only if the task calls for that filter.
- Trivy can return exit code `0` with vulnerabilities unless configured otherwise. For an exercise requiring a failure status on findings, learn `--exit-code`.
- A failed database download or incomplete scan is not a clean scan.
- When the task asks about what is running, relate the report to the running image/digest and workload, not just a similarly named tag.
- Count the requested item correctly: unique CVEs, affected packages, images, and Pods are different quantities.

## 7. Monitoring, Logging and Runtime Security (MLRS, 20%)

**KodeKloud chapter:** [KK07 — Monitoring, Logging and Runtime Security][kk07], with syscall-investigation background in [KK04 — System Hardening][kk04].

### MLRS1. Falco and behavioral threat detection

**KodeKloud lessons and labs**

- [KK07 02 — Behavioral Analytics of Syscall Processes][kk07-02] · [03 — Falco Overview and Installation][kk07-03].
- [04 — Use Falco to Detect Threats][kk07-04] · [05 — Falco Configuration Files][kk07-05] · [06 — Falco lab][kk07-06].

**Exam objectives**

- Perform behavioral analytics to detect malicious activities.
- Detect threats within physical infrastructure, apps, networks, data, users and workloads.

**Killercoda exercises**

- [ ] [Falco Change Rule](https://killercoda.com/killer-shell-cks/scenario/falco-change-rule)
- [ ] [Syscall Activity Strace](https://killercoda.com/killer-shell-cks/scenario/syscall-activity-strace) — supporting host/process observation.

For correlating detections with workload identities and attack phases, continue with MLRS2.

**Cover**

- Falco observes events and evaluates rules; syscall/eBPF/kernel-driver support and other event sources depend on the installation.
- Rule fields: `rule`, `desc`, `condition`, `output`, `priority`; reusable macros and lists; exceptions and overrides.
- Conditions versus output fields: `proc.name` in a condition, `%proc.name` in output.
- Find the active configuration, rule load order, custom-rule location, enabled outputs, and service/DaemonSet managing Falco.
- Alerts such as shells inside containers, unexpected sensitive-file access, unusual network activity, and writes to protected locations.
- Falco normally **detects and reports**; a rule alone does not block a syscall or delete a Pod.

Simple practice rule using a syscall event source:

```yaml
- rule: Lab shell started in container
  desc: Detect a shell process being started in a container
  condition: >
    evt.type in (execve, execveat) and
    container.id != host and
    proc.name in (sh, bash, dash, ash, zsh)
  output: >
    Container shell detected
    (user=%user.name command=%proc.cmdline container_id=%container.id)
  priority: WARNING
```

**Practise:** inspect an existing rule, change its condition/output or exception as required, validate it with the installed Falco version, reload/restart the actual service, and trigger a benign matching event in a lab container.

```bash
falco --help
sudo systemctl status falco --no-pager
sudo journalctl -u falco -n 100 --no-pager
```

The systemd unit may have a different name, or Falco may run as a DaemonSet. For rules using shared macros, validation must include the base rules that define them. Confirm the engine loads the rule and emits the required fields; merely saving a YAML file is insufficient.

**References:** [Falco rule basics](https://falco.org/docs/concepts/rules/basic-elements/), [overriding rules](https://falco.org/docs/concepts/rules/overriding/).

### MLRS2. Runtime investigation and attack timeline

**KodeKloud lessons and labs**

- [KK04 17 — Linux Syscalls and strace][kk04-17] · [18 — AquaSec Tracee][kk04-18].
- [KK07 02 — Behavioral Analytics][kk07-02] · [04 — Falco Threat Detection][kk07-04] · [05 — Falco Configuration][kk07-05] · [06 — Falco lab][kk07-06].
- These support process investigation and alert interpretation; complete the multi-source incident-correlation drill below to practise an evidence-backed attack timeline.

**Exam objective:** Investigate and identify phases of attack and bad actors within the environment. This also supports threat detection across infrastructure and workloads.

**Killercoda exercises**

- [ ] [Syscall Activity Strace](https://killercoda.com/killer-shell-cks/scenario/syscall-activity-strace)
- [ ] [Falco Change Rule](https://killercoda.com/killer-shell-cks/scenario/falco-change-rule) — related alert-investigation practice.
- [ ] **Supplemental exercise — Incident correlation:** combine an audit event, runtime alert, and workload identity into a short evidence-backed timeline.

**Cover**

- Relate Kubernetes namespace/Pod/container identity to a runtime container ID and host process.
- Read process command lines, parents, open files, sockets, and syscall activity.
- Correlate timestamps from audit events, Falco output, application logs, and host logs.
- Recognize a possible sequence: initial access → execution → privilege escalation → credential access → lateral movement → persistence/exfiltration.
- Select evidence that supports your conclusion; do not infer an attack solely from a high-CPU process or an unfamiliar name.

```bash
k -n team-a get pod suspicious -o wide
k -n team-a get pod suspicious -o jsonpath='{.status.containerStatuses[*].containerID}{"\n"}'
sudo crictl ps -a
sudo crictl inspect CONTAINER_ID
ps -eo pid,ppid,user,cmd
sudo ss -tpn
sudo lsof -p PID
sudo strace -f -p PID -e trace=network,file
```

Use the tools and observation duration requested by the exercise. `strace` attaches to a process and can add overhead; stop tracing when you have the required evidence.

**Practise:** find which Pod triggered an alert, identify the process and accessed file/destination, save the requested evidence, and apply the requested containment to the correct workload.

**Trap:** deleting a Pod managed by a controller commonly results in a replacement. Know whether the task requires a Pod action, controller change, credential change, or policy change.

### MLRS3. Immutability of containers at runtime

**KodeKloud lessons and labs**

- [KK07 07 — Mutable vs Immutable Infrastructure][kk07-07].
- [08 — Ensure Immutability of Containers at Runtime][kk07-08] · [09 — Container Immutability lab][kk07-09].

**Exam objective:** Ensure immutability of containers at runtime.

**Killercoda exercises**

- [ ] [Immutability Readonly Filesystem](https://killercoda.com/killer-shell-cks/scenario/immutability-readonly-fs)

**Cover:** read-only root filesystem, controlled writable volumes, dropped capabilities, non-root execution, no privilege escalation, minimal runtime tools, and replacement through trusted images rather than ad hoc mutation.

Container fragment:

```yaml
securityContext:
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: [ALL]
volumeMounts:
  - name: tmp
    mountPath: /tmp
```

Add the corresponding Pod volume, usually an `emptyDir` for temporary data, and ensure the workload's UID/GID can write where required. For NGINX-like applications, inspect their configured PID/cache/temp paths rather than mounting a writable volume over the entire root filesystem.

**Practise:** harden an existing Deployment, use logs to discover necessary writable directories, provide only those writable mounts, and verify both successful startup and failed writes to the image filesystem.

**Verify:** controller template, new Pods, application response, forbidden write, allowed temporary write. Distinguish Kubernetes immutable object fields from the security goal of an immutable container filesystem.

### MLRS4. Kubernetes audit policy and logs

**KodeKloud lessons and labs**

- [KK07 10 — Use Audit Logs to Monitor Access][kk07-10] · [11 — Audit Logs lab][kk07-11].

**Exam objective:** Use Kubernetes audit logs to monitor access.

**Killercoda exercises**

- [ ] [Auditing Enable Audit Logging](https://killercoda.com/killer-shell-cks/scenario/auditing-enable-audit-logs)

**Cover**

- Audit levels: `None`, `Metadata`, `Request`, `RequestResponse`.
- Policy matching by users/groups, verbs, resources/subresources, namespaces, and non-resource URLs.
- **First matching rule wins.** Put specific rules ahead of a broad catch-all.
- Audit stages and `omitStages`; log path, retention/rotation flags, and optional webhook backends.
- Audit events describe API activity, including actor, verb, target, time, source address, and result. They do not automatically record every process/filesystem event inside a container.

Example policy: record Secret access metadata without logging Secret bodies, detailed ConfigMap changes in `team-a`, and metadata for other requests:

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - RequestReceived
rules:
  - level: Metadata
    resources:
      - group: ""
        resources: [secrets]
  - level: RequestResponse
    verbs: [create, update, patch, delete]
    namespaces: [team-a]
    resources:
      - group: ""
        resources: [configmaps]
  - level: Metadata
```

Use exactly the levels and scopes requested by the task. Higher-detail levels can include sensitive bodies.

**Enable-and-verify drill**

1. Save the policy on the designated control-plane host.
2. Add the policy-file mount, writable audit-log directory mount, and appropriate hostPath volumes to the API-server manifest.
3. Set `--audit-policy-file`, `--audit-log-path`, and the requested `--audit-log-maxage`, `--audit-log-maxbackup`, and `--audit-log-maxsize` values.
4. Confirm the API server is healthy after the change.
5. Generate a matching API request; inspect the expected log file and fields.
6. Confirm a differently scoped request produces the intended lower-detail/no-log behavior.

```bash
k -n team-a get secrets
sudo tail -n 20 /var/log/kubernetes/audit.log

sudo jq -r 'select(.objectRef.resource == "secrets") |
  [.requestReceivedTimestamp, .user.username, .verb,
   .objectRef.namespace, .objectRef.name, .responseStatus.code] |
  @tsv' /var/log/kubernetes/audit.log
```

Use the configured log path. `list` requests need not identify a single resource name. Correlate `auditID` and stage when multiple records describe one request.

**Trap:** creating a policy file without adding the API-server argument and mounts does not enable auditing. Kubernetes audit logs, `kubectl logs`, and Falco events are different evidence sources.

**Reference:** [Kubernetes auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/).

## 8. Cluster Setup (CS, 15%)

**KodeKloud chapter:** [KK03 — Cluster Setup and Hardening][kk03]. KodeKloud combines this domain and Cluster Hardening in one chapter; the links below select the Cluster Setup material.

### CS1. NetworkPolicy

**KodeKloud lessons and labs**

- [KK03 40 — Network Policy][kk03-40] · [41 — Developing Network Policies][kk03-41] · [42 — Network Policies lab][kk03-42].

**Exam objective:** Use Network security policies to restrict cluster level access.

**Killercoda exercises**

- [ ] [NetworkPolicy Create Default Deny](https://killercoda.com/killer-shell-cks/scenario/networkpolicy-create-default-deny)
- [ ] [NetworkPolicy Namespace Selector](https://killercoda.com/killer-shell-cks/scenario/networkpolicy-namespace-communication)
- [ ] [Cilium Network Policy Playground](https://killercoda.com/killer-shell-cks/scenario/playground-cilium-network-policy)

**Cover**

- Ingress and egress isolation are independent. Policies select Pods in the policy's own namespace.
- No selecting policy for a direction means the Pod is not isolated in that direction by standard NetworkPolicy.
- Allow rules are **additive** across policies; a default-deny policy does not override another policy's allow rule.
- `podSelector`, `namespaceSelector`, `ipBlock`, `except`, ports, protocols, and explicit `policyTypes`.
- A namespace selector and Pod selector in the **same peer item** are ANDed; separate peer items are ORed.
- A peer `podSelector` without `namespaceSelector` selects Pods in the policy namespace.
- Empty selector versus empty rule list versus an empty rule object: `podSelector: {}`, `ingress: []`, and `ingress: [{}]` have very different meanings.
- A policy-capable CNI is required. Standard NetworkPolicy does not implement application-layer authorization or encryption.

Default deny both directions:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: backend
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
```

Allow only `app=frontend` Pods in namespace `frontend` to reach `app=api` Pods on TCP 8080:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-api
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes: [Ingress]
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: frontend
          podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

If the source Pod is egress-isolated, it also needs an egress rule for that destination. When egress is isolated, account for the actual DNS path, usually UDP **and TCP 53**. Discover DNS Pod labels and any NodeLocal DNS setup rather than copying selectors blindly.

**Practise:** default deny, same-namespace allow, cross-namespace allow, destination-specific egress, DNS access, and CIDR exceptions. If using the Cilium playground, recognize `CiliumNetworkPolicy` as a separate CRD with additional features.

**Verify:** use at least an allowed source, wrong-label source, and wrong-namespace source; test both permitted and unpermitted ports. Confirm the application is listening before attributing every timeout to a policy.

**Reference:** [NetworkPolicy documentation](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

### CS2. CIS benchmarks and kube-bench

**KodeKloud lessons and labs**

- [KK03 02 — What Are CIS Benchmarks?][kk03-02] · [03 — CIS Assessment on Ubuntu lab][kk03-03].
- [04 — CIS Benchmark for Kubernetes][kk03-04] · [05 — kube-bench][kk03-05] · [06 — kube-bench lab][kk03-06].

**Exam objective:** Use CIS benchmark to review the security configuration of Kubernetes components (etcd, kubelet, kubedns, kubeapi).

**Killercoda exercises**

- [ ] [CIS Benchmarks Fix Controlplane](https://killercoda.com/killer-shell-cks/scenario/cis-benchmarks-kube-bench-fix-controlplane)

**Cover**

- CIS benchmarks describe secure configuration checks. `kube-bench` checks configuration; it is not an image vulnerability scanner.
- Read check IDs, findings, rationale, and remediation. Understand `PASS`, `FAIL`, and checks requiring manual review.
- Locate the effective API-server/controller-manager/scheduler/etcd arguments and kubelet configuration.
- Check authentication, authorization, TLS, file ownership/permissions, and exposure of component endpoints.
- Benchmark/check IDs depend on the benchmark and Kubernetes version. Match the provided benchmark to the environment.

**Workflow**

1. Run the provided assessment command and inspect the specific failing checks.
2. Back up the affected configuration outside watched directories.
3. Fix the relevant flag, configuration field, or file permission.
4. Wait for/restart the correct component as required.
5. Check cluster health, then rerun the affected checks.

```bash
sudo kube-bench --help
sudo kube-bench run --targets master
sudo stat /etc/kubernetes/pki
sudo journalctl -u kubelet -n 100 --no-pager
k get --raw='/readyz?verbose'
```

`master` is an example target used by some benchmark definitions; use the target supported by the installed version and lab. Do not copy a blanket recursive `chmod` over PKI files—directories, public certificates, and private keys have different requirements.

**Complete when:** you can remediate a few named findings while keeping the API server and nodes healthy.

### CS3. Ingress with TLS

**KodeKloud lessons and labs**

- TLS foundations: [KK03 11 — TLS Introduction][kk03-11] · [12 — TLS Basics][kk03-12] · [13 — TLS in Kubernetes][kk03-13].
- Certificates: [14 — Certificate Creation][kk03-14] · [15 — View Certificate Details][kk03-15] · [16 — Certificate Inspection lab][kk03-16].
- Ingress: [43 — Ingress][kk03-43] · [44 — Ingress lab 1][kk03-44] · [45 — Annotations and Rewrite Target][kk03-45] · [46 — Ingress lab 2][kk03-46].

**Exam objective:** Properly set up Ingress with TLS.

**Killercoda exercises**

- [ ] [Ingress Create](https://killercoda.com/killer-shell-cks/scenario/ingress-create)
- [ ] [Ingress Secure](https://killercoda.com/killer-shell-cks/scenario/ingress-secure)

**Cover:** IngressClass/controller, routing by host/path, Service port versus target port, TLS Secret format, certificate SANs/expiry, and where TLS terminates.

```bash
k -n web create secret tls web-tls --cert=server.crt --key=server.key
k get ingressclass
k -n web get svc,endpointslices
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
  namespace: web
spec:
  ingressClassName: nginx
  tls:
    - hosts: [web.example.com]
      secretName: web-tls
  rules:
    - host: web.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
```

Use the actual IngressClass. The Secret belongs in the **Ingress namespace**. `tls.hosts`, the routing host, and the certificate's SAN must align.

**Verify:** `curl --resolve web.example.com:443:INGRESS_IP --cacert ca.crt https://web.example.com/`. Supplying the hostname preserves SNI and hostname verification. `curl -k` can help diagnose routing, but does not prove certificate trust.

**Trap:** Ingress TLS commonly protects client-to-controller traffic; it does not automatically encrypt controller-to-backend traffic. Know which connection the task asks you to secure.

### CS4. Node metadata and endpoint protection

**Related KodeKloud lessons**

- [KK03 27 — Kubelet Security][kk03-27] · [40 — Network Policy][kk03-40] · [41 — Developing Network Policies][kk03-41].
- [KK04 13 — Minimize IAM Roles][kk04-13] · [14 — Minimize External Network Access][kk04-14] · [15 — UFW Firewall Basics][kk04-15].
- No dedicated node-metadata lesson appears in these public notes. Use these endpoint/network/IAM foundations with the metadata-protection Killercoda task and verification checks below.

**Exam objective:** Protect node metadata and endpoints.

**Killercoda exercises**

- [ ] [NetworkPolicy Metadata Protection](https://killercoda.com/killer-shell-cks/scenario/networkpolicy-metadata-protection)

**Cover**

- Cloud metadata services can expose instance identity and credentials; `169.254.169.254` is a common IPv4 metadata address.
- Restrict workload access while preserving required egress. Understand provider-side protections and least-privilege node IAM.
- Identify exposure of kubelet, API server, etcd, dashboards, NodePorts, and runtime sockets. Pod NetworkPolicy and node firewalls operate at different boundaries.

Example: allow IPv4 egress except the common metadata address for selected Pods:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-metadata
  namespace: apps
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes: [Egress]
  egress:
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
            except: [169.254.169.254/32]
```

This example deliberately permits other IPv4 destinations. It is not a general least-privilege egress policy. Another matching allow policy can reallow metadata traffic. Check IPv6/provider-specific endpoints, CNI behavior, NAT, and host-network workloads when applicable.

**Verify:** metadata requests from the selected Pod fail while explicitly required traffic still works. A host-level `curl` does not demonstrate Pod-level policy enforcement.

### CS5. Verify platform binaries

**KodeKloud lessons and labs**

- [KK03 34 — Verify Platform Binaries Before Deploying][kk03-34] · [35 — Platform Binary Verification lab][kk03-35].

**Exam objective:** Verify platform binaries before deploying.

**Killercoda exercises**

- [ ] [Verify Platform Binaries](https://killercoda.com/killer-shell-cks/scenario/verify-platform-binaries-kubelet)

**Cover:** exact binary version, operating system/architecture, SHA-256, trusted published checksums, and integrity versus publisher authenticity.

```bash
command -v kubelet
readlink -f "$(command -v kubelet)"
kubelet --version
sha256sum /usr/bin/kubelet

# For a downloaded lab binary and its trusted checksum file:
printf '%s  %s\n' "$(cat kubelet.sha256)" kubelet | sha256sum --check
```

Use the actual binary location. A checksum mismatch can result from comparing a different version or architecture. A matching checksum is only meaningful if the reference checksum is trusted; a hash does not by itself prove who published the file.

**Complete when:** you can identify a changed binary and compare a correct replacement against the expected trusted checksum.

## 9. Cluster Hardening (CH, 15%)

**KodeKloud chapter:** [KK03 — Cluster Setup and Hardening][kk03]. The links below focus on identities, API access, component exposure, and upgrades.

### CH1. RBAC and least privilege

**KodeKloud lessons and labs**

- [KK03 22 — Authorization][kk03-22] · [23 — RBAC][kk03-23] · [24 — RBAC lab][kk03-24].
- [25 — Cluster Roles and Role Bindings][kk03-25] · [26 — Cluster Roles/Bindings lab][kk03-26].

**Exam objective:** Use Role Based Access Controls to minimize exposure.

**Killercoda exercises**

- [ ] [RBAC ServiceAccount Permissions](https://killercoda.com/killer-shell-cks/scenario/rbac-serviceaccount-permissions)
- [ ] [RBAC User Permissions](https://killercoda.com/killer-shell-cks/scenario/rbac-user-permissions)

**Cover**

- Role/ClusterRole versus RoleBinding/ClusterRoleBinding; subjects, `roleRef`, verbs, API groups, resources, subresources, and `resourceNames`.
- Core resources use `apiGroups: [""]`; Deployments use `apps`.
- A RoleBinding may reference a ClusterRole but grants its namespaced permissions only in the binding namespace.
- `pods/log` and `pods/exec` are subresources. Creating an exec session commonly needs `create` on `pods/exec`.
- Permissions are additive. Remove or narrow the actual overly broad grant; adding a narrow Role does not cancel `cluster-admin` elsewhere.
- Recognize escalation through Secret access, workload creation with powerful service accounts, impersonation, and `bind`/`escalate` privileges.
- A binding's `roleRef` is immutable; changing the referenced role requires replacing the binding.

```bash
k -n team-a create role pod-reader --verb=get,list,watch --resource=pods
k -n team-a create rolebinding pod-reader \
  --role=pod-reader --serviceaccount=team-a:reader

k auth can-i list pods -n team-a --as=system:serviceaccount:team-a:reader
k auth can-i delete pods -n team-a --as=system:serviceaccount:team-a:reader
k auth can-i list pods -n team-b --as=system:serviceaccount:team-a:reader
```

**Practise:** namespace-scoped read access, a narrowly scoped cluster-level permission, access to a named resource, and finding/removing an excessive binding.

**Trap:** `resourceNames` does not simply restrict every verb in the same way; collection operations and resource creation have important limitations. Consult the RBAC documentation for the exact operation.

### CH2. Service accounts and tokens

**KodeKloud lessons and labs**

- [KK03 09 — Service Accounts][kk03-09] · [10 — Service Accounts lab][kk03-10].
- For permissions, also use [23 — RBAC][kk03-23] and [24 — RBAC lab][kk03-24]. Apply the projected-token and automount guidance below when course examples show older token behavior.

**Exam objective:** Exercise caution in using service accounts e.g. disable defaults, minimize permissions on newly created ones.

**Killercoda exercises**

- [ ] [ServiceAccount Token Mounting](https://killercoda.com/killer-shell-cks/scenario/serviceaccount-token-mounting)
- [ ] [RBAC ServiceAccount Permissions](https://killercoda.com/killer-shell-cks/scenario/rbac-serviceaccount-permissions) — minimize permissions.
- [ ] [Secret ServiceAccount Pod](https://killercoda.com/killer-shell-cks/scenario/secret-serviceaccount-pod) — related Secret integration; also listed under MMV3.

**Cover:** namespace-scoped identities; `serviceAccountName`; token automount; TokenRequest/projected tokens; token audience/expiry; RBAC; and image-pull credentials.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app
  namespace: team-a
automountServiceAccountToken: false
```

Pod-spec fragment:

```yaml
spec:
  serviceAccountName: app
  automountServiceAccountToken: false
```

- The Pod's explicit automount setting takes precedence over the service account's setting.
- Modern Kubernetes normally uses projected, short-lived service-account tokens; do not assume every service account has an automatically created token Secret.
- Turning off automount does not remove existing RBAC permissions. RBAC and credential exposure are separate checks.
- Updating a service account's automount setting does not remove a token volume from existing Pods; recreate/roll out affected workloads when required.
- Applications that need API access should receive only the necessary permissions and appropriate credentials.

```bash
k -n team-a create token app --duration=10m
k -n team-a get pod web -o yaml
k auth can-i list secrets -n team-a --as=system:serviceaccount:team-a:app
```

**Verify:** inspect the new Pod's projected volumes/mounts and test the intended API operations. `imagePullSecrets` authenticate image pulls and are not the same as API-access tokens.

### CH3. Authentication, TLS, CSR, and kubeconfig

**KodeKloud lessons and labs**

- [KK03 08 — Authentication][kk03-08] · [13 — TLS in Kubernetes][kk03-13] · [14 — Certificate Creation][kk03-14].
- [15 — View Certificate Details][kk03-15] · [16 — Certificate Inspection lab][kk03-16].
- [17 — Certificates API][kk03-17] · [18 — Certificates API lab][kk03-18].
- [19 — KubeConfig][kk03-19] · [20 — KubeConfig lab][kk03-20].

**Related exam objective:** Restrict access to Kubernetes API; these identity and certificate foundations also support least-privilege RBAC.

**Killercoda exercises**

- [ ] [CSRs: Sign Manually](https://killercoda.com/killer-shell-cks/scenario/certificate-signing-requests-sign-manually)
- [ ] [CSRs: Sign via API](https://killercoda.com/killer-shell-cks/scenario/certificate-signing-requests-sign-k8s)

**Cover**

- Client versus serving certificates; CA trust; SANs; expiry; client certificate common name and organization mapping to user/groups.
- Kubeconfig clusters, users, contexts, CA data, client certificate/key data, and namespace selection.
- The `certificates.k8s.io/v1` CSR workflow, `signerName`, usages, approval, and certificate retrieval.
- Authentication establishes identity; RBAC still controls what that identity can do.

```bash
openssl genrsa -out alice.key 2048
openssl req -new -key alice.key -out alice.csr -subj '/CN=alice/O=developers'
base64 -w0 alice.csr

k get csr
k describe csr alice
k certificate approve alice
k get csr alice -o jsonpath='{.status.certificate}' | base64 -d > alice.crt
openssl x509 -in alice.crt -noout -subject -issuer -dates
```

For a client CSR object, practise `signerName: kubernetes.io/kube-apiserver-client` and `usages: [client auth]`. Approval is not permission to use every Kubernetes resource.

**Complete when:** you can authenticate with a new lab identity, grant only the requested rights, and demonstrate that additional operations are denied.

### CH4. API-server and kubelet hardening; recovery

**KodeKloud lessons and labs**

- [KK03 07 — Kubernetes Security Primitives][kk03-07] · [08 — Authentication][kk03-08] · [22 — Authorization][kk03-22].
- [27 — Kubelet Security][kk03-27] · [28 — Kubelet Security lab][kk03-28].
- [29 — Kubectl Proxy and Port Forward][kk03-29] · [30 — Proxy/Port Forward lab][kk03-30].
- [31 — Kubernetes Dashboard][kk03-31] · [32 — Securing Kubernetes Dashboard][kk03-32] · [33 — Dashboard Security lab][kk03-33] — supporting endpoint-exposure material; follow the supplied environment rather than assuming a Dashboard is installed.

**Exam objective:** Restrict access to Kubernetes API. Component hardening also supports the CIS and endpoint-protection objectives in Cluster Setup.

**Killercoda exercises**

- [ ] [Apiserver Crash](https://killercoda.com/killer-shell-cks/scenario/apiserver-crash)
- [ ] [Apiserver Misconfigured](https://killercoda.com/killer-shell-cks/scenario/apiserver-misconfigured)
- [ ] [Apiserver NodeRestriction](https://killercoda.com/killer-shell-cks/scenario/apiserver-node-restriction)

**Cover**

- API-server authentication, RBAC/Node authorization, admission configuration, and network exposure.
- Kubelet anonymous authentication, webhook authorization, client CA, serving TLS, and the read-only port.
- Node authorizer and NodeRestriction: the admission plugin constrains authenticated kubelet identities; it is not a replacement for RBAC or network controls.
- API-server-to-etcd TLS and client credentials; protect etcd endpoints and data from direct access.
- Binding addresses and authentication for dashboards/proxies. Prefer narrowly scoped access rather than exposing privileged interfaces.
- Recognize obsolete API-server flags in old training examples. Inspect the installed version's options and existing configuration.

Example kubelet configuration fragment:

```yaml
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
authorization:
  mode: Webhook
readOnlyPort: 0
```

Merge required changes into the real configuration; retain the existing CA and other required settings. Common API-server settings to recognize include `--authorization-mode=Node,RBAC`, `--anonymous-auth=false`, and enabling `NodeRestriction`, but use the configuration mechanism and requirements of the installed version/task.

**Static-Pod recovery drill**

```bash
sudo mkdir -p /root/cks-backups
sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml \
  /root/cks-backups/kube-apiserver.yaml

sudo crictl ps -a --name kube-apiserver
sudo crictl logs CONTAINER_ID
sudo journalctl -u kubelet -n 100 --no-pager
```

1. Fix YAML indentation, an invalid argument, an incorrect path, or a broken etcd endpoint.
2. Confirm referenced host files exist and are mounted at the container paths used by flags.
3. Wait for kubelet to recreate the static Pod after a manifest change. Restarting kubelet alone does not inherently restart every healthy static Pod.
4. Verify `k get --raw='/readyz?verbose'`, node readiness, and relevant workloads.

**Trap:** deleting a mirror Pod through the API does not change the static Pod's source manifest. Use node-side tools when the API is unavailable.

### CH5. Upgrade Kubernetes to address vulnerabilities

**KodeKloud lessons and labs**

- [KK03 36 — Kubernetes Software Versions][kk03-36] · [37 — Cluster Upgrade Process][kk03-37].
- [38 — Cluster Upgrade Demo][kk03-38] · [39 — Cluster Upgrade lab][kk03-39]. Use the version-specific official upgrade guide below for the actual target release and package repositories.

**Exam objective:** Upgrade Kubernetes to avoid vulnerabilities.

**Killercoda coverage:** no dedicated upgrade scenario appears in the current Killer Shell CKS catalog. Use a suitable kubeadm lab or the [Playground](https://killercoda.com/killer-shell-cks/scenario/playground) only if its environment supports the required upgrade workflow; follow the version-specific guide linked below.

- [ ] **Supplemental exercise — Upgrade:** perform a supported kubeadm upgrade, return nodes to Ready/schedulable state, and check applications.

**Cover:** supported version skew, patch/minor upgrades, package repositories/versions, `kubeadm upgrade`, node draining, kubelet restart, and post-upgrade checks.

**Practise this sequence using the version-specific kubeadm guide:**

1. Record component versions and identify the target version from the task.
2. Make required backups and check health before beginning.
3. Configure the correct package repository and upgrade `kubeadm` to the required version.
4. On the first control-plane node, use `kubeadm upgrade plan`, then `kubeadm upgrade apply` for the specified version.
5. For additional control-plane/worker nodes, use the appropriate `kubeadm upgrade node` workflow.
6. Drain a node before upgrading/restarting its kubelet, respecting workload constraints and PodDisruptionBudgets.
7. Upgrade the required kubelet/kubectl packages, reload systemd if needed, restart kubelet, verify, and uncordon.
8. Repeat one node at a time as applicable.

```bash
k get nodes
sudo kubeadm upgrade plan
k drain node01 --ignore-daemonsets
sudo systemctl daemon-reload
sudo systemctl restart kubelet
k get nodes
k uncordon node01
```

The command block is a reminder, not a complete upgrade script. Resolve local-storage or unmanaged-Pod blockers according to the task rather than automatically adding destructive drain flags. Minor-version skipping is not supported by the normal kubeadm upgrade workflow.

**Reference:** [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/).

## 10. System Hardening (SH, 10%)

**KodeKloud chapter:** [KK04 — System Hardening][kk04]. Topic-specific links cover host access, host/network footprint, syscall restrictions, and AppArmor.

### SH1. Host hardening, least privilege, SSH, and firewall

**KodeKloud lessons and labs**

- Host/access foundations: [KK04 02 — Least Privilege][kk04-02] · [03 — Minimize Host OS Footprint][kk04-03] · [04 — Limit Node Access][kk04-04] · [05 — Node Access lab][kk04-05].
- SSH/sudo: [06 — SSH Hardening][kk04-06] · [07 — Privilege Escalation in Linux][kk04-07] · [08 — SSH Hardening and sudo lab][kk04-08].
- Attack surface: [09 — Remove Obsolete Packages and Services][kk04-09] · [10 — Restrict Kernel Modules][kk04-10] · [11 — Identify and Disable Open Ports][kk04-11] · [12 — Ports/Packages/Services lab][kk04-12].
- Identity/network exposure: [13 — Minimize IAM Roles][kk04-13] · [14 — Minimize External Network Access][kk04-14].
- Firewall: [15 — UFW Basics][kk04-15] · [16 — UFW lab][kk04-16].

**Exam objectives**

- Minimize host OS footprint (reduce attack surface).
- Using least-privilege identity and access management.
- Minimize external access to the network.

**Killercoda exercises**

- [ ] [Manage Packages](https://killercoda.com/killer-shell-cks/scenario/system-hardening-manage-packages) — host OS footprint.
- [ ] [Close Open Ports](https://killercoda.com/killer-shell-cks/scenario/system-hardening-close-open-ports) — external network exposure.

**Additional practice:** no dedicated host SSH/sudo/IAM scenario appears in this catalog; use the least-privilege checks and commands below in a suitable host lab. Kubernetes identity permissions are covered separately in [CH1](#ch1-rbac-and-least-privilege).

**Cover**

- Identify unnecessary packages, services, open ports, privileged users, and kernel modules.
- Understand `stop` versus `disable` versus `mask` for systemd services.
- SSH key authentication, root-login restrictions, password authentication, and configuration validation.
- Sudo policies, least-privilege users/groups, file ownership/modes, and cloud IAM permissions.
- Host firewall rules with the tool provided: UFW, iptables, or nftables. Understand inbound/outbound direction and rule ordering.
- Keep required management, Kubernetes, DNS, and CNI traffic working while reducing exposure.

```bash
sudo ss -lntup
sudo lsof -iTCP -sTCP:LISTEN -nP
sudo systemctl list-units --type=service --state=running
sudo systemctl status SERVICE --no-pager
dpkg -l
sudo -l
sudo visudo -c
sudo sshd -t
sudo ufw status numbered
lsmod
```

**Practise:** map a listening port to its process and owning service; stop and disable the named unwanted service; remove the named package when requested; verify the listener is gone and the intended change persists.

**Traps:** killing a PID can be temporary because systemd restarts it. Deleting a firewall rule by number changes the remaining numbers. Appending an SSH setting may not override an earlier effective setting or an included file; inspect effective configuration before reload.

### SH2. seccomp and syscall restrictions

**KodeKloud lessons and labs**

- [KK04 17 — Linux Syscalls][kk04-17] · [18 — AquaSec Tracee][kk04-18] — observation/background.
- [19 — Restrict Syscalls with seccomp][kk04-19] · [20 — Implement seccomp in Kubernetes][kk04-20] · [21 — seccomp lab][kk04-21] — enforcement practice.

**Exam objective:** Appropriately use kernel hardening tools such as AppArmor, seccomp.

**Related Killercoda exercise**

- [ ] [Syscall Activity Strace](https://killercoda.com/killer-shell-cks/scenario/syscall-activity-strace) — observe syscalls; this does not configure or verify seccomp enforcement.

**Killercoda coverage:** no dedicated seccomp scenario appears in this catalog. Complete the [official seccomp tutorial](https://kubernetes.io/docs/tutorials/security/seccomp/) and the profile drills below.

- [ ] **Supplemental exercise — seccomp:** use RuntimeDefault, then a provided Localhost profile; verify one blocked syscall and a functioning application.

**Cover**

- seccomp filters system calls. It does not replace filesystem permissions, RBAC, or NetworkPolicy.
- `RuntimeDefault`, `Localhost`, and `Unconfined`; Pod-level defaults and container overrides.
- Localhost JSON profile structure: `defaultAction`, syscall names, actions, and architecture requirements.
- A local profile must exist on each node that can run the workload, under the kubelet's seccomp profile directory.

Pod-spec fragment:

```yaml
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
```

For a provided profile stored as `/var/lib/kubelet/seccomp/profiles/custom.json` with the default kubelet root:

```yaml
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/custom.json
```

**Practise:** apply RuntimeDefault, load a provided local profile, diagnose a missing profile, and demonstrate one prohibited syscall while normal application behavior still works.

**Verify:** inspect Pod events and runtime errors; where tooling is available, check `/proc/1/status` and use `strace` to understand blocked operations. Merely seeing `Seccomp: 2` indicates filtering, not that the intended custom policy is correct.

**Reference:** [Kubernetes seccomp tutorial](https://kubernetes.io/docs/tutorials/security/seccomp/).

### SH3. AppArmor

**KodeKloud lessons and labs**

- [KK04 22 — AppArmor][kk04-22] · [23 — Creating AppArmor Profiles][kk04-23] · [24 — AppArmor in Kubernetes][kk04-24].
- [25 — Linux Capabilities][kk04-25] · [26 — AppArmor lab][kk04-26]. Recognize legacy annotations in older notes, but use the structured API and verification guidance below for current Kubernetes versions.

**Exam objective:** Appropriately use kernel hardening tools such as AppArmor, seccomp.

**Killercoda exercises**

- [ ] [AppArmor](https://killercoda.com/killer-shell-cks/scenario/apparmor)

**Cover:** path-based application confinement; loaded profile name versus filename; complain versus enforce mode; node availability; Pod/container attachment.

```bash
sudo aa-status
sudo apparmor_parser -r /etc/apparmor.d/cks-profile
sudo journalctl -k --no-pager | grep -i apparmor
```

Container security-context fragment using the current structured API:

```yaml
securityContext:
  appArmorProfile:
    type: Localhost
    localhostProfile: cks-profile
```

Use the **profile name declared inside the loaded profile**, not automatically its filename. Load it on every eligible node or ensure the workload is scheduled only where it exists. Recognize the legacy annotation form in old exercises, but check the current API with `kubectl explain`.

**Practise:** load a supplied profile, attach it, fix a misspelled profile reference, and confirm a forbidden file write fails while allowed work succeeds.

**Verify:** profile mode/load status, Pod events, and `/proc/1/attr/current` in a suitable test container. Complain mode records ordinary violations instead of enforcing them; verify actual denial behavior rather than relying on the profile name alone.

**Reference:** [Kubernetes AppArmor tutorial](https://kubernetes.io/docs/tutorials/security/apparmor/).

## 11. Miscellaneous

**Related KodeKloud preparation:** [Course Introduction][kk01-01] · [Mock Exams Introduction][kk08-01]. No dedicated Vim or exam-desktop setup lesson appears in the public CKS notes; use the Killercoda setup exercises below for those workflows.

These exercises support general preparation rather than one specific weighted exam domain. All other public catalog entries are linked under the related exam topics above.

### General environment and editor practice

- [ ] [Playground](https://killercoda.com/killer-shell-cks/scenario/playground) — general Kubernetes practice environment.
- [ ] [Exam Desktop](https://killercoda.com/killer-shell-cks/scenario/playground-exam-desktop) — practise the exam-style desktop workflow.
- [ ] [Vim Setup](https://killercoda.com/killer-shell-cks/scenario/vim-setup) — editor configuration and speed.

Related preparation material: [Tips](#1-tips-first) and [kubectl, shell, and editor tricks](#2-normal-kubectl-shell-and-editor-tricks).

## 12. Killercoda practice catalog

**KodeKloud counterpart:** each topic above links its related course lesson/lab notes. For additional mixed practice beyond this Killercoda index, use [KodeKloud Mock Exams][kk08] and [CKS Challenges][kk09].

This table maps all **42 public entries** returned by the [Killer Shell CKS profile][kc] when researched. Several entries are playgrounds/setup exercises. Follow the task shown inside each scenario; availability and contents can change.

The live profile was rechecked during this reorganization and still lists the same 42 entries. This consolidated index is retained in addition to the task checkboxes inside each topic; its groups follow the same highest-to-lowest domain order.

| Topics | Scenarios |
| --- | --- |
| **Minimize Microservice Vulnerabilities — 20%** | |
| MMV1 | [Privileged Containers](https://killercoda.com/killer-shell-cks/scenario/privileged-containers), [Privilege Escalation Containers](https://killercoda.com/killer-shell-cks/scenario/privilege-escalation-containers) |
| MMV3, CH2 | [Secret Access in Pods](https://killercoda.com/killer-shell-cks/scenario/secret-pod-access), [Secret Read and Decode](https://killercoda.com/killer-shell-cks/scenario/secret-read-secrets), [Secret ServiceAccount Pod](https://killercoda.com/killer-shell-cks/scenario/secret-serviceaccount-pod), [Secret ETCD Encryption](https://killercoda.com/killer-shell-cks/scenario/secret-etcd-encryption) |
| F0, MMV4 | [Container Namespaces Docker](https://killercoda.com/killer-shell-cks/scenario/container-namespaces-docker), [Container Namespaces Podman](https://killercoda.com/killer-shell-cks/scenario/container-namespaces-podman) |
| MMV4 | [Sandbox gVisor](https://killercoda.com/killer-shell-cks/scenario/sandbox-gvisor) |
| **Supply Chain Security — 20%** | |
| SCS1 | [Container Image Footprint User](https://killercoda.com/killer-shell-cks/scenario/container-image-footprint-user), [Container Hardening](https://killercoda.com/killer-shell-cks/scenario/container-hardening) |
| SCS3, MMV6 | [ImagePolicyWebhook Setup](https://killercoda.com/killer-shell-cks/scenario/image-policy-webhook-setup), [Image Use Digest](https://killercoda.com/killer-shell-cks/scenario/image-use-digest) |
| SCS4 | [Static Manual Analysis K8s](https://killercoda.com/killer-shell-cks/scenario/static-manual-analysis-k8s), [Static Manual Analysis Docker](https://killercoda.com/killer-shell-cks/scenario/static-manual-analysis-docker) |
| SCS5 | [Image Vulnerability Scanning Trivy](https://killercoda.com/killer-shell-cks/scenario/image-vulnerability-scanning-trivy) |
| **Monitoring, Logging and Runtime Security — 20%** | |
| MLRS1 | [Falco Change Rule](https://killercoda.com/killer-shell-cks/scenario/falco-change-rule) |
| MLRS2, SH2 | [Syscall Activity Strace](https://killercoda.com/killer-shell-cks/scenario/syscall-activity-strace) |
| MLRS3 | [Immutability Readonly Filesystem](https://killercoda.com/killer-shell-cks/scenario/immutability-readonly-fs) |
| MLRS4 | [Auditing Enable Audit Logging](https://killercoda.com/killer-shell-cks/scenario/auditing-enable-audit-logs) |
| **Cluster Setup — 15%** | |
| CS1 | [NetworkPolicy Create Default Deny](https://killercoda.com/killer-shell-cks/scenario/networkpolicy-create-default-deny), [NetworkPolicy Namespace Selector](https://killercoda.com/killer-shell-cks/scenario/networkpolicy-namespace-communication), [Cilium Network Policy Playground](https://killercoda.com/killer-shell-cks/scenario/playground-cilium-network-policy) |
| CS2 | [CIS Benchmarks Fix Controlplane](https://killercoda.com/killer-shell-cks/scenario/cis-benchmarks-kube-bench-fix-controlplane) |
| CS3 | [Ingress Create](https://killercoda.com/killer-shell-cks/scenario/ingress-create), [Ingress Secure](https://killercoda.com/killer-shell-cks/scenario/ingress-secure) |
| CS4 | [NetworkPolicy Metadata Protection](https://killercoda.com/killer-shell-cks/scenario/networkpolicy-metadata-protection) |
| CS5 | [Verify Platform Binaries](https://killercoda.com/killer-shell-cks/scenario/verify-platform-binaries-kubelet) |
| **Cluster Hardening — 15%** | |
| CH1 | [RBAC ServiceAccount Permissions](https://killercoda.com/killer-shell-cks/scenario/rbac-serviceaccount-permissions), [RBAC User Permissions](https://killercoda.com/killer-shell-cks/scenario/rbac-user-permissions) |
| CH2 | [ServiceAccount Token Mounting](https://killercoda.com/killer-shell-cks/scenario/serviceaccount-token-mounting) |
| CH3 | [CSRs: Sign Manually](https://killercoda.com/killer-shell-cks/scenario/certificate-signing-requests-sign-manually), [CSRs: Sign via API](https://killercoda.com/killer-shell-cks/scenario/certificate-signing-requests-sign-k8s) |
| CH4 | [Apiserver Crash](https://killercoda.com/killer-shell-cks/scenario/apiserver-crash), [Apiserver Misconfigured](https://killercoda.com/killer-shell-cks/scenario/apiserver-misconfigured), [Apiserver NodeRestriction](https://killercoda.com/killer-shell-cks/scenario/apiserver-node-restriction) |
| **System Hardening — 10%** | |
| SH1 | [Close Open Ports](https://killercoda.com/killer-shell-cks/scenario/system-hardening-close-open-ports), [Manage Packages](https://killercoda.com/killer-shell-cks/scenario/system-hardening-manage-packages) |
| SH3 | [AppArmor](https://killercoda.com/killer-shell-cks/scenario/apparmor) |
| **Miscellaneous — general setup** | |
| General practice | [Playground](https://killercoda.com/killer-shell-cks/scenario/playground), [Exam Desktop](https://killercoda.com/killer-shell-cks/scenario/playground-exam-desktop), [Vim Setup](https://killercoda.com/killer-shell-cks/scenario/vim-setup) |

The Cilium **policy** playground is useful for network-policy practice; it is not evidence that you have completed an encryption exercise. Use the MMV5 verification drill separately.

### Additional exercises to complete the coverage

- [ ] **PSA:** enforce Restricted in a test namespace; repair a rejected workload; verify compliant Pods run.
- [ ] **seccomp:** use RuntimeDefault, then a provided Localhost profile; verify one blocked syscall and a functioning application.
- [ ] **Cilium encryption:** enable the requested encryption mode in a suitable lab; prove cross-node transport is encrypted.
- [ ] **Istio mTLS:** enroll the test workloads, require STRICT mTLS, and verify authorized meshed traffic plus plaintext rejection.
- [ ] **SBOM:** generate an SPDX document, locate a package/version, and associate it with the correct artifact.
- [ ] **Signing:** sign a disposable practice artifact if tooling/registry are available; verify the expected signer and demonstrate rejection of an untrusted artifact in the supplied admission system.
- [ ] **Upgrade:** perform a supported kubeadm upgrade, return nodes to Ready/schedulable state, and check applications.
- [ ] **Static analysis:** use at least one actual analyzer in addition to the manual-analysis scenarios.
- [ ] **Incident correlation:** combine an audit event, runtime alert, and workload identity into a short evidence-backed timeline.

## 13. Recommended study order and readiness checks

**KodeKloud mixed practice:** [Mock Exams Introduction][kk08-01] · [Mock Exam 1][kk08-02] · [Mock Exam 2][kk08-03] · [Mock Exam 3][kk08-04] · [CKS Challenges][kk09]. Use the topic-level KodeKloud lessons/labs above for targeted review before repeating a mock.

### 13.1 Four-week example plan

The original dependency-first example plan is retained below. For weight-prioritized revision instead, follow sections 5–10 in order: the three 20% domains, the two 15% domains, then System Hardening at 10%.

Adjust the pace to your CKA/Linux familiarity. Advance when you can complete the exercises, not just when you have watched the material.

| Phase | Topics and work | Exit criterion |
| --- | --- | --- |
| Week 1: cluster foundations | F0; CS1–CS5; CH1–CH4; shell/Vim practice. | Write and test policies/RBAC, configure TLS Ingress, fix CIS findings, recover a broken API server. |
| Week 2: hosts and workloads | CH5; SH1–SH3; MMV1–MMV4 and MMV6. | Upgrade a lab node, apply profiles, enforce PSA, use sandbox runtimes, encrypt existing Secrets. |
| Week 3: supply chain and encryption | MMV5; SCS1–SCS5; begin MLRS1/MLRS4. | Verify Pod-to-Pod encryption, generate SBOMs, analyse/scan images, verify signatures/admission, enable auditing. |
| Week 4: runtime and mixed practice | MLRS1–MLRS4; KodeKloud mocks/challenges; eligible killer.sh simulator attempts. | Complete timed mixed work, diagnose failures, and independently verify every change. |

For each topic use three passes:

1. **Learning pass:** documentation and hints allowed; understand the mechanism.
2. **Independent pass:** reset the lab; solve using only official documentation/help.
3. **Timed pass:** mix it with unrelated tasks and record the time and mistakes.

### 13.2 High-value repeat drills

These are practice targets, not claims about which questions will appear:

| Drill | What to prove |
| --- | --- |
| Default deny + narrow allow | Allowed client works; another client fails; required DNS still works. |
| RBAC least privilege | Correct subject can do exactly the required operation in the required scope. |
| Disable token automount | New Pod has no unwanted API-token mount; application still runs. |
| Harden a Deployment | New Pods are non-root, restricted, and healthy. |
| PSA rejection and repair | Invalid Pod denied; corrected Pod admitted and functioning. |
| AppArmor/seccomp | Correct profile is active on the actual node; prohibited action fails. |
| Encrypt existing Secrets | Raw etcd data is encrypted; authorized API access still works. |
| ImagePolicyWebhook | Approved image admitted; disallowed image rejected; failure behavior matches requirements. |
| Trivy/SBOM | Correct image and output file; findings/components interpreted correctly. |
| Falco rule change | Engine accepts rule; matching event produces required output. |
| Audit logging | API server stays healthy; request appears at the intended audit level. |
| Read-only root filesystem | Forbidden write fails; required writable paths and application work. |
| Broken API-server recovery | API readiness and nodes recover without relying on a functioning API to debug. |
| Pod-to-Pod encryption | Transport is demonstrably encrypted; required application communication survives. |

### 13.3 Final readiness checklist

- [ ] I can explain every official competency in section 3 and implement a representative task for it.
- [ ] I have completed the relevant KodeKloud labs/mocks and the Killercoda scenarios linked in sections 5–11 and cataloged in section 12.
- [ ] I have separately practised the coverage gaps: PSA, seccomp, SBOM, signing, encryption, and upgrades.
- [ ] I can find official examples quickly using permitted documentation.
- [ ] I can generate YAML, read errors/events, and fix indentation without losing several minutes.
- [ ] I can edit a static-Pod manifest and recover if the API server stops.
- [ ] I know which changes need a Pod rollout, service restart, or object recreation.
- [ ] I verify both allowed and denied behavior for security policies.
- [ ] I can correlate image/container/Pod/node identities during an investigation.
- [ ] I have completed a full two-hour mixed practice session and reviewed every missed task.
- [ ] I have rechecked the current exam version, allowed resources, host workflow, and system requirements.

### 13.4 Last-minute reminders

**Correct host → correct context → correct namespace → persistent change → positive and negative checks → application/cluster health → required output file → return to base.**

Common expensive mistakes:

- Writing a correct manifest without applying it when live state is required.
- Editing the Pod instead of the controller template.
- Forgetting init containers or updating only one replica/node.
- Confusing an admission-policy object with actual enforcement.
- Putting API-server configuration files on the host without mounting them into the container.
- Leaving static-manifest backups inside the watched directory.
- Putting `identity` first in encryption providers or failing to rewrite old Secrets.
- Forgetting egress DNS or misunderstanding NetworkPolicy AND/OR selection.
- Treating a checksum, digest, signature, SBOM, and vulnerability report as interchangeable.
- Assuming a successful application request proves TLS/mTLS or policy enforcement.

## 14. Sources and maintenance

### Primary preparation sources

1. [KodeKloud public CKS documentation][kk] — chapter/lesson mapping and supporting material.
2. [Killer Shell CKS on Killercoda][kc] — public scenario catalog and exercise descriptions.
3. [Killer Shell course scenario index](https://github.com/killer-sh/cks-course-environment/blob/master/Scenarios.md) — additional mapping of topics to scenario links; the live catalog can contain additional entries.
4. [Killercoda CKS preparation page](https://killercoda.com/cks) — practice guidance and simulator context.
5. [Linux Foundation training-portal course page][lf-course] — the supplied exam reference. Its app-rendered body was not readable through the text fetch, so exam details were checked against the public LF page, FAQ, and instructions below.

### Authoritative exam references

6. [CKS curriculum repository][curriculum] and [current v1.37 PDF][curriculum-pdf] — competencies and weights.
7. [Linux Foundation public CKS certification page][lf-public] — domains and exam overview.
8. [CKA/CKAD/CKS FAQ][lf-faq] — pass score, versions, prerequisite, validity, and simulator entitlements.
9. [Important instructions: CKS][lf-instructions] — task-host workflow, tools, remote desktop, and technical requirements.
10. [Resources allowed during CKS][lf-resources] — authoritative documentation-access rules.
11. [Candidate handbook][lf-handbook] and [LF certification FAQ](https://training.linuxfoundation.org/about/faqs/certification-faq/) — preparation/check-in and bookmark policy.
12. [CNCF CKS certification page][cncf-cks] — prerequisite clarification; its weight discrepancy is noted in section 3.

Topic-specific implementation references appear alongside the relevant sections. Tool commands and examples are study aids; use the task's supplied versions, files, and requirements when practising or taking the exam.

**Before the exam:** reopen the curriculum repository, FAQ, instructions, and allowed-resource list. Refresh this guide if the software version or competencies change.

[kk]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/tree/main/docs
[kk02]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/tree/main/docs/02-Understanding-the-Kubernetes-Attack-Surface
[kk03]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/tree/main/docs/03-Cluster-Setup-and-Hardening
[kk04]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/tree/main/docs/04-System-Hardening
[kk05]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/tree/main/docs/05-Minimize-Microservice-Vulnerabilities
[kk06]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/tree/main/docs/06-Supply-Chain-Security
[kk07]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/tree/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security
[kk08]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/tree/main/docs/08-Mock-Exams
[kk09]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/tree/main/docs/09-cks-challenges
[kk01-01]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/01-Introduction/01-Course-Introduction.md
[kk01-02]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/01-Introduction/02-Exam-Information.md
[kk02-01]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/02-Understanding-the-Kubernetes-Attack-Surface/01-The-Attack.md
[kk02-02]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/02-Understanding-the-Kubernetes-Attack-Surface/02-The-4C%27s-of-Cloud-Native-security.md
[kk03-02]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/02-What-are-CIS-Benchmarks.md
[kk03-03]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/03-Lab-Run-CIS-Benchmark-Assessment-tool-on-Ubuntu.md
[kk03-04]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/04-CIS-benchmark-for-Kubernetes.md
[kk03-05]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/05-Kube-bench.md
[kk03-06]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/06-Lab-Kube-bench.md
[kk03-07]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/07-Kubernetes-Security-Primitives.md
[kk03-08]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/08-Authentication.md
[kk03-09]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/09-Service-Accounts.md
[kk03-10]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/10-Lab-Service-Accounts.md
[kk03-11]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/11-TLS-Introduction.md
[kk03-12]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/12-TLS-Basics.md
[kk03-13]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/13-TLS-in-Kubernetes.md
[kk03-14]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/14-TLS-in-Kubernetes-Certificate-Creation.md
[kk03-15]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/15-View-Certificate-Details.md
[kk03-16]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/16-Labs-View-Certificates.md
[kk03-17]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/17-Certificates-API.md
[kk03-18]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/18-Labs-Certificates-API.md
[kk03-19]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/19-KubeConfig.md
[kk03-20]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/20-Labs-KubeConfig.md
[kk03-22]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/22-Authorization.md
[kk03-23]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/23-RBAC.md
[kk03-24]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/24-Labs-RBAC.md
[kk03-25]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/25-Cluster-Roles-and-Role-Bindings.md
[kk03-26]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/26-Labs-Cluster-Roles-and-Role-Bindings.md
[kk03-27]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/27-Kubelet-Security.md
[kk03-28]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/28-Labs-Kubelet-Security.md
[kk03-29]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/29-Kubectl-Proxy-Port-Forward.md
[kk03-30]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/30-Labs-Kubectl-Proxy-Port-Forward.md
[kk03-31]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/31-Kubernetes-Dashboard.md
[kk03-32]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/32-Securing-Kubernetes-Dashboard.md
[kk03-33]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/33-Labs-Securing-Kubernetes-Dashboard.md
[kk03-34]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/34-Verify-platform-binaries-before-deploying.md
[kk03-35]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/35-Labs-Verify-platform-binaries-before-deploying.md
[kk03-36]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/36-Kubernetes-Software-Versions.md
[kk03-37]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/37-Cluster-Upgrade-Process.md
[kk03-38]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/38-Demo-Cluster-Upgrade-Process.md
[kk03-39]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/39-Labs-Cluster-Upgrade-Process.md
[kk03-40]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/40-Network-Policy.md
[kk03-41]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/41-Developing-Network-Policies.md
[kk03-42]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/42-Labs-Network-Policies.md
[kk03-43]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/43-Ingress.md
[kk03-44]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/44-Labs-Ingress-1.md
[kk03-45]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/45-Ingress-Annotations-and-rewrite-target.md
[kk03-46]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/03-Cluster-Setup-and-Hardening/46-Labs-Ingress-2.md
[kk04-02]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/02-Least-Privilege-Principle.md
[kk04-03]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/03-Minimize-host-OS-footprint-Intro.md
[kk04-04]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/04-Limit-Node-Access.md
[kk04-05]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/05-lab-Limit-Node-Access.md
[kk04-06]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/06-SSH-Hardening.md
[kk04-07]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/07-Privilege-Escalation-in-Linux.md
[kk04-08]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/08-Lab-SSH-Hardening-and-sudo.md
[kk04-09]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/09-Remove-Obsolete-Packages-and-Services.md
[kk04-10]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/10-Restrict-Kernel-Modules.md
[kk04-11]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/11-Identify-and-Disable-Open-Ports.md
[kk04-12]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/12-Lab-Identify-open-ports%2C-remove-packages-services.md
[kk04-13]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/13-Minimize-IAM-roles.md
[kk04-14]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/14-Minimize-external-access-to-the-network.md
[kk04-15]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/15-UFW-Firewall-Basics.md
[kk04-16]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/16-Lab-UFW-Firewall-Basics.md
[kk04-17]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/17-Linux-Syscalls.md
[kk04-18]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/18-AquaSecTracee.md
[kk04-19]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/19-Restrict-syscalls-using-seccomp.md
[kk04-20]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/20-Implement-Seccomp-in-Kubernetes.md
[kk04-21]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/21-Lab-Seccomp.md
[kk04-22]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/22-AppArmor.md
[kk04-23]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/23-Creating-AppArmor-Profiles.md
[kk04-24]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/24-AppArmor-in-Kubernetes.md
[kk04-25]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/25-Linux-Capabilities.md
[kk04-26]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/04-System-Hardening/26-Lab-AppArmor.md
[kk05-02]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/02-Security-Contexts.md
[kk05-03]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/03-Labs-Security-Contexts.md
[kk05-04]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/04-Admission-Controllers.md
[kk05-05]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/05-Labs-Admission-Controllers.md
[kk05-06]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/06-Validating-and-Mutating-Admission-Controllers.md
[kk05-07]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/07-Labs-Validating-and-Mutating-Admission-Controllers.md
[kk05-09]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/09-PSA-and-PSS.md
[kk05-11]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/11-Labs-PSA.md
[kk05-12]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/12-Open-Policy-Agent-%28OPA%29.md
[kk05-13]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/13-Labs-OPA.md
[kk05-14]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/14-OPA-Gatekeeper-in-Kubernetes.md
[kk05-15]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/15-Labs-OPA-in-Kubernetes.md
[kk05-16]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/16-Manage-Kubernetes-secrets.md
[kk05-17]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/17-Lab-Manage-Kubernetes-secrets.md
[kk05-18]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/18-Container-Sandboxing.md
[kk05-19]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/19-gVisor.md
[kk05-20]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/20-kata-Containers.md
[kk05-21]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/21-Runtime-Classes.md
[kk05-22]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/22-Using-Runtimes-in-Kubernetes.md
[kk05-23]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/23-Lab-Using-Runtimes-in-Kubernetes.md
[kk05-24]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/24-One-way-SSL-vs-Mutual-SSL.md
[kk05-25]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/05-Minimize-Microservice-Vulnerabilities/25-Implement-pod-to-pod-encryption-by-use-of-mTLS.md
[kk06-02]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/06-Supply-Chain-Security/02-Minimize-base-image-footprint.md
[kk06-03]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/06-Supply-Chain-Security/03-Image-Security.md
[kk06-04]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/06-Supply-Chain-Security/04-Labs-Image-Security.md
[kk06-05]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/06-Supply-Chain-Security/05-Whitelist-Allowed-Registries-Image-Policy-Webhook.md
[kk06-06]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/06-Supply-Chain-Security/06-Labs-Whitelist-Allowed-Registries-ImagePolicyWebhook.md
[kk06-07]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/06-Supply-Chain-Security/07-Use-static-analysis-of-user-workloads.md
[kk06-08]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/06-Supply-Chain-Security/08-Labs-kubesec.md
[kk06-09]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/06-Supply-Chain-Security/09-Scan-images-for-known-vulnerabilities-%28Trivy%29.md
[kk06-10]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/06-Supply-Chain-Security/10-Labs-Trivy.md
[kk07-02]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security/02-Perform-behavioral-analytics-of-syscall-process.md
[kk07-03]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security/03-Falco-Overview-and-Installation.md
[kk07-04]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security/04-Use-Falco-to-Detect-Threats.md
[kk07-05]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security/05-Falco-Configuration-Files.md
[kk07-06]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security/06-Labs-Use-Falco-to-detect-threats.md
[kk07-07]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security/07-Mutable-vs-Immutable-Infrastructure-Mutable-vs-Immutable-Infrastructure.md
[kk07-08]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security/08-Ensure-Immutability-of-Containers-at-Runtime.md
[kk07-09]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security/09-Lab-Ensure-Immutability-of-Containers-at-Runtime.md
[kk07-10]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security/10-Use-Audit-Logs-to-monitor-access.md
[kk07-11]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/07-Monitoring%2C-Logging-and-Runtime-Security/11-Labs-Use-Audit-Logs-to-monitor-access.md
[kk08-01]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/08-Mock-Exams/01-Mock-Exams-Introduction.md
[kk08-02]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/08-Mock-Exams/02-Mock-Exam-1.md
[kk08-03]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/08-Mock-Exams/03-Mock-Exam-2.md
[kk08-04]: https://github.com/kodekloudhub/certified-kubernetes-security-specialist-cks-course/blob/main/docs/08-Mock-Exams/04-Mock-Exam-3.md
[kc]: https://killercoda.com/killer-shell-cks
[lf-course]: https://trainingportal.linuxfoundation.org/courses/certified-kubernetes-security-specialist-cks
[lf-public]: https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist-cks-2/
[lf-faq]: https://docs.linuxfoundation.org/tc-docs/certification/faq-cka-ckad-cks
[lf-instructions]: https://docs.linuxfoundation.org/tc-docs/certification/important-instructions-cks
[lf-resources]: https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-security-specialist-cks
[lf-handbook]: https://docs.linuxfoundation.org/tc-docs/certification/lf-handbook2
[curriculum]: https://github.com/cncf/curriculum
[curriculum-pdf]: https://github.com/cncf/curriculum/blob/main/CKS_Curriculum%20v1.37.pdf
[cncf-cks]: https://www.cncf.io/training/certification/cks/
