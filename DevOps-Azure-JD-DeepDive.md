# 🚀 DevOps / SRE trên Azure (Hybrid) — Deep Dive theo JD

> Tài liệu này **mổ xẻ toàn bộ công nghệ trong JD DevOps Engineer (Azure, hybrid on-prem + cloud)**, giải thích sâu từng công nghệ, use case thực tế, và **khi có nhiều công cụ giải quyết cùng 1 bài toán thì best-practice nên chọn cái nào**.
>
> Đối tượng: bạn **rất mạnh on-prem** nhưng **ít kinh nghiệm cloud** → nên tài liệu luôn có phần **"ánh xạ tư duy on-prem → cloud"** để bạn bắc cầu từ cái đã biết sang cái mới.
>
> Cách học tốt nhất: đọc **Phần 1 (bức tranh lớn - big picture)** trước để có khung, rồi đào sâu từng phần. Mọi thứ đều bám vào **1 bài toán xuyên suốt: xây dựng nền tảng cho công ty "ShopVN"**.

---

## 📑 Mục lục

- [Phần 0 — Bóc tách công nghệ từ JD](#phần-0--bóc-tách-công-nghệ-từ-jd)
- [Phần 1 — Bài toán xuyên suốt "ShopVN" + Big Picture](#phần-1--bài-toán-xuyên-suốt-shopvn--big-picture)
- [Phần 2 — Ánh xạ tư duy On-Prem → Cloud (dành riêng cho bạn)](#phần-2--ánh-xạ-tư-duy-on-prem--cloud)
- [Phần 3 — CI/CD: Azure DevOps vs Jenkins vs GitHub Actions](#phần-3--cicd-azure-devops-vs-jenkins-vs-github-actions)
- [Phần 4 — IaC: Terraform vs Bicep vs ARM vs CloudFormation (+ Ansible ở đâu?)](#phần-4--iac-terraform-vs-bicep-vs-arm-vs-cloudformation)
- [Phần 5 — Container & Orchestration: Docker, Kubernetes/AKS, VMSS](#phần-5--container--orchestration-docker-kubernetesaks-vmss)
- [Phần 6 — Azure Networking: VNet, NSG, VPN, Private Endpoint](#phần-6--azure-networking-vnet-nsg-vpn-private-endpoint)
- [Phần 7 — Observability: OpenTelemetry, Prometheus, Grafana, ELK, Azure Monitor](#phần-7--observability-opentelemetry-prometheus-grafana-elk-azure-monitor)
- [Phần 8 — SLI / SLO / Error Budget](#phần-8--sli--slo--error-budget)
- [Phần 9 — DevSecOps: quét bảo mật trong pipeline](#phần-9--devsecops-quét-bảo-mật-trong-pipeline)
- [Phần 10 — Disaster Recovery & Backup](#phần-10--disaster-recovery--backup)
- [Phần 11 — FinOps: tối ưu chi phí](#phần-11--finops-tối-ưu-chi-phí)
- [Phần 12 — Config Management & Version Control (Ansible, Git)](#phần-12--config-management--version-control)
- [Phần 13 — Bảng quyết định nhanh "chọn công cụ nào"](#phần-13--bảng-quyết-định-nhanh)
- [Phần 14 — Lộ trình học + ánh xạ chứng chỉ AZ-104/305/400](#phần-14--lộ-trình-học--chứng-chỉ)

---

## Phần 0 — Bóc tách công nghệ từ JD

Đọc kỹ JD, mình gom nhóm toàn bộ công nghệ/khái niệm thành **9 domain**. Đây chính là "bản đồ" của cả tài liệu:

| # | Domain | Công nghệ trong JD | Bản chất bài toán |
|---|--------|--------------------|-------------------|
| 1 | **CI/CD** | Azure DevOps, Jenkins, GitHub Actions, Bitbucket | Tự động build → test → deploy |
| 2 | **IaC** | Terraform, CloudFormation, ARM templates, Bash | Hạ tầng khai báo bằng code |
| 3 | **Config Mgmt** | Ansible, Software Configuration Management, version control | Cấu hình đồng nhất trên nhiều máy |
| 4 | **Container/Orchestration** | Docker, Kubernetes, microservices, VMSS | Đóng gói & chạy app quy mô lớn |
| 5 | **Azure Networking** | VNet, NSG, VPN, Private Endpoint | Kết nối & cô lập mạng (hybrid) |
| 6 | **Observability** | OpenTelemetry, Grafana, Prometheus, ELK, logs/metrics/traces | Nhìn thấy hệ thống đang làm gì |
| 7 | **Reliability (SRE)** | SLI/SLO, incident response, performance tuning | Đo & cam kết độ tin cậy |
| 8 | **Security/Compliance** | Security scanning, dependency checks, compliance | Bảo mật nhúng vào pipeline (DevSecOps) |
| 9 | **DR / Backup / FinOps** | Disaster recovery, backup, cost optimization | Chịu lỗi & tối ưu tiền |

> 💡 **Nhận xét về JD**: đây là JD **DevOps thiên Azure + hybrid + observability nặng (OpenTelemetry)**. Ba từ khóa "đắt giá" nhất — cũng là thứ phân biệt ứng viên senior — là: **(1) hybrid on-prem↔cloud**, **(2) OpenTelemetry end-to-end**, **(3) SLI/SLO**. Đây đúng là điểm bạn cần đầu tư vì bạn mạnh on-prem sẵn.
>
> Chứng chỉ JD nhắc: **AZ-104** (Azure Admin) → **AZ-305** (Solutions Architect) → **AZ-400** (DevOps Expert). Xem [Phần 14](#phần-14--lộ-trình-học--chứng-chỉ).

---

## Phần 1 — Bài toán xuyên suốt "ShopVN" + Big Picture

Để thấy **bức tranh lớn**, ta không học rời rạc. Ta xây dựng **một hệ thống thật** và mọi công nghệ trong JD sẽ "gắn" vào đúng chỗ của nó.

### 🎯 Đề bài — Công ty "ShopVN"

> ShopVN là sàn thương mại điện tử. Hiện tại họ chạy **hoàn toàn on-prem** (VMware, vài con vật lý, Oracle DB, deploy tay bằng script). Vấn đề:
> - Mỗi lần sale lớn (12.12) hệ thống **sập vì không scale kịp**.
> - Deploy tay → hay lỗi, downtime, ban đêm mới dám release.
> - Không ai biết hệ thống "khỏe hay yếu" cho tới khi khách hàng la.
> - Sếp muốn **lên Azure** nhưng **kho hàng (warehouse/ERP) phải giữ on-prem** vì lý do pháp lý → **bắt buộc hybrid**.
>
> **Nhiệm vụ của bạn (DevOps):** thiết kế nền tảng để: deploy tự động nhiều lần/ngày, tự scale khi tải cao, quan sát được toàn hệ thống, bảo mật, chịu được thảm họa, và **không đốt tiền vô tội vạ**.

### 🗺️ Bức tranh lớn — kiến trúc mục tiêu (ASCII)

```
                                 INTERNET (khách hàng)
                                        │
                                        ▼
                          ┌──────────────────────────┐
                          │   Azure Front Door / WAF  │  ← CDN + chống DDoS + WAF
                          └────────────┬─────────────┘
                                        │
        ┌───────────────────────────────────────────────────────────────┐
        │                     AZURE (Cloud - Public)                     │
        │                                                                 │
        │   VNet: 10.10.0.0/16                                            │
        │   ┌──────────────────────┐   ┌────────────────────────────┐    │
        │   │ Subnet: AKS (app)    │   │ Subnet: data / private     │    │
        │   │ 10.10.1.0/24         │   │ 10.10.2.0/24               │    │
        │   │                      │   │                            │    │
        │   │  ┌────────────────┐  │   │  ┌──────────────────────┐  │    │
        │   │  │  AKS Cluster   │  │   │  │ Azure SQL / PostgreSQL│ │    │
        │   │  │ ┌────┐ ┌────┐  │  │   │  │  (Private Endpoint)   │ │    │
        │   │  │ │pod │ │pod │  │◄─┼───┼──┤ Azure Cache (Redis)   │ │    │
        │   │  │ └────┘ └────┘  │  │   │  │ Storage / Blob        │ │    │
        │   │  │ cart  payment  │  │   │  └──────────────────────┘  │    │
        │   │  │ catalog ...    │  │   │       ▲ NSG chặn public     │    │
        │   │  └───────┬────────┘  │   └───────┼────────────────────┘    │
        │   └──────────┼───────────┘           │                         │
        │              │ OTel traces/metrics/logs                        │
        │              ▼                                                  │
        │   ┌─────────────────────────────────────────┐                  │
        │   │   Observability stack                    │                  │
        │   │   Prometheus(metrics) + Loki/ELK(logs)   │                  │
        │   │   + Tempo/Jaeger(traces) → Grafana        │                  │
        │   │   (song song: Azure Monitor / App Insights)│                 │
        │   └─────────────────────────────────────────┘                  │
        │                                                                 │
        └───────────────────────────────┬─────────────────────────────────┘
                                         │  Site-to-Site VPN / ExpressRoute
                                         │  (kênh riêng, mã hóa)
        ┌────────────────────────────────┼────────────────────────────────┐
        │                    ON-PREM (Data center của ShopVN)              │
        │   ┌──────────────────┐   ┌──────────────────┐                    │
        │   │ ERP / Warehouse  │   │ Oracle DB (legacy)│                   │
        │   │ (giữ lại - luật) │   │                   │                   │
        │   └──────────────────┘   └──────────────────┘                    │
        │   Ansible quản cấu hình VM on-prem + Prometheus node_exporter    │
        └──────────────────────────────────────────────────────────────────┘


   ═══════════ VÒNG ĐỜI CODE (chạy song song với hạ tầng trên) ═══════════

   Dev push code
        │
        ▼
   ┌─────────────┐   ┌──────────────────────────────────────────────┐
   │ Git (repo)  │──►│  CI/CD Pipeline (Azure DevOps / GitHub Actions)│
   │ Bitbucket/  │   │  1. Build  2. Unit test                       │
   │ GitHub      │   │  3. SAST + SCA + secret scan (DevSecOps)      │
   └─────────────┘   │  4. Build Docker image → scan image (Trivy)   │
                     │  5. Push image → Azure Container Registry     │
                     │  6. Terraform plan/apply (hạ tầng)            │
                     │  7. Deploy lên AKS (Helm/manifest)            │
                     │  8. Smoke test → promote                     │
                     └──────────────────────────────────────────────┘
```

### 🧩 "Ai làm gì" — bảng ánh xạ công nghệ vào bài toán ShopVN

| Bài toán ShopVN | Công nghệ giải quyết | Phần chi tiết |
|-----------------|----------------------|---------------|
| Deploy tự động, nhiều lần/ngày | CI/CD (Azure DevOps / GitHub Actions) | [P3](#phần-3--cicd-azure-devops-vs-jenkins-vs-github-actions) |
| Hạ tầng tái lập, không "bấm tay" | Terraform / Bicep | [P4](#phần-4--iac-terraform-vs-bicep-vs-arm-vs-cloudformation) |
| Cấu hình VM on-prem đồng nhất | Ansible | [P12](#phần-12--config-management--version-control) |
| App scale theo tải, chạy nhiều bản | Docker + AKS (K8s), VMSS | [P5](#phần-5--container--orchestration-docker-kubernetesaks-vmss) |
| Nối cloud ↔ on-prem an toàn | VPN/ExpressRoute + Private Endpoint + NSG | [P6](#phần-6--azure-networking-vnet-nsg-vpn-private-endpoint) |
| Nhìn thấy hệ thống end-to-end | OpenTelemetry + Prometheus + Grafana + ELK | [P7](#phần-7--observability-opentelemetry-prometheus-grafana-elk-azure-monitor) |
| Cam kết "uptime 99.9%" | SLI/SLO + error budget | [P8](#phần-8--sli--slo--error-budget) |
| Không để lỗ hổng lọt production | SAST/SCA/secret/image scan | [P9](#phần-9--devsecops-quét-bảo-mật-trong-pipeline) |
| Sập 1 vùng vẫn sống | DR multi-region + backup | [P10](#phần-10--disaster-recovery--backup) |
| Không đốt tiền cloud | FinOps | [P11](#phần-11--finops-tối-ưu-chi-phí) |

Giờ đào sâu từng phần. Mỗi phần theo cấu trúc: **Bản chất → Deep dive → Use case ShopVN → Nếu nhiều tool thì chọn cái nào (best practice) → Bẫy thường gặp.**

---

## Phần 2 — Ánh xạ tư duy On-Prem → Cloud

Bạn mạnh on-prem, nên đây là **cây cầu quan trọng nhất**. Cloud không phải "phép màu" — nó là **on-prem được API hóa và tính tiền theo giờ**. Mọi thứ bạn đã biết đều có bản sao trên cloud:

```
   ON-PREM (bạn đã biết)              AZURE (khái niệm tương đương)
   ─────────────────────             ────────────────────────────
   Máy vật lý / VMware VM       →     Azure VM
   Nhiều VM giống nhau + LB     →     VM Scale Set (VMSS)
   Trung tâm dữ liệu (site)     →     Region  (vd: Southeast Asia)
   Phòng máy / rack riêng       →     Availability Zone (AZ)
   Switch/VLAN                  →     VNet + Subnet
   Firewall / ACL              →     NSG (Network Security Group)
   Load balancer (F5)          →     Azure Load Balancer / App Gateway / Front Door
   SAN / NAS                   →     Managed Disk / Azure Files / Blob Storage
   Oracle/MSSQL tự quản        →     Azure SQL (PaaS - Microsoft lo patch/backup)
   Đường leased line 2 site     →     VPN Gateway (S2S) hoặc ExpressRoute
   Active Directory             →     Entra ID (Azure AD)
   Cron + script deploy         →     CI/CD pipeline
   "Cài tay theo checklist"     →     IaC (Terraform/Bicep) + Ansible
   Nagios/Zabbix                →     Prometheus + Grafana / Azure Monitor
   Băng từ backup               →     Azure Backup + GRS storage
```

### 🔑 3 thay đổi tư duy lớn nhất khi lên cloud

1. **Từ "vật nuôi" sang "gia súc" (Pets → Cattle).**
   On-prem: server `db-prod-01` là "thú cưng", hỏng thì bạn *chữa*. Cloud: server là "gia súc" — hỏng thì *giết đi tạo con mới* (immutable). Đây là lý do IaC + container ra đời. Đừng SSH vào sửa tay production trên cloud.

2. **Từ CAPEX sang OPEX (mua đứt → thuê theo giờ).**
   On-prem mua server 1 lần dùng 5 năm → tâm lý "để chạy cho đáng tiền". Cloud tính theo giây → **bật lúc cần, tắt lúc không cần** = tiết kiệm. Đây là gốc rễ của FinOps ([P11](#phần-11--finops-tối-ưu-chi-phí)).

3. **Từ "tự lo mọi thứ" sang Shared Responsibility (trách nhiệm chia sẻ).**
   ```
   Bạn quản          │ IaaS(VM) │ PaaS(App/AKS) │ SaaS │
   ──────────────────┼──────────┼───────────────┼──────┤
   Data/App          │   BẠN    │     BẠN       │ BẠN  │
   OS / patch        │   BẠN    │  Microsoft*   │  MS  │
   Phần cứng/mạng vật lý│  MS   │     MS        │  MS  │
   ```
   Chọn PaaS (vd Azure SQL thay vì tự cài Oracle trên VM) = **đẩy việc patch/backup/HA cho Microsoft** → bạn tập trung vào app. Best practice hybrid: **ưu tiên PaaS trên cloud, chỉ giữ IaaS/on-prem khi bắt buộc.**

---

## Phần 3 — CI/CD: Azure DevOps vs Jenkins vs GitHub Actions

### Bản chất
CI/CD = tự động hóa đường đi của code: **Continuous Integration** (mỗi commit tự build + test để phát hiện lỗi sớm) và **Continuous Delivery/Deployment** (tự đưa lên môi trường). Thay cho việc bạn on-prem hay làm: "viết bash + cron + deploy tay ban đêm".

### Deep dive — 3 công cụ trong JD

```
   JENKINS                    AZURE DEVOPS PIPELINES        GITHUB ACTIONS
   ───────                    ─────────────────────         ──────────────
   Tự host (bạn quản          SaaS của Microsoft,           SaaS gắn liền repo
   server master+agent)       tích hợp Boards/Repos/        GitHub, workflow
   Plugin cực nhiều           Artifacts                     YAML trong repo
   (cũng là điểm yếu:         YAML pipeline                 Marketplace actions
   bảo trì, xung đột)         Self/MS-hosted agents         Runner MS/self-host
   Groovy pipeline            Tích hợp Azure sâu nhất        Cộng đồng lớn nhất
```

| Tiêu chí | Jenkins | Azure DevOps | GitHub Actions |
|----------|---------|--------------|----------------|
| Kiểu | Self-hosted | SaaS (host self được) | SaaS |
| Bảo trì | **Bạn tự lo** (nặng) | Microsoft lo | Microsoft lo |
| Tích hợp Azure | Qua plugin | **Native, sâu nhất** | Tốt (OIDC) |
| Hybrid/on-prem | **Mạnh nhất** (agent chạy đâu cũng được) | Self-hosted agent | Self-hosted runner |
| Hệ sinh thái | Plugin khổng lồ (rủi ro bảo mật) | Trọn bộ ALM | Marketplace + cộng đồng |
| Đường cong học | Dốc (Groovy) | Vừa | Dễ nhất |
| Chi phí | "Miễn phí" nhưng tốn người vận hành | Theo user/parallel job | Theo phút chạy |

### 🎯 Use case ShopVN
- Code microservice `payment` push lên Git → pipeline: build → test → quét bảo mật → build image → push ACR → deploy AKS staging → smoke test → chờ approve → deploy production.
- Có job on-prem (deploy ERP) → cần **self-hosted agent** đặt trong data center để chạm được máy nội bộ.

### ✅ Best practice — nhiều tool thì chọn cái nào?

> **Quy tắc vàng: chọn CI/CD theo "code của bạn đang ở đâu" và "team đã ở đâu", đừng chọn theo tính năng.**

- **Code trên GitHub + team nhỏ/vừa, cloud-native → GitHub Actions.** Đơn giản, không phải nuôi server, tích hợp OIDC với Azure (không cần lưu secret dài hạn). Đây là xu hướng mặc định 2025-2026 cho dự án mới.
- **Đã dùng nguyên bộ Azure DevOps (Boards + Repos + Artifacts), tổ chức lớn cần governance/audit chặt → Azure DevOps Pipelines.** Tích hợp Azure và quản trị doanh nghiệp mạnh nhất. Rất hợp JD này vì JD "Azure-centric".
- **Có nhu cầu on-prem nặng, pipeline phức tạp cũ, hoặc air-gapped → Jenkins.** Nhưng **đừng chọn Jenkins cho dự án mới chỉ vì "quen"** — chi phí vận hành master/agent/plugin rất lớn. Xu hướng là **migrate Jenkins → GitHub Actions/Azure DevOps**, giữ Jenkins cho phần legacy.

**Cho ShopVN (Azure hybrid):** khuyến nghị **Azure DevOps hoặc GitHub Actions làm chính** (SaaS, đỡ nuôi hạ tầng), cắm thêm **self-hosted agent/runner trong on-prem** cho các job cần chạm ERP. Không dựng Jenkins mới.

### ⚠️ Bẫy thường gặp
- Cắm **secret cứng** (password, key) vào pipeline → dùng **Key Vault + OIDC/Workload Identity** thay vì secret dài hạn.
- Pipeline "build được là xong" → thiếu bước **quét bảo mật** ([P9](#phần-9--devsecops-quét-bảo-mật-trong-pipeline)) và **rollback**.
- Một pipeline khổng lồ → nên tách CI (build/test) và CD (deploy) rõ ràng, môi trường staging → prod có **cổng phê duyệt (approval gate)**.

---

## Phần 4 — IaC: Terraform vs Bicep vs ARM vs CloudFormation

### Bản chất
**Infrastructure as Code**: mô tả hạ tầng (VNet, VM, AKS, DB...) bằng **file khai báo** thay vì bấm chuột trên Portal. Lợi ích: tái lập được, review qua Git, không "drift" (lệch cấu hình), tạo lại môi trường trong vài phút.

**2 trường phái:**
- **Declarative (khai báo "tôi muốn kết quả gì")**: Terraform, Bicep, ARM, CloudFormation. Bạn nói *what*, tool tự lo *how*.
- **Imperative/procedural (mô tả "làm từng bước")**: Bash script, một phần Ansible. Bạn nói *how*.

### Deep dive — so sánh 4 công cụ khai báo

```
   TERRAFORM              BICEP               ARM templates        CloudFormation
   /OpenTofu              (Azure-native)      (Azure gốc)          (AWS-only)
   ─────────              ─────────────       ─────────────        ──────────────
   Đa cloud (Azure,       Chỉ Azure           Chỉ Azure            Chỉ AWS
   AWS, GCP, on-prem,     DSL gọn, biên       JSON dài dòng,       JSON/YAML
   SaaS...)               dịch ra ARM         "assembly" của       Native AWS
   HCL - dễ đọc           Microsoft đẩy       Bicep - ít viết tay  → KHÔNG dùng
   Có state file          mạnh, không cần     nữa                  cho Azure
   Provider khổng lồ      state riêng
```

| Tiêu chí | Terraform / OpenTofu | Bicep | ARM (JSON) | CloudFormation |
|----------|----------------------|-------|------------|----------------|
| Phạm vi cloud | **Đa cloud** | Chỉ Azure | Chỉ Azure | Chỉ AWS |
| Ngôn ngữ | HCL (dễ) | DSL gọn (dễ) | JSON (rối) | JSON/YAML |
| State | **File state** (phải quản) | Không cần (Azure lo) | Không cần | Không cần |
| Hợp với JD? | ✅ hybrid/đa cloud | ✅ thuần Azure | ⚠️ chỉ khi buộc | ❌ (AWS) |
| Cộng đồng/module | Rất lớn (Registry) | Đang lớn nhanh | Ít | Lớn (trong AWS) |

> 📝 **Lưu ý pháp lý quan trọng (2023→nay):** HashiCorp đổi Terraform sang giấy phép **BSL** (không còn open-source thuần). Cộng đồng fork ra **OpenTofu** (thuộc Linux Foundation, open-source, tương thích Terraform). 2025-2026 nhiều tổ chức chọn **OpenTofu** để tránh rủi ro license. Về mặt kỹ thuật gần như giống hệt Terraform.

### 🧭 Ansible nằm ở đâu? (Provisioning vs Configuration)

Đây là chỗ **rất hay nhầm**. Phân biệt 2 việc khác nhau:

```
   PROVISIONING (tạo hạ tầng)          CONFIGURATION (cấu hình bên trong)
   ────────────────────────           ──────────────────────────────────
   "Tạo 5 VM, 1 VNet, 1 LB"           "Trên mỗi VM: cài nginx, mở port,
   → Terraform / Bicep                 copy config, tạo user, patch OS"
                                       → Ansible
   Ví như XÂY NHÀ                     Ví như DỌN NỘI THẤT trong nhà
```

- **Terraform/Bicep**: giỏi *tạo* tài nguyên cloud (declarative, idempotent, có state).
- **Ansible**: giỏi *cấu hình* bên trong OS (agentless, chạy qua SSH/WinRM), quản cả **on-prem lẫn cloud**, không cần agent.
- **Bash**: keo dán — glue script cho việc nhỏ, một lần, hoặc bootstrap.

### ✅ Best practice — chọn cái nào?

> **Quy tắc: Terraform (hoặc OpenTofu) để TẠO hạ tầng; Ansible để CẤU HÌNH OS; đừng bắt tool làm việc của tool kia.**

- **Thuần Azure, team .NET/Microsoft, muốn không quản state → Bicep.** Microsoft support ngày phát hành dịch vụ mới (day-0), gọn hơn ARM rất nhiều.
- **Hybrid hoặc đa cloud, hoặc muốn 1 ngôn ngữ IaC cho tất cả (Azure + on-prem VMware + SaaS như GitHub/Datadog) → Terraform/OpenTofu.** Đây là lựa chọn hợp JD "hybrid" nhất.
- **ARM JSON**: chỉ đụng khi buộc (một tính năng chưa có trong Bicep) — không viết mới bằng tay.
- **CloudFormation**: chỉ dùng nếu bạn ở AWS. Với JD Azure này → **không dùng**.
- **Ansible cho on-prem của ShopVN** (cấu hình ERP VM, patch OS đồng loạt) là hoàn hảo vì agentless, hợp môi trường on-prem bạn đã quen.

**Combo thực chiến ShopVN:** `Terraform tạo AKS + VNet + SQL` → `Ansible cấu hình các VM on-prem + node đặc biệt` → `Bash` cho vài script bootstrap nhỏ. Với phần thuần Azure có thể thay Terraform bằng Bicep nếu team ngại quản state.

### ⚠️ Bẫy thường gặp
- **State file để lung tung / commit vào Git** → phải để **remote backend** (Azure Storage + state locking) và **không** commit (chứa secret).
- **Sửa tay trên Portal → drift** với code. Luôn sửa qua code, chạy `terraform plan` để phát hiện drift.
- Trộn provisioning và config vào 1 script bash 500 dòng → khó bảo trì. Tách đúng vai trò.

---

## Phần 5 — Container & Orchestration: Docker, Kubernetes/AKS, VMSS

### Bản chất
- **Docker (container)**: đóng gói app + toàn bộ dependency vào 1 "hộp" chạy giống nhau ở mọi nơi → hết cảnh "máy tôi chạy được mà server thì không".
- **Kubernetes (K8s)**: "hệ điều hành của data center" — điều phối hàng trăm container: tự khởi động lại khi chết, tự scale, rolling update, service discovery, load balancing nội bộ.
- **AKS (Azure Kubernetes Service)**: K8s do Azure quản control plane (bạn không phải tự vá master).
- **VMSS (VM Scale Set)**: nhóm nhiều VM giống hệt nhau, tự tăng/giảm số lượng theo tải + đặt sau load balancer.

### Deep dive — Container vs VM vs VMSS vs K8s

```
   VM đơn         VMSS                  Kubernetes / AKS
   ──────         ────                  ────────────────
   1 máy          N máy giống nhau      Điều phối CONTAINER trên nhiều node
   Scale tay      auto-scale VM         auto-scale POD + node, self-heal,
                  theo CPU/metric       rolling update, service mesh...
   ┌───┐          ┌───┐┌───┐┌───┐       ┌─────────────────────────┐
   │app│          │app││app││app│       │ node1   node2   node3   │
   └───┘          └───┘└───┘└───┘       │ ┌─┐┌─┐  ┌─┐┌─┐  ┌─┐     │
                     ▲                  │ └─┘└─┘  └─┘└─┘  └─┘     │
                  Load Balancer         │ pods tự dời khi node chết │
                                        └─────────────────────────┘
   Đơn giản       Vẫn là "1 app/VM"     Nhiều microservice, mật độ cao
```

### 🎯 Use case ShopVN
- App gồm nhiều microservice (`cart`, `catalog`, `payment`, `search`) → đóng Docker, chạy trên **AKS**. Ngày sale 12.12, HPA (Horizontal Pod Autoscaler) tự tăng pod `payment` từ 3 → 30, Cluster Autoscaler thêm node.
- Một dịch vụ **batch xử lý ảnh** không cần K8s → có thể chạy trên **VMSS** (đơn giản hơn).

### ✅ Best practice — khi nào K8s/AKS, khi nào VMSS, khi nào đừng dùng K8s?

> **Đừng dùng Kubernetes chỉ vì "thời thượng". K8s trả giá bằng độ phức tạp vận hành.**

- **Nhiều microservice, cần scale/self-heal/rolling update tinh vi, team đủ năng lực → AKS.** Managed control plane free (chỉ trả tiền node), tích hợp Entra ID, ACR, Azure Monitor.
- **App đơn khối (monolith) hoặc ít service, chỉ cần "nhân bản + auto scale" → VMSS** hoặc **Azure App Service / Container Apps**. Đơn giản hơn K8s nhiều.
- **Chỉ 1-2 container, không muốn quản K8s → Azure Container Apps** (serverless K8s, scale-to-zero) hoặc **App Service**. Đây thường là lựa chọn đúng bị bỏ quên vì người ta nhảy thẳng vào AKS.
- **Self-managed K8s (tự dựng bằng kubeadm)**: chỉ khi air-gapped/on-prem đặc thù. Trên Azure → luôn ưu tiên **AKS** (đỡ vá control plane).

**Thang leo độ phức tạp (chọn bậc thấp nhất đủ dùng):**
```
   App Service / Container Apps  <  VMSS  <  AKS  <  self-managed K8s
   (đơn giản, ít quyền kiểm soát) ──────────────► (mạnh, phức tạp, tốn người)
```

### ⚠️ Bẫy thường gặp
- Nhảy vào AKS khi chỉ có 2 service → over-engineering. Bắt đầu từ Container Apps/App Service.
- Không đặt **resource requests/limits** cho pod → 1 service ngốn hết RAM làm sập node.
- Quên **liveness/readiness probe** → K8s không biết pod chết, hoặc gửi traffic vào pod chưa sẵn sàng.

---

## Phần 6 — Azure Networking: VNet, NSG, VPN, Private Endpoint

### Bản chất (ánh xạ on-prem)
```
   VNet          = mạng LAN ảo của bạn trên cloud (dải IP riêng)
   Subnet        = chia VLAN trong VNet
   NSG           = firewall/ACL gắn ở subnet hoặc NIC (allow/deny theo IP+port)
   VPN Gateway   = "đường hầm" mã hóa nối on-prem ↔ Azure qua Internet (S2S)
   ExpressRoute  = leased line riêng (không qua Internet) - nhanh, ổn định, đắt
   Private Endpoint = kéo dịch vụ PaaS (SQL, Storage) vào VNet với IP private
                      → không đi ra Internet công cộng
```

### Deep dive — bức tranh mạng hybrid ShopVN

```
   ON-PREM 192.168.0.0/16                    AZURE VNet 10.10.0.0/16
   ┌────────────────────┐                    ┌───────────────────────────────┐
   │  ERP, Oracle DB    │                    │ ┌──────────┐   ┌────────────┐  │
   │                    │   S2S VPN /        │ │AKS subnet│   │data subnet │  │
   │  ┌──────────────┐  │◄══ ExpressRoute ══►│ │10.10.1/24│   │10.10.2/24  │  │
   │  │ VPN device    │  │   (mã hóa IPsec)   │ └──────────┘   └─────┬──────┘  │
   │  └──────────────┘  │                    │        ▲              │         │
   └────────────────────┘                    │        │      Private Endpoint  │
                                             │        │      ┌──────────────┐   │
                                             │    NSG chặn ──►│ Azure SQL    │   │
                                             │    port lạ    │ (IP private) │   │
                                             │               └──────────────┘   │
                                             └───────────────────────────────┘
```

### 🎯 Use case ShopVN
- App trên AKS cần đọc dữ liệu tồn kho từ **Oracle on-prem** → đi qua **S2S VPN** (hoặc ExpressRoute nếu cần băng thông/độ trễ ổn định).
- **Azure SQL không được phép lộ ra Internet** → dùng **Private Endpoint**: SQL có IP trong subnet data, chỉ app trong VNet truy cập được. **NSG** chặn mọi port ngoài luồng cho phép.

### ✅ Best practice — VPN vs ExpressRoute, NSG vs Firewall?
- **Nối hybrid băng thông vừa, ngân sách hạn chế → Site-to-Site VPN.** Nhanh triển khai, đi qua Internet nhưng mã hóa IPsec.
- **Cần độ trễ thấp/ổn định/băng thông lớn/tuân thủ (không qua Internet) → ExpressRoute.** Đắt hơn, dùng cho tải nặng, DB replication, hệ thống tài chính.
- **Best practice bảo mật PaaS: luôn ưu tiên Private Endpoint** thay vì để dịch vụ mở public + firewall IP. "Private by default".
- **NSG** cho lọc L3/L4 cơ bản (đủ cho hầu hết case); **Azure Firewall** khi cần lọc L7, FQDN filtering, threat intel tập trung. Đừng mua Azure Firewall nếu NSG đủ dùng (FinOps).

### ⚠️ Bẫy
- Mở NSG `0.0.0.0/0` cho port 22/3389 → cả Internet SSH được. Chỉ mở qua **Bastion** hoặc IP whitelist.
- Trùng dải IP giữa on-prem và VNet → VPN không route được. Quy hoạch IP từ đầu.

---

## Phần 7 — Observability: OpenTelemetry, Prometheus, Grafana, ELK, Azure Monitor

Đây là **phần đắt giá nhất của JD** ("Implement end-to-end observability using OpenTelemetry"). Nắm chắc phần này bạn hơn rất nhiều ứng viên.

### Bản chất — Monitoring vs Observability, và "3 trụ cột"
- **Monitoring** (cũ, kiểu Nagios/Zabbix bạn quen): "CPU > 80% thì báo" — trả lời câu hỏi **đã biết trước**.
- **Observability** (mới): thu đủ dữ liệu để trả lời **câu hỏi chưa biết trước** ("tại sao đơn hàng của khách A chậm 3s lúc 2h chiều?").

**3 trụ cột (three pillars):**
```
   METRICS (số liệu)        LOGS (nhật ký)          TRACES (dấu vết)
   ────────────────         ──────────────          ────────────────
   "bao nhiêu?"             "chuyện gì xảy ra?"      "đi qua đâu, chậm ở đâu?"
   CPU 80%, 1200 req/s,     "ERROR: null pointer    request → gateway(2ms)
   p99 latency 300ms        at PaymentService..."   → cart(5ms) → payment(2800ms!)
                                                     → DB(50ms)
   → Prometheus             → Loki / ELK             → Jaeger / Tempo
```

### OpenTelemetry (OTel) — "nhân vật chính" của JD

> **Vấn đề nó giải quyết:** trước đây mỗi tool (Datadog, New Relic, AppD) có SDK riêng → code bạn bị **khóa vào 1 vendor**. Đổi tool = viết lại instrumentation.
>
> **OTel = tiêu chuẩn mở, trung lập vendor** để sinh & thu thập metrics/logs/traces. Bạn nhúng OTel SDK **1 lần** vào app, rồi tự do đổi backend (Prometheus, Grafana, Datadog, Azure Monitor...) mà **không sửa code app**. Đây là lý do nó thắng.

```
   APP (nhúng OTel SDK 1 lần)
        │  (traces + metrics + logs, chuẩn OTLP)
        ▼
   ┌──────────────────────────┐
   │  OTel Collector          │  ← trạm trung chuyển: nhận, xử lý, định tuyến
   │  (receive→process→export)│
   └───┬─────────┬────────┬───┘
       │         │        │      Đổi backend chỉ cần đổi "exporter" ở Collector,
       ▼         ▼        ▼      KHÔNG đụng code app.
   Prometheus   Loki    Tempo/Jaeger  (hoặc Azure Monitor, Datadog...)
   (metrics)   (logs)   (traces)
       └─────────┼────────┘
                 ▼
            ┌─────────┐
            │ GRAFANA │  ← 1 màn hình nhìn cả 3 trụ cột, tương quan với nhau
            └─────────┘
```

### Deep dive — ai làm gì trong stack

| Thành phần | Vai trò | Ví như |
|-----------|---------|--------|
| **OpenTelemetry** | Chuẩn + SDK + Collector để *sinh & thu* dữ liệu | "Ổ cắm điện chuẩn quốc tế" |
| **Prometheus** | Lưu & truy vấn **metrics** (pull model, PromQL) | Kho số liệu |
| **Loki / ELK** | Lưu & tìm **logs** | Kho nhật ký |
| **Jaeger / Tempo** | Lưu & xem **traces** phân tán | Bản đồ hành trình request |
| **Grafana** | **Dashboard** hợp nhất + alerting, nhìn cả 3 | Buồng lái (cockpit) |
| **Azure Monitor / App Insights** | Bản "all-in-one" của Azure (metrics+logs+traces+APM) | Combo có sẵn của Azure |

### ✅ Best practice — Prometheus+Grafana+ELK (self-host) HAY Azure Monitor?

> **Quy tắc: instrument bằng OpenTelemetry (trung lập) trước; backend chọn theo bối cảnh hybrid, chi phí, và kỹ năng team.**

- **Metrics → Prometheus** gần như mặc định cho K8s (chuẩn CNCF, PromQL mạnh). Trên Azure có **Azure Managed Prometheus** nếu ngại tự vận hành.
- **Logs → chọn Loki hay ELK?**
  - **Loki**: rẻ, nhẹ, tích hợp Grafana mượt, index theo label (không index full-text) → hợp khi log nhiều, ngân sách hạn chế.
  - **ELK (Elasticsearch)**: tìm kiếm full-text mạnh, phân tích log phức tạp, nhưng **nặng và tốn tài nguyên**. Chọn khi cần search/analytics log sâu.
- **Dashboard/alert → Grafana** (nhìn được cả Prometheus, Loki, Tempo, kể cả Azure Monitor làm data source).
- **Azure Monitor / Application Insights**: mạnh nhất cho **tài nguyên Azure** và APM .NET, tích hợp sẵn không phải dựng gì. Nhưng **chi phí ingest log có thể đắt** ở quy mô lớn, và **không phủ tốt on-prem**.

**Khuyến nghị hybrid cho ShopVN (best of both):**
```
   Instrument mọi thứ bằng OpenTelemetry
        ├─► Azure Monitor/App Insights: cho phần Azure-native + APM nhanh gọn
        └─► Prometheus + Loki + Grafana: cho AKS + ON-PREM (Azure Monitor phủ
            on-prem kém) → 1 Grafana nhìn xuyên cloud + on-prem
```
Lý do: on-prem là điểm mạnh của bạn nhưng là điểm yếu của Azure Monitor → **Prometheus/Grafana** trám chỗ đó, còn Azure Monitor lo phần Azure. OTel làm keo dán để không khóa vendor.

### ⚠️ Bẫy
- Thu **mọi log ở mức DEBUG** → hóa đơn ingest nổ + nhiễu. Lấy mẫu (sampling) traces, chọn log level hợp lý.
- Có metrics nhưng **không có traces** → biết "chậm" nhưng không biết "chậm ở service nào". Traces là thứ phân biệt observability thật.
- Dashboard đẹp nhưng **không ai đọc**; điều quan trọng là **alert dựa trên SLO** ([P8](#phần-8--sli--slo--error-budget)), không phải trên CPU thô.

---

## Phần 8 — SLI / SLO / Error Budget

### Bản chất (khái niệm SRE của Google)
- **SLI (Indicator)**: *thước đo* độ khỏe thật, theo góc nhìn **người dùng**. Vd: tỉ lệ request thành công, p99 latency.
- **SLO (Objective)**: *mục tiêu* cho SLI. Vd: "99.9% request thành công trong 30 ngày".
- **SLA (Agreement)**: *cam kết hợp đồng* với khách (kèm phạt tiền). SLA luôn **lỏng hơn** SLO nội bộ.
- **Error Budget (ngân sách lỗi)**: `100% - SLO`. Với SLO 99.9% → được phép lỗi **0.1%** ≈ 43 phút/tháng.

```
   SLO 99.9% uptime  ⇒  Error budget = 0.1% ≈ 43 phút downtime/tháng
   ┌──────────────────────────────────────────────┐
   │ Error budget tháng: [■■■■■■■■□□]  còn 20%       │
   └──────────────────────────────────────────────┘
         │                                  │
   Còn budget → thoải mái            Hết budget → ĐÓNG BĂNG release
   release feature mới,              tính năng mới, dồn toàn lực
   chạy nhanh                        vào ổn định/độ tin cậy
```

### 🎯 Vì sao error budget hay?
Nó **hòa giải mâu thuẫn muôn thuở** giữa Dev (muốn ship nhanh) và Ops (muốn ổn định): biến "độ tin cậy" thành **ngân sách đo được**. Còn budget thì Dev cứ ship; hết budget thì cả team dừng tính năng, sửa reliability. Khách quan, không cãi nhau cảm tính.

### 🎯 Use case ShopVN
- SLI: `tỉ lệ đơn hàng checkout thành công`, `p99 thời gian tải trang sản phẩm < 500ms`.
- SLO: checkout thành công 99.95%/tháng. → error budget ≈ 21 phút/tháng.
- Alert **theo tốc độ đốt budget (burn rate)**: nếu đang đốt budget nhanh gấp 14 lần bình thường → page người trực ngay, thay vì đợi sập mới báo.

### ✅ Best practice
- **Chọn SLI theo trải nghiệm người dùng**, không theo tài nguyên. "Checkout thành công" tốt hơn "CPU < 80%".
- **Đừng đặt SLO = 100%** — bất khả thi và cực đắt. 99.9% là điểm cân bằng phổ biến.
- Alert dựa trên **multi-window burn rate** (kết hợp cửa sổ ngắn + dài) để vừa nhạy vừa ít báo động giả — thay cho alert CPU thô.

---

## Phần 9 — DevSecOps: quét bảo mật trong pipeline

### Bản chất — "Shift Left"
Bảo mật **dịch sang trái** (làm sớm trong vòng đời) thay vì kiểm tra lúc cuối. Nhúng các bước quét ngay trong CI/CD → bắt lỗ hổng khi còn rẻ để sửa.

### Deep dive — các loại quét (đặt vào pipeline)

```
   commit          build            image             deploy         runtime
     │               │                │                 │              │
     ▼               ▼                ▼                 ▼              ▼
  ┌──────┐      ┌────────┐       ┌─────────┐       ┌────────┐     ┌────────┐
  │secret│      │  SAST  │       │container│       │  IaC   │     │  DAST  │
  │ scan │      │ + SCA  │       │  scan   │       │  scan  │     │(chạy   │
  └──────┘      └────────┘       └─────────┘       └────────┘     │thật)   │
  lộ key?       lỗ hổng          lỗ hổng OS trong  Terraform      └────────┘
                trong code +     base image        sai cấu hình?
                thư viện
```

| Loại | Quét cái gì | Tool phổ biến |
|------|-------------|---------------|
| **Secret scanning** | key/password lỡ commit | Gitleaks, TruffleHog, GitHub secret scanning |
| **SAST** (static) | lỗ hổng trong *code của bạn* | SonarQube, Semgrep, CodeQL |
| **SCA** (dependency) | lỗ hổng trong *thư viện bên thứ 3* | Trivy, Snyk, Dependabot, OWASP Dep-Check |
| **Container scan** | CVE trong image (OS + lib) | Trivy, Grype, Snyk |
| **IaC scan** | Terraform/K8s cấu hình sai (S3 public...) | Checkov, tfsec, Trivy |
| **DAST** (dynamic) | tấn công app đang chạy | OWASP ZAP |

### ✅ Best practice — nhiều tool, chọn cái nào?
- **Muốn 1 tool phủ nhiều loại, open-source, miễn phí, chạy nhanh trong CI → Trivy.** Nó làm được SCA + container scan + IaC scan + secret → gọn cho startup/team vừa. Đây thường là lựa chọn "bắt đầu" tốt nhất.
- **Doanh nghiệp cần báo cáo, ưu tiên vá, hỗ trợ, tích hợp SLA → Snyk** (thương mại, developer-friendly, gợi ý fix).
- **SAST chuyên sâu chất lượng code + security gate → SonarQube** (hoặc CodeQL nếu ở GitHub, miễn phí cho public).
- **Secret**: bật **cả 2 lớp** — quét lúc commit (pre-commit hook Gitleaks) *và* ở CI (chặn merge).

> **Nguyên tắc "fail the build":** đặt ngưỡng — vd chặn merge nếu có lỗ hổng **HIGH/CRITICAL**. Nhưng đừng chặn mọi thứ (kể cả LOW) → team sẽ tắt scan cho đỡ phiền. Cân bằng tín hiệu/nhiễu.

### ⚠️ Bẫy
- Quét nhưng **không ai xử kết quả** → "security theater". Phải có quy trình triage.
- Chỉ quét image cuối, quên **base image** → kéo theo hàng trăm CVE. Dùng base image nhỏ (distroless/alpine) + cập nhật thường xuyên.

---

## Phần 10 — Disaster Recovery & Backup

### Bản chất — 2 con số phải thuộc lòng
```
   RPO (Recovery Point Objective) = "mất tối đa BAO NHIÊU DỮ LIỆU?"
        └─► quyết định tần suất BACKUP / replication
   RTO (Recovery Time Objective)  = "được phép SẬP BAO LÂU?"
        └─► quyết định KIẾN TRÚC DR (backup vs standby nóng)

   Ví dụ: RPO=5 phút, RTO=1 giờ → replicate liên tục + có sẵn kịch bản failover
```

### Deep dive — 4 chiến lược DR (đắt dần, RTO nhỏ dần)

```
   1. BACKUP & RESTORE      2. PILOT LIGHT        3. WARM STANDBY      4. MULTI-SITE
   ────────────────────     ──────────────        ───────────────     ────────────
   RTO: giờ→ngày            RTO: chục phút        RTO: phút           RTO: ~0 (active-active)
   Rẻ nhất                  Chạy phần lõi          Chạy bản thu nhỏ    Chạy full 2 vùng
   ┌──┐   backup            ┌──┐  ┌·┐ (DB replicate ┌──┐ ┌▪┐          ┌──┐ ┌──┐
   │██│─────────►[kho]      │██│  │·│  chờ scale up)│██│ │▪│          │██│ │██│
   └──┘                     └──┘  └·┘               └──┘ └▪┘          └──┘ └──┘
   phục hồi lâu             bật nhanh khi cần      chỉ cần scale       chuyển traffic tức thì
```

### 🎯 Use case ShopVN (hybrid)
- **DB Azure SQL**: bật **geo-replication** sang region phụ (Southeast Asia → East Asia). RPO nhỏ.
- **Storage/Blob**: dùng **GRS** (Geo-Redundant Storage) — tự nhân bản sang region khác.
- **Oracle on-prem**: backup + replicate lên **Azure (blob)** làm nơi trú ẩn → nếu data center cháy vẫn còn.
- **App AKS**: manifest/Helm trong Git (IaC) → dựng lại cluster region phụ trong vài chục phút.

### ✅ Best practice
- **Chọn chiến lược theo RTO/RPO của từng hệ thống, không "một cỡ cho tất cả".** Trang bán hàng chính → warm standby; hệ thống report nội bộ → backup & restore là đủ (tiết kiệm).
- **Backup phải được TEST khôi phục định kỳ** — "backup không test = không có backup". Diễn tập DR (game day).
- Dùng **IaC** để tái dựng hạ tầng vùng phụ → đây là lý do IaC và DR gắn chặt nhau.
- Quy tắc **3-2-1**: 3 bản sao, 2 loại lưu trữ, 1 bản off-site (Azure là "off-site" tự nhiên cho on-prem).

---

## Phần 11 — FinOps: tối ưu chi phí

### Bản chất
**FinOps** = văn hóa + thực hành để **mọi người chịu trách nhiệm về chi phí cloud**. Gốc rễ: cloud là OPEX (thuê theo giờ) → dễ "rò rỉ tiền" nếu không quản. Vòng lặp: **Inform (nhìn thấy) → Optimize (tối ưu) → Operate (vận hành, tự động hóa)**.

### Deep dive — các đòn bẩy tiết kiệm (theo thứ tự "dễ & lời nhất")

```
   1. TẮT cái không dùng     → dev/test tắt ban đêm, xóa disk/IP mồ côi   (lời ngay)
   2. RIGHT-SIZING           → VM D8s dùng 10% CPU → hạ xuống D2s
   3. AUTOSCALE              → scale theo tải thật (VMSS/HPA) thay vì over-provision
   4. Reserved / Savings Plan→ cam kết 1-3 năm cho tải ổn định (giảm tới ~60-70%)
   5. Spot instances         → tải chịu gián đoạn (batch) rẻ tới ~90%
   6. Storage tiering        → data cũ → Cool/Archive tier
   7. Tagging + budget alert  → gắn tag theo team/dự án để quy trách nhiệm
```

### ✅ Best practice — dùng công cụ gì?
- **Nhìn (Inform)**: **Azure Cost Management + Budgets** (native, miễn phí) — đặt ngân sách + cảnh báo. Bổ sung **Infracost** trong CI để **ước lượng chi phí ngay trong PR Terraform** (thấy tiền *trước khi* deploy — rất mạnh).
- **Khuyến nghị (Optimize)**: **Azure Advisor** gợi ý right-sizing/reserved tự động.
- **Cam kết**: tải ổn định 24/7 → **Reserved Instances / Savings Plan**; tải biến động → **autoscale**; tải chịu gián đoạn → **Spot**.
- **Quy trách nhiệm**: bắt buộc **tagging** (team, env, cost-center) qua **Azure Policy** — không tag thì không cho tạo.

### 🎯 Use case ShopVN
- Môi trường `dev/staging` tự tắt 19h–7h + cuối tuần → tiết kiệm ~65% giờ chạy.
- Node AKS baseline (chạy 24/7) mua **Savings Plan**; node scale cho ngày sale dùng **Spot**.
- Ảnh sản phẩm cũ > 90 ngày → chuyển Blob sang **Cool/Archive**.

### ⚠️ Bẫy
- Tối ưu quá tay làm giảm độ tin cậy (cắt HA để tiết kiệm) → cân với SLO.
- Mua Reserved cho tải chưa ổn định → "khóa" tiền sai. Right-sizing **trước**, cam kết **sau**.

---

## Phần 12 — Config Management & Version Control

### Ansible — cấu hình đồng nhất (điểm mạnh on-prem của bạn)
- **Agentless**: chạy qua SSH/WinRM, không cần cài agent trên máy đích → hợp on-prem lẫn cloud.
- **Idempotent**: chạy playbook nhiều lần cho cùng kết quả (không "cài lại nginx 3 lần").
- **Use case ShopVN**: patch OS đồng loạt cho VM ERP on-prem, cài & cấu hình node_exporter (để Prometheus thu metrics), quản user/ssh key.

**Ansible vs Terraform (nhắc lại cho chắc):** Terraform *tạo* hạ tầng; Ansible *cấu hình bên trong*. Với cloud thuần, xu hướng là **immutable** (build image sẵn bằng Packer + Terraform) thay cho cấu hình runtime; nhưng **on-prem thì Ansible vẫn là vua**.

### Version Control best practices (JD nhắc "establish version control best practices")
```
   ✔ Trunk-based hoặc GitHub Flow (nhánh ngắn, merge sớm) cho tốc độ cao
   ✔ Branch protection: bắt buộc PR review + CI xanh mới merge
   ✔ Conventional Commits (feat:, fix:...) → tự sinh changelog/version
   ✔ MỌI THỨ vào Git: app code, IaC, pipeline YAML, Ansible playbook, dashboard
     → "GitOps": Git là nguồn sự thật duy nhất (single source of truth)
   ✔ KHÔNG commit secret → dùng Key Vault + secret scanning
   ✔ Tag/release + semantic versioning cho artifact
```

**GitOps** (nâng cao, rất hợp K8s): trạng thái mong muốn của cluster nằm trong Git; công cụ (**ArgoCD/Flux**) tự đồng bộ cluster khớp Git. Deploy = merge PR. Rollback = git revert.

---

## Phần 13 — Bảng quyết định nhanh

> "Bài toán X, có mấy tool, chọn cái nào?" — tra bảng này.

| Bài toán | Mặc định nên chọn | Chọn cái khác khi... |
|----------|-------------------|----------------------|
| CI/CD dự án mới, cloud-native | GitHub Actions | Đã dùng bộ Azure DevOps → Azure DevOps; on-prem legacy nặng → Jenkins |
| IaC thuần Azure | Bicep | Hybrid/đa cloud → Terraform/OpenTofu |
| IaC hybrid/đa cloud | Terraform/OpenTofu | — |
| Cấu hình OS/máy | Ansible | Cloud immutable → Packer + image |
| Chạy nhiều microservice | AKS | Ít service/đơn giản → Container Apps/VMSS |
| Chạy app đơn, cần auto-scale | VMSS / App Service | Nhiều service phức tạp → AKS |
| Nối on-prem ↔ Azure | S2S VPN | Cần băng thông/độ trễ ổn định → ExpressRoute |
| Bảo vệ PaaS (SQL/Storage) | Private Endpoint | — (đừng để public) |
| Instrument app | OpenTelemetry (luôn) | — |
| Lưu metrics | Prometheus | Azure-native gọn → Azure Monitor |
| Lưu logs | Loki (rẻ/nhẹ) | Cần full-text search sâu → ELK |
| Dashboard | Grafana | Thuần Azure → Azure Monitor Workbooks |
| Quét bảo mật all-in-one, free | Trivy | Doanh nghiệp cần support/fix → Snyk |
| SAST | SonarQube / CodeQL | — |
| DR cho hệ thống quan trọng | Warm standby + geo-replication | Hệ ít quan trọng → Backup & restore |
| Xem chi phí | Azure Cost Management + Infracost (PR) | — |
| Cam kết chi phí tải ổn định | Reserved/Savings Plan | Tải biến động → autoscale; chịu gián đoạn → Spot |

### 🧠 Nguyên tắc chung khi phân vân giữa các tool
1. **Đơn giản nhất mà đủ dùng thắng** (App Service < VMSS < AKS). Đừng over-engineer.
2. **Trung lập vendor ở tầng chuẩn** (OpenTelemetry, Terraform) để không bị khóa.
3. **Managed/PaaS thắng self-hosted** trừ khi có lý do rõ (air-gapped, tuân thủ, chi phí quy mô cực lớn).
4. **Chọn theo "team & code đang ở đâu"**, không theo bảng tính năng.
5. **Đo được thì quản được**: mọi quyết định (reliability, cost) phải có số (SLO, budget).

---

## Phần 14 — Lộ trình học + chứng chỉ

### Ánh xạ chứng chỉ trong JD
```
   AZ-104 (Administrator)   → nền tảng: VM, VNet, Storage, Entra ID, monitor
        │  "vận hành được Azure"
        ▼
   AZ-305 (Solutions Architect) → thiết kế: HA, DR, hybrid, security, cost
        │  "thiết kế kiến trúc đúng"
        ▼
   AZ-400 (DevOps Engineer Expert) → CI/CD, IaC, observability, DevSecOps
        "tự động hóa & vận hành ở quy mô"
   (AZ-400 yêu cầu có AZ-104 hoặc AZ-204 trước)
```

### Lộ trình 8 tuần (tận dụng thế mạnh on-prem của bạn)
```
   Tuần 1-2  Nền tảng Azure + ánh xạ on-prem→cloud (Phần 2)   → thi thử AZ-104
             Dựng 1 VNet + VM + NSG bằng Portal rồi bằng Terraform
   Tuần 3    IaC: Terraform tạo AKS + SQL; Ansible cấu hình 1 VM
   Tuần 4    Container: đóng Docker 1 app, deploy lên AKS, HPA
   Tuần 5    CI/CD: pipeline GitHub Actions/Azure DevOps đủ 8 bước (Phần 1)
   Tuần 6    Observability: OTel → Prometheus + Grafana; đặt 1 SLO + alert
   Tuần 7    DevSecOps (Trivy) + DR (geo-replication) + FinOps (Cost + Infracost)
   Tuần 8    Ráp toàn bộ thành "ShopVN thu nhỏ" → đây là PROJECT để phỏng vấn
```

### 💼 Cách nói trong phỏng vấn (biến on-prem thành lợi thế)
> "Tôi có nền tảng on-prem vững (mạng, OS, Ansible, backup). Khi lên Azure, tôi ánh xạ từng khái niệm on-prem sang dịch vụ tương đương, nên tôi hiểu **cả hai phía của hybrid** — đó chính là thứ JD này cần. Tôi đã dựng end-to-end: IaC (Terraform) → CI/CD → AKS → OpenTelemetry/Grafana → SLO/error budget → DR → FinOps."

---

## 🎁 Tổng kết — Big Picture 1 hình

```
   ┌────────────────────────── VÒNG ĐỜI DEVOPS (khép kín) ──────────────────────────┐
   │                                                                                 │
   │  CODE ──► BUILD ──► TEST ──► SCAN(sec) ──► RELEASE ──► DEPLOY ──► OPERATE ──► ┐  │
   │  Git      CI        unit      SAST/SCA     artifact    AKS        run         │  │
   │  Bitbucket          test      Trivy        ACR         VMSS                   │  │
   │  ▲                                                        │                    │  │
   │  │                                                        ▼                    │  │
   │  │                                              MONITOR (OpenTelemetry)         │  │
   │  │                                              Prometheus/Grafana/ELK          │  │
   │  │                                              SLI/SLO/error budget            │  │
   │  │                                                        │                    │  │
   │  └──────────────── PLAN (dựa trên metrics/incident) ◄─────┘                    │  │
   │                                                                                 │  │
   │  NỀN MÓNG XUYÊN SUỐT:                                                           │  │
   │   • IaC (Terraform/Bicep) + Ansible  → mọi hạ tầng là code                     │  │
   │   • Networking (VNet/NSG/VPN/PrivateEndpoint) → hybrid an toàn                 │  │
   │   • DR/Backup → chịu thảm họa   • FinOps → tối ưu tiền   • Security → shift-left│  │
   └─────────────────────────────────────────────────────────────────────────────────┘
```

**Một câu để nhớ toàn bộ JD:**
> *Dùng **IaC** dựng hạ tầng **hybrid** an toàn (**VNet/Private Endpoint**), đóng gói app bằng **Docker** chạy trên **AKS**, đưa lên bằng **CI/CD** có **quét bảo mật**, quan sát bằng **OpenTelemetry → Grafana**, cam kết độ tin cậy bằng **SLO/error budget**, sống sót thảm họa bằng **DR/backup**, và tối ưu tiền bằng **FinOps** — tất cả version-controlled trong **Git**.*

---

> 📌 *Tài liệu bổ sung cho repo. Bạn có thể học kèm các file AWS SAA sẵn có (nhiều khái niệm networking/HA/DR/cost là chung cho mọi cloud, chỉ khác tên dịch vụ). Cần mình đào sâu thêm phần nào (vd hands-on lab Terraform+AKS thực tế, hay chi tiết OpenTelemetry Collector config), cứ nói.*
