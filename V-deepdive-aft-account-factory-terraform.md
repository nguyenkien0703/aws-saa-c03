# 🏭 V. DEEP DIVE: AFT — ACCOUNT FACTORY FOR TERRAFORM (THỰC CHIẾN)

> **Chủ đề hẹp #1** trong loạt đào sâu cho JD Cloud Ops. Đây là *differentiator* tuyển dụng số 1 của JD (yêu cầu *"proven hands-on experience deploying Control Tower & AFT"*).
>
> **Cho ai:** người mạnh on-prem, ít kinh nghiệm cloud, cần hiểu **thực tế vận hành** AFT chứ không chỉ khái niệm.

---

### ⚠️ Ghi chú về độ tin cậy (đọc trước)

Tài liệu này được tổng hợp từ một lần chạy **deep-research** (fan-out nhiều nguồn → trích claim → kiểm chứng đối kháng). Bước kiểm chứng tự động bị **ngắt giữa chừng do đụng giới hạn sử dụng phiên**, nên trạng thái mỗi nhóm claim được đánh dấu rõ:

- ✅ **[VERIFIED 3-0]** = đã được 3 agent độc lập xác nhận, trích thẳng tài liệu **AWS chính thức**.
- 📄 **[SOURCED]** = trích từ nguồn chính thức (AWS docs/blog, HashiCorp) nhưng **chưa qua vòng kiểm chứng đối kháng** (bị ngắt vì rate limit). Mình đã đối chiếu với kiến thức chuyên môn và các claim này khớp với hành vi AWS được ghi trong tài liệu — độ tin cậy cao, nhưng bạn nên xác nhận lại trên tài liệu gốc khi triển khai thật.

