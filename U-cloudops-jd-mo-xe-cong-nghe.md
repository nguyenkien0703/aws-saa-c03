# 🚀 U. MỔ XẺ JD "CLOUD OPS" — TỪ ON-PREM LÊN CLOUD (DEEP DIVE)

> **Dành cho:** Người có nền tảng vững về On-Premise (mạng, hệ thống, firewall vật lý) nhưng **ít kinh nghiệm thực chiến Cloud**, muốn xây nền tảng vững và hiểu cách giải bài toán thực tế.
>
> **Cách đọc tài liệu này:**
> 1. Đọc **Phần 0 (Big Picture)** trước để thấy bức tranh tổng thể — tất cả công nghệ ráp vào một hệ thống ngân hàng giả định.
> 2. Sau đó đọc từng phần deep-dive. Mỗi công nghệ đều có: **Bản chất → So sánh với On-Prem bạn đã biết → Use case → Nếu nhiều tool giải được thì chọn cái nào (Best Practice)**.
> 3. Cuối cùng đọc **Phần 8 (Decision Cheat-Sheet)** để tra nhanh.

---

## 📖 MỤC LỤC

- [Phần 0 — BIG PICTURE: Bài toán ngân hàng ACloudBank](#phần-0)
- [Phần 1 — Multi-Account & Landing Zone](#phần-1) *(Organizations, Control Tower, AFT, SCP, OU, IAM Identity Center)*
- [Phần 2 — Shared Services & Network Hub](#phần-2) *(Transit Gateway, Route 53 Resolver, VPC Endpoints, PrivateLink, S2S VPN)*
- [Phần 3 — Network Security Appliances](#phần-3) *(Palo Alto PAN-OS, F5 BIG-IP, GWLB, CAB)*
- [Phần 4 — Monitoring & Observability](#phần-4) *(Prometheus, Grafana, Loki, CloudWatch, EventBridge, SNS, PagerDuty)*
- [Phần 5 — Automation & IaC](#phần-5) *(Terraform, GitLab CI, GitHub Actions, GitOps, Python/Bash)*
- [Phần 6 — Incident Management & BCP/DR](#phần-6)
- [Phần 7 — Một ngày vận hành + Luồng xử lý sự cố end-to-end](#phần-7)
- [Phần 8 — Bảng quyết định nhanh (Best Practice Cheat-Sheet)](#phần-8)
- [Phần 9 — Lộ trình học từ On-Prem lên Cloud](#phần-9)

---

## 🧾 DANH SÁCH CÔNG NGHỆ ĐƯỢC TRÍCH TỪ JD

Trước khi đi sâu, đây là **toàn bộ công nghệ/khái niệm** mình soi ra được trong JD, gom theo nhóm:

| # | Nhóm | Công nghệ / Khái niệm |
|---|------|----------------------|
| 1 | **Governance / Multi-Account** | AWS Organizations, Control Tower, Account Factory for Terraform (AFT), Service Control Policies (SCP), Organizational Units (OU), Account Baseline, Landing Zone |
| 2 | **Identity** | IAM (roles, policies, permission boundaries), AWS SSO = IAM Identity Center |
| 3 | **Network Hub** | Transit Gateway (TGW), Route 53 Resolver, Private Hosted Zones (PHZ), DNS forwarding rules, VPC Endpoints (Gateway + Interface), PrivateLink, Site-to-Site VPN, VPC/Subnetting |
| 4 | **Security Appliances** | Palo Alto Networks PAN-OS (policies, zones, NAT, threat prevention), F5 BIG-IP (virtual servers, pools, health monitors, SSL offload), Gateway Load Balancer (GWLB — ngầm định), CAB / Change Management, SOC/NOC |
| 5 | **Observability** | Grafana, Prometheus, Loki, CloudWatch (Logs / Metrics / Alarms), EventBridge, SNS, PagerDuty, Slack, VPC Flow Logs |
| 6 | **IaC / Automation** | Terraform (modules, state, remote backend, workspaces), GitLab CI, GitHub Actions, GitOps, Python, Bash |
| 7 | **Resilience / Process** | BCP/DR, Incident Management, On-call, Runbook, SOP, Well-Architected Framework |

> 👉 Toàn bộ 7 nhóm này sẽ được **ráp lại thành 1 hệ thống duy nhất** ở Phần 0, để bạn thấy chúng ăn khớp với nhau thế nào — đúng như JD mô tả một **Cloud Platform / Shared Services team** của ngân hàng.

---

<a name="phần-0"></a>
## 🏦 PHẦN 0 — BIG PICTURE: BÀI TOÁN "ACloudBank"

### Đề bài (mô phỏng đúng JD)

> **ACloudBank** là một ngân hàng số. Ban đầu chỉ có 1 tài khoản AWS, dev tự bấm tay trên Console. Giờ ngân hàng bùng nổ: 4 team sản phẩm (Core Banking, Mobile App, Data/BI, Internal Tools), yêu cầu bảo mật khắc nghiệt (PCI-DSS, quy định NHNN), phải kết nối với **Data Center on-prem** (nơi đặt Core Banking cũ) và với **đối tác** (cổng thanh toán, CIC — trung tâm thông tin tín dụng).
>
> Nhiệm vụ của team Cloud Ops (chính là vị trí trong JD): **Xây "sân bay" (Landing Zone) cho toàn ngân hàng — nơi mọi account mới hạ cánh đều tự động tuân thủ chuẩn bảo mật, mạng, logging.**

### Vì sao "1 account" là sai lầm? (Góc nhìn On-Prem)

Ở on-prem, bạn không nhét cả ngân hàng vào **một VLAN, một Windows Domain, một firewall** duy nhất. Bạn chia **zone** (DMZ, Internal, Restricted), chia **VRF**, phân quyền theo phòng ban. Trên Cloud cũng vậy — nhưng đơn vị cô lập mạnh nhất **không phải VPC, mà là AWS Account**.

```
On-Prem                         AWS
─────────────────────────────   ─────────────────────────────
Tủ rack / khu vực vật lý      →  AWS Account   (ranh giới bảo mật + billing mạnh nhất)
VLAN / VRF                    →  VPC + Subnet
Firewall vật lý (Palo Alto)   →  Security Group + NACL + GWLB + Palo Alto ảo
Active Directory              →  IAM Identity Center (SSO) + IAM
Core switch / router hub      →  Transit Gateway
Leased line tới đối tác       →  Site-to-Site VPN / Direct Connect / PrivateLink
```

> 💡 **Nguyên tắc vàng:** *Account là "blast radius" (bán kính vụ nổ).* Một account bị hack/nhầm lệnh `terraform destroy` thì các account khác vẫn sống. Vì vậy best practice AWS là **multi-account** — mỗi môi trường/mỗi team/mỗi mục đích một account riêng.

### Sơ đồ tổng thể ACloudBank (toàn bộ công nghệ trong JD nằm ở đây)

```
                                 ┌───────────────────────────────────────────────────┐
                                 │              AWS ORGANIZATION (root)               │
                                 │        Quản lý bởi Control Tower + AFT             │
                                 └───────────────────────────────────────────────────┘
                                                       │  (SCP áp từ trên xuống)
        ┌───────────────────────┬──────────────────────┼───────────────────────┬───────────────────────┐
        │                       │                      │                       │                       │
   ┌────▼─────┐          ┌──────▼──────┐        ┌───────▼──────┐        ┌───────▼───────┐       ┌───────▼───────┐
   │  OU:     │          │  OU:        │        │  OU:         │        │  OU:          │       │  OU:          │
   │Security  │          │Infrastructure│       │  Workloads   │        │  Workloads    │       │  Sandbox      │
   │          │          │             │        │  (Prod)      │        │  (Non-Prod)   │       │               │
   ├──────────┤          ├─────────────┤        ├──────────────┤        ├───────────────┤       ├───────────────┤
   │Log Archive│         │ Network Hub │        │ CoreBank-Prod│        │ CoreBank-Dev  │       │ dev-sandbox   │
   │Audit     │          │ Shared Svcs │        │ Mobile-Prod  │        │ Mobile-Dev    │       │               │
   │(GuardDuty,│         │(TGW, R53,   │        │ Data-Prod    │        │ Data-Dev      │       │               │
   │Security  │          │ Palo Alto,  │        │              │        │               │       │               │
   │ Hub)     │          │ F5, Egress) │        │              │        │               │       │               │
   └──────────┘          └──────┬──────┘        └──────┬───────┘        └───────┬───────┘       └───────────────┘
                                │                      │                        │
                                │        ┌─────────────┴────────────┐           │
                                │        │   TRANSIT GATEWAY (hub)  │◄──────────┘
                                │        │  (mọi VPC + on-prem cắm  │
                                └───────►│   vào đây như core-switch)│
                                         └───────┬──────────┬────────┘
                                                 │          │
                        ┌────────────────────────┘          └───────────────────────┐
                        │ Site-to-Site VPN / DX                   PrivateLink        │
                   ┌────▼─────────┐                          ┌────────▼──────────┐
                   │ DATA CENTER  │                          │  ĐỐI TÁC          │
                   │ ON-PREM      │                          │ (Payment Gateway, │
                   │ Core Banking │                          │  CIC, 3rd-party)  │
                   └──────────────┘                          └───────────────────┘

   OBSERVABILITY (xuyên suốt, đổ về Infra/Monitoring account):
   ┌───────────────────────────────────────────────────────────────────────────────┐
   │  Mọi account → CloudWatch Logs + Metrics → (cross-account) →  Central Account   │
   │  Prometheus (scrape metrics) → Grafana (dashboard) ; Loki (gom log)             │
   │  Alarm/EventBridge → SNS → PagerDuty (gọi điện) / Slack (chat) → On-call        │
   └───────────────────────────────────────────────────────────────────────────────┘

   AUTOMATION (cách mọi thứ trên được tạo ra):
   ┌───────────────────────────────────────────────────────────────────────────────┐
   │  Git repo (Terraform modules)  →  GitLab CI / GitHub Actions  →  terraform      │
   │  apply  →  AFT pipeline vend account mới tự động theo baseline                  │
   └───────────────────────────────────────────────────────────────────────────────┘
```

**Đọc sơ đồ này một lần nữa, chậm rãi.** Mọi thứ còn lại trong tài liệu chỉ là "zoom vào" từng khối của bức tranh này. Giờ ta đi từng phần.

---

<a name="phần-1"></a>
## 🏛️ PHẦN 1 — MULTI-ACCOUNT & LANDING ZONE

Đây là **xương sống** của vị trí. Nếu chỉ học được 1 phần, hãy học phần này.

### 1.1. AWS Organizations — "Sở chỉ huy"

**Bản chất:** Dịch vụ để **gom nhiều AWS account về dưới một quyền quản lý trung tâm**, có 1 account "mẹ" gọi là **Management account** (trước gọi là Master/Payer).

**So với On-Prem:** Giống việc bạn có **một Active Directory Forest** quản lý nhiều domain con, hoặc một **tenant** quản lý nhiều phòng ban. Organizations cho bạn:
- **Consolidated Billing** — gộp hóa đơn tất cả account, hưởng chiết khấu theo volume (giống mua sỉ điện toàn công ty thay vì từng phòng mua lẻ).
- **SCP** — luật chặn áp toàn tổ chức.
- **OU** — cây thư mục nhóm account.

```
Management Account (chỉ dùng để quản trị, KHÔNG chạy workload)
│
├── OU: Security ────────── Log Archive Acct, Audit Acct
├── OU: Infrastructure ──── Network Hub Acct, Shared Services Acct
├── OU: Workloads
│    ├── OU: Prod ───────── CoreBank-Prod, Mobile-Prod
│    └── OU: Non-Prod ───── CoreBank-Dev, Mobile-Dev
└── OU: Sandbox ─────────── dev-sandbox-01, dev-sandbox-02
```

> ⚠️ **Best practice quan trọng:** *KHÔNG chạy workload trong Management account.* Nó có quyền tối thượng — nếu bị chiếm là mất cả tổ chức. Chỉ dùng nó để quản trị Organizations.

### 1.2. Organizational Units (OU) — "Nhóm theo mục đích, không theo phòng ban"

**Use case thực tế ACloudBank:**
- OU **Security** chứa `Log Archive` (kho log bất biến) + `Audit` (nơi chạy GuardDuty, Security Hub).
- OU **Prod** áp SCP nghiêm ngặt: cấm tắt CloudTrail, cấm tạo resource ngoài `ap-southeast-1`.
- OU **Sandbox** cho dev nghịch thoải mái nhưng chặn tạo resource đắt (cấm launch instance > `xlarge`).

> 💡 **Best practice:** Nhóm OU theo **chức năng/độ nhạy cảm bảo mật** (Prod, Non-Prod, Security, Sandbox) — **KHÔNG** theo sơ đồ tổ chức công ty (phòng A, phòng B). Vì luật bảo mật (SCP) áp theo mức độ rủi ro, không theo phòng ban.

### 1.3. Service Control Policies (SCP) — "Lan can chặn, không phải cầu cho quyền"

**Bản chất — điểm này cực dễ hiểu nhầm:** SCP là **guardrail (lan can)**. Nó **KHÔNG cấp quyền**. Nó chỉ định **giới hạn tối đa** (permission ceiling) mà account trong OU đó được phép làm. Quyền thực tế = **giao của (SCP) ∩ (IAM policy)**.

**So với On-Prem:** Giống **GPO (Group Policy) cấp domain** chặn "không ai được cài phần mềm lạ" — dù user có là local admin đi nữa. SCP đứng **trên** cả `AdministratorAccess`: một user Admin trong account con vẫn không thể vượt SCP.

**Ví dụ SCP thực tế cho OU Prod của ngân hàng:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyRegionOutsideSingapore",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": { "aws:RequestedRegion": ["ap-southeast-1"] }
      }
    },
    {
      "Sid": "ProtectCloudTrailAndGuardDuty",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail",
        "guardduty:DeleteDetector",
        "config:DeleteConfigurationRecorder"
      ],
      "Resource": "*"
    }
  ]
}
```

Dịch nghĩa: *Trong OU Prod, dù bạn là ai, bạn KHÔNG thể tạo tài nguyên ngoài Singapore, và KHÔNG thể tắt các dịch vụ giám sát bảo mật.* Đây chính là "compliance và governance" mà JD nhắc tới.

> 💡 **Best practice:** SCP nên viết theo kiểu **deny-list** (chặn vài hành động nguy hiểm) thay vì allow-list (chỉ cho vài hành động) — vì allow-list rất dễ vỡ khi AWS ra service mới. Ngoại lệ: môi trường cực nhạy cảm mới dùng allow-list.

### 1.4. Landing Zone — "Sân bay chuẩn hoá"

**Bản chất:** Landing Zone = **toàn bộ thiết lập nền** (multi-account, network, security, logging, identity) được dựng sẵn theo best practice, để **mỗi account mới "hạ cánh" là đã tuân thủ chuẩn ngay**. Bạn không dựng từng account bằng tay nữa.

Landing Zone gồm 3 thứ luôn có sẵn cho account mới:
1. **Security baseline** — CloudTrail bật, Config bật, GuardDuty bật, không có Security Group mở toang.
2. **Network baseline** — VPC theo chuẩn IP, gắn sẵn vào Transit Gateway, route ra Internet qua egress tập trung.
3. **Identity baseline** — role quản trị, role read-only, tích hợp SSO.

### 1.5. AWS Control Tower — "Máy dựng & canh gác Landing Zone"

**Bản chất:** Control Tower là dịch vụ **được quản lý (managed)** của AWS để **tạo và vận hành Landing Zone tự động**. Nó tự dựng: Organizations + OU mẫu (Security, Sandbox) + Log Archive account + Audit account + SSO + hàng loạt **guardrails** (chính là SCP + AWS Config rules đóng gói sẵn).

**Ba loại guardrail:**
- **Preventive** (SCP) — *chặn* trước khi làm (vd cấm tắt CloudTrail).
- **Detective** (Config rules) — *phát hiện* sau khi làm (vd cảnh báo có S3 bucket public).
- **Proactive** (CloudFormation Hooks) — chặn ngay lúc deploy nếu cấu hình sai.

```
              ┌──────────────────────────────────────────┐
              │            AWS CONTROL TOWER              │
              │  (bảng điều khiển + tự động hoá)          │
              └───────────────┬──────────────────────────┘
        tự động tạo & quản lý │
   ┌──────────────┬───────────┼───────────┬───────────────┐
   ▼              ▼           ▼           ▼               ▼
Organizations  OU mẫu   Log Archive   Audit acct     Guardrails
+ SCP          (Security, (bất biến)   (GuardDuty,    (preventive/
               Sandbox)                Security Hub)  detective)
                            + IAM Identity Center (SSO)
```

### 1.6. Account Factory for Terraform (AFT) — "Dây chuyền đẻ account bằng code"

**Đây là từ khoá "đắt giá" nhất trong JD** (JD yêu cầu *proven hands-on experience*).

**Vấn đề:** Control Tower có sẵn **Account Factory** (bấm nút tạo account qua Console/Service Catalog). Nhưng ngân hàng cần **tạo account bằng code, qua pipeline, có review, có audit** — không ai được bấm tay. Đó là lý do có **AFT**.

**Bản chất:** AFT là một **framework Terraform + pipeline** (do AWS phát hành) chạy trên Control Tower. Bạn khai báo account mới trong 1 file Terraform → push Git → pipeline tự động: tạo account → áp customization (network, IAM, tag, tool) → account sẵn sàng.

```
 Developer muốn account mới cho team "Payments"
        │
        │ 1. Thêm 1 block vào Git (account request)
        ▼
 ┌──────────────────────┐   account_requests.tf
 │  module "payments" { │   name = "payments-prod"
 │    email = ...        │   ou   = "Prod"
 │    ou    = "Prod"     │   customizations = "banking-baseline"
 │  }                    │
 └──────────┬───────────┘
            │ 2. git push → MR/PR review (4-eyes)
            ▼
 ┌───────────────────────────────────────────────────────┐
 │                  AFT PIPELINE (CodePipeline)            │
 │  ┌──────────┐  ┌───────────────┐  ┌──────────────────┐ │
 │  │ Control  │→ │ Global        │→ │ Account-specific │ │
 │  │ Tower    │  │ Customizations│  │ Customizations   │ │
 │  │ tạo acct │  │(áp cho MỌI acct)│ │(riêng payments)  │ │
 │  └──────────┘  └───────────────┘  └──────────────────┘ │
 └───────────────────────────┬───────────────────────────┘
                             ▼
            Account "payments-prod" ra đời, ĐÃ có sẵn:
            VPC chuẩn + gắn TGW + IAM roles + logging + tag chuẩn
```

**Bốn "khối" customization của AFT (nhớ để phỏng vấn):**
| Khối | Áp cho | Ví dụ |
|------|--------|-------|
| **Account Request** | account cụ thể | khai báo tên, email, OU |
| **Global Customizations** | **mọi** account | gắn VPC vào TGW, bật Flow Logs, tag chuẩn |
| **Account Customizations** | nhóm account | baseline riêng cho "banking" vs "data" |
| **AFT Account Provisioning** | trước khi tạo | logic kiểm tra tiền điều kiện |

> 💡 **Best practice — Chọn cách tạo account nào?**
>
> | Nhu cầu | Chọn | Lý do |
> |---------|------|-------|
> | Vài account, không cần automation | **Control Tower Account Factory** (Console) | Đơn giản, có sẵn |
> | Ngân hàng, cần GitOps + audit + review | **AFT** ✅ | Account là code, có pipeline, 4-eyes, reproducible |
> | Đã có Landing Zone custom rất phức tạp, không muốn Control Tower | Landing Zone Accelerator (LZA) hoặc tự build | Linh hoạt tối đa nhưng tự gánh vận hành |
>
> Với JD ngân hàng → **AFT là đáp án chuẩn.**

### 1.7. Landing Zone: Control Tower vs Tự Build vs LZA (Best Practice)

Đây là câu hỏi kinh điển "nhiều tool cùng giải một bài":

| Phương án | Khi nào chọn | Đánh đổi |
|-----------|-------------|----------|
| **AWS Control Tower + AFT** ✅ | Đa số doanh nghiệp/ngân hàng vừa & lớn | AWS lo phần khó, bạn tùy biến qua AFT. **Mặc định nên chọn.** |
| **Landing Zone Accelerator (LZA)** | Yêu cầu cực đặc thù (chính phủ, defense), cần config sâu hơn Control Tower cho phép | Mạnh nhưng nặng, cần team giỏi |
| **Tự build bằng Terraform từ đầu** | Team rất giỏi, muốn kiểm soát 100% | Bạn tự gánh mọi guardrail, mọi bản vá — tốn người |

> 🎯 **Kết luận:** Với ngân hàng như ACloudBank → **Control Tower làm nền + AFT làm dây chuyền** là best practice công nghiệp.

### 1.8. IAM: Roles, Policies, Permission Boundaries

**So với On-Prem:** IAM giống AD nhưng "mịn" hơn nhiều. Điểm khác biệt cốt lõi:
- On-prem: bạn nghĩ theo **user + group**.
- Cloud: bạn nghĩ theo **role** (danh tính tạm, không có mật khẩu vĩnh viễn, dịch vụ/người "assume" vào để lấy quyền tạm thời).

**Ba khái niệm phải phân biệt:**

```
┌─────────────────────────────────────────────────────────────────┐
│ IAM Policy (Identity-based) : Cấp quyền "được làm gì"            │
│    → gắn vào user/role/group                                     │
├─────────────────────────────────────────────────────────────────┤
│ Permission Boundary : "Trần" quyền tối đa cho 1 IAM entity       │
│    → dùng khi cho phép team tự tạo role, nhưng không cho họ      │
│      tạo role quyền cao hơn boundary. Chống leo thang đặc quyền. │
├─────────────────────────────────────────────────────────────────┤
│ SCP : "Trần" quyền cho cả ACCOUNT (từ Organizations)            │
│    → đứng trên tất cả                                            │
└─────────────────────────────────────────────────────────────────┘

Quyền thực tế = SCP  ∩  Permission Boundary  ∩  Identity Policy
                (nếu có bất kỳ Deny nào → chặn ngay)
```

**Use case permission boundary:** Ngân hàng cho phép dev tự tạo IAM role cho Lambda của họ (để họ tự chủ), NHƯNG gắn permission boundary để họ **không thể tạo role có quyền `iam:*` hoặc đụng vào production database** → tự phục vụ mà vẫn an toàn.

### 1.9. AWS SSO = IAM Identity Center — "AD của Cloud, đăng nhập một lần cho mọi account"

**Bản chất:** IAM Identity Center (tên cũ AWS SSO) cho phép **1 danh tính đăng nhập vào TẤT CẢ account** qua một cổng duy nhất, với **quyền tạm thời** (không tạo IAM user vĩnh viễn trong từng account nữa).

**So với On-Prem:** Chính xác là **SSO qua ADFS/Okta**. Bạn login một lần bằng tài khoản công ty → chọn "vào account CoreBank-Prod với role ReadOnly" → nhận credential tạm 1 giờ.

```
      Nhân viên (login bằng AD/Okta 1 lần)
                │
                ▼
    ┌───────────────────────────┐
    │  IAM Identity Center       │  ← đồng bộ user/group từ AD/Okta/Entra ID
    │  (SSO portal)              │
    └───────┬──────────┬─────────┘
            │          │      Permission Set = "template quyền" (vd ReadOnly, PowerUser)
   assume   │          │ assume
            ▼          ▼
   CoreBank-Prod   Mobile-Dev   ... (mọi account)
   [role: ReadOnly] [role: Admin]
   credential tạm    credential tạm
```

> 💡 **Best practice — IAM Users vs Identity Center?**
>
> | Tình huống | Chọn |
> |-----------|------|
> | Con người đăng nhập, môi trường multi-account | **IAM Identity Center** ✅ (không dùng IAM user cho người nữa) |
> | Ứng dụng/CI-CD cần key lâu dài | IAM Role + OIDC (GitHub Actions/GitLab dùng OIDC, **không** hardcode access key) |
> | Legacy tool bắt buộc access key | IAM user (hạn chế tối đa, xoay key thường xuyên) |
>
> **Nguyên tắc:** *Con người → SSO (quyền tạm). Máy → Role/OIDC (không key vĩnh viễn).* Access key vĩnh viễn là "nợ bảo mật".

---

<a name="phần-2"></a>
## 🌐 PHẦN 2 — SHARED SERVICES & NETWORK HUB

Phần này là **sở trường on-prem của bạn** — nhưng làm trên cloud. Bạn sẽ thấy quen thuộc.

### 2.1. Ôn nhanh: VPC & Subnetting (nền tảng)

**VPC** = mạng riêng ảo của bạn trong AWS = tương đương **một khu mạng có VRF riêng** ở on-prem. Trong VPC bạn chia **subnet** (public/private) theo AZ (Availability Zone = trung tâm dữ liệu vật lý).

```
VPC 10.0.0.0/16  (CoreBank-Prod)
├── AZ-a
│   ├── public-subnet  10.0.0.0/24   → có route ra Internet Gateway
│   └── private-subnet 10.0.10.0/24  → chỉ route nội bộ / qua TGW
└── AZ-b (dự phòng — HA)
    ├── public-subnet  10.0.1.0/24
    └── private-subnet 10.0.11.0/24
```

> ⚠️ **Bài học đắt giá cho multi-account:** **Lập kế hoạch IP (IPAM) từ đầu.** Nếu 2 VPC trùng dải IP (cùng `10.0.0.0/16`) thì **không route với nhau qua TGW được**. Ngân hàng phải có bảng phân bổ IP tập trung (AWS IPAM). Đây là lỗi kinh điển khi lên multi-account.

### 2.2. Transit Gateway (TGW) — "Core-switch/Router của cả tổ chức"

**Vấn đề:** Có 20 VPC + on-prem + đối tác. Nếu nối từng cặp bằng **VPC Peering** thì cần `n*(n-1)/2` kết nối = mạng nhện không quản nổi (full-mesh). 20 VPC = 190 peering. Ác mộng.

**Giải pháp:** **Transit Gateway** = một **hub trung tâm** (như core-switch/router hub-and-spoke ở on-prem). Mọi VPC + VPN + Direct Connect chỉ cắm **một lần** vào TGW.

```
       VPC Peering (full-mesh)          Transit Gateway (hub-and-spoke)
       ─────────────────────            ──────────────────────────────
        VPC-A ─── VPC-B                    VPC-A     VPC-B     VPC-C
          │  ╲   ╱  │                         ╲        │        ╱
          │   ╲ ╱   │                          ╲       │       ╱
        VPC-D ─── VPC-C                         ┌──────▼──────┐
       (rối, khó mở rộng)                       │     TGW     │
                                                └──┬───────┬──┘
                                                   │       │
                                               on-prem   đối tác
                                            (đơn giản, dễ mở rộng)
```

**Tính năng đắt giá của TGW — Route Tables / Segmentation:** TGW có **nhiều route table** → bạn phân đoạn ("micro-segmentation"). Ví dụ: VPC Prod **không** được route thẳng sang VPC Dev; đối tác chỉ tới được VPC "DMZ", không đụng Core Banking.

> 💡 **Best practice — TGW vs VPC Peering vs PrivateLink?**
>
> | Nhu cầu | Chọn | Vì sao |
> |---------|------|--------|
> | Nối **nhiều** VPC + hybrid, cần định tuyến & phân đoạn | **Transit Gateway** ✅ | Hub trung tâm, mở rộng dễ, có route table phân đoạn |
> | Chỉ nối **2 VPC**, lưu lượng lớn, muốn rẻ nhất | **VPC Peering** | Không phí/GB qua hub, nhưng không transitive (A-B, B-C thì A KHÔNG tự thấy C) |
> | Chỉ expose **1 service/1 port** (không muốn nối cả mạng) | **PrivateLink** | Chia sẻ service mà không mở toàn bộ network — bảo mật nhất |
>
> Với ngân hàng nhiều VPC + on-prem → **TGW là trục chính**, PrivateLink cho các kết nối service lẻ.

### 2.3. Route 53 Resolver, Private Hosted Zones, DNS Forwarding

**Vấn đề DNS lai (hybrid):** App trên AWS cần phân giải `core-banking.acloudbank.local` (nằm ở **on-prem DNS**). Ngược lại, server on-prem cần phân giải tên nội bộ trên AWS. Ai forward cho ai?

**Các mảnh ghép:**
- **Private Hosted Zone (PHZ):** vùng DNS **riêng tư** cho tên nội bộ trong VPC (vd `*.aws.acloudbank.local`).
- **Route 53 Resolver Inbound Endpoint:** cho phép **on-prem → hỏi vào AWS** (on-prem resolve được tên AWS).
- **Route 53 Resolver Outbound Endpoint + Forwarding Rules:** cho phép **AWS → hỏi ra on-prem** (forward query tên `.local` về DNS on-prem).

```
   ON-PREM DNS (BIND/AD)                 AWS (VPC + Route 53 Resolver)
   ─────────────────────                 ─────────────────────────────
   *.corp.acloudbank.local               *.aws.acloudbank.local (PHZ)

   App on-prem hỏi "x.aws..."     ──────►  INBOUND Endpoint  ──► Route 53 trả lời
                                            (10.0.0.53)

   App AWS hỏi "core.corp..."     ◄──────  OUTBOUND Endpoint ◄── Forwarding Rule
                                            forward về DNS on-prem (192.168.1.53)
```

**Use case ACloudBank:** Mobile App (trên AWS) gọi Core Banking (on-prem). Nó resolve `core-banking.corp.acloudbank.local` → Route 53 Resolver thấy rule "domain `corp.acloudbank.local` → forward tới 192.168.1.53" → trả về IP on-prem → gói tin đi qua VPN/TGW.

> 💡 **Best practice:** Đặt **Route 53 Resolver Endpoints + các PHZ trong Network Hub account** (shared), rồi **share PHZ** cho các VPC khác qua RAM (Resource Access Manager) hoặc associate. Tập trung DNS về một chỗ → dễ quản trị, tránh mỗi account tự cấu hình lệch nhau.

### 2.4. VPC Endpoints (Gateway vs Interface) & PrivateLink

**Vấn đề chi phí & bảo mật:** EC2 trong private subnet muốn gọi **S3** hoặc **DynamoDB**. Mặc định traffic đi ra **Internet qua NAT Gateway** → tốn tiền NAT + đi qua public Internet (không đẹp cho ngân hàng).

**Giải pháp — VPC Endpoint:** traffic tới dịch vụ AWS đi **hoàn toàn trong mạng AWS**, không qua Internet.

**Hai loại — PHẢI phân biệt (hay hỏi thi & phỏng vấn):**

```
┌──────────────────────────────────────────────────────────────────────┐
│ GATEWAY ENDPOINT                 │ INTERFACE ENDPOINT (PrivateLink)    │
├──────────────────────────────────┼─────────────────────────────────────┤
│ Chỉ cho: S3 và DynamoDB          │ Cho: hầu hết dịch vụ AWS khác        │
│                                  │ (SSM, ECR, CloudWatch, KMS, SQS...) │
│ Cơ chế: thêm route vào route     │ Cơ chế: tạo ENI (card mạng ảo) có   │
│ table (prefix list)              │ IP riêng trong subnet của bạn        │
│ Giá: MIỄN PHÍ                    │ Giá: TRẢ PHÍ theo giờ + GB           │
│ Không có DNS riêng               │ Có Private DNS (tên service trỏ ENI)│
└──────────────────────────────────┴─────────────────────────────────────┘
```

**PrivateLink** là công nghệ nền của Interface Endpoint. Nó còn dùng để **đối tác expose service cho bạn** (hoặc bạn expose cho đối tác) qua **1 ENI riêng tư**, không cần peering, không lộ ra Internet.

```
   Bạn (VPC)                            Đối tác (VPC khác / account khác)
   ─────────                            ──────────────────────────────
   App ──► Interface Endpoint (ENI) ══► NLB ──► Service của đối tác
           10.0.10.20 (IP riêng)        (PrivateLink)
   → Bạn "thấy" service đối tác như 1 IP nội bộ, KHÔNG mở cả mạng.
```

> 💡 **Best practice — S3 nên dùng Gateway hay Interface Endpoint?**
> - Traffic **trong cùng Region**, chỉ cần tiết kiệm NAT → **Gateway Endpoint** (miễn phí). ✅ mặc định.
> - Cần truy cập S3 **từ on-prem** qua Direct Connect/VPN, hoặc cần IP riêng → **Interface Endpoint** (S3 giờ hỗ trợ cả 2).
>
> 💡 **Best practice — kết nối tới service đối tác:**
> - Chỉ cần 1 service/1 port → **PrivateLink** (an toàn nhất, không mở network).
> - Cần route cả dải mạng 2 chiều → **VPN/TGW peering**.

### 2.5. Site-to-Site VPN & (bonus) Direct Connect

**Site-to-Site VPN:** đường hầm **IPsec mã hoá qua Internet** nối on-prem/đối tác với AWS. Chính là **IPsec VPN** bạn đã quen ở on-prem (Cisco/Palo Alto/Fortigate) — chỉ khác đầu kia là **Virtual Private Gateway** hoặc **TGW**.

```
  On-Prem Firewall (Palo Alto)                    AWS
  Customer Gateway (IP tĩnh) ══[IPsec Tunnel 1]══► TGW / VGW
                             ══[IPsec Tunnel 2]══► (2 tunnel để HA)
                             BGP trao đổi route động
```

> 💡 **Best practice — VPN vs Direct Connect (DX)?**
> | Yếu tố | Site-to-Site VPN | Direct Connect |
> |--------|------------------|----------------|
> | Đường truyền | Qua Internet (mã hoá) | Cáp riêng vật lý tới AWS |
> | Băng thông/độ trễ | Thấp hơn, biến động | Cao, ổn định, độ trễ thấp |
> | Thời gian setup | Vài giờ | Vài tuần (kéo cáp) |
> | Chi phí | Rẻ | Đắt |
> | **Ngân hàng dùng khi** | Backup / kết nối đối tác nhỏ | **Kết nối chính** Core Banking on-prem ↔ AWS (cần ổn định) |
>
> **Kiến trúc chuẩn ngân hàng:** **Direct Connect làm đường chính + Site-to-Site VPN làm backup** (nếu cáp DX đứt thì tự fail-over sang VPN). JD chỉ nhắc VPN nên có thể họ dùng VPN cho đối tác, DX cho core.

---

<a name="phần-3"></a>
## 🛡️ PHẦN 3 — NETWORK SECURITY APPLIANCES

Đây là chỗ **on-prem của bạn tỏa sáng** — Palo Alto & F5 là thiết bị bạn có thể đã đụng ở on-prem. Trên cloud, chúng chạy dưới dạng **máy ảo (virtual appliance)** từ AWS Marketplace.

### 3.1. Palo Alto PAN-OS trên AWS — "Firewall L7 tập trung"

**Bản chất:** Palo Alto VM-Series là **firewall thế hệ mới (NGFW)** chạy như EC2. Nó làm điều mà Security Group/NACL **không** làm được: **inspect Layer 7, threat prevention (IPS/IDS), URL filtering, giải mã SSL, App-ID**.

**So với On-Prem:** Y hệt Palo Alto vật lý bạn biết — cùng khái niệm **Security Zones, Policies, NAT rules, Threat Prevention profiles**. Chỉ khác: interface là ENI ảo, HA dùng nhiều AZ.

**Vấn đề kiến trúc:** Làm sao ép **mọi** traffic của 20 VPC đi qua Palo Alto để kiểm tra? → Dùng **Gateway Load Balancer (GWLB)**.

### 3.2. Gateway Load Balancer (GWLB) — "Bộ chia tải cho firewall" (ngầm định trong JD)

**Bản chất:** GWLB cho phép **chèn (insert) một cụm firewall ảo vào đường đi của gói tin một cách trong suốt**, đồng thời **scale + HA** cho cụm firewall đó. Nó dùng giao thức GENEVE để "bọc" gói tin gửi tới firewall rồi nhận lại.

```
        Kiến trúc "Inspection VPC" tập trung (Security Hub account)

   Internet
      │
      ▼
 ┌─────────────────────────────────────────────────────────────┐
 │  INSPECTION / SECURITY VPC                                   │
 │                                                             │
 │   IGW ──► GWLB Endpoint ──► [ GWLB ] ──► Palo Alto Fleet    │
 │                               (chia tải) │  ├ PA-VM (AZ-a)   │
 │                                          │  └ PA-VM (AZ-b)   │
 │   ◄─────── traffic đã được inspect ──────┘  (threat prevent)│
 └───────────────────────────┬─────────────────────────────────┘
                             │ qua TGW
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                     ▼
   CoreBank-Prod VPC    Mobile-Prod VPC       Data-Prod VPC
   (traffic Đông-Tây & Bắc-Nam đều bị ép qua Palo Alto)
```

**Use case ACloudBank:** Traffic từ Internet vào Mobile App, và traffic giữa Mobile ↔ Core Banking, đều bị **định tuyến qua Inspection VPC** (nhờ TGW route table) → Palo Alto kiểm tra threat, chặn mã độc, log lại → mới cho đi tiếp. Đây là "network security policies, zones, NAT, threat prevention" trong JD.

### 3.3. F5 BIG-IP — "Load Balancer cao cấp + WAF/SSL"

**Bản chất:** F5 BIG-IP là **Application Delivery Controller (ADC)** — load balancer L4-L7 rất mạnh, có: **Virtual Servers, Pools, Health Monitors, SSL offloading, iRules (kịch bản traffic), WAF (ASM), LTM/GTM**.

**So với On-Prem:** Đúng con F5 bạn biết. Trên AWS chạy dạng VM-Series.

**Câu hỏi lớn — vậy dùng F5 hay dùng luôn ALB/NLB của AWS?**

> 💡 **Best practice — F5 BIG-IP vs AWS ELB (ALB/NLB)?**
>
> | Tình huống | Chọn | Vì sao |
> |-----------|------|--------|
> | App cloud-native mới, cần LB HTTP đơn giản, tự scale, rẻ | **ALB** (L7) / **NLB** (L4) ✅ | Managed, không phải vá, rẻ, tích hợp WAF/ACM sẵn |
> | Cần iRules phức tạp, tính năng F5 đặc thù, **đồng nhất chính sách với on-prem** | **F5 BIG-IP** | Team đã giỏi F5, migrate cấu hình cũ, feature ALB không có |
> | Ngân hàng đang có sẵn chuẩn F5, đội ngũ quen | **F5** (như JD) | Nhất quán vận hành, kỹ năng sẵn có, kiểm soát sâu |
>
> **Thực tế JD:** Ngân hàng chọn F5 vì đã có chuẩn/kỹ năng on-prem và cần tính năng nâng cao + đồng nhất chính sách bảo mật. **Bài học:** "Best practice" không phải luôn chọn dịch vụ AWS — đôi khi **tận dụng công cụ enterprise sẵn có + kỹ năng đội ngũ** là quyết định đúng. Đây chính là lý do họ tuyển người **vừa biết on-prem (F5/Palo Alto) vừa biết cloud** như bạn.

### 3.4. CAB & Change Management — "Không ai đổi firewall lúc nửa đêm mà không ai duyệt"

**CAB (Change Advisory Board):** hội đồng duyệt thay đổi. Mọi thay đổi rủi ro cao (sửa rule Palo Alto, đổi routing TGW) phải: **có change ticket → đánh giá tác động → CAB duyệt → thực hiện trong cửa sổ bảo trì → có rollback plan**.

> 💡 **Best practice:** Ở ngân hàng, **infrastructure = code (Terraform)** giúp CAB dễ duyệt vì thay đổi hiện rõ trong **Merge Request diff** ("thêm 1 rule, xóa 1 route"). Đây là lý do IaC + CAB đi cùng nhau. Vai trò của bạn: đánh giá tác động (impact assessment), chuẩn bị rollback, trình bày ở CAB.

---

<a name="phần-4"></a>
## 📊 PHẦN 4 — MONITORING & OBSERVABILITY

JD yêu cầu **2 thế giới song song**: stack AWS-native (**CloudWatch**) và stack open-source (**Prometheus + Grafana + Loki**). Hiểu vì sao có cả hai là chìa khóa.

### 4.1. Ba trụ cột Observability (nhớ khái niệm nền)

```
┌─────────────┬──────────────────────────┬───────────────────────────┐
│ METRICS     │ LOGS                     │ TRACES                    │
│ (số liệu)   │ (bản ghi sự kiện)        │ (dấu vết 1 request)       │
├─────────────┼──────────────────────────┼───────────────────────────┤
│ CPU 80%     │ "ERROR payment failed"   │ request đi qua 5 service  │
│ req/s = 1200│ "user X logged in"       │ mất tổng 800ms ở đâu      │
│ → Prometheus│ → Loki / CloudWatch Logs │ → X-Ray / Tempo / Jaeger  │
│   CloudWatch│                          │                           │
└─────────────┴──────────────────────────┴───────────────────────────┘
       ▲                    ▲
       └──── GRAFANA (một màn hình nhìn tất cả) ────┘
```

### 4.2. Prometheus — "Cỗ máy thu thập & lưu metrics"

**Bản chất:** Prometheus **chủ động "cào" (pull/scrape)** số liệu từ các endpoint `/metrics` của app/exporter theo chu kỳ, lưu vào time-series DB, và có ngôn ngữ truy vấn **PromQL** để tính toán/cảnh báo.

**So với On-Prem:** Giống Zabbix/Nagios nhưng theo mô hình **pull + time-series + label** hiện đại, chuẩn cho Kubernetes/container.

```
   node_exporter (CPU/RAM máy)      ┐
   app /metrics (req/s, latency)    ├──scrape (pull)──► PROMETHEUS ──► lưu TSDB
   blackbox_exporter (ping/http)    ┘                       │
                                                            ├─ PromQL (query)
                                                            └─ Alertmanager (bắn cảnh báo)
```

### 4.3. Loki — "Prometheus cho log"

**Bản chất:** Loki gom log tập trung, nhưng **chỉ index metadata/label** (không index toàn văn như Elasticsearch) → **rẻ hơn nhiều**, ghép ăn ý với Grafana. Agent thu log thường là **Promtail / Grafana Alloy**.

**So với On-Prem:** Là đối thủ "nhẹ ký & rẻ" của **ELK (Elasticsearch-Logstash-Kibana)**.

### 4.4. Grafana — "Một màn hình nhìn tất cả"

**Bản chất:** Grafana là lớp **hiển thị (dashboard) + cảnh báo** đứng trên nhiều nguồn dữ liệu: Prometheus (metrics), Loki (logs), CloudWatch, và cả DB. Một dashboard có thể trộn số liệu từ nhiều nguồn.

```
   ┌────────────┐   ┌──────┐   ┌────────────┐
   │ Prometheus │   │ Loki │   │ CloudWatch │   (nhiều data source)
   └─────┬──────┘   └──┬───┘   └─────┬──────┘
         └─────────────┼─────────────┘
                       ▼
                  ┌─────────┐
                  │ GRAFANA │  dashboards + alerts + on-call view
                  └─────────┘
```

### 4.5. CloudWatch — "Observability AWS-native"

**Bản chất:** CloudWatch là bộ đồ nghề gốc của AWS: **Metrics** (mọi dịch vụ AWS tự đẩy vào), **Logs** (log group), **Alarms** (ngưỡng → hành động), **Dashboards**, **Logs Insights** (query log).

> 💡 **Best practice — CloudWatch vs Prometheus/Grafana/Loki? (câu hỏi "nhiều tool 1 bài" điển hình)**
>
> | Tiêu chí | CloudWatch | Prometheus + Grafana + Loki |
> |----------|-----------|------------------------------|
> | Metrics **dịch vụ AWS** (ELB, RDS, Lambda) | ✅ Tự có, sâu, không setup | Phải cài CloudWatch exporter |
> | Metrics **app/K8s/container** | Được nhưng đắt/kém linh hoạt | ✅ Chuẩn công nghiệp, PromQL mạnh |
> | Chi phí ở quy mô lớn | Đắt dần theo custom metrics & log | ✅ Rẻ hơn (tự vận hành) |
> | Đa cloud / hybrid / on-prem | Chỉ AWS | ✅ Chạy mọi nơi, đồng nhất on-prem + cloud |
> | Công sức vận hành | ✅ Managed, không lo | Phải tự vận hành (hoặc dùng AMP/AMG managed) |
>
> **Kiến trúc lai chuẩn (chính là điều JD muốn):**
> - Dùng **CloudWatch** làm nguồn cho **hạ tầng AWS** (alarm nhanh, log group, EventBridge).
> - Dùng **Prometheus/Loki** cho **app + container + on-prem** để đồng nhất giám sát lai và tiết kiệm.
> - **Grafana đứng trên gom cả hai** → một màn hình duy nhất. (Ngân hàng có thể dùng **AMP** = Amazon Managed Prometheus và **AMG** = Amazon Managed Grafana để đỡ vận hành.)

### 4.6. Central Logging cross-account (điều JD nhấn mạnh)

**Vấn đề:** 20 account, mỗi cái log một nơi → điều tra sự cố bất khả thi. Phải **gom log về 1 account trung tâm** (Log Archive account — do Control Tower dựng sẵn).

```
  CoreBank-Prod ─┐
  Mobile-Prod   ─┤ CloudWatch Logs ──subscription filter──► Kinesis Firehose ─┐
  Data-Prod     ─┘                                                            │
                                                    (hoặc Promtail → )        ▼
                                              ┌──────────────────────────────────┐
                                              │ CENTRAL LOGGING ACCOUNT           │
                                              │  S3 (bất biến, giữ 7 năm - NHNN)  │
                                              │  + Loki (điều tra nhanh)          │
                                              │  + OpenSearch (nếu cần full-text) │
                                              └──────────────────────────────────┘
```

### 4.7. Alerting: EventBridge, SNS, PagerDuty, Slack

**Chuỗi cảnh báo end-to-end:**

```
 Sự kiện/ngưỡng                Định tuyến              Kênh báo             Con người
 ─────────────                ──────────              ─────────            ──────────
 CloudWatch Alarm ─┐
 (CPU>90%, 5xx cao)│
 EventBridge Rule ─┼──► SNS Topic ──┬──► PagerDuty ──► gọi điện on-call (P1 nửa đêm)
 (root login,      │                ├──► Slack #alerts (P3 giờ hành chính)
  GuardDuty finding)│               └──► Email / Lambda (auto-remediation)
 Prometheus        │
 Alertmanager ─────┘
```

- **EventBridge:** "bus sự kiện" — bắt sự kiện AWS (vd "ai đó login root", "GuardDuty phát hiện đào coin") → kích hoạt hành động.
- **SNS:** hệ thống pub/sub để fan-out một thông báo ra nhiều nơi.
- **PagerDuty:** quản lý **on-call rotation + escalation** — nếu người trực không ack trong 5 phút, tự leo thang lên người tiếp theo. Dùng cho sự cố **nghiêm trọng**.
- **Slack:** cảnh báo mức thấp/thông tin, cộng tác trong lúc xử lý sự cố.

> 💡 **Best practice — phân tầng cảnh báo (chống "alert fatigue"):**
> - **P1/P2 (nguy cấp)** → PagerDuty **gọi điện** (đánh thức người).
> - **P3 (cần chú ý)** → Slack.
> - **P4 (thông tin)** → email/log.
> Đừng bắn **mọi thứ** vào PagerDuty — người trực sẽ chai lì và bỏ lỡ cái quan trọng. Mỗi alert phải **actionable** (kèm runbook link).

### 4.8. VPC Flow Logs — "Xem ai nói chuyện với ai" (Network analysis trong JD)

**Bản chất:** Flow Logs ghi lại **metadata mọi luồng IP** trong VPC (src, dst, port, ACCEPT/REJECT, bytes). Đây là công cụ số 1 để **điều tra kết nối** ("vì sao Mobile không gọi được Core Banking?").

**Use case:** Thấy nhiều dòng `REJECT` từ Mobile-Prod → Core-Prond port 8443 → biết ngay Security Group/NACL đang chặn → sửa. Kết hợp packet capture (traffic mirroring) khi cần soi sâu payload.

---

<a name="phần-5"></a>
## 🤖 PHẦN 5 — AUTOMATION & INFRASTRUCTURE AS CODE

Triết lý cốt lõi: **KHÔNG bấm tay trên Console.** Mọi thứ = code, review qua Git, apply qua pipeline.

### 5.1. Terraform — "Bản thiết kế hạ tầng bằng code"

**Bản chất:** Terraform khai báo hạ tầng mong muốn (**declarative**), so sánh với thực tế (**state**), rồi tạo/sửa/xóa để khớp. Đa cloud (không khóa vào AWS).

**Bốn khái niệm JD yêu cầu (advanced level):**

**1) Modules — đóng gói tái sử dụng:**
```hcl
# Thay vì copy-paste 200 dòng VPC cho mỗi account, viết 1 module:
module "banking_vpc" {
  source   = "git::https://gitlab.acloudbank/modules/vpc?ref=v1.4.0"
  cidr     = "10.20.0.0/16"
  azs      = ["ap-southeast-1a", "ap-southeast-1b"]
  attach_tgw = true
}
# → tái dùng cho Core, Mobile, Data... Sửa 1 chỗ, áp mọi nơi.
```

**2) State + Remote Backend — "bộ nhớ" của Terraform:**
```
   Terraform State = bản đồ "cái gì mình đang quản lý + ID thực tế"
   ┌────────────────────────────────────────────────────────┐
   │ REMOTE BACKEND (bắt buộc khi làm nhóm)                  │
   │  S3 bucket   → lưu file state (mã hoá, versioned)       │
   │  DynamoDB    → State Locking (khoá, chống 2 người apply │
   │                cùng lúc làm hỏng state)                 │
   └────────────────────────────────────────────────────────┘
```
> ⚠️ **Bài học máu:** Đừng để state ở máy local. Mất file state = Terraform "quên" nó quản lý gì = thảm họa. Luôn dùng remote backend + locking.

**3) Workspaces — tách state theo môi trường:** một codebase, nhiều state (dev/staging/prod) → không giẫm chân nhau.

> 💡 **Best practice — Workspaces vs thư mục riêng cho môi trường?**
> - **Workspaces** hợp khi khác biệt dev/prod **nhỏ** (chỉ khác vài biến).
> - **Thư mục/repo riêng + backend riêng** (pattern phổ biến hơn ở doanh nghiệp) khi môi trường khác nhau **nhiều** và cần **blast radius tách biệt** + quyền khác nhau. Ngân hàng thường chọn **tách backend theo account/môi trường** cho an toàn.

**4) State per account:** Trong multi-account, **mỗi account nên có state riêng** (không nhét cả tổ chức vào 1 state khổng lồ → chậm & rủi ro).

### 5.2. Terraform vs CloudFormation vs CDK (Best Practice)

> 💡 **Nhiều tool 1 bài — chọn IaC nào?**
>
> | Tool | Chọn khi | Đánh đổi |
> |------|---------|----------|
> | **Terraform** ✅ (JD yêu cầu) | Đa cloud, hệ sinh thái module lớn, chuẩn công nghiệp, AFT dùng nó | Tự quản state; cần kỷ luật |
> | **CloudFormation** | Thuần AWS, muốn AWS quản state hộ, dùng StackSets multi-account | Chỉ AWS, cú pháp dài dòng |
> | **AWS CDK** | Dev thích viết bằng ngôn ngữ lập trình (TS/Python) | Sinh ra CloudFormation, vẫn khóa AWS |
>
> Ngân hàng + AFT + đa dạng công cụ (Palo Alto, F5, Grafana provider) → **Terraform** là chuẩn.

### 5.3. CI/CD: GitLab CI & GitHub Actions + GitOps

**Bản chất pipeline IaC:** Mỗi thay đổi Terraform đi qua các "cổng":

```
  Dev sửa .tf ──► git push ──► Merge Request
        │
        ▼
 ┌──────────────────────────────────────────────────────────────┐
 │  PIPELINE (GitLab CI / GitHub Actions)                        │
 │  1. terraform fmt + validate   (cú pháp đúng?)                │
 │  2. tflint / checkov / tfsec   (bảo mật? tuân thủ?)           │
 │  3. terraform PLAN             (đăng diff vào MR để CAB xem)  │
 │  4. ⛔ MANUAL APPROVAL (4-eyes / CAB)                          │
 │  5. terraform APPLY            (chỉ chạy sau khi duyệt)       │
 └──────────────────────────────────────────────────────────────┘
        │
        ▼  Xác thực bằng OIDC (KHÔNG hardcode AWS key!)
   AWS account
```

**GitOps:** Git là **nguồn chân lý duy nhất (single source of truth)**. Muốn đổi hạ tầng → sửa Git, không SSH vào sửa tay. Trạng thái thật luôn khớp Git.

> 💡 **Best practice — GitLab CI vs GitHub Actions?**
> Cả hai đều tốt; **chọn theo nơi code đang ở**. Điểm chung phải làm đúng:
> - Dùng **OIDC federation** để pipeline lấy quyền AWS **tạm thời**, tuyệt đối không lưu Access Key trong CI secret.
> - **PLAN** hiển thị ở MR để review; **APPLY** phải qua **manual approval** cho môi trường Prod.
> - Bảo vệ branch `main`, bắt buộc review.

### 5.4. Python & Bash — "Chất keo tự động hoá"

- **Bash:** việc nhanh, gọi CLI, cron nhỏ (vd script backup, health-check).
- **Python (boto3):** logic phức tạp — vd Lambda tự động: khi phát hiện Security Group mở port 22 ra `0.0.0.0/0` → tự thu hồi và báo Slack. Hoặc script gom báo cáo chi phí đa account.

> 💡 **Best practice:** Việc **thay đổi hạ tầng** → dùng **Terraform** (declarative, có state). Python/Bash chỉ dùng cho **automation vận hành/glue/remediation** (event-driven, xử lý dữ liệu). Đừng dùng script để "tạo hạ tầng bừa" ngoài IaC → sẽ lệch (drift).

---

<a name="phần-6"></a>
## 🚨 PHẦN 6 — INCIDENT MANAGEMENT & BCP/DR

### 6.1. Bốn chiến lược DR (nhớ theo trục Chi phí ↔ Thời gian phục hồi)

```
  Rẻ, chậm phục hồi ◄──────────────────────────────────► Đắt, phục hồi tức thì

  ① Backup & Restore   ② Pilot Light      ③ Warm Standby    ④ Multi-Site Active/Active
  RTO: nhiều giờ        RTO: chục phút     RTO: vài phút     RTO: ~0
  RPO: tuỳ backup       Data đã sync,      Bản thu nhỏ chạy  Cả 2 vùng chạy thật,
  Chỉ có backup,        core tắt sẵn,      sẵn, scale up      chia tải, fail-over
  cần thì dựng lại      bật khi cần        khi cần            tức thì
  ────────────────      ─────────────      ──────────────    ─────────────────────
  Data ít quan trọng    Vừa phải           Hệ thống quan      Core Banking, thanh
                                           trọng              toán (không được chết)
```

- **RTO** (Recovery Time Objective): mất bao lâu để phục hồi.
- **RPO** (Recovery Point Objective): mất tối đa bao nhiêu **dữ liệu** (tính theo thời gian).

> 💡 **Best practice ngân hàng:** Core Banking/thanh toán → **Warm Standby hoặc Multi-Site** (multi-Region). Hệ thống nội bộ ít quan trọng → **Pilot Light/Backup-Restore** để tiết kiệm. Chọn theo **giá trị nghiệp vụ**, không phải "cái gì cũng active-active" (quá đắt).

### 6.2. Incident Management, On-call, Runbook

```
  Alert (PagerDuty gọi) ──► Ack ──► Điều tra ──► Khắc phục ──► Khôi phục
        │                                │
        │                                ▼
        │                         RUNBOOK (SOP)
        │                     "Khi RDS Prod đầy connection:
        │                      1) check CloudWatch metric X
        │                      2) chạy query Y
        │                      3) scale read-replica / kill session"
        ▼
  Post-Incident Review (blameless) ──► cập nhật runbook, thêm alert, sửa gốc rễ
```

- **On-call rotation:** lịch trực xoay vòng (PagerDuty quản), có escalation.
- **Runbook:** "công thức nấu ăn" xử lý từng loại sự cố — để **người trực lúc 3h sáng làm theo được ngay**, không cần chuyên gia.
- **BCP:** kế hoạch duy trì kinh doanh; phải **diễn tập DR định kỳ** (game day) — DR không test = DR không tồn tại.

---

<a name="phần-7"></a>
## 🎬 PHẦN 7 — RÁP TẤT CẢ: MỘT NGÀY VẬN HÀNH & LUỒNG SỰ CỐ

Giờ ta "chạy phim" để thấy **tất cả công nghệ phối hợp**.

### Cảnh 1 — Onboard team mới "Payments" (Automation + Governance)

```
1. Team Payments cần account Prod + Dev.
2. Bạn thêm 2 block vào Git repo AFT (account_requests).
3. Tạo Merge Request → CAB review diff → approve.
4. GitLab CI chạy → AFT pipeline:
     - Control Tower tạo 2 account, đặt vào OU Prod / Non-Prod
     - SCP tự áp (chặn region ngoài Singapore, cấm tắt CloudTrail)
     - Global customization: tạo VPC 10.30.0.0/16, gắn TGW, bật Flow Logs
     - IAM Identity Center: gán Permission Set (Payments-Admin cho Dev, ReadOnly cho Prod)
5. 20 phút sau, 2 account sẵn sàng, ĐÃ tuân thủ chuẩn. Dev login qua SSO.
```
→ *Bao phủ: Organizations, OU, SCP, Control Tower, AFT, Terraform, GitLab CI, IAM Identity Center, TGW, Flow Logs.*

### Cảnh 2 — Payments gọi đối tác thanh toán & Core Banking (Network)

```
   Mobile App (Mobile-Prod VPC)
        │ resolve "core.corp.acloudbank.local"
        │ → Route 53 Resolver forward về DNS on-prem
        ▼
   TGW ──► (route ép qua) Inspection VPC ──► Palo Alto (threat check)
        │                                         │ sạch, cho qua
        │                                         ▼
        ├──► Direct Connect/VPN ──► Core Banking ON-PREM
        └──► PrivateLink ──► Payment Gateway đối tác (chỉ 1 service, không mở mạng)

   Trước khi tới app: Internet ──► F5 BIG-IP (SSL offload, WAF, LB) ──► Mobile pods
```
→ *Bao phủ: Route 53 Resolver, TGW, GWLB+Palo Alto, Direct Connect/VPN, PrivateLink, F5.*

### Cảnh 3 — 2h47 sáng, sự cố P1 (Observability + Incident)

```
  ① Prometheus thấy latency thanh toán vọt 2s; đồng thời CloudWatch Alarm:
     RDS Prod "DatabaseConnections" chạm trần.
  ② Alertmanager + EventBridge → SNS → PagerDuty GỌI ĐIỆN cho on-call (bạn).
     (song song: Slack #incident-payments báo P1)
  ③ Bạn mở GRAFANA: 1 dashboard thấy metrics (Prometheus) + log lỗi (Loki)
     "too many connections". VPC Flow Logs xác nhận traffic bình thường
     → không phải mạng, là DB.
  ④ Mở RUNBOOK "RDS connection exhaustion":
       - kill idle sessions, tăng max_connections tạm, scale read-replica.
  ⑤ Latency về bình thường. Đóng P1.
  ⑥ Sáng hôm sau: Post-mortem (blameless). Root cause: 1 deploy làm rò rỉ
     connection pool. Hành động: thêm alarm sớm hơn, cập nhật runbook,
     fix code qua pipeline. Nếu DB region chết → kích hoạt Warm Standby (DR).
```
→ *Bao phủ: Prometheus, CloudWatch, EventBridge, SNS, PagerDuty, Slack, Grafana, Loki, Flow Logs, Runbook, Incident Mgmt, DR.*

**Đó là toàn bộ JD, gói trong một đêm trực.**

---

<a name="phần-8"></a>
## ⚡ PHẦN 8 — BẢNG QUYẾT ĐỊNH NHANH (BEST PRACTICE CHEAT-SHEET)

| Bài toán | Các lựa chọn | ✅ Nên chọn (khi nào) |
|----------|-------------|----------------------|
| Tạo & quản account hàng loạt | Console / Account Factory / **AFT** / LZA | **AFT** cho ngân hàng (GitOps, audit); Account Factory nếu ít account |
| Guardrail toàn tổ chức | IAM policy / **SCP** / Config | **SCP** để *chặn* (preventive) + Config để *phát hiện* (detective) |
| Đăng nhập con người | IAM user / **IAM Identity Center** | **Identity Center** (quyền tạm, SSO). IAM user cho người = tránh |
| CI/CD lấy quyền AWS | Access key / **OIDC role** | **OIDC** (không key vĩnh viễn) |
| Nối nhiều VPC + hybrid | Peering / **TGW** / PrivateLink | **TGW** (hub, phân đoạn). Peering cho 2 VPC. PrivateLink cho 1 service |
| Truy cập S3 từ VPC | NAT / **Gateway Endpoint** | **Gateway Endpoint** (miễn phí, không qua Internet) |
| Truy cập SSM/ECR/KMS riêng tư | NAT / **Interface Endpoint** | **Interface Endpoint** (PrivateLink) |
| Kết nối tới service đối tác | VPN / Peering / **PrivateLink** | **PrivateLink** nếu chỉ 1 service (an toàn nhất) |
| Kết nối on-prem chính | **Direct Connect** / VPN | **DX** làm chính + **VPN** backup |
| Firewall L7 / threat prevention | SG-NACL / **Palo Alto + GWLB** | **Palo Alto qua GWLB** cho inspection tập trung |
| Load balancer | **ALB/NLB** / F5 | **ALB/NLB** cho cloud-native; **F5** khi cần feature nâng cao + đồng nhất on-prem |
| Metrics hạ tầng AWS | **CloudWatch** / Prometheus | **CloudWatch** (native, sâu) |
| Metrics app/container/hybrid | CloudWatch / **Prometheus** | **Prometheus** (PromQL, rẻ, đa nền); dùng **AMP** để đỡ vận hành |
| Log tập trung | CloudWatch Logs / **Loki** / OpenSearch | **Loki** (rẻ) + **S3** (lưu trữ dài hạn/bất biến) |
| Dashboard | CloudWatch Dashboard / **Grafana** | **Grafana** (gom mọi nguồn về 1 màn hình) |
| Cảnh báo nguy cấp (đánh thức người) | Email / Slack / **PagerDuty** | **PagerDuty** cho P1/P2; Slack P3; email P4 |
| IaC | **Terraform** / CloudFormation / CDK | **Terraform** (đa cloud, module, AFT dùng nó) |
| Lưu Terraform state | Local / **S3 + DynamoDB** | **S3 (versioned) + DynamoDB lock** — bắt buộc khi làm nhóm |
| DR cho hệ trọng yếu | Backup / Pilot Light / **Warm Standby** / Multi-Site | **Warm Standby / Multi-Site** cho Core Banking; Pilot Light cho hệ phụ |

---

<a name="phần-9"></a>
## 🎯 PHẦN 9 — LỘ TRÌNH HỌC TỪ ON-PREM LÊN CLOUD

Bạn mạnh mạng/hệ thống on-prem — hãy tận dụng, đừng học lại từ số 0. Thứ tự đề xuất:

**Giai đoạn 1 — Nền tảng account & mạng (2–3 tuần).** Dựng thử: 1 Organization + 2–3 account bằng **Control Tower** (dùng free tier/sandbox), viết vài **SCP**, bật **IAM Identity Center**. Đây là thứ khác on-prem nhất → học trước.

**Giai đoạn 2 — Mạng (nhanh với bạn, vì đã giỏi).** Map kiến thức cũ: VLAN→VPC, core-switch→**TGW**, IPsec→**S2S VPN**, DNS forwarding→**Route 53 Resolver**. Làm lab nối 2 VPC qua TGW + 1 VPN giả lập on-prem.

**Giai đoạn 3 — IaC (quan trọng bậc nhất cho JD).** Học **Terraform** cho tới mức viết **module + remote backend (S3/DynamoDB)**. Đưa mọi thứ giai đoạn 1–2 vào code. Gắn vào **GitHub Actions/GitLab CI** với OIDC.

**Giai đoạn 4 — Observability.** Dựng **Prometheus + Grafana + Loki** bằng Docker Compose ở nhà, rồi bản cloud (**AMP + AMG**). Nối CloudWatch làm data source cho Grafana.

**Giai đoạn 5 — Security appliances (bạn đã có nền).** Tìm hiểu **Palo Alto VM-Series + GWLB** và **F5 BIG-IP** trên AWS Marketplace — kỹ năng on-prem của bạn chuyển thẳng sang, chỉ khác cách "chèn" vào mạng cloud (GWLB).

**Giai đoạn 6 — Vận hành thực chiến.** Viết **runbook**, cấu hình alert phân tầng (PagerDuty/Slack), diễn tập **DR game day**, tham gia mô phỏng **CAB**.

**Chứng chỉ hỗ trợ (theo thứ tự giá trị cho JD này):**
1. **AWS SAA-C03** (bạn đang học — nền tảng) →
2. **AWS Advanced Networking Specialty** (TGW, DX, Route 53, hybrid — rất trúng JD) →
3. **HashiCorp Terraform Associate** →
4. **AWS Security Specialty** (multi-account, SCP, IAM sâu).

> 💬 **Lời khuyên cuối:** JD này tuyển người **"lai" giữa on-prem và cloud** — chính là lợi thế của bạn. Đừng tự ti vì ít kinh nghiệm cloud: rất nhiều khái niệm (firewall zone, LB, DNS forward, routing, HA, DR) bạn **đã hiểu bản chất**, chỉ cần học **cách AWS gọi tên và ráp lại**. Tài liệu này chính là cây cầu đó. Hãy dựng một **Landing Zone mini trong tài khoản cá nhân** và tự tay đi hết Cảnh 1→3 ở Phần 7 — làm được là bạn tự tin phỏng vấn.

---

> 📚 **Liên kết trong repo này:** xem thêm `04-networking-services.md` (VPC/TGW/Route53), `05-security-services.md` (IAM/SCP), `06-management-governance.md` (Organizations/Control Tower), `B-bao-mat-compliance.md`, `A-nen-tang-kien-truc.md` (Well-Architected), `P-architecture-diagrams.md` (nhiều sơ đồ chi tiết hơn).
