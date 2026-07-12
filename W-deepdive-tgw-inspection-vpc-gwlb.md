# 🕸️ W. DEEP DIVE: TRANSIT GATEWAY + INSPECTION VPC + GWLB (THỰC CHIẾN)

> **Chủ đề hẹp #2** trong loạt đào sâu cho JD Cloud Ops. Đây là nơi **kỹ năng mạng/firewall on-prem của bạn nối thẳng vào cloud** — hub-and-spoke tập trung + chèn firewall Palo Alto kiểm tra mọi luồng traffic.
>
> **Cho ai:** người mạnh mạng on-prem (routing, firewall zone, NAT, IPsec), cần hiểu cách AWS "ráp" các mảnh này lại và các bẫy vận hành + chi phí.

---

### ⚠️ Ghi chú về độ tin cậy (đọc trước)

Báo cáo này tổng hợp từ một lần chạy **deep-research**. Vòng **kiểm chứng đối kháng bị chặn hoàn toàn do đụng giới hạn sử dụng phiên** (reset 6:20am UTC) — nên **không claim nào có nhãn VERIFIED tự động**. Tuy nhiên toàn bộ claim được **trích từ 3 nguồn có thẩm quyền**:
- 📄 **Palo Alto Networks official docs** (VM-Series + GWLB integration) — *primary*
- 📄 **AWS APN Blog** (Centralized traffic inspection with GWLB) — *primary*
- 📝 **Blog thực chiến** (haitmg.pl — Palo Alto VM-Series + TGW + GWLB) — *practitioner*

Mình đã **đối chiếu từng claim với kiến thức chuyên môn** và tất cả khớp với hành vi AWS/Palo Alto được ghi trong tài liệu. Nhãn dùng trong bài:
- 📄 **[SOURCED-PRIMARY]** = từ AWS/Palo Alto docs, độ tin cậy cao.
- 📝 **[SOURCED-BLOG]** = từ blog thực chiến, hợp lý về kỹ thuật nhưng **nên xác nhận lại** khi triển khai thật.

---

## 1. Bức tranh & ánh xạ On-Prem

**Bài toán:** ngân hàng có hàng chục VPC + kết nối on-prem + đối tác. Cần: (a) mọi VPC nối nhau có kiểm soát, (b) **mọi** traffic (Bắc-Nam ra Internet và Đông-Tây giữa các VPC) đều bị **firewall Palo Alto kiểm tra tập trung** — giống mô hình "mọi thứ đi qua cụm firewall lõi" ở on-prem.

```
On-Prem bạn đã biết                 AWS tương đương
────────────────────                ─────────────────────────────
Core router / L3 switch hub    →     Transit Gateway (TGW)
VRF / zone tách biệt           →     TGW Route Tables (phân đoạn)
Cụm firewall lõi (HA pair)     →     Palo Alto VM-Series fleet sau GWLB
"Đẩy traffic qua firewall"     →     GWLB Endpoint (GWLBE) + route table
Đối xứng luồng (stateful)      →     TGW Appliance Mode
IPsec tới site khác            →     Site-to-Site VPN
Leased line / MPLS             →     Direct Connect (DX)
```

---

## 2. Transit Gateway: hub + phân đoạn bằng Route Tables

**TGW = core-switch của cả tổ chức.** Mọi VPC, VPN, DX chỉ **cắm một lần** vào TGW (attachment), thay vì full-mesh peering.

Sức mạnh thật nằm ở **nhiều TGW Route Table** → **micro-segmentation**. Mỗi attachment (VPC) **associate** với một route table và **propagate** route vào các route table khác một cách có chọn lọc → bạn quyết định "ai thấy ai".

```
                    ┌───────────────── TRANSIT GATEWAY ─────────────────┐
                    │                                                   │
   Prod VPC ───────►│ assoc: RT-Prod      ┌──────────────────────────┐ │
   Non-Prod VPC ───►│ assoc: RT-NonProd   │ Nguyên tắc phân đoạn:     │ │
   Shared VPC ─────►│ assoc: RT-Shared    │ • Prod ↔ Shared: CHO      │ │
   Inspection VPC ─►│ assoc: RT-Inspect   │ • Prod ↔ Non-Prod: CẤM    │ │
                    │                     │ • Mọi thứ ra Internet:    │ │
                    │                     │   ÉP qua Inspection VPC   │ │
                    │                     └──────────────────────────┘ │
                    └───────────────────────────────────────────────────┘
```

**Ánh xạ on-prem:** TGW route table = **VRF**. "Prod không route thẳng sang Non-Prod" = bạn không leak route giữa 2 VRF. Khác biệt: trên AWS bạn làm bằng association/propagation thay vì `route-target import/export`.