Nguồn gốc nằm ở [mục Nguồn](#nguồn) cuối bài.

---

## 1. AFT là gì và nằm ở đâu trong bức tranh?

**Một câu:** AFT là một **framework Terraform + pipeline do AWS phát hành**, chạy **bên trên** AWS Control Tower, để bạn **tạo và tùy biến AWS account bằng code theo mô hình GitOps** — thay vì bấm tay trong Console.

**Điểm cốt lõi phải khắc cốt ghi tâm:** ✅ **[VERIFIED 3-0]** *AFT bắt buộc phải có sẵn một Control Tower landing zone trước khi cài — AFT ngồi TRÊN Control Tower, không thay thế nó.* Control Tower vẫn là bên **thực sự tạo account**; AFT chỉ **điều phối + tùy biến**.

```
   ┌──────────────────────────────────────────────────────────────────┐
   │                      AWS ORGANIZATION                             │
   │                                                                  │
   │  ┌────────────────────────┐     ┌────────────────────────────┐   │
   │  │ Control Tower          │     │ AFT MANAGEMENT ACCOUNT      │   │
   │  │ MANAGEMENT ACCOUNT     │     │ (account RIÊNG, do AFT tạo) │   │
   │  │                        │     │                            │   │
   │  │ • Nơi Account Factory  │     │ • CodePipeline / CodeBuild  │   │
   │  │   THỰC SỰ tạo account  │◄────│ • DynamoDB (metadata + hàng │   │
   │  │   [VERIFIED 3-0]       │     │   đợi request)              │   │
   │  │ • Guardrails (SCP)     │     │ • SQS (queue FIFO)          │   │
   │  └────────────────────────┘     │ • Lambda / Step Functions   │   │
   │                                 └────────────────────────────┘   │
   └──────────────────────────────────────────────────────────────────┘
```

> ✅ **[VERIFIED 3-0]** Hai management account **tách biệt**: *"AFT management account KHÔNG phải là Control Tower management account"*. Đây là best practice bảo mật — bạn không chạy automation nặng trong CT management account (account quyền tối thượng). AFT có account riêng của nó.

**Ánh xạ on-prem:** Nếu bạn từng dùng một **provisioning server** (Ansible Tower / vRealize) để "đẻ" VM theo template chuẩn — AFT chính là "provisioning server để đẻ **cả AWS account**", còn Control Tower là hypervisor/nền tảng bên dưới thực sự tạo tài nguyên.

---

## 2. Luồng end-to-end: từ `git push` tới account sẵn sàng

📄 **[SOURCED]** Chuỗi sự kiện đầy đủ (nguồn: AWS provisioning-framework docs):

```
 1. Dev thêm/sửa 1 account request (file .tf) → git push vào repo aft-account-request
         │
         ▼
 2. Pipeline nhận request → đẩy vào DynamoDB + SQS (hàng đợi FIFO)   [VERIFIED 3-0: FIFO]
         │   → có thể submit NHIỀU request cùng lúc, xử lý lần lượt
         ▼
 3. Control Tower Account Factory TẠO account   ◄── chạy trong CT MGMT account [VERIFIED 3-0]
         │
         ▼
 4. AFT provisioning framework chạy chuỗi bước (trong AFT mgmt account):  [SOURCED]
      a) validate input của account request
      b) lấy account ID mới
      c) ghi metadata account vào DynamoDB
      d) tạo IAM role  AWSAFTExecution  trong account MỚI
      e) gắn tags
      f) áp các "feature options" đã chọn
      g) chạy PROVISIONING customizations (Step Functions, nếu có)
      h) TẠO + kích hoạt một CodePipeline RIÊNG cho account đó
         │
         ▼
 5. Trong per-account pipeline chạy theo thứ tự CỐ ĐỊNH:   [SOURCED]
      GLOBAL customizations  ──►  ACCOUNT customizations
      (áp cho MỌI account)        (áp riêng nhóm/account này)
         │
         ▼
 6. Account sẵn sàng: đã có VPC chuẩn, gắn TGW, IAM, logging, tag... đúng baseline.
```

> ✅ **[VERIFIED 3-0]** *"Global customizations chạy trong các pipeline được tạo cho từng vended account"* — tức mỗi account có pipeline riêng của nó, không phải một pipeline chung. Đây là lý do AFT **scale ngang** tốt: 200 account = 200 pipeline độc lập.

**Điểm hay cho phỏng vấn:** thứ tự **Global → Account** là **cố định, tự động** 📄 **[SOURCED]**. Bạn không đảo được. Vậy nên: thứ gì áp cho *tất cả* account (bật Flow Logs, gắn TGW, tag bắt buộc) → để ở **Global**; thứ gì đặc thù (baseline riêng cho nhóm "banking" vs "data") → để ở **Account**.

---

## 3. Cấu trúc Git: 4 lớp tùy biến (thực ra là 4–5 repo)

Đây là phần JD hỏi thẳng ("the four customization layers"). 📄 **[SOURCED]** AFT theo **GitOps**, dùng **4 repo tùy biến** (+ 1 repo deploy chính AFT = tổng 5 repo theo HashiCorp):

| Repo | Lớp | Áp cho | Chạy bằng | Ví dụ ngân hàng |
|------|-----|--------|-----------|-----------------|
| `aft-account-request` | **Account Request** | 1 account cụ thể | file `.tf` khai báo | tên, email, OU, chọn customization nào |
| `aft-global-customizations` | **Global** | **MỌI** account | Terraform / Python / Bash / AWS CLI | bật Flow Logs, gắn TGW, tag bắt buộc, IAM baseline |
| `aft-account-customizations` | **Account** | nhóm/account chọn qua tên | Terraform / script | baseline riêng "banking" (KMS, backup vault) vs "data" |
| `aft-account-provisioning-customizations` | **Provisioning** | trước customization, mọi account | **Step Functions** (`customizations.asl.json`) | gọi CMDB on-prem, xin IP từ IPAM, chờ phê duyệt ITSM |

```
   aft-account-request/                 ← "TÔI MUỐN account gì"
     └── payments-prod.tf  { name, email, ou="Prod",
                             account_customizations_name = "banking" }  [SOURCED]
                                              │
                                              ▼ chọn folder
   aft-account-customizations/
     ├── ACCOUNT_TEMPLATE/   (mẫu gốc để copy)
     ├── banking/            ← baseline riêng ngân hàng
     └── data/               ← baseline riêng data
   aft-global-customizations/           ← áp cho TẤT CẢ (chạy TRƯỚC account)
   aft-account-provisioning-customizations/  ← Step Functions (integrate on-prem)
```

> 📄 **[SOURCED]** *`account_customizations_name` trong account request phải KHỚP tên folder* (copy từ folder `ACCOUNT_TEMPLATE`) trong repo account-customizations. Sai tên = customization không áp. Đây là lỗi "gõ nhầm" hay gặp.

**Provisioning customizations — lớp mạnh nhất & bị bỏ quên nhất:** 📄 **[SOURCED]** nó là một **Step Functions state machine**. Nếu không định nghĩa → stage này **no-op** (bỏ qua). Nhưng nó cho phép tích hợp **Lambda, ECS/Fargate, và Step Functions Activities với worker ON-PREM** + SNS/SQS. → *Đây chính là điểm nối cloud với thế giới on-prem của bạn:* khi vend account, gọi ngược về hệ thống ITSM/CMDB/IPAM on-prem để xin IP, tạo ticket, chờ duyệt CAB — rồi mới tiếp tục. Rất "banking".

> 💡 **Best practice (AWS official)** 📄 **[SOURCED]**: *Hãy viết **wrapper module** bao quanh module gốc của AFT, ĐỪNG sửa nội tạng AFT.* Nhờ vậy khi nâng cấp AFT/Control Tower lên phiên bản mới, bạn **không** phát sinh xung đột. Đây là bài học vận hành quan trọng nhất để AFT "sống lâu".

---

## 4. Vận hành ở quy mô lớn (vend nhiều account)

- ✅ **[VERIFIED 3-0]** Có thể **submit nhiều account request cùng lúc**; AFT xử lý theo **FIFO**. → vend hàng loạt được, nhưng tuần tự (không song song vô hạn) — lập kế hoạch thời gian khi onboard 20 account một đợt.
- 📄 **[SOURCED]** AFT dùng **DynamoDB + SQS** làm hàng đợi → khắc phục giới hạn "Control Tower tạo từng-account-một". Bạn đẩy request vào queue, AFT rút ra xử lý.
- ✅ **[VERIFIED 3-0]** **Mỗi account một pipeline riêng** → thay đổi customization cho account A không đụng account B. Blast radius nhỏ.
- 📄 **[SOURCED]** Customization chạy được **Terraform, Python, Bash, hoặc AWS CLI** — linh hoạt, không bó buộc thuần Terraform.

**Re-run (áp lại customization):** 📄 **[SOURCED]** provisioning framework + customizations chạy cho **mọi** account được vend, và **re-run theo 2 cách**:
1. **Sửa bất kỳ thứ gì** trong account request repo của account đó (kể cả một tag) → kích hoạt lại pipeline.
2. Vend một account mới.

> Thực tế: nếu bạn cập nhật **global customization** (vd thêm một Config rule cho tất cả), bạn cần **kích lại pipeline của từng account** để áp thay đổi. AWS/cộng đồng thường viết một job "re-run all accounts" quét bảng DynamoDB và trigger lần lượt. Đây là "tác vụ vận hành" điển hình bạn sẽ tự động hoá bằng Python/Bash (đúng như JD mô tả).

---

## 5. Drift & những cái bẫy thực chiến (phần "vàng" khi phỏng vấn)

> ⚠️ **[SOURCED] Điểm mù drift lớn nhất:** *AFT KHÔNG phát hiện drift khi ai đó **đổi OU của account thủ công**.* Lý do rất tinh tế: AFT quản lý một **item trong DynamoDB (account request)**, **không** quản lý account trực tiếp. Bạn đổi OU trên Console → *account request trong DynamoDB không đổi* → AFT nghĩ "mọi thứ vẫn khớp" → **không** báo drift, **không** kéo về. (nguồn: AWS troubleshooting guide)
>
> **Hệ quả cho ngân hàng:** đừng tin AFT là "chốt chặn drift". Phải bổ sung **AWS Config / detective guardrails / kiểm tra định kỳ** để bắt thay đổi ngoài luồng. Đây là lỗ hổng governance nếu không biết.

**Các bẫy khác:**

- ⚠️ **[SOURCED]** **Trùng email hoặc tên account** → provisioning fail (`ConditionalCheckFailedException` / lỗi Service Catalog). Chi tiết lỗi hiện qua CloudWatch log group + SNS topic **`aft-failure-notifications`**. → *Best practice:* subscribe topic này vào Slack/PagerDuty để biết ngay khi vend fail; quản lý email account tập trung (email alias theo pattern, vd `aws+payments-prod@bank.com`).

- ⚠️ **[SOURCED] Bug `IndexError`** (GitHub issue #404 của repo AFT): pipeline customization có thể lỗi `IndexError` khi pipeline của account đó **chưa từng chạy trong ~1 năm** (giới hạn lưu lịch sử execution của CodePipeline) — hàm `pipeline_is_running` giả định luôn có ít nhất 1 lần chạy trước. → account "ngủ đông" lâu ngày có thể vấp lỗi này khi re-run. Bài học: AFT là **phần mềm có bug như mọi phần mềm** — theo dõi GitHub issues của `aws-ia/terraform-aws-control_tower_account_factory`.

- **Nâng cấp AFT** (kinh nghiệm cộng đồng): AFT phát hành theo version; nâng cấp cần đọc release notes kỹ vì đôi khi đổi biến/format. → củng cố lời khuyên **wrapper module** ở Mục 3: giữ code của bạn tách khỏi nội tạng AFT.

---

## 6. Quyết định lớn: AFT vs LZA vs Control Tower Account Factory

Câu "nhiều tool 1 bài" kinh điển. Tổng hợp từ các nguồn so sánh (Meshcloud, Sierra-Cedar, towardsthecloud) 📄 **[SOURCED]** + phân tích của mình:

| Tiêu chí | **Control Tower Account Factory** (gốc) | **AFT** | **Landing Zone Accelerator (LZA)** |
|----------|------------------------------------------|---------|-------------------------------------|
| Cách tạo account | Bấm Console / Service Catalog | **GitOps (Terraform + pipeline)** | GitOps (CloudFormation/CDK là chính) |
| IaC | Không (thủ công) | **Terraform** ✅ | Thiên về CloudFormation/CDK |
| Độ phức tạp cài đặt | Thấp | Trung bình | **Cao** |
| Tùy biến sâu (mạng, security phức tạp, nhiều Region) | Hạn chế | Tốt | **Rất sâu** (chính phủ, defense, healthcare) |
| Ai vận hành | AWS lo phần lõi | Bạn + AWS (AFT là open-source AWS-IA) | Bạn (nặng) |
| Hợp với | Bắt đầu nhanh, ít account | **Doanh nghiệp/ngân hàng dùng Terraform** ✅ | Yêu cầu compliance cực đặc thù |

```
   Ít tùy biến, nhanh ◄──────────────────────────────► Tùy biến tối đa, nặng

   Account Factory          AFT                         LZA
   (Console)                (Terraform GitOps)          (CFN/CDK, guardrails
                                                         đóng gói cực nhiều)
                             ▲ JD ACloudBank ở đây
```

> 🎯 **Kết luận cho JD ngân hàng của bạn:**
> - Nếu tổ chức **chuẩn hoá Terraform** (như JD nêu) → **AFT** là lựa chọn tự nhiên: account = code, có pipeline, review, audit.
> - Chọn **LZA** khi bạn cần bộ guardrail/khung compliance **đóng gói sẵn cực rộng** (nhiều framework: NIST, PCI, HIPAA...) và chấp nhận vận hành nặng, và đội ngũ mạnh về CloudFormation/CDK.
> - **Account Factory gốc** chỉ hợp giai đoạn đầu / tổ chức nhỏ.
>
> **AFT và LZA giải cùng bài toán "landing zone at scale" nhưng khác triết lý công cụ** (Terraform-first vs CFN/CDK-first). Chọn theo **kỹ năng đội ngũ + hệ sinh thái IaC hiện có**, không theo "cái nào mới hơn".

---

## 7. Góc nhìn Banking/Regulated (vì sao JD tuyển AFT)

Ghép AFT với yêu cầu ngân hàng:

- **4-eyes / CAB:** account = Merge Request → review diff → duyệt → pipeline apply. Mọi account "sinh ra" đều có **dấu vết Git** (ai xin, ai duyệt, khi nào). Kiểm toán viên thích điều này.
- **Baseline bắt buộc:** Global customizations ép **mọi** account mới có sẵn CloudTrail, Config, GuardDuty, KMS, backup, tag phân loại dữ liệu — **không account nào "lọt" ra ngoài chuẩn**.
- **Tích hợp on-prem:** Provisioning customizations (Step Functions + worker on-prem) nối vào ITSM/CMDB/IPAM hiện có — đúng thế mạnh của bạn.
- **Guardrail phòng ngừa:** AFT áp account vào **OU đúng** → OU đó gắn **SCP** → chặn từ gốc (region, tắt logging...). AFT + SCP + Config = 3 lớp governance.

---

## 8. Việc thực hành đề xuất (để nói được "hands-on" khi phỏng vấn)

1. Bật **Control Tower** trong một Organization sandbox (dùng account cá nhân + vài email alias).
2. Cài **AFT** theo `aws-ia/terraform-aws-control_tower_account_factory` (Terraform module chính thức).
3. Vend 1 account qua `git push` một account request.
4. Viết 1 **Global customization** đơn giản (vd gắn tag + bật Flow Logs) và xem nó áp tự động.
5. Cố tình vend trùng email → xem lỗi hiện ở `aft-failure-notifications`.
6. Thử **wrapper module** thay vì sửa AFT trực tiếp.

Làm xong 6 bước này, bạn kể được "trải nghiệm vận hành thật" — thứ nhà tuyển dụng banking muốn nghe.

---

## Nguồn

**Tài liệu chính thức (primary):**
1. AWS Control Tower — AFT Architecture: https://docs.aws.amazon.com/controltower/latest/userguide/aft-architecture.html *(nguồn của các claim VERIFIED 3-0)*
2. AWS — AFT Overview: https://docs.aws.amazon.com/controltower/latest/userguide/aft-overview.html
3. AWS — AFT Provisioning Framework: https://docs.aws.amazon.com/controltower/latest/userguide/aft-provisioning-framework.html
4. AWS — AFT Account Customization Options: https://docs.aws.amazon.com/controltower/latest/userguide/aft-account-customization-options.html
5. AWS — Account Troubleshooting Guide (drift, lỗi vend): https://docs.aws.amazon.com/controltower/latest/userguide/account-troubleshooting-guide.html
6. AWS MT Blog — Deploy and customize accounts using AFT: https://aws.amazon.com/blogs/mt/deploy-and-customize-aws-accounts-using-account-factory-for-terraform-in-aws-control-tower/
7. AWS MT Blog — Manage your multi-account environment with AFT (wrapper module practice): https://aws.amazon.com/blogs/mt/manage-your-aws-multi-account-environment-with-account-factory-for-terraform-aft/
8. HashiCorp — AWS Control Tower + AFT tutorial: https://developer.hashicorp.com/terraform/tutorials/aws/aws-control-tower-aft
9. GitHub Issue #404 (bug IndexError): https://github.com/aws-ia/terraform-aws-control_tower_account_factory/issues/404

**So sánh / thực chiến (blog — tham khảo, chưa kiểm chứng độc lập):**
10. Meshcloud — AWS Landing Zone comparison: https://www.meshcloud.io/en/blog/aws-landing-zone-comparison/
11. Sierra-Cedar — AFT vs LZA: https://www.sierra-cedar.com/2023/06/13/aws-account-factory-for-terraform-vs-aws-landing-zone-accelerator/
12. towardsthecloud — Control Tower alternatives: https://towardsthecloud.com/blog/aws-control-tower-alternatives
13. Rackspace — Scaling landing zone customizations: https://www.rackspace.com/blog/scaling-landing-zone-customizations-on-aws
14. OpenCredo, Presidio, Medium (practical-aws) — lessons from the trenches (xem danh sách đầy đủ trong transcript research).

> *Trạng thái xác minh: 6 claim lõi VERIFIED 3-0 (AWS docs). Phần còn lại SOURCED từ nguồn chính thức; vòng kiểm chứng đối kháng bị ngắt do giới hạn phiên (reset 6:20am UTC) — có thể chạy lại `deep-research` để xác minh đầy đủ nếu cần.*
