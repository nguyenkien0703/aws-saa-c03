# 🎓 Ôn Thi CKA & CKS — Cẩm Nang Đầy Đủ (2025/2026)

> Tài liệu ôn thi **CKA (Certified Kubernetes Administrator)** và **CKS (Certified Kubernetes Security Specialist)** — hai chứng chỉ Kubernetes danh giá nhất, thi **hands-on trên terminal thật** (không trắc nghiệm).
>
> ⚠️ **Đính chính quan trọng:** CKA và CKS là chứng chỉ của **CNCF / Linux Foundation**, **KHÔNG phải của AWS**. (AWS có chứng chỉ riêng như "AWS Certified Kubernetes..." — nhưng CKA/CKS thì thuộc CNCF, thi vendor-neutral, dùng được cho mọi cloud và on-prem.)
>
> 📎 File song hành: **[X-kubernetes-thuc-chien-troubleshooting.md](./X-kubernetes-thuc-chien-troubleshooting.md)** — kinh nghiệm troubleshooting thực chiến (bổ trợ trực tiếp domain Troubleshooting 30% của CKA).

---

## 📑 Mục lục

- [Phần 0 — CKA vs CKS: chọn cái nào, thi theo thứ tự nào](#phần-0--cka-vs-cks-chọn-cái-nào-thi-theo-thứ-tự-nào)
- [Phần 1 — Logistics kỳ thi (đăng ký, môi trường, proctor)](#phần-1--logistics-kỳ-thi-đăng-ký-môi-trường-proctor)
- [Phần 2 — Setup 60 giây đầu tiên (alias, vimrc)](#phần-2--setup-60-giây-đầu-tiên-alias-vimrc)
- [Phần 3 — CKA: Domains & Command Cookbook](#phần-3--cka-domains--command-cookbook)
- [Phần 4 — CKS: Domains & Tool Cookbook](#phần-4--cks-domains--tool-cookbook)
- [Phần 5 — killer.sh & chiến thuật quản lý thời gian](#phần-5--killersh--chiến-thuật-quản-lý-thời-gian)
- [Phần 6 — Lộ trình học 30/60/90 ngày](#phần-6--lộ-trình-học-306090-ngày)
- [Phần 7 — Checklist "phải thuộc lòng" trước khi thi](#phần-7--checklist-phải-thuộc-lòng-trước-khi-thi)

---

## Phần 0 — CKA vs CKS: chọn cái nào, thi theo thứ tự nào

| | **CKA** | **CKS** |
|--|---------|---------|
| **Tên đầy đủ** | Certified Kubernetes Administrator | Certified Kubernetes Security Specialist |
| **Điều kiện tiên quyết** | Không | **Phải có CKA còn hiệu lực** trước |
| **Trọng tâm** | Vận hành, quản trị cluster | Bảo mật, hardening cluster |
| **Thời lượng** | 2 giờ | 2 giờ |
| **Số câu** | 15–20 task hands-on | ~15–17 task hands-on |
| **Điểm đậu** | **66%** | **67%** |
| **Giá** | **$445** (kèm 1 lần thi lại free) | **$445** (kèm 1 lần thi lại free) |
| **Hiệu lực** | 2 năm | 2 năm |
| **Phiên bản k8s** | v1.35 (chu kỳ 2026) | v1.35 (chu kỳ 2026) |
| **Simulator** | 2 lượt killer.sh free | 2 lượt killer.sh free |

> 🔑 **Thứ tự bắt buộc:** CKA trước → CKS sau. CKS **vô hiệu** nếu CKA hết hạn. Lời khuyên: học CKA cho vững nền (nhất là Troubleshooting 30%), thi CKA, rồi mới sang CKS (CKS giả định bạn đã thành thạo kubectl và không tốn thời gian cho thao tác cơ bản).
>
> ⚠️ **Luôn kiểm tra lại phiên bản k8s và trọng số domain trên trang Linux Foundation ngay trước khi đăng ký** — mỗi chu kỳ cập nhật (gần đây chạy 1.30 → 1.35).

---

## Phần 1 — Logistics kỳ thi (đăng ký, môi trường, proctor)

### Môi trường thi
- **Online, giám sát từ xa (remote-proctored)**, hoàn toàn **hands-on** trên terminal Linux thật, cluster đa-node thật. Không có trắc nghiệm.
- CKA/CKAD/CKS đã đổi sang **remote desktop Linux (XFCE)** đầy đủ (từ 2023) — bên trong có **terminal** và **Firefox**.
- Proctor qua PSI (nền tảng "Bridge" + Secure Browser), giám sát webcam/mic/màn hình. Cần **CMND/hộ chiếu có ảnh**, bàn sạch, không màn hình phụ.

### Tài liệu được phép mở (1 tab Firefox phụ)
- **kubernetes.io/docs** (kèm kubernetes.io/blog)
- **helm.sh/docs**
- Với CKS còn được mở docs của các tool: **Falco, Trivy, AppArmor, gVisor**.
- ❌ Không được mở site khác. ❌ Không được paste dotfile/alias từ nguồn ngoài — **phải tự gõ alias** đầu giờ.

### Copy/paste trong exam
- Trong terminal: `Ctrl+Shift+C` / `Ctrl+Shift+V` (hoặc chuột phải).
- Trong Firefox: `Ctrl+C` / `Ctrl+V` bình thường.

### 🔴 Quy tắc sống còn: đổi context mỗi task
Mỗi task chạy trên **cluster/context riêng**. **BẮT BUỘC** copy-paste và chạy lệnh `kubectl config use-context <ctx>` được cho ở **đầu MỖI task**. Làm nhầm cluster = **0 điểm**.

---

## Phần 2 — Setup 60 giây đầu tiên (alias, vimrc)

Gõ ngay đầu giờ (tiết kiệm 20–30 phút tích lũy — **phải thuộc lòng vì không được paste**):

```bash
alias k=kubectl
export do="--dry-run=client -o yaml"      # sinh manifest
export now="--force --grace-period=0"     # xóa tức thì
source <(kubectl completion bash); complete -o default -F __start_kubectl k
```
`~/.vimrc` (cực quan trọng cho thụt lề YAML):
```vim
set number tabstop=2 shiftwidth=2 expandtab autoindent
```
> ⚠️ **Lưu ý alias:** alias chỉ chạy trong shell tương tác. Khi task yêu cầu **ghi manifest ra file**, dùng lệnh đầy đủ (không dùng alias `k` bên trong nội dung file).

---

## Phần 3 — CKA: Domains & Command Cookbook

### Trọng số domain (bản cập nhật Feb 2025 — hiện hành)

| Domain | Trọng số |
|--------|----------|
| **Troubleshooting** | **30%** ⭐ lớn nhất |
| **Cluster Architecture, Installation & Configuration** | **25%** |
| **Services & Networking** | **20%** |
| **Workloads & Scheduling** | **15%** |
| **Storage** | **10%** |

**Thay đổi từ bản Feb 2025 (đừng dùng guide cũ trước 2025):**
- **Thêm:** Gateway API (GatewayClass/Gateway/HTTPRoute), **Helm** & **Kustomize**, **CRD & Operators**, hiểu interface mở rộng **CNI/CSI/CRI**, nhấn mạnh **dynamic volume provisioning** (StorageClass), Pod Security Standards & RBAC nâng cao.
- **Giảm nhẹ (vẫn thi):** etcd backup/restore và cluster upgrade — vẫn học vì giá trị cao.
- **Bỏ:** PodSecurityPolicy (PSP), alpha API, Docker shim.

> 💡 Troubleshooting chiếm 30% → **[file X-kubernetes-thuc-chien-troubleshooting.md](./X-kubernetes-thuc-chien-troubleshooting.md) chính là phần ôn domain này.**

### 3.1. Workloads & Scheduling

```bash
# Pods
k run nginx --image=nginx                              # pod nhanh
k run nginx --image=nginx $do > pod.yaml              # sinh manifest rồi sửa
k run busybox --image=busybox --command -- sleep 3600
k run nginx --image=nginx --port=80 --labels="tier=frontend,env=prod" $do > pod.yaml
k run tmp --image=busybox --rm -it --restart=Never -- sh   # pod debug dùng 1 lần

# Deployments (create deploy CÓ hỗ trợ --replicas)
k create deploy nginx --image=nginx --replicas=3
k scale deploy nginx --replicas=5

# Rollouts
k set image deploy/nginx nginx=nginx:1.25
k rollout status deploy nginx
k rollout undo deploy nginx --to-revision=2

# Labels
k label node worker-1 disk=ssd
k label node worker-1 disk-        # xóa label (dấu gạch cuối)
```

**Taints & Tolerations:**
```bash
k taint nodes worker-1 key=value:NoSchedule       # thêm
k taint nodes worker-1 key=value:NoSchedule-      # xóa (dấu gạch cuối)
```
```yaml
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

**Node affinity / nodeSelector** (sửa tay vào pod spec):
```yaml
nodeSelector:
  disk: ssd
# hoặc:
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: disk, operator: In, values: [ssd]}
```

**DaemonSet** (không có lệnh imperative): mẹo — `k create deploy ... $do > ds.yaml` → đổi `kind: Deployment` thành `kind: DaemonSet`, bỏ `replicas`/`strategy`/`status`.

**Static Pod:** nằm ở `/etc/kubernetes/manifests/` trên node; kubelet tự tạo/xóa pod khi thêm/bớt file. Tên static pod có hậu tố tên node.
```bash
grep staticPodPath /var/lib/kubelet/config.yaml    # tìm đường dẫn static pod
```

### 3.2. Services & Networking
```bash
k expose deploy nginx --port=80 --target-port=80              # ClusterIP, tái dùng label làm selector
k expose deploy nginx --port=80 --type=NodePort
k expose pod redis --port=6379 --name redis-service
k create svc clusterip my-svc --tcp=80:8080 $do > svc.yaml
# Ingress / Gateway API / NetworkPolicy: viết tay từ docs (bookmark sẵn ví dụ)
```

### 3.3. RBAC
```bash
k create role pod-reader --verb=get,list,watch --resource=pods -n dev
k create rolebinding read-pods --role=pod-reader --user=jane -n dev
k create rolebinding read-pods --role=pod-reader --serviceaccount=dev:app-sa -n dev
k create clusterrole node-reader --verb=get,list,watch --resource=nodes
k create clusterrolebinding read-nodes --clusterrole=node-reader --user=jane
# Verify (rất hay được yêu cầu):
k auth can-i get pods --as=jane -n dev
k auth can-i list nodes --as=system:serviceaccount:dev:app-sa
```

### 3.4. Cluster Architecture — Node Maintenance
```bash
k drain <node> --ignore-daemonsets --delete-emptydir-data --force   # evict pod an toàn
k cordon <node>       # đánh dấu không schedule
k uncordon <node>     # cho schedule lại
```

### 3.5. ⭐ etcd Backup & Restore (thuộc lòng cờ cert)
Tìm endpoint/cert trước: `cat /etc/kubernetes/manifests/etcd.yaml`
```bash
# BACKUP
ETCDCTL_API=3 etcdctl snapshot save /opt/backup/etcd.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify
ETCDCTL_API=3 etcdctl snapshot status /opt/backup/etcd.db --write-out=table

# RESTORE sang data-dir MỚI
ETCDCTL_API=3 etcdctl snapshot restore /opt/backup/etcd.db \
  --data-dir=/var/lib/etcd-from-backup
```
Sau đó sửa `/etc/kubernetes/manifests/etcd.yaml` → đổi `hostPath` volume `etcd-data` sang `/var/lib/etcd-from-backup`. Static pod etcd tự restart.

### 3.6. ⭐ Cluster Upgrade với kubeadm
```bash
# --- Trên node CONTROL PLANE ---
# 1. Upgrade kubeadm trước
sudo apt-mark unhold kubeadm
sudo apt-get update && sudo apt-get install -y kubeadm='1.36.x-*'
sudo apt-mark hold kubeadm

# 2. Plan & apply
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.36.x

# 3. Drain node NÀY (control plane drain SAU apply)
kubectl drain <node> --ignore-daemonsets

# 4. Upgrade kubelet + kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt-get update && sudo apt-get install -y kubelet='1.36.x-*' kubectl='1.36.x-*'
sudo apt-mark hold kubelet kubectl

# 5. Restart kubelet
sudo systemctl daemon-reload && sudo systemctl restart kubelet

# 6. Uncordon
kubectl uncordon <node>
```
**Node worker / control-plane phụ:** giống hệt, nhưng thay `kubeadm upgrade apply` bằng **`sudo kubeadm upgrade node`** (bỏ `upgrade plan`).

> 🔑 **Quy tắc thứ tự sống còn:**
> - Upgrade **kubeadm trước**, rồi apply, rồi kubelet/kubectl. Không bao giờ upgrade kubelet trước control plane.
> - **KHÔNG drain trước `kubeadm upgrade apply`** trên control plane — drain đến *sau* apply, ngay trước upgrade kubelet. Trên **worker thì drain trước**.
> - Repo đã chuyển sang `pkgs.k8s.io`; nếu không tìm thấy version, sửa `/etc/apt/sources.list.d/kubernetes.list` trỏ đúng minor mới trước khi `apt-get update`.

### 3.7. Troubleshooting (30% — xem file X)
Flow chung: `kubectl get nodes/pods -A` → `describe` → `logs` → node-level `systemctl`/`journalctl`.
```bash
# kubelet down / node NotReady
ssh <node>; sudo -i
systemctl status kubelet; journalctl -u kubelet -f
systemctl restart kubelet
# Service không endpoint
kubectl get endpoints <svc>          # rỗng = selector lệch
kubectl get pods --show-labels
# CoreDNS
kubectl run dnstest --image=busybox:1.28 --rm -it --restart=Never -- nslookup kubernetes.default
# Control plane component down = static pod
ls /etc/kubernetes/manifests/; crictl ps -a | grep <component>; crictl logs <id>
```

---

## Phần 4 — CKS: Domains & Tool Cookbook

### Trọng số domain (bản cập nhật 15/10/2024 — hiện hành)

| Domain | Trọng số |
|--------|----------|
| **Minimize Microservice Vulnerabilities** | **20%** |
| **Supply Chain Security** | **20%** |
| **Monitoring, Logging & Runtime Security** | **20%** |
| **Cluster Setup** | **15%** |
| **Cluster Hardening** | **15%** |
| **System Hardening** | **10%** |

Công cụ/chuẩn chính thức tham chiếu: **CIS Benchmark (kube-bench), NetworkPolicy, RBAC, TLS, AppArmor, seccomp, Cilium, Istio (mTLS), Kubesec, KubeLinter, Trivy, Falco, SBOM, audit logs.**

### 4.1. Trivy — quét lỗ hổng image (Supply Chain)
```bash
trivy image --severity HIGH,CRITICAL nginx:1.19
trivy image --severity CRITICAL --ignore-unfixed myimage:tag
trivy config <dir>        # quét manifest/config
trivy fs .                # quét filesystem

# Task hay gặp: tìm pod chạy image có CRITICAL rồi XÓA pod đó
kubectl get pods -n <ns> -o custom-columns='NAME:.metadata.name,IMAGE:.spec.containers[*].image'
trivy image --severity CRITICAL <image>    # nếu có CRITICAL -> kubectl delete pod ...
```

### 4.2. Falco — phát hiện runtime (Monitoring/Runtime)
- Thư mục config: **`/etc/falco/`**. Config chính `falco.yaml`; rule mặc định `falco_rules.yaml` (**đừng sửa** — bị ghi đè khi upgrade); **rule tùy chỉnh → `falco_rules.local.yaml`** hoặc `/etc/falco/rules.d/`.
- Thứ tự nạp: `falco_rules.yaml` → `falco_rules.local.yaml` → `rules.d/`.
```yaml
# /etc/falco/falco_rules.local.yaml — phát hiện shell spawn trong container
- rule: Detect shell in container
  desc: Alert if a shell is spawned inside a container
  condition: container.id != host and proc.name in (bash, sh, zsh)
  output: "Shell in container (user=%user.name container=%container.name image=%container.image.repository proc=%proc.cmdline)"
  priority: WARNING
  tags: [container, shell]
```
```bash
systemctl restart falco
journalctl -u falco -f            # đọc alert
# Task hay gặp: sửa output của rule có sẵn (thêm %evt.time, %container.name), đổi priority, rồi restart
```
> Priority: EMERGENCY > ALERT > CRITICAL > ERROR > WARNING > NOTICE > INFORMATIONAL > DEBUG. Thành phần: **rules**, **macros** (điều kiện tái dùng), **lists**.

### 4.3. AppArmor (System Hardening)
```bash
# Trên node: nạp profile
apparmor_parser -q /etc/apparmor.d/k8s-deny-write
aa-status                        # kiểm tra profile đã nạp
```
```yaml
# K8s 1.30+ dùng field NATIVE (ưu tiên):
spec:
  securityContext:
    appArmorProfile:
      type: Localhost           # hoặc RuntimeDefault / Unconfined
      localhostProfile: k8s-deny-write
  containers:
  - { name: app, image: busybox }
# Pre-1.30 (annotation cũ, vẫn có thể gặp trên cluster cũ):
# container.apparmor.security.beta.kubernetes.io/<container>: localhost/k8s-deny-write
```

### 4.4. seccomp (System Hardening)
- Profile nằm ở **`/var/lib/kubelet/seccomp/`** (đường dẫn localhost tương đối so với thư mục này).
```yaml
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault          # profile mặc định của runtime (chặn syscall nguy hiểm)
  containers:
  - name: app
    image: nginx
    securityContext:
      seccompProfile:
        type: Localhost
        localhostProfile: profiles/audit.json   # => /var/lib/kubelet/seccomp/profiles/audit.json
```
> Dạng annotation (`seccomp.security.alpha...`) đã **bị bỏ** — dùng `securityContext.seccompProfile`.

### 4.5. Pod Security Admission (Cluster Hardening) — thay PodSecurityPolicy
Level: **privileged / baseline / restricted**. Mode: **enforce / audit / warn**.
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-ns
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```
```bash
kubectl label ns secure-ns pod-security.kubernetes.io/enforce=baseline --overwrite
```

### 4.6. NetworkPolicy (Minimize Microservice Vulns)
```yaml
# Default deny TOÀN BỘ ingress + egress trong namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: default-deny, namespace: prod }
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
---
# Cho phép egress DNS (BẮT BUỘC sau khi deny egress, nếu không DNS chết)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: allow-dns, namespace: prod }
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to: []
    ports: [{protocol: UDP, port: 53}, {protocol: TCP, port: 53}]
---
# Cho phép ingress vào backend CHỈ từ pod frontend, cổng 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: backend-from-frontend, namespace: prod }
spec:
  podSelector: { matchLabels: { app: backend } }
  policyTypes: [Ingress]
  ingress:
  - from:
    - podSelector: { matchLabels: { app: frontend } }
    ports: [{port: 8080}]
```
> ⚠️ **Bẫy:** `namespaceSelector` + `podSelector` trong **cùng một item `from`** (không có `-`) = AND; tách thành item riêng = OR.

### 4.7. kube-bench — CIS Benchmark (Cluster Setup)
```bash
kube-bench run --targets master         # hoặc node / etcd / controlplane
kube-bench run --check 1.2.5
```
Sửa finding bằng cách sửa static-pod manifest ở **`/etc/kubernetes/manifests/`** (apiserver tự restart khi file đổi).

### 4.8. Hardening API Server (`/etc/kubernetes/manifests/kube-apiserver.yaml`)
```yaml
- --anonymous-auth=false
- --authorization-mode=Node,RBAC          # không bao giờ AlwaysAllow
- --profiling=false
- --enable-admission-plugins=NodeRestriction,PodSecurity
- --audit-log-path=/var/log/kubernetes/audit/audit.log
- --audit-policy-file=/etc/kubernetes/audit-policy.yaml
```

### 4.9. Hardening kubelet (`/var/lib/kubelet/config.yaml`)
```yaml
authentication:
  anonymous:
    enabled: false          # tắt truy cập ẩn danh
authorization:
  mode: Webhook             # không AlwaysAllow
readOnlyPort: 0
```
`systemctl restart kubelet`. CIS: bảo vệ quyền file config/cert (`chmod 600`, `chown root:root`).

### 4.10. Audit Logging (Cluster Setup)
```yaml
# /etc/kubernetes/audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: RequestResponse
  resources:
  - group: ""
    resources: ["secrets"]
- level: Metadata          # catch-all
```
Cờ apiserver: `--audit-policy-file`, `--audit-log-path`, `--audit-log-maxage`, `--audit-log-maxbackup`, `--audit-log-maxsize`. Nhớ mount cả file policy và thư mục log dưới dạng `hostPath` vào static pod. Level: **None, Metadata, Request, RequestResponse**.

### 4.11. ⭐ Mã hóa Secret at-rest (Cluster Hardening)
```yaml
# /etc/kubernetes/enc/enc.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources: ["secrets"]
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <base64-32-byte-key>
  - identity: {}                  # fallback
```
```bash
# Cờ apiserver:
- --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
# Mã hóa lại toàn bộ secret hiện có SAU khi bật:
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
# Kiểm tra raw trong etcd (phải thấy 'k8s:enc:aescbc...'):
ETCDCTL_API=3 etcdctl get /registry/secrets/default/<name> --cacert ... | hexdump -C
```

### 4.12. gVisor / runsc qua RuntimeClass (Minimize Microservice Vulns)
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata: { name: gvisor }
handler: runsc          # phải khớp handler trong containerd config
---
apiVersion: v1
kind: Pod
metadata: { name: sandboxed }
spec:
  runtimeClassName: gvisor
  containers: [{name: app, image: nginx}]
```
containerd (`/etc/containerd/config.toml`): thêm runtime `runsc` (`runtime_type = "io.containerd.runsc.v1"`), rồi `systemctl restart containerd`.

### 4.13. ImagePolicyWebhook (Supply Chain — admission control)
```yaml
# /etc/kubernetes/admission/admission-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  configuration:
    imagePolicy:
      kubeConfigFile: /etc/kubernetes/admission/kubeconfig.yaml
      defaultAllow: false        # fail-closed = từ chối nếu backend không tới được
```
Cờ apiserver: `--enable-admission-plugins=...,ImagePolicyWebhook` và `--admission-control-config-file=...`.

### 4.14. ServiceAccount Hardening (Cluster Hardening)
```yaml
apiVersion: v1
kind: ServiceAccount
metadata: { name: restricted-sa, namespace: prod }
automountServiceAccountToken: false      # tắt tự mount token
---
apiVersion: v1
kind: Pod
metadata: { name: app }
spec:
  serviceAccountName: restricted-sa
  automountServiceAccountToken: false     # pod-level ghi đè SA-level
```

### 4.15. securityContext hardening (Minimize Microservice Vulns)
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      privileged: false
      capabilities:
        drop: ["ALL"]
        add: ["NET_BIND_SERVICE"]   # chỉ khi cần
```

### 4.16. RBAC Least Privilege (Cluster Hardening)
```bash
kubectl auth can-i list secrets --as=system:serviceaccount:prod:app -n prod
kubectl get rolebindings,clusterrolebindings -A -o wide | grep <sa>
```
> Task hay gặp: thay binding quá quyền (bind vào `cluster-admin` hoặc wildcard) bằng Role/RoleBinding hẹp. Rule không bao giờ dùng `verbs: ["*"]`, tránh `resources: ["secrets"]`.

### 4.17. Static analysis (Supply Chain)
```bash
kubesec scan pod.yaml          # chấm điểm manifest, gợi ý sửa
kube-linter lint ./manifests/  # lint policy (privileged, thiếu limit...)
```

---

## Phần 5 — killer.sh & chiến thuật quản lý thời gian

### Tư duy về killer.sh
- Simulator **cố tình KHÓ hơn và ít thời gian hơn** đề thật ("designed to break you"). Đạt **~45–65%** trên killer.sh là bình thường; nhiều người đậu đề thật (80–95%) một tuần sau. Giá trị thật là **lời giải chi tiết** — học đến khi tự làm lại được mọi task từ đầu.
- Có **2 lượt, mỗi lượt truy cập 36 giờ** (17 câu). Câu giống nhau giữa 2 lượt → lượt 1 để học, lượt 2 để **speed-run**.

### Chiến thuật thời gian (~120 phút)
- Trung bình **~7 phút/câu**. Câu dễ 2–3 phút, trung bình 5–7, khó 8–12.
- **Ngưỡng bỏ qua:** nếu 1 task cần >10–15 phút chỉ để bắt đầu → **flag và bỏ qua** (ExamUI có nút flag/notepad). Mọi task **cân bằng điểm** — câu dễ đáng giá ngang câu khó. Có **điểm từng phần** (partial credit).
- Chừa **15–20 phút cuối** để quay lại câu đã flag.
- **Sao lưu manifest trước khi sửa:** `cp kube-apiserver.yaml ~/apiserver.bak` — gõ sai 1 cờ là apiserver sập.
- Verify từng đáp án (`kubectl get`, `describe`, check endpoints/rollout) trước khi qua câu khác.
- "**Nhớ cấu trúc của docs, đừng nhớ nội dung**" — dùng `kubectl explain <res> --recursive`, `-o yaml`, `--help`.
- Nhiều task CKS cần **SSH vào node** (`ssh cks-node`) + `sudo -i`. Sửa `/etc/kubernetes/manifests/*` thì static pod tự restart — chờ và verify bằng `crictl ps`.

### 12 dạng task tần suất cao (CKS)
1. Sửa finding kube-bench/CIS trên apiserver/kubelet/etcd (cờ + quyền file).
2. Tạo/siết **NetworkPolicy** (default-deny + allow chọn lọc, nhớ DNS).
3. Viết/sửa **Falco rule** và đọc alert từ log.
4. Gắn label **Pod Security Admission** cho namespace; sửa pod vi phạm `restricted`.
5. Harden **securityContext** pod (drop caps, runAsNonRoot, readOnlyRootFilesystem).
6. **Trivy** scan image → xóa pod chạy image CRITICAL.
7. Bật **mã hóa secret at-rest** và re-encrypt secret hiện có.
8. Cấu hình **audit policy** + cờ audit apiserver.
9. Gắn profile **AppArmor**/**seccomp** vào pod.
10. Dọn **RBAC least-privilege**; tắt **automountServiceAccountToken**.
11. Tạo **gVisor RuntimeClass** và gán pod.
12. Nối **ImagePolicyWebhook** / admission config.

---

## Phần 6 — Lộ trình học 30/60/90 ngày

### 🗓️ CKA — 6 tuần (nếu học ~1–2h/ngày)
| Tuần | Trọng tâm | Thực hành |
|------|-----------|-----------|
| 1 | Kiến trúc, Pod/Deployment/Service, kubectl imperative | Dựng cluster kubeadm 1 master + 2 worker (hoặc kind/minikube) |
| 2 | Scheduling (taint/toleration/affinity), DaemonSet, static pod | Làm bài tập từng loại |
| 3 | Services & Networking, NetworkPolicy, Ingress, Gateway API, CoreDNS | Tự phá & sửa DNS/service |
| 4 | Storage (PV/PVC/StorageClass), ConfigMap/Secret, Helm/Kustomize | Dynamic provisioning |
| 5 | **Cluster admin**: etcd backup/restore, kubeadm upgrade, RBAC, cert | Làm đi làm lại tới khi thuộc cờ |
| 6 | **Troubleshooting (30%)** + killer.sh 2 lượt | Speed-run, mô phỏng đề thật 2h |

### 🗓️ CKS — 4 tuần (SAU khi có CKA)
| Tuần | Trọng tâm |
|------|-----------|
| 1 | Cluster Setup + Hardening: kube-bench/CIS, apiserver/kubelet flags, audit, encryption at-rest, RBAC |
| 2 | Microservice: securityContext, PSA, NetworkPolicy, ServiceAccount, gVisor |
| 3 | Supply Chain: Trivy, ImagePolicyWebhook, kubesec/kube-linter, SBOM; Runtime: Falco, AppArmor, seccomp |
| 4 | killer.sh 2 lượt + luyện tốc độ SSH-vào-node, sửa static pod, verify |

### Tài nguyên học (miễn phí + trả phí)
- **Chính thức:** CNCF curriculum (GitHub), Linux Foundation candidate handbook, kubernetes.io/docs (làm quen tìm kiếm trong docs vì được mở khi thi).
- **Thực hành free:** killercoda.com (scenario CKA/CKS), kodekloud (lab), play-with-k8s.
- **Simulator:** killer.sh (đi kèm khi mua exam — 2 lượt).
- **Sách/khóa:** KodeKloud CKA/CKS (Mumshad), "Kubernetes Up & Running".

---

## Phần 7 — Checklist "phải thuộc lòng" trước khi thi

**CKA:**
- [ ] Gõ được alias + vimrc trong 60s không nhìn.
- [ ] etcd `snapshot save`/`restore` với đủ 3 cờ `--cacert/--cert/--key` từ trí nhớ.
- [ ] Thứ tự kubeadm upgrade (kubeadm → apply → drain → kubelet → uncordon).
- [ ] Static pod ở `/etc/kubernetes/manifests/`; sửa = tự restart.
- [ ] `k create role/rolebinding/clusterrole` + `k auth can-i --as=`.
- [ ] `drain --ignore-daemonsets --delete-emptydir-data`, cordon/uncordon.
- [ ] Debug: endpoints rỗng → selector; node NotReady → `journalctl -u kubelet`.
- [ ] Luôn `use-context` đầu mỗi task.

**CKS:**
- [ ] Đường dẫn: Falco `/etc/falco/`, seccomp `/var/lib/kubelet/seccomp/`, manifest `/etc/kubernetes/manifests/`.
- [ ] NetworkPolicy default-deny + **allow DNS egress**.
- [ ] EncryptionConfiguration + re-encrypt bằng `kubectl replace`.
- [ ] PSA label 3 mode (enforce/warn/audit).
- [ ] securityContext hardening đầy đủ (drop ALL caps, runAsNonRoot, readOnly root FS).
- [ ] Trivy severity CRITICAL → xóa pod.
- [ ] Sao lưu manifest trước khi sửa cờ apiserver/kubelet.
- [ ] Verify sau mỗi thay đổi (`crictl ps`, `kubectl get`).

---

> 📚 **Nguồn:** Linux Foundation (trang CKA/CKS + candidate handbook + FAQ), CNCF curriculum, killer.sh, kubernetes.io/docs, Falco/Trivy/kube-bench docs, killercoda, KodeKloud, cùng nhiều blog cộng đồng (DevOpsCube, sailor.sh, k8s.guide, stackrox study guide). **Luôn xác nhận phiên bản k8s và trọng số domain trên trang Linux Foundation ngay trước khi đăng ký thi.**