---

## 3. Inspection VPC + GWLB + GENEVE: cơ chế chèn firewall

Đây là phần "ma thuật" cần hiểu kỹ. **Gateway Load Balancer (GWLB)** cho phép chèn một **fleet firewall ảo** vào đường đi gói tin một cách **trong suốt**, đồng thời **scale + HA** cho fleet đó.

📄 **[SOURCED-PRIMARY]** GWLB và các firewall đích **trao đổi traffic bằng giao thức GENEVE** (encapsulation). Firewall phải **decapsulate** (bóc vỏ) để thấy gói tin gốc. 📄 **[SOURCED-PRIMARY]** GENEVE **giữ nguyên header + payload gói gốc** → firewall thấy **danh tính nguồn thật**, inspect đầy đủ.

📝 **[SOURCED-BLOG]** GENEVE chạy trên **UDP port 6081**.

```
   Cách traffic bị "lái" qua firewall (dùng GWLB Endpoint — GWLBE):

   [Nguồn] ──► route table trỏ tới ──► GWLBE (trong subnet riêng)
                                          │  bọc GENEVE (UDP 6081)
                                          ▼
                                    ┌───────────┐
                                    │   GWLB    │ (chia tải theo flow)
                                    └─────┬─────┘
                            ┌─────────────┼─────────────┐
                            ▼             ▼             ▼
                        PA-VM (AZ-a)  PA-VM (AZ-b)  PA-VM (AZ-c)
                        (bóc GENEVE, threat prevention, rồi trả lại)
                                          │
                                          ▼
                                    [Đích] (đi tiếp sau khi sạch)
```

📄 **[SOURCED-PRIMARY]** Hai luồng điển hình:

**Bắc-Nam (ra Internet):**
```
App VPC ──► TGW ──► Security/Inspection VPC: GWLBE ──► VM-Series
                                                          │ (sạch)
                                                          ▼
                                                    NAT Gateway ──► Internet Gateway ──► Internet
```
**Đông-Tây (VPC ↔ VPC):**
```
App-A VPC ──► TGW ──► GWLBE ──► VM-Series (inspect) ──► TGW ──► App-B VPC
```

---

## 4. Appliance Mode: chống asymmetric routing (điểm dễ sai nhất)

**Vấn đề chí mạng:** firewall là **stateful** — chiều đi và chiều về của **cùng một flow phải qua CÙNG một firewall instance**. Nếu chiều đi qua PA-VM ở AZ-a nhưng chiều về qua PA-VM ở AZ-b → firewall AZ-b không có "state" của flow đó → **drop gói** (asymmetric routing). Ở on-prem bạn xử lý bằng HA active/passive hoặc session sync; trên AWS/TGW có cơ chế riêng.

📄 **[SOURCED-PRIMARY]** **Giải pháp: bật Appliance Mode trên TGW attachment của Inspection VPC.** Khi bật:
- 📄 TGW **hash mỗi flow tới một ENI cố định** trong security VPC **suốt vòng đời flow** → đảm bảo 2 chiều đi qua **cùng AZ / cùng instance** → hết asymmetric.
- 📝 **[SOURCED-BLOG]** Appliance Mode **mặc định TẮT**, phải **bật thủ công**. Nó chuyển TGW từ **AZ-affinity forwarding** sang **flow-hash forwarding trên 4-tuple**.

```
   KHÔNG appliance mode                 CÓ appliance mode
   ────────────────────                 ────────────────────
   đi:  qua PA-VM (AZ-a)                đi:  qua PA-VM (AZ-a)
   về:  qua PA-VM (AZ-b)  ✗ DROP        về:  qua PA-VM (AZ-a)  ✓ OK
   (firewall b không có state)          (flow hash cố định 1 ENI)
```

> ⚠️ **[SOURCED-BLOG] Bẫy kiến trúc quan trọng:** Appliance Mode set **per TGW-VPC attachment**, và **mỗi VPC chỉ được 1 attachment**. Vì vậy một số thiết kế tách **HAI security VPC**: appliance mode **BẬT** cho VPC xử lý Đông-Tây, **TẮT** cho VPC xử lý Bắc-Nam. → nếu gộp chung dễ vấp giới hạn này. (đối chiếu thêm khi thiết kế thật)

---

## 5. Egress tập trung vs phân tán + NAT ở đâu?

**Centralized egress:** mọi traffic ra Internet của tất cả VPC đi qua **một chỗ** (Inspection/Egress VPC) → 1 nơi đặt NAT Gateway + firewall + kiểm soát. Ngược lại **distributed egress** = mỗi VPC có NAT riêng.

