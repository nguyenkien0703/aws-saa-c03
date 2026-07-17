# 🚢 Kubernetes Thực Chiến — Troubleshooting, Tips & Best Practices

> Tài liệu này là **cẩm nang thực chiến (field reference)** cho kỹ sư vận hành Kubernetes trên production: cách **chẩn đoán và xử lý sự cố thật**, những **cạm bẫy (gotcha)** hay gặp, và **best practice** đúc kết từ nhiều blog SRE, engineering blog, và issue thực tế của cộng đồng.
>
> Đối tượng: bạn đã biết k8s cơ bản (Pod/Deployment/Service), giờ muốn **xử lý sự cố nhanh trong đêm on-call** và **vận hành production đúng chuẩn**.
>
> Cách dùng: mỗi mục **bắt đầu bằng lệnh cần gõ** để chẩn đoán, và **kết thúc bằng cách fix thật**. Khi gặp sự cố, `Ctrl+F` đúng triệu chứng (VD `CrashLoopBackOff`, `ImagePullBackOff`, `Pending`, `502`) rồi làm theo.
>
> 📎 File song hành: **[Y-cka-cks-cert-prep.md](./Y-cka-cks-cert-prep.md)** — ôn thi chứng chỉ CKA & CKS.

---

## 📑 Mục lục

- [Phần 0 — Bức tranh lớn: kiến trúc k8s cần nhớ khi debug](#phần-0--bức-tranh-lớn-kiến-trúc-k8s-cần-nhớ-khi-debug)
- [Phần 1 — Bộ công cụ chẩn đoán (Diagnostic Toolkit)](#phần-1--bộ-công-cụ-chẩn-đoán-diagnostic-toolkit)
- [Phần 2 — Các bước đầu tiên luôn chạy (Universal First Moves)](#phần-2--các-bước-đầu-tiên-luôn-chạy-universal-first-moves)
- [Phần 3 — Sự cố Pod (Pod Issues)](#phần-3--sự-cố-pod-pod-issues)
- [Phần 4 — Sự cố Mạng (Networking)](#phần-4--sự-cố-mạng-networking)
- [Phần 5 — Sự cố Node](#phần-5--sự-cố-node)
- [Phần 6 — Quản lý tài nguyên & Autoscaling](#phần-6--quản-lý-tài-nguyên--autoscaling)
- [Phần 7 — Sự cố Storage](#phần-7--sự-cố-storage)
- [Phần 8 — Control Plane & etcd](#phần-8--control-plane--etcd)
- [Phần 9 — Những cạm bẫy kinh điển (Gotchas)](#phần-9--những-cạm-bẫy-kinh-điển-gotchas)
- [Phần 10 — Best Practices Production](#phần-10--best-practices-production)
- [Phần 11 — GitOps, Deployment & Observability](#phần-11--gitops-deployment--observability)
- [Phần 12 — Anti-patterns cần tránh](#phần-12--anti-patterns-cần-tránh)
- [Phần 13 — Bảng tra cứu nhanh triệu chứng → nguyên nhân](#phần-13--bảng-tra-cứu-nhanh-triệu-chứng--nguyên-nhân)

---

## Phần 0 — Bức tranh lớn: kiến trúc k8s cần nhớ khi debug

Khi debug, bạn phải biết **thành phần nào chịu trách nhiệm gì** để nhắm đúng chỗ. Đây là bản đồ tối thiểu:

| Nhóm | Thành phần | Nhiệm vụ | Khi hỏng thì thấy gì |
|------|-----------|----------|----------------------|
| **Control Plane** | `kube-apiserver` | Cổng vào mọi thao tác (REST API) | `kubectl` treo/timeout, cả cluster "đơ" |
| | `etcd` | Database key-value lưu toàn bộ state | apiserver chậm, mất dữ liệu, write bị từ chối |
| | `kube-scheduler` | Chọn node cho pod mới | Pod kẹt `Pending` mãi |
| | `kube-controller-manager` | Vòng lặp điều hòa (Deployment→RS→Pod...) | Deployment không tạo pod, node NotReady không evict |
| **Node** | `kubelet` | Agent trên mỗi node, chạy pod, report sức khỏe | Node `NotReady`, pod không start |
| | `kube-proxy` | Lập trình iptables/IPVS cho Service | Service không truy cập được (dù pod OK) |
| | Container runtime (`containerd`) | Chạy container thật | Pod `ContainerCreating`, `PLEG not healthy` |
| | CNI (Calico/Cilium/Flannel) | Cấp IP + mạng cho pod | Pod kẹt `ContainerCreating`, không thông mạng |
| **Add-ons** | CoreDNS | Phân giải DNS nội bộ | Service resolve fail, app không gọi được nhau |
| | metrics-server | Cấp CPU/mem cho `kubectl top` & HPA | `top` lỗi, HPA hiện `<unknown>` |

**Điều quan trọng nhất khi debug:** control plane component (`apiserver`, `scheduler`, `controller-manager`, `etcd`) chạy dưới dạng **static pod** — file manifest nằm ở `/etc/kubernetes/manifests/` trên node master. Sửa file → kubelet **tự restart** pod đó. Đây là cách bạn "chữa" control plane.

---

## Phần 1 — Bộ công cụ chẩn đoán (Diagnostic Toolkit)

### 1.1. Alias & autocomplete (cài 1 lần, tiết kiệm cả đời)

```bash
# ~/.bashrc  (zsh: đổi 'bash' -> 'zsh')
source <(kubectl completion bash)
alias k=kubectl
complete -o default -F __start_kubectl k    # để completion chạy qua alias 'k'
export KUBE_EDITOR="vim"
```

### 1.2. Các flag `kubectl` đáng thuộc lòng

| Flag | Tác dụng |
|------|----------|
| `-o wide` | Thêm cột NODE, IP — biết pod nằm ở node nào |
| `-o yaml` / `-o json` | Xem toàn bộ spec + status |
| `-o jsonpath='{...}'` | Trích đúng field (dùng trong script) |
| `-o custom-columns=...` | Bảng tùy biến cột |
| `--sort-by=` | Sắp xếp (VD theo restartCount, theo thời gian event) |
| `--field-selector` | Lọc phía server (rẻ hơn `grep`) |
| `-w` / `--watch` | Stream thay đổi realtime |
| `--show-labels` | Hiện label — cực quan trọng khi debug selector |

```bash
# Ví dụ thực chiến
kubectl get pods --sort-by=.status.containerStatuses[0].restartCount   # pod restart nhiều nhất
kubectl get events --sort-by=.lastTimestamp                            # event mới nhất cuối cùng
kubectl get pods --field-selector=status.phase!=Running               # pod đang có vấn đề
kubectl get pods -o custom-columns='NAME:.metadata.name,NODE:.spec.nodeName,IMAGE:.spec.containers[0].image'
kubectl explain pod.spec.securityContext --recursive                  # tra schema offline
```

### 1.3. Krew plugins & TUI (dùng hằng ngày)

Cài `krew` trước, rồi `kubectl krew install <tên>`:

| Công cụ | Công dụng |
|---------|-----------|
| **ctx / ns** (kubectx/kubens) | Đổi context/namespace tức thì (interactive với `fzf`) |
| **stern** | Tail log nhiều pod/nhiều container 1 lúc, regex + màu: `stern -l app=api --since 5m` |
| **view-secret** | Decode Secret không cần base64 thủ công: `kubectl view-secret my-secret` |
| **neat** | Bỏ managedFields/status → manifest sạch để commit git |
| **tree** | Xem cây sở hữu (Deployment→RS→Pod, hoặc CRD): `kubectl tree deploy api` |
| **kube-ps1** | Hiện context/namespace ngay trên prompt (tránh "lỡ tay gõ nhầm prod") |
| **k9s** (binary riêng) | TUI thống trị: điều hướng pod/log/exec/port-forward bằng phím tắt |

### 1.4. Debug pod đang chạy: `kubectl debug` (ephemeral container)

Cách debug **hiện đại** — chèn container công cụ vào pod đang chạy (kể cả image distroless không có shell):

```bash
# Gắn "hộp đồ nghề" mạng vào pod đang chạy, dùng chung network namespace
kubectl debug -it <pod> --image=nicolaka/netshoot --target=<container>

# Nhân bản 1 pod đang CrashLoop thành pod có shell để mổ xẻ
kubectl debug <pod> -it --image=busybox --copy-to=<pod>-debug --share-processes

# Mở shell trên chính NODE (pod privileged, mount host vào /host)
kubectl debug node/<node> -it --image=ubuntu
```

> `nicolaka/netshoot` = image debug mạng "quốc dân": có `curl`, `dig`, `tcpdump`, `ss`, `iproute2`, `mtr`, `iperf3`.

### 1.5. Debug ở tầng node: `crictl` + `journalctl`

Khi `kubectl` bó tay (apiserver không tới được node, pod chưa đăng ký):

```bash
crictl ps -a                    # tất cả container kể cả đã thoát
crictl logs <container-id>
crictl inspect <container-id>
crictl rmi --prune              # dọn image rác (giải phóng disk)

journalctl -u kubelet -f                    # log kubelet realtime
journalctl -u kubelet --since "30m ago"
systemctl status kubelet containerd
```

---

## Phần 2 — Các bước đầu tiên luôn chạy (Universal First Moves)

Trước khi làm gì phức tạp, luôn chạy bộ này — **80% câu trả lời nằm ở đây**:

```bash
kubectl get pods -n <ns> -o wide                         # STATUS, RESTARTS, node
kubectl describe pod <pod> -n <ns>                       # khối Events ở cuối = vàng
kubectl get events -n <ns> --sort-by='.lastTimestamp'
kubectl logs <pod> -n <ns> --previous                    # log của container ĐÃ CHẾT
kubectl logs <pod> -n <ns> -c <container>                # pod nhiều container
```

> 🔑 **Thói quen giá trị nhất:** đọc khối **Events** ở cuối `describe`, và **luôn dùng `--previous`** với pod đang restart. Log hiện tại chỉ cho thấy container mới (trống rỗng), không phải container vừa chết.

---

## Phần 3 — Sự cố Pod (Pod Issues)

### 3.1. `CrashLoopBackOff` — container khởi động rồi chết lặp lại

Container start → crash → restart với backoff tăng dần (10s → 20s → 40s → … → tối đa 5 phút). Đây là **triệu chứng**, không phải nguyên nhân.

**Chẩn đoán:**
```bash
kubectl logs <pod> --previous          # ← stack trace của lần crash
kubectl describe pod <pod>             # Last State + Exit Code + Reason
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[0].lastState.terminated.exitCode}'
```

**Đọc Exit Code — đây là chìa khóa:**

| Exit Code | Ý nghĩa | Hướng xử lý |
|-----------|---------|-------------|
| **0** | Thoát "thành công" nhưng `restartPolicy: Always` vẫn restart | Entrypoint không phải tiến trình chạy dài (script chạy xong, thiếu `CMD`, chạy nền) |
| **1** | Lỗi ứng dụng chung | Đọc log `--previous` (exception, thiếu env/config, không kết nối được dependency lúc boot) |
| **126** | Entrypoint không có quyền thực thi | Sai permission, sai kiến trúc binary |
| **127** | Command not found | Sai đường dẫn ENTRYPOINT/CMD, image distroless không có shell |
| **137** | SIGKILL — gần như luôn là **OOMKilled** | Xem mục 3.3, hoặc liveness probe giết pod |
| **139** | SIGSEGV (segfault) | Bug native/thư viện C |
| **143** | SIGTERM (shutdown "êm") | Thường là bình thường |

**Nguyên nhân hay gặp:** bug/misconfig lúc boot, thiếu ConfigMap/Secret/env, migration DB fail, dependency chưa sẵn sàng lúc start, **liveness probe giết app khởi động chậm** (xem Phần 9), memory limit quá thấp.

**Mẹo thực chiến khi log trống & crash quá nhanh không kịp `exec`:** ghi đè command để giữ container sống rồi vào mổ:
```bash
# Sửa command của deployment thành ["sleep","3600"], rồi:
kubectl exec -it <pod> -- sh
# ...chạy tay entrypoint thật để xem lỗi hiện ra
```

### 3.2. `ImagePullBackOff` / `ErrImagePull` — không kéo được image

`ErrImagePull` = lỗi lần đầu; `ImagePullBackOff` = kubelet đang backoff retry.

**Chẩn đoán:** `kubectl describe pod <pod>` → đọc **đúng dòng message trong Events**, nó nói rõ lỗi gì:

| Message | Nguyên nhân |
|---------|-------------|
| `manifest unknown` / `not found` | **Sai tag hoặc tên image** (phổ biến nhất, đơn giản nhất) |
| `401 Unauthorized` / `403` | Registry private thiếu/sai `imagePullSecret` |
| `no such host` / timeout | DNS/mạng/firewall tới registry, hoặc node air-gapped |
| `pull rate limit` | Docker Hub giới hạn pull ẩn danh |

```bash
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].image}'   # xem đang kéo image gì
```

**Fix auth:**
```bash
kubectl create secret docker-registry regcred \
  --docker-server=<registry> --docker-username=<u> \
  --docker-password=<p> --docker-email=<e> -n <ns>
```
Gắn vào pod spec, hoặc gắn vào ServiceAccount để áp dụng cho mọi pod:
```bash
kubectl patch serviceaccount default -n <ns> \
  -p '{"imagePullSecrets":[{"name":"regcred"}]}'
```

**Cạm bẫy hay dính:** secret nằm **sai namespace** (imagePullSecrets có phạm vi namespace); secret `type: Opaque` thay vì `kubernetes.io/dockerconfigjson`; `imagePullPolicy: Always` với tag chỉ tồn tại local trên vài node; CA registry khác nhau giữa các node → chỗ chạy chỗ không. Kiểm tra creds:
```bash
kubectl get secret regcred -o jsonpath='{.data.\.dockerconfigjson}' | base64 -d
```

### 3.3. `OOMKilled` (exit 137) — bị kernel giết vì vượt memory limit

SIGKILL = 128 + 9 = 137. Kernel giết container vì vượt **memory limit**.

**Chẩn đoán:**
```bash
kubectl describe pod <pod>    # Last State: Terminated, Reason: OOMKilled
kubectl top pod <pod> --containers                          # cần metrics-server
# Trên node:
dmesg -T | grep -i oom       # hoặc: journalctl -k | grep -i "killed process"
```

**Phân biệt 2 trường hợp:** nếu **chỉ container của bạn** chết → nó vượt **limit của chính nó**. Nếu **nhiều pod trên cùng node** chết một lúc và node báo MemoryPressure → **cạn RAM ở tầng node** (eviction).

**Fix:** tăng `resources.limits.memory`, hoặc sửa leak/workload quá lớn. Đặt `requests == limits` cho memory để có QoS **Guaranteed** (được bảo vệ khỏi eviction).

> ⚠️ **Cạm bẫy runtime kinh điển:** JVM/Node/Go **không tự thấy cgroup limit** — JVM tính heap theo RAM của cả host rồi bị OOMKilled. Fix: `-XX:MaxRAMPercentage` (JVM đời mới đã container-aware) hoặc set `-Xmx` thấp hơn pod limit, chừa headroom cho non-heap/metaspace/thread.

### 3.4. Pod `Pending` — không bao giờ được schedule

`describe pod` chứa **lý do nguyên văn từ scheduler**. Đọc đúng từng chữ:

| Message trong Events | Nguyên nhân & Fix |
|----------------------|-------------------|
| `Insufficient cpu` / `Insufficient memory` | Không node nào đủ **requests**. Giảm requests, thêm node, hoặc kiểm tra requests bị set nhầm quá cao |
| `node(s) had untolerated taint {...}` | Thêm **toleration** phù hợp, hoặc node đang bị taint (control-plane, GPU pool...) |
| `didn't match Pod's node affinity/selector` | `nodeSelector`/`nodeAffinity` khớp 0 node (sai label, node pool scale về 0) |
| `unbound immediate PersistentVolumeClaims` | PVC chưa bound (xem Phần 7) |
| `too many pods` | Node đạt max-pods (EKS: cạn IP của CNI) |

```bash
kubectl describe pod <pod> | grep -A20 Events
kubectl describe nodes | grep -A5 "Allocated resources"
```

### 3.5. Init container fail

Init container chạy tuần tự đến hết trước khi app container start. Một cái fail chặn tất cả; pod hiện `Init:CrashLoopBackOff` hoặc `Init:Error`.
```bash
kubectl logs <pod> -c <init-container>          # thêm --previous nếu đã restart
```
Nguyên nhân thường gặp: init "wait-for-dependency" chờ mãi (DB/Service chưa lên → thực ra là vấn đề Service/DNS), migration fail, sai permission volume dùng chung. → Sửa dependency hoặc logic init.

### 3.6. Kẹt `ContainerCreating`

Init xong hết; kubelet đang gắn volume/mạng/secret và bị chặn.
```bash
kubectl describe pod <pod>          # tìm FailedMount / FailedCreatePodSandBox
journalctl -u kubelet --since "15 minutes ago" | grep -iE "sandbox|cni|volume|mount|secret"
```

| Event | Nguyên nhân & Fix |
|-------|-------------------|
| `secret/configmap "x" not found` | Object thiếu hoặc sai namespace. Tạo nó, hoặc đánh dấu volume `optional: true` |
| `FailedCreatePodSandBox` / CNI errors | CNI DaemonSet (calico/cilium/aws-node) chưa healthy, hoặc node vừa join (config cần 30–60s). EKS: cạn ENI/IP |
| `MountVolume.SetUp failed` / `Multi-Attach error` | Volume RWO còn gắn ở node đã chết. Xóa VolumeAttachment cũ (xem dưới) |

```bash
kubectl get volumeattachment
kubectl delete volumeattachment <name>       # gỡ RWO volume kẹt ở node chết
```

### 3.7. Pod `Evicted`

kubelet chủ động giết pod để giành lại tài nguyên node (memory, disk, inode). Pod còn xác với `Status: Evicted`.
```bash
kubectl get pods -A --field-selector=status.phase=Failed | grep Evicted
kubectl describe node <node> | grep -iE "pressure|evict"
```

| Loại pressure | Nguyên nhân & Fix |
|---------------|-------------------|
| **DiskPressure** | Disk root/ephemeral đầy: log container, image, `emptyDir`. Fix: xoay log, `crictl rmi --prune`, disk to hơn, set `ephemeral-storage` requests/limits |
| **Inode exhaustion** | `df -h` còn trống nhưng vẫn evict → kiểm tra `df -i`. Hàng triệu file nhỏ (log tệ) |
| **MemoryPressure** | Pod không limit hoặc quá to. Set requests/limits; QoS Guaranteed bị evict cuối cùng |

```bash
kubectl delete pod -A --field-selector=status.phase=Failed    # dọn xác pod Evicted
```

---

## Phần 4 — Sự cố Mạng (Networking)

### 4.1. Service không truy cập được — kiểm tra ENDPOINTS là tất cả

```bash
kubectl get endpoints <svc> -n <ns>      # hoặc: kubectl get endpointslices
kubectl describe svc <svc> -n <ns>
```

**Nếu ENDPOINTS rỗng**, Service trỏ vào hư không — nhưng **DNS vẫn resolve bình thường** (đây là hiểu lầm số 1: tên resolve được, kết nối vẫn fail). Nguyên nhân:
- **Selector không khớp** — `svc.spec.selector` không match label pod. So sánh:
```bash
kubectl get svc <svc> -o jsonpath='{.spec.selector}'
kubectl get pods --show-labels -n <ns>
```
- **Pod chưa Ready** — readiness probe fail sẽ loại pod khỏi Endpoints. Sửa probe/app.
- **Tất cả replica đều chết.**

**Nếu có endpoints mà vẫn fail:**
- **`targetPort` ≠ port container đang lắng nghe** — lỗi kinh điển. `port` là cổng client gọi; `targetPort` phải bằng `containerPort` thật.
```bash
kubectl exec <pod> -- ss -tlnp          # app đang nghe cổng nào?
```
- App bind `127.0.0.1` thay vì `0.0.0.0` → không tới được từ ngoài pod.
- Test trực tiếp pod IP để cô lập lỗi Service hay pod: `kubectl exec <pod-khác> -- curl <podIP>:<port>`.

### 4.2. DNS resolve fail (CoreDNS)

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns      # CoreDNS healthy?
kubectl logs -n kube-system -l k8s-app=kube-dns
kubectl run -it --rm dns --image=busybox:1.36 --restart=Never -- \
  nslookup kubernetes.default.svc.cluster.local
```
Từ trong pod: xem `/etc/resolv.conf` (phải trỏ ClusterIP của kube-dns, có `search` domains, `ndots:5`). Nếu CoreDNS `CrashLoopBackOff`, nguyên nhân hay gặp là **loop resolv.conf** (`Loop ... detected`) — sửa `forward` trong Corefile trỏ upstream thật, không trỏ về stub local. Sau khi upgrade, kiểm tra **RBAC của CoreDNS** có quyền list endpointslices không.

### 4.3. Độ trễ DNS 5 giây huyền thoại ⭐

Lookup thỉnh thoảng mất **đúng 5 giây**. Nguyên nhân gốc: **race trong module conntrack của Linux kernel** khi pod bắn song song query A + AAAA qua cùng socket có SNAT/DNAT; một packet bị drop, client retry sau timeout 5s. Đây là **bug kernel/mạng, không phải lỗi app bạn**.

**Cách fix (theo thứ tự ưu tiên):**
1. **Triển khai NodeLocal DNSCache** — agent cache trên mỗi node, dùng TCP upstream, né race DNAT của conntrack. **Đây là fix chuẩn production.**
2. `options single-request-reopen` (hoặc `use-vc` cho TCP) trong `dnsConfig` pod — tách A/AAAA sang socket khác nhau.
3. Tắt IPv6/AAAA nếu không dùng.

### 4.4. Thuế trễ từ `ndots:5`

Kubernetes chèn `ndots:5`, nên bất kỳ tên nào **ít hơn 5 dấu chấm** sẽ bị thử với mọi `search` domain trước (`svc.cluster.local`, `cluster.local`, …) rồi mới lookup thật — lãng phí 5+ query cho mỗi hostname ngoài. Fix: dùng FQDN có **dấu chấm cuối** (`api.example.com.`) cho host ngoài, hoặc giảm `ndots` per-pod qua `dnsConfig.options`. NodeLocal DNSCache cũng che bớt chi phí này.

### 4.5. NetworkPolicy chặn traffic

Triệu chứng: kết nối **timeout** (không phải refused) giữa 2 pod đáng lẽ nói chuyện được. NetworkPolicy là **allow-list** — một khi có policy chọn pod, mọi thứ không được cho phép rõ ràng đều bị chặn.
```bash
kubectl get networkpolicy -n <ns>
kubectl describe networkpolicy <name> -n <ns>
```
> ⚠️ **Bẫy kinh điển:** thêm policy *ingress* rồi **DNS chết** cho các pod đó — vì default-deny cũng chặn luôn **egress UDP/53 tới CoreDNS**. Luôn nhớ allow egress DNS. (Xem mẫu YAML default-deny + allow-DNS trong file CKS.)

### 4.6. Ingress 502 / 503 / 504

Đọc **log của ingress controller trước tiên** — nó nói rõ lỗi gì.
```bash
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --tail=100
```

| Mã | Ý nghĩa & Fix |
|----|---------------|
| **502 Bad Gateway** | `connect() failed (111: Connection refused)`: backend không nghe, hoặc **targetPort sai**, backend crash. Thường kèm `no endpoints available` → quay lại 4.1 |
| **503 Service Unavailable** | Không có endpoint healthy / backend fail readiness, hoặc Ingress trỏ Service không tồn tại |
| **504 Gateway Timeout** | Backend nhận nhưng quá chậm. Tăng `nginx.ingress.kubernetes.io/proxy-read-timeout` (mặc định 60s); điều tra app/DB chậm |

### 4.7. Sự cố MTU ("It's not DNS, it's MTU")

Khó chịu vì request nhỏ chạy được, payload lớn/TLS handshake thì treo. Overlay/encapsulation (VXLAN, IPIP, WireGuard) làm giảm MTU hiệu dụng; nếu MTU của CNI set cao hơn underlay, packet lớn bị **drop im lặng**. Chẩn đoán:
```bash
ping -M do -s 1472 <target>      # do-not-fragment, tìm ngưỡng MTU
```
Fix: hạ MTU của CNI/pod cho khớp underlay (trừ đi overhead encapsulation).

### 4.8. kube-proxy

Nếu Service **lúc được lúc không trên toàn cluster**, nghi kube-proxy (nó lập trình iptables/IPVS).
```bash
kubectl logs -n kube-system -l k8s-app=kube-proxy --tail=100
iptables-save | grep <svc-clusterip>     # trên node — rule còn không?
```
Rule cũ tồn đọng, lệch mode IPVS vs iptables, hoặc kube-proxy crashloop sau upgrade đều phá routing Service dù pod vẫn ổn.

---

## Phần 5 — Sự cố Node

### 5.1. Node `NotReady`

```bash
kubectl get nodes
kubectl describe node <node> | grep -A10 Conditions   # Disk/Memory/PID Pressure/Ready
# SSH vào node:
systemctl status kubelet
journalctl -u kubelet -n 200 --no-pager
df -h ; df -i ; free -m
```

| Nguyên nhân | Dấu hiệu & Fix |
|-------------|----------------|
| **kubelet down/crash** | Node ngừng heartbeat → NotReady sau ~40s, pod bị evict. Restart/sửa kubelet; xem log tìm lỗi cert, config, full disk |
| **`PLEG is not healthy`** | Runtime (containerd) treo/quá tải. Kiểm tra `crictl ps`, `systemctl status containerd`, I/O node |
| **DiskPressure** | Log/image chạy loạn. `crictl rmi --prune`, sửa log rotation |
| **PIDPressure** | Bùng nổ process (fork bomb, zombie leak). Hiếm |
| **MemoryPressure** | Cạn RAM node; kubelet đang evict |
| **Network/CNI down** | Node không tới được control plane; kiểm tra routing, security group, CNI pod trên node đó |

---

## Phần 6 — Quản lý tài nguyên & Autoscaling

### 6.1. requests vs limits — hiểu đúng để không "tự bắn chân"

- **Requests** = cam kết scheduling (giữ chỗ, quyết định QoS). **Limits** = trần cứng (memory vượt → OOMKill; CPU vượt → throttle, **không bao giờ bị giết**).
- Không set requests → QoS **BestEffort** → bị evict đầu tiên.
- Requests quá cao → mật độ pod thấp, lãng phí tiền, báo "insufficient resources" giả.
- Requests quá thấp → node oversubscribe, OOM/throttle do hàng xóm ồn.
- `requests == limits` (memory) → QoS **Guaranteed**, an toàn nhất cho workload quan trọng.

### 6.2. CPU throttling — kẻ giết người thầm lặng ⭐

CPU limit áp **quota CFS mỗi chu kỳ 100ms**; chạm quota là kernel **throttle** (đóng băng) container tới chu kỳ sau, **kể cả khi node đang rảnh**. Biểu hiện là **latency spike / tail latency cao**, KHÔNG phải CPU usage cao.
```bash
kubectl exec <pod> -- cat /sys/fs/cgroup/cpu/cpu.stat    # nr_throttled, throttled_time
# Prometheus: rate(container_cpu_cfs_throttled_periods_total[5m])
```
> ⚠️ **Cái bẫy "death spiral":** CPU limit thấp → throttle cao nhưng *usage quan sát thấp* → HPA/VPA scale thiếu hoặc khuyến nghị limit còn thấp hơn → throttle tệ hơn. Nhiều nơi **bỏ hẳn CPU limit** (giữ CPU *request*) để tránh throttle. Luôn xem metric throttle trước khi tin khuyến nghị CPU của VPA.

### 6.3. HPA không hoạt động

```bash
kubectl get hpa -n <ns>          # TARGETS hiện <unknown>/50% là hỏng
kubectl describe hpa <name> -n <ns>
```

| Triệu chứng | Nguyên nhân |
|-------------|-------------|
| Target `<unknown>` | **metrics-server chưa cài/hỏng** (`kubectl top` cũng fail), hoặc pod **không set CPU/mem requests** (utilization % tính theo requests) |
| Custom metrics không lên | Adapter (Prometheus Adapter/KEDA) không phục vụ metric |
| Scale up nhưng không nhanh hơn | Pod bị **CPU throttle** → mọi replica throttle như nhau. Fix throttle, đừng thêm replica |
| Flapping (lên xuống liên tục) | Chỉnh `behavior` stabilization window |

> Đừng chạy **HPA + VPA cùng một metric** (CPU/mem) → xung đột, dao động.

### 6.4. Cluster Autoscaler không scale up

```bash
kubectl logs -n kube-system -l app=cluster-autoscaler --tail=200
kubectl get configmap cluster-autoscaler-status -n kube-system -o yaml
```
Lý do không thêm node: pod requests **vượt sức chứa của mọi loại node** (sẽ không bao giờ vừa); node group đạt `--max`; pod có `nodeSelector`/affinity/taint không group nào thỏa; PVC pin vào zone đã đầy; thiếu IAM sửa ASG. Status configmap & log nói rõ lý do.

### 6.5. Bộ autoscaling hiện đại (tham khảo chọn công cụ)

| Công cụ | Scale cái gì | Khi nào dùng |
|---------|--------------|--------------|
| **HPA** | Số replica (theo CPU/mem/custom) | Mặc định cho web/API |
| **VPA** | requests/limits mỗi container | Right-size (thường chỉ chạy chế độ khuyến nghị) |
| **Cluster Autoscaler** | Số node (qua ASG/node group) | Chuẩn đa-cloud |
| **Karpenter** | Provision node đúng size trực tiếp | **EKS/AWS**: nhanh (<30s), bin-packing tốt hơn |
| **KEDA** | Replica theo sự kiện (Kafka lag, SQS, cron), **scale-to-zero** | Worker hàng đợi, workload bursty/async |

---

## Phần 7 — Sự cố Storage

### 7.1. PVC kẹt `Pending`
```bash
kubectl describe pvc <pvc> -n <ns>     # Events nói rõ lý do
kubectl get storageclass ; kubectl get pv
```
- StorageClass `volumeBindingMode: WaitForFirstConsumer` → PVC **Pending là bình thường** cho tới khi pod dùng nó được schedule. → Cứ schedule pod.
- `provisioning failed` → lỗi CSI driver, hết quota cloud, sai tên StorageClass, thiếu default StorageClass.
- Lệch access mode / size / zone với PV có sẵn (static provisioning).

### 7.2. Mount volume fail
`FailedMount` / `FailedAttachVolume` trong events. Volume **RWO chỉ gắn được 1 node** — lỗi **Multi-Attach** nghĩa là node cũ còn giữ (node chết đột ngột). Xóa `volumeattachment` cũ (xem 3.6). Ngoài ra: permission (`fsGroup`/`securityContext`), CSI node plugin DaemonSet không chạy trên node, cloud API throttle lúc attach.

### 7.3. Pod kẹt `Terminating` (finalizers)
```bash
kubectl get pod <pod> -o jsonpath='{.metadata.finalizers}'
```
- Node đã chết, kubelet không graceful shutdown xong được: `kubectl delete pod <pod> --grace-period=0 --force` (nhưng **finalizer vẫn chặn** cho tới khi được gỡ).
- Nếu **finalizer** của controller đã chết đang giữ, gỡ finalizer:
```bash
kubectl patch pod <pod> -p '{"metadata":{"finalizers":null}}' --type=merge
```
> ⚠️ Chỉ gỡ finalizer khi **hiểu rõ** — chúng tồn tại để đảm bảo cleanup (detach volume, deregister load balancer).

---

## Phần 8 — Control Plane & etcd

### 8.1. Certificate của kubeadm hết hạn ⭐

Cert client/serving của kubeadm hết hạn sau **1 năm**. Triệu chứng: `kubectl` báo `x509: certificate has expired`, kubelet NotReady, apiserver từ chối TLS.
```bash
kubeadm certs check-expiration                   # bảng tất cả cert + ngày hết hạn
```
**Gia hạn (trên MỌI node control-plane):**
```bash
kubeadm certs renew all
# Sau đó BẮT BUỘC restart control plane để nạp cert mới:
systemctl restart kubelet
# (hoặc di chuyển manifest static pod ra/vào /etc/kubernetes/manifests/)
cp /etc/kubernetes/admin.conf ~/.kube/config     # cập nhật kubeconfig admin
```
> 🔴 **Lỗi số 1:** gia hạn cert xong **quên restart** apiserver/controller-manager/scheduler/etcd — gia hạn vô nghĩa cho tới khi tiến trình reload. **Phòng ngừa:** upgrade cluster ít nhất mỗi năm (upgrade tự gia hạn cert). CA 10 năm là riêng — CA hết hạn = xây lại cluster.

### 8.2. etcd đầy disk / chậm

etcd nhạy cảm với latency và dung lượng. Triệu chứng: apiserver chậm/timeout, `etcdserver: mvcc: database space exceeded`, write bị từ chối.
```bash
ETCDCTL_API=3 etcdctl endpoint status --write-out=table --endpoints=... --cacert=... --cert=... --key=...
```
Fix: etcd chạm `--quota-backend-bytes` mặc định **2GB** → compact revision cũ và **defrag**:
```bash
etcdctl compact <rev>
etcdctl defrag --endpoints=...
etcdctl alarm disarm
```
Sau đó tăng quota và — quan trọng nhất — cho etcd **SSD/NVMe riêng nhanh** (etcd bị chặn bởi fsync; disk chậm → apiserver toàn cluster chậm). Theo dõi `etcd_disk_wal_fsync_duration_seconds` và `etcd_server_leader_changes_seen_total` (leader nhảy liên tục = disk/network đau).

### 8.3. Component control plane down (apiserver/scheduler/controller-manager)
Đây là **static pod**:
```bash
ls /etc/kubernetes/manifests/
crictl ps -a | grep <component>
crictl logs <container-id>
# sửa manifest yaml (flag/image/port sai) → kubelet tự restart
```

---

## Phần 9 — Những cạm bẫy kinh điển (Gotchas)

### 9.1. Liveness vs Readiness probe — gây outage nhiều hơn là ngăn ⭐⭐

- **Liveness fail → container BỊ GIẾT và restart.** **Readiness fail → pod bị loại khỏi Service endpoints** (không restart).
- **KHÔNG BAO GIỜ đặt check dependency (DB, API downstream) trong liveness probe.** War story: DB chớp nháy → liveness fail trên *mọi* pod → cả fleet restart đồng loạt → không reconnect được trong bão → CrashLoopBackOff toàn cluster. Check dependency thuộc về **readiness**, hoặc lý tưởng là không check.
- **App khởi động chậm không có startup probe:** liveness bắn lúc boot, giết app trước khi lên → trông như crash. Thêm **`startupProbe`** (`failureThreshold × periodSeconds` rộng rãi) để hoãn liveness tới khi app lên.
- **`timeoutSeconds: 1` quá gắt** giết pod khỏe trong lúc GC pause. Cho timeout/threshold thực tế.
- **Dùng chung `/health` cho cả 2 probe** (mặc định Spring Boot): dependency chậm làm `/health` unhealthy → liveness giết pod hoàn toàn ổn. Tách `/livez` (chỉ process sống) và `/readyz` (sẵn sàng phục vụ).

**Mẫu probe chuẩn:**
```yaml
startupProbe:                    # cổng chặn cho app khởi động chậm
  httpGet: { path: /healthz, port: 8080 }
  failureThreshold: 30
  periodSeconds: 10              # cho phép tới 300s để lên
livenessProbe:                   # CHỈ restart khi state không cứu được (chỉ self)
  httpGet: { path: /healthz, port: 8080 }
  periodSeconds: 10
  failureThreshold: 3
readinessProbe:                  # gate traffic; được phép flap an toàn
  httpGet: { path: /ready, port: 8080 }
  periodSeconds: 5
  failureThreshold: 3
```

### 9.2. `terminationGracePeriodSeconds` & graceful shutdown

Khi xóa/evict, pod nhận **SIGTERM**, sau `terminationGracePeriodSeconds` (mặc định **30s**) thì **SIGKILL**. Cạm bẫy:
- App **phớt lờ SIGTERM** → chết cứng, rớt kết nối. → Phải bắt SIGTERM để drain.
- Endpoint removal và SIGTERM xảy ra **đồng thời** → thêm `preStop: sleep 5` để traffic LB kịp drain trước:
```yaml
lifecycle:
  preStop:
    exec: { command: ["sh","-c","sleep 5"] }
terminationGracePeriodSeconds: 45
```
> ~90% lỗi "graceful shutdown" là do app phớt lờ SIGTERM. Grace period **bao gồm** thời gian chạy preStop.

### 9.3. PodDisruptionBudget chặn drain

`kubectl drain` treo / node kẹt `Ready,SchedulingDisabled`:
```bash
kubectl get pdb -A
kubectl describe pdb <name> -n <ns>     # ALLOWED DISRUPTIONS: 0 = bị chặn
```
Nếu `ALLOWED DISRUPTIONS = 0`, drain không evict được. **Đừng force mù quáng** — vấn đề thật thường là pod thay thế không lên Ready (thiếu tài nguyên, image pull, probe fail). Sửa pod thay thế, hoặc tạm nới PDB. **PDB `minAvailable == số replica` chặn vĩnh viễn mọi eviction tự nguyện** — misconfig phổ biến. PDB **không** bảo vệ khỏi node crash (disruption không tự nguyện).

### 9.4. Rolling update kẹt
```bash
kubectl rollout status deploy/<name> -n <ns>
kubectl get rs -n <ns>
```
Pod ReplicaSet mới không lên Ready → readiness fail, ImagePullBackOff, CrashLoop, hoặc thiếu tài nguyên; `maxUnavailable/maxSurge` quá gắt. Pod cũ ở lại vì cái mới chưa Ready (đúng thiết kế — sửa pod mới). `kubectl rollout undo deploy/<name>` để thoát.

### 9.5. Namespace kẹt `Terminating` mãi (finalizers)
```bash
kubectl get ns <ns> -o json | jq '.status'
kubectl api-resources --verbs=list --namespaced -o name \
  | xargs -n1 kubectl get --show-kind --ignore-not-found -n <ns>   # tìm object còn sót
```
Nguyên nhân thường gặp: **APIService orphan / custom resource kẹt** mà controller đã chết. Fix tốt nhất: xóa resource còn sót / sửa APIService fail. Force cuối cùng (nguy hiểm, có thể orphan tài nguyên):
```bash
kubectl get ns <ns> -o json | jq 'del(.spec.finalizers[])' \
  | kubectl replace --raw "/api/v1/namespaces/<ns>/finalize" -f -
```

---

## Phần 10 — Best Practices Production

### 10.1. securityContext — baseline hạn chế tối đa
Drop tất cả, chỉ add lại cái cần:
```yaml
securityContext:                 # pod-level
  runAsNonRoot: true
  runAsUser: 10001
  fsGroup: 10001
  seccompProfile:
    type: RuntimeDefault
containers:
- name: app
  securityContext:               # container-level (ghi đè pod)
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    privileged: false
    capabilities:
      drop: ["ALL"]              # add lại NET_BIND_SERVICE nếu cần bind cổng <1024
```
> App cần ghi → mount `emptyDir` tại `/tmp` thay vì tắt `readOnlyRootFilesystem`. Secret: base64 **không phải** mã hóa — bật encryption-at-rest + dùng External Secrets/Vault/Sealed Secrets thay vì raw Secret trong git.

### 10.2. HA & chống gián đoạn
- **PodDisruptionBudget** bảo vệ khỏi gián đoạn *tự nguyện* (drain, upgrade). Đừng set `minAvailable == replicas`.
- **topologySpreadConstraints** — cách hiện đại (thay anti-affinity) để trải pod qua zone/node:
```yaml
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: topology.kubernetes.io/zone
  whenUnsatisfiable: ScheduleAnyway    # hoặc DoNotSchedule (đảm bảo cứng hơn)
  labelSelector: { matchLabels: { app: api } }
```
- Tách prod và non-prod thành **cluster khác nhau**, không chỉ namespace (blast radius, RBAC, resource starvation).

### 10.3. Nhãn chuẩn (recommended labels)
Dùng bộ chuẩn để selector/cost allocation/dashboard hoạt động: `app.kubernetes.io/name`, `/instance`, `/version`, `/component`, `/part-of`, `/managed-by`.

### 10.4. Image hygiene
- **Không bao giờ `:latest` trên production** — không xác định, phá rollback/audit. Pin tag bất biến (semver hoặc git-SHA), lý tưởng là **digest** (`@sha256:...`). Scan image (Trivy/Grype), ký (cosign/sigstore).

---

## Phần 11 — GitOps, Deployment & Observability

### 11.1. GitOps: ArgoCD vs FluxCD

| | ArgoCD | FluxCD |
|--|--------|--------|
| **UI** | Web UI mạnh, RBAC theo team | Không bundle UI (dùng Weave GitOps/Headlamp) |
| **Mô hình** | Hub-and-spoke, đa cluster | Pull per-cluster, module hóa |
| **Hợp với** | Đa số team (dễ onboard) | Platform team muốn building block tách rời, cô lập per-cluster |

### 11.2. Helm vs Kustomize
- **Helm** — template + đóng gói + quản lý dependency + **rollback** (`helm rollback`). Hợp app bên thứ 3, hệ phức tạp. Best practice: pin version chart, `values-{dev,staging,prod}.yaml`, `helm diff` trước khi apply, `--atomic --timeout`, lint trong CI.
- **Kustomize** — patch YAML thuần qua base + overlay, minh bạch, không ngôn ngữ template, tích hợp sẵn kubectl (`-k`). Hợp app của chính bạn.
- **Kết hợp:** Helm đóng gói + Kustomize overlay lên trên (ArgoCD & Flux đều hỗ trợ).

### 11.3. Chiến lược rollout
| Chiến lược | Bản chất | Khi dùng |
|-----------|----------|----------|
| **RollingUpdate** (mặc định) | Thay dần, chỉnh `maxSurge`/`maxUnavailable` | Đa số (zero-downtime nếu probe + PDB đúng) |
| **Recreate** | Giết hết rồi tạo mới (có downtime) | Singleton, schema không tương thích |
| **Blue-Green** | 2 stack đầy đủ, flip traffic tức thì | Rollback nhanh, chấp nhận x2 chi phí tạm |
| **Canary** | Dịch % traffic, xem metric, promote/abort | Argo Rollouts (hợp ArgoCD) / Flagger (hợp Flux) |

### 11.4. Observability — 3 tầng metric
- **metrics-server** — cấp `kubectl top` + HPA (chỉ metric ngắn hạn).
- **kube-prometheus-stack** (Helm) — bộ chuẩn: Prometheus Operator + Prometheus + Alertmanager + Grafana + node-exporter + **kube-state-metrics**.
- Phân biệt: **kube-state-metrics** = *trạng thái object* (replica, pod phase, restart count); **metrics-server** = *CPU/mem sống*.
- **Log:** Loki (index theo label, rẻ, hợp Grafana) vs EFK/ELK (full-text, nặng/đắt hơn). Collector: Fluent Bit / Promtail / Alloy.
- **Trace:** OpenTelemetry → Tempo/Jaeger. Đặt **OTel Collector** trước mọi thứ để trung lập vendor.
- **Events hết hạn ~1h** → ship ra ngoài (event-exporter) để không mất dấu eviction/OOMKill/FailedScheduling:
```bash
kubectl get events -A --sort-by=.lastTimestamp --field-selector type=Warning
```

---

## Phần 12 — Anti-patterns cần tránh

| Anti-pattern | Hậu quả |
|--------------|---------|
| Không set requests/limits | Scheduling kém, noisy neighbor, OOM |
| Dùng tag `:latest` | Deploy không tái lập, không rollback được |
| Liveness probe check dependency ngoài | **Bão restart dây chuyền** toàn cluster |
| Chạy root / privileged / không securityContext | Nguy cơ leo thang & host escape |
| Lưu state trong pod / emptyDir | Mất dữ liệu → dùng PV/PVC + StatefulSet |
| Coi Secret là đã mã hóa | Chỉ base64 → bật encryption-at-rest + secret manager |
| Nhồi nhiều concern vào 1 container | 1 lỗi giết tất cả; sai mô hình scaling |
| Trộn prod & non-prod chung cluster | Resource starvation & blast radius |
| Single replica gọi là "HA" | Không HA thật; thêm PDB + topology spread |
| PDB `minAvailable == replicas` | Chặn vĩnh viễn drain/upgrade |
| HPA + VPA tranh nhau cùng metric | Dao động |
| Phớt lờ SIGTERM | Rớt kết nối mỗi lần deploy/scale-down |
| CPU limit trên app nhạy latency | Throttle → p99 spike |
| `kubectl apply` tay trên prod | Config drift → dùng GitOps làm nguồn chân lý |

---

## Phần 13 — Bảng tra cứu nhanh triệu chứng → nguyên nhân

| Triệu chứng | Nghi ngờ đầu tiên | Lệnh chốt |
|-------------|-------------------|-----------|
| `CrashLoopBackOff` | Bug app / probe / OOM | `kubectl logs --previous`; đọc exit code |
| Exit code `137` | OOMKilled hoặc liveness giết | `describe` → Reason; `dmesg \| grep oom` |
| `ImagePullBackOff` | Sai tag / thiếu secret | `describe` → đọc message Events |
| `Pending` | Thiếu tài nguyên / taint / PVC | `describe pod \| grep -A20 Events` |
| `ContainerCreating` kẹt | CNI / secret thiếu / volume kẹt | `describe` → FailedMount/SandBox |
| `Evicted` | Node disk/mem/inode pressure | `describe node \| grep pressure`; `df -i` |
| Service không tới | Endpoints rỗng / targetPort sai | `kubectl get endpoints <svc>` |
| DNS resolve fail | CoreDNS down / RBAC | `logs -n kube-system -l k8s-app=kube-dns` |
| DNS chậm đúng 5s | conntrack race | Triển khai NodeLocal DNSCache |
| Ingress 502/503 | Backend/targetPort/endpoint | log ingress controller |
| Kết nối timeout giữa 2 pod | NetworkPolicy chặn | `kubectl get netpol -n <ns>` |
| Node `NotReady` | kubelet / pressure / CNI | `journalctl -u kubelet`; `describe node` |
| Latency spike, CPU thấp | CPU throttling | `cat .../cpu.stat` → nr_throttled |
| HPA `<unknown>` | metrics-server / thiếu requests | `kubectl top pods`; `describe hpa` |
| PVC `Pending` | WaitForFirstConsumer / CSI | `describe pvc`; `get storageclass` |
| Pod kẹt `Terminating` | Node chết / finalizer | `--grace-period=0 --force`; check finalizers |
| Namespace kẹt `Terminating` | APIService/CR orphan | tìm object sót; gỡ finalizer |
| `x509: certificate expired` | kubeadm cert hết hạn | `kubeadm certs check-expiration` + renew + **restart** |
| apiserver chậm toàn cluster | etcd disk/fsync | `etcdctl endpoint status`; defrag + SSD |

---

> 📚 **Nguồn tham khảo:** Kubernetes.io docs, Komodor, Sysdig, Datadog, groundcover, ScaleOps, kubernetes/kubernetes issue #56903 (5s DNS race), CNCF blog, Falco/Trivy docs, AWS EKS Best Practices, Teleport & Sedai (anti-patterns). Tài liệu này tổng hợp và Việt hóa từ nhiều nguồn — với sự cố production, luôn đối chiếu doc chính thức của phiên bản k8s bạn đang chạy.