| | Centralized egress | Distributed egress |
|--|--------------------|--------------------|
| Kiểm soát/inspect | ✅ Một điểm, dễ áp chính sách | Rải rác, khó |
| Chi phí NAT | ✅ Ít NAT Gateway hơn | Nhiều NAT → tốn |
| Chi phí TGW | ❌ Trả phí TGW data processing cho traffic egress | Không qua TGW |
| Độ trễ / blast radius | Thêm 1 chặng; điểm tập trung | Ngắn hơn, phân tán rủi ro |

**NAT nên đặt ở đâu — firewall hay NAT Gateway?** Điểm cực dễ sai:

- 📝 **[SOURCED-BLOG]** ⚠️ **Đừng cấu hình DNAT trên VM-Series behind GWLB** — nó **âm thầm phá GWLB**, vì GWLB kiểm tra **5-tuple của gói trả về** so với bảng state; NAT trên appliance làm lệch 5-tuple. **Khuyến nghị: dùng AWS NAT Gateway cho outbound translation**, không NAT trên firewall.
- 📄 **[SOURCED-PRIMARY]** Ngược lại, AWS APN blog nêu một phương án tối ưu chi phí: **để firewall tự NAT (overlay routing)** thay vì route tới NAT Gateway riêng → giảm chi phí NAT Gateway.

> 🎯 **Kết luận thực dụng:** hai nguồn nói hai hướng vì **phụ thuộc mô hình triển khai** (GWLB inline vs overlay routing / phiên bản PAN-OS). **Best practice an toàn cho người mới:** đi theo mẫu tham chiếu chính thức của Palo Alto cho đúng phiên bản PAN-OS bạn dùng, và **mặc định dùng NAT Gateway** cho outbound (tránh DNAT-trên-appliance) trừ khi mẫu của vendor nói khác. Test kỹ luồng 2 chiều trước khi lên Prod.

---

## 6. DNS tập trung (Route 53 Resolver) — hybrid

Trong hub tập trung, DNS cũng nên tập trung ở **Shared Services / Network Hub account**:
- **Inbound Endpoint:** on-prem resolve được tên riêng trên AWS.
- **Outbound Endpoint + Forwarding Rules:** AWS forward query tên on-prem (`*.corp.local`) về DNS on-prem.
- **Private Hosted Zones** share cho các VPC qua RAM.

*(Chi tiết cơ chế DNS hybrid đã có trong `U-cloudops-jd-mo-xe-cong-nghe.md` Phần 2.3 — ở đây chỉ nhấn: đặt resolver endpoints tập trung, đừng để mỗi account tự cấu hình lệch nhau.)*

---

## 7. Hybrid: Direct Connect vs Site-to-Site VPN

📝 **[SOURCED-BLOG]** Một lý do quan trọng chọn **GWLB thay vì mô hình cũ**: cách cũ nối firewall vào TGW qua **Site-to-Site VPN + BGP** bị **giới hạn 1.25 Gbps mỗi tunnel** — GWLB gỡ bỏ nút thắt băng thông này (inspection chạy trong VPC, không qua tunnel VPN).

| | Site-to-Site VPN | Direct Connect (DX) |
|--|------------------|---------------------|
| Đường | Internet, mã hoá IPsec | Cáp riêng vật lý |
| Băng thông/tunnel | ~1.25 Gbps/tunnel 📝 | tới 100 Gbps, ổn định |
| Setup | Giờ | Tuần (kéo cáp) |
| Vai trò ngân hàng | Backup / đối tác nhỏ | **Đường chính** Core Banking |

> **Mẫu chuẩn ngân hàng:** DX làm chính + VPN backup (fail-over khi cáp đứt).

---

## 8. Bẫy vận hành & chi phí (phần "vàng")

- ⚠️ 📝 **[SOURCED-BLOG] VPC Flow Logs "mù" với traffic inspection:** GENEVE encapsulation (UDP 6081) **che địa chỉ thật** → Flow Logs của Inspection VPC **kém hữu ích** để debug. → phải debug bằng **log trên chính Palo Alto** (traffic log, threat log), không dựa Flow Logs ở khúc GENEVE.
- ⚠️ 📝 **[SOURCED-BLOG] GWLB fail-open:** khi **tất cả** firewall target **unhealthy**, GWLB có thể vào chế độ **fail-open = bỏ qua inspection** (traffic vẫn chạy nhưng KHÔNG được kiểm tra) → **rủi ro bảo mật âm thầm**. → phải có **alarm health của target group** bắn PagerDuty ngay; cân nhắc fail-close cho môi trường nhạy cảm (tùy cấu hình/vendor).
- 💰 📝 **[SOURCED-BLOG] Chi phí ẩn:**
  - **Cross-AZ data transfer**: ~**$0.01/GB mỗi chiều** — inspection tập trung dễ tạo traffic chéo AZ.
  - **TGW data processing**: ~**$0.02/GB** — mọi gói qua TGW đều tính phí. Ở quy mô ngân hàng con số này **rất lớn**.
  - → *Best practice:* thiết kế để **firewall cùng AZ với nguồn/đích** khi có thể (giảm cross-AZ), và cân nhắc egress phân tán cho traffic cực lớn nếu phí TGW vượt lợi ích tập trung.
- ⚠️ 📝 **[SOURCED-BLOG] Multi-account & AZ-ID:** tên AZ (`ap-southeast-1a`) **khác nhau giữa các account** cho cùng một AZ vật lý. Thiết kế appliance-mode/inspection phải tham chiếu **AZ ID** (vd `use1-az1` / `apse1-az1`) **chứ không phải AZ name**, để firewall nằm đúng AZ vật lý với workload. → lỗi kinh điển khi làm cross-account.
- 📄 **[SOURCED-PRIMARY] SSL/TLS decryption:** VM-Series behind GWLB **hỗ trợ decrypt cả forward & inbound**, gồm **TLS 1.2 và TLS 1.3 (DHE/ECDHE)** → làm được threat prevention trên traffic mã hoá (yêu cầu điển hình của ngân hàng). Lưu ý tải CPU tăng khi bật decrypt.

---

## 9. Tổng hợp: kiến trúc tham chiếu ACloudBank

```
                                   Internet
                                      │
   ┌──────────────────────────────────┼─────────────────────────────────────┐
   │  INSPECTION / EGRESS VPC (Network Hub account)                          │
   │     IGW ── NAT GW ── VM-Series fleet (PA) ◄─ GWLB ◄─ GWLBE              │
   │            (outbound NAT)   ↑ threat prevent, TLS decrypt                │
   │     Appliance Mode BẬT trên TGW attachment (chống asymmetric)           │
   └───────────────────────────────────┬─────────────────────────────────────┘
                                       │ TGW attachment
                        ┌────────── TRANSIT GATEWAY ──────────┐
                        │  RT phân đoạn: Prod/NonProd/Shared  │
                        │  mọi egress + E-W ÉP qua Inspection │
                        └───┬───────────┬───────────┬─────────┘
                            │           │           │
                    Prod VPC     Non-Prod VPC   Shared Svc VPC
                    (CoreBank)   (Dev/Test)     (R53 Resolver, DNS)
                            │
                 DX (chính) + VPN (backup)
                            ▼
                    DATA CENTER ON-PREM
```

---

## Nguồn

**Primary:**
1. Palo Alto Networks — VM-Series integration with Gateway Load Balancer: https://docs.paloaltonetworks.com/vm-series/10-1/vm-series-deployment/set-up-the-vm-series-firewall-on-aws/vm-series-integration-with-gateway-load-balancer *(GENEVE giữ header, appliance mode, TLS 1.2/1.3 decrypt, luồng N-S/E-W)*
2. AWS APN Blog — Centralized traffic inspection with Gateway Load Balancer on AWS: https://aws.amazon.com/blogs/apn/centralized-traffic-inspection-with-gateway-load-balancer-on-aws/ *(GENEVE decap, appliance mode flow-hash, 2 security VPC, VPN 1.25Gbps, firewall NAT overlay)*

**Practitioner (blog — hợp lý kỹ thuật, nên xác nhận lại):**
3. haitmg.pl — Palo Alto VM-Series + AWS Transit Gateway + GWLB: https://haitmg.pl/blog/palo-alto-vm-series-aws-transit-gateway-gwlb/ *(GENEVE UDP 6081, Flow Logs mù, DNAT phá GWLB, fail-open, chi phí cross-AZ/TGW, AZ-ID vs AZ-name)*

> *Trạng thái xác minh: vòng kiểm chứng đối kháng bị chặn hoàn toàn do giới hạn phiên (reset 6:20am UTC) → không có nhãn VERIFIED tự động. Toàn bộ claim trích từ nguồn nêu trên và đã được đối chiếu thủ công với kiến thức chuyên môn; các claim [SOURCED-BLOG] nên được xác nhận lại trên tài liệu vendor cho đúng phiên bản PAN-OS khi triển khai thật. Có thể chạy lại `deep-research` sau khi hết giới hạn để có bản kiểm chứng đầy đủ.*
