# 🛠️ Hands-On Lab — Xây "ShopVN" trên Azure từ số 0

> Đây là **phần thực hành đi kèm** [DevOps-Azure-JD-DeepDive.md](./DevOps-Azure-JD-DeepDive.md).
> Deep-dive dạy *tại sao*; file này dạy *làm thế nào* bằng **code thật, gõ được, chạy được**.
>
> Mục tiêu: đi hết 1 vòng — từ hạ tầng (Terraform) → app (Docker) → chạy (AKS) → tự động (CI/CD) → quan sát (OpenTelemetry) → bảo mật + tối ưu. Sau lab này bạn có **1 project để mang đi phỏng vấn**.
>
> ⚠️ Lab tốn tiền Azure thật (AKS, VPN...). Dùng **free trial/credit** và **`terraform destroy` sau khi tập** để khỏi cháy ví (bài học FinOps đầu tiên 😄).

---

## 📋 Chuẩn bị (prerequisites)

```bash
# Cài công cụ (Linux/macOS)
az --version           # Azure CLI
terraform --version    # >= 1.6 (hoặc dùng OpenTofu: tofu --version)
kubectl version --client
helm version
docker --version

# Đăng nhập Azure
az login
az account set --subscription "<SUBSCRIPTION_ID>"
```

**Bản đồ lab (10 bước):**
```
   Bước 1  Remote state backend (Storage cho Terraform)
   Bước 2  VNet + Subnet + NSG            ─┐
   Bước 3  AKS cluster                     │  HẠ TẦNG (Terraform)
   Bước 4  Azure SQL + Private Endpoint    │
   Bước 5  Container Registry (ACR)       ─┘
   Bước 6  Dockerize app + push ACR
   Bước 7  Deploy lên AKS (Helm/manifest)
   Bước 8  CI/CD pipeline (GitHub Actions)  ← tự động hóa bước 6-7
   Bước 9  Observability (OpenTelemetry + Prometheus + Grafana)
   Bước 10 DevSecOps + FinOps guardrails
```

---

## Bước 1 — Remote state backend

**Vì sao:** Terraform lưu "trạng thái thật" vào state file. Để state trên máy = mất là toi + không team-work được. Best practice: để trên **Azure Storage + lock**.

```bash
# Tạo storage chứa state (chạy 1 lần, bằng CLI vì "con gà - quả trứng")
az group create -n rg-tfstate -l southeastasia
az storage account create -n stshopvntfstate -g rg-tfstate -l southeastasia --sku Standard_LRS
az storage container create -n tfstate --account-name stshopvntfstate
```

`providers.tf`:
```hcl
terraform {
  required_version = ">= 1.6"
  required_providers {
    azurerm = { source = "hashicorp/azurerm", version = "~> 3.100" }
  }
  backend "azurerm" {
    resource_group_name  = "rg-tfstate"
    storage_account_name = "stshopvntfstate"
    container_name       = "tfstate"
    key                  = "shopvn.tfstate"    # ← lock tự động qua blob lease
  }
}
provider "azurerm" { features {} }
```

---

## Bước 2 — VNet + Subnet + NSG

**Ánh xạ on-prem:** VNet = LAN ảo, Subnet = VLAN, NSG = firewall.

```
   VNet 10.10.0.0/16
   ├── snet-aks   10.10.1.0/24   (chứa node AKS)
   └── snet-data  10.10.2.0/24   (chứa Private Endpoint của SQL)
```

`network.tf`:
```hcl
resource "azurerm_resource_group" "main" {
  name     = "rg-shopvn"
  location = "southeastasia"
}

resource "azurerm_virtual_network" "main" {
  name                = "vnet-shopvn"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  address_space       = ["10.10.0.0/16"]
}

resource "azurerm_subnet" "aks" {
  name                 = "snet-aks"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.10.1.0/24"]
}

resource "azurerm_subnet" "data" {
  name                 = "snet-data"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.10.2.0/24"]
}

# NSG: mặc định Azure cho phép trong VNet, chặn từ Internet. Ta siết thêm.
resource "azurerm_network_security_group" "aks" {
  name                = "nsg-aks"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "deny-ssh-from-internet"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Deny"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "Internet"
    destination_address_prefix = "*"
  }
}

resource "azurerm_subnet_network_security_group_association" "aks" {
  subnet_id                 = azurerm_subnet.aks.id
  network_security_group_id = azurerm_network_security_group.aks.id
}
```

---

## Bước 3 — AKS cluster

`aks.tf`:
```hcl
resource "azurerm_kubernetes_cluster" "main" {
  name                = "aks-shopvn"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  dns_prefix          = "shopvn"

  default_node_pool {
    name                = "system"
    vm_size             = "Standard_D2s_v5"
    vnet_subnet_id      = azurerm_subnet.aks.id
    auto_scaling_enabled = true          # ← Cluster Autoscaler (FinOps + scale)
    min_count           = 1
    max_count           = 3
  }

  identity { type = "SystemAssigned" }   # ← không dùng service principal + secret

  network_profile {
    network_plugin = "azure"
    service_cidr   = "10.20.0.0/16"      # tránh trùng dải VNet
    dns_service_ip = "10.20.0.10"
  }
}

output "kube_config" {
  value     = azurerm_kubernetes_cluster.main.kube_config_raw
  sensitive = true
}
```

Áp dụng & lấy credential:
```bash
terraform init
terraform plan     # ← luôn xem plan trước khi apply
terraform apply
az aks get-credentials -g rg-shopvn -n aks-shopvn
kubectl get nodes  # thấy node là thành công
```

---

## Bước 4 — Azure SQL + Private Endpoint

**Điểm mấu chốt bảo mật:** SQL **không** có public endpoint; app chỉ chạm qua IP private trong VNet.

`sql.tf`:
```hcl
resource "azurerm_mssql_server" "main" {
  name                         = "sql-shopvn-01"
  resource_group_name          = azurerm_resource_group.main.name
  location                     = azurerm_resource_group.main.location
  version                      = "12.0"
  administrator_login          = "shopadmin"
  administrator_login_password = var.sql_password   # ← truyền qua Key Vault/TF_VAR, KHÔNG hardcode
  public_network_access_enabled = false             # ← khóa public
}

resource "azurerm_mssql_database" "main" {
  name      = "shopdb"
  server_id = azurerm_mssql_server.main.id
  sku_name  = "S0"
}

resource "azurerm_private_endpoint" "sql" {
  name                = "pe-sql-shopvn"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  subnet_id           = azurerm_subnet.data.id      # ← IP private nằm ở snet-data

  private_service_connection {
    name                           = "psc-sql"
    private_connection_resource_id = azurerm_mssql_server.main.id
    subresource_names              = ["sqlServer"]
    is_manual_connection           = false
  }
}
```
> 🔑 Bài học: `public_network_access_enabled = false` + Private Endpoint = "private by default". Đây là câu trả lời chuẩn cho mọi câu hỏi "làm sao để DB không lộ ra Internet".

---

## Bước 5 — Azure Container Registry (ACR)

```hcl
resource "azurerm_container_registry" "main" {
  name                = "acrshopvn01"     # phải unique toàn cầu, chỉ chữ+số
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  sku                 = "Standard"
}

# Cho AKS quyền kéo image từ ACR (không cần password)
resource "azurerm_role_assignment" "aks_acr" {
  scope                = azurerm_container_registry.main.id
  role_definition_name = "AcrPull"
  principal_id         = azurerm_kubernetes_cluster.main.kubelet_identity[0].object_id
}
```

---

## Bước 6 — Dockerize app + push ACR

App mẫu `payment` (Node.js). `Dockerfile` (multi-stage + non-root + distroless = nhỏ, ít CVE):
```dockerfile
# ---- build stage ----
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .

# ---- runtime stage ----
FROM gcr.io/distroless/nodejs20-debian12
WORKDIR /app
COPY --from=build /app .
USER 1000                      # ← không chạy root
EXPOSE 3000
CMD ["server.js"]
```

Build & push:
```bash
az acr login -n acrshopvn01
docker build -t acrshopvn01.azurecr.io/payment:v1 .
docker push acrshopvn01.azurecr.io/payment:v1
```

---

## Bước 7 — Deploy lên AKS

`k8s/payment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: payment }
spec:
  replicas: 3
  selector: { matchLabels: { app: payment } }
  template:
    metadata: { labels: { app: payment } }
    spec:
      containers:
        - name: payment
          image: acrshopvn01.azurecr.io/payment:v1
          ports: [{ containerPort: 3000 }]
          resources:                       # ← BẮT BUỘC: tránh 1 pod ăn hết node
            requests: { cpu: "100m", memory: "128Mi" }
            limits:   { cpu: "500m", memory: "256Mi" }
          readinessProbe:                  # ← chỉ nhận traffic khi sẵn sàng
            httpGet: { path: /health, port: 3000 }
            initialDelaySeconds: 5
          livenessProbe:                   # ← tự restart khi treo
            httpGet: { path: /health, port: 3000 }
            initialDelaySeconds: 10
---
apiVersion: v1
kind: Service
metadata: { name: payment }
spec:
  selector: { app: payment }
  ports: [{ port: 80, targetPort: 3000 }]
---
apiVersion: autoscaling/v2               # ← HPA: tự scale pod theo CPU
kind: HorizontalPodAutoscaler
metadata: { name: payment }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: payment }
  minReplicas: 3
  maxReplicas: 30                         # ← ngày sale 12.12 tự phình ra
  metrics:
    - type: Resource
      resource: { name: cpu, target: { type: Utilization, averageUtilization: 70 } }
```
```bash
kubectl apply -f k8s/payment.yaml
kubectl get pods,hpa
```

**Luồng lúc sale (ASCII):**
```
   Tải tăng → CPU pod > 70% → HPA thêm pod (3→30)
   Node hết chỗ → Cluster Autoscaler thêm node (1→3)
   Tải giảm → thu pod & node lại → tiết kiệm tiền
```

---

## Bước 8 — CI/CD pipeline (GitHub Actions)

Tự động hóa bước 6-7 + chèn bảo mật. `.github/workflows/deploy.yml`:
```yaml
name: build-scan-deploy
on:
  push: { branches: [main] }

permissions:
  id-token: write        # ← OIDC: đăng nhập Azure KHÔNG cần lưu secret dài hạn
  contents: read

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # --- DevSecOps: quét TRƯỚC khi build ---
      - name: Secret scan
        uses: gitleaks/gitleaks-action@v2

      - name: SCA + IaC scan (Trivy)
        uses: aquasecurity/trivy-action@master
        with: { scan-type: 'fs', severity: 'HIGH,CRITICAL', exit-code: '1' }

      # --- Build image ---
      - name: Azure login (OIDC)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - run: az acr login -n acrshopvn01
      - run: docker build -t acrshopvn01.azurecr.io/payment:${{ github.sha }} .

      # --- Quét image trước khi push ---
      - name: Scan image (Trivy)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: acrshopvn01.azurecr.io/payment:${{ github.sha }}
          severity: 'HIGH,CRITICAL'
          exit-code: '1'          # ← có CVE nặng → FAIL build, không cho lên prod

      - run: docker push acrshopvn01.azurecr.io/payment:${{ github.sha }}

  cd:
    needs: ci
    runs-on: ubuntu-latest
    environment: production        # ← cổng approval trước khi vào prod
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - run: az aks get-credentials -g rg-shopvn -n aks-shopvn
      - run: |
          kubectl set image deployment/payment \
            payment=acrshopvn01.azurecr.io/payment:${{ github.sha }}
          kubectl rollout status deployment/payment   # ← chờ rollout xong, tự rollback nếu lỗi
```
> 🔑 3 best practice gói trong file này: **OIDC (không secret dài hạn)**, **quét bảo mật chặn build**, **approval gate + rollout status (rollback tự động)**.

---

## Bước 9 — Observability (OpenTelemetry → Prometheus + Grafana)

### 9a. Cài stack bằng Helm
```bash
# kube-prometheus-stack = Prometheus + Grafana + Alertmanager
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# OpenTelemetry Collector
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm install otel open-telemetry/opentelemetry-collector -n monitoring -f otel-values.yaml
```

### 9b. OTel Collector: nhận → xử lý → định tuyến
`otel-values.yaml` (rút gọn — đây là "trái tim" của observability trung lập vendor):
```yaml
config:
  receivers:
    otlp:                          # nhận từ app (chuẩn OTLP)
      protocols: { grpc: {}, http: {} }
  processors:
    batch: {}                      # gom lô cho hiệu quả
    memory_limiter: { check_interval: 1s, limit_percentage: 80 }
  exporters:
    prometheus: { endpoint: "0.0.0.0:8889" }     # metrics → Prometheus
    otlphttp/tempo: { endpoint: "http://tempo:4318" }  # traces → Tempo
    # Muốn đổi sang Azure Monitor? chỉ thêm 1 exporter, KHÔNG sửa app:
    # azuremonitor: { connection_string: "${APPINSIGHTS_CONNECTION_STRING}" }
  service:
    pipelines:
      metrics: { receivers: [otlp], processors: [batch], exporters: [prometheus] }
      traces:  { receivers: [otlp], processors: [batch], exporters: [otlphttp/tempo] }
```
> 🔑 Đây chính là lý do OTel thắng: muốn thêm Azure Monitor / Datadog → **thêm 1 dòng exporter**, code app không đổi. Chống vendor lock-in.

### 9c. Instrument app (ví dụ Node.js — auto-instrument, gần như 0 code)
```javascript
// tracing.js — nạp trước app: node -r ./tracing.js server.js
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-http');

new NodeSDK({
  traceExporter: new OTLPTraceExporter({ url: 'http://otel-collector:4318/v1/traces' }),
  instrumentations: [getNodeAutoInstrumentations()],  // tự bắt HTTP, DB, gRPC...
}).start();
```

### 9d. Kết quả — 1 request checkout hiện thành trace
```
   Trace id: a1b2c3...  (tổng 2912ms  ← CHẬM!)
   ├─ gateway        2ms
   ├─ cart-service   8ms
   ├─ payment-service ────────────────────── 2850ms  ⚠️ thủ phạm ở đây
   │   └─ call bank API   2800ms  (timeout retry)
   └─ order-service  50ms
```
Không có traces → chỉ biết "checkout chậm". Có traces → biết **chậm ở `payment` gọi bank API**. Đó là khác biệt observability thật.

---

## Bước 10 — SLO + DevSecOps + FinOps guardrails

### 10a. Đặt 1 SLO + alert theo burn-rate (PrometheusRule)
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata: { name: shopvn-slo, namespace: monitoring }
spec:
  groups:
    - name: slo.rules
      rules:
        # SLI: tỉ lệ request 5xx của payment
        - alert: PaymentErrorBudgetBurnFast
          # đốt budget nhanh gấp 14x trong 1h → page ngay
          expr: |
            (sum(rate(http_requests_total{job="payment",code=~"5.."}[1h]))
             / sum(rate(http_requests_total{job="payment"}[1h]))) > (14 * 0.001)
          for: 2m
          labels: { severity: page }
          annotations: { summary: "Payment đốt error budget quá nhanh" }
```
> SLO 99.9% → budget lỗi 0.1%. Alert khi **tốc độ đốt** vượt ngưỡng, không phải khi CPU cao. Đây là alert "đáng để dựng người dậy lúc 3h sáng".

### 10b. FinOps guardrails
```hcl
# Bắt buộc tag (Azure Policy) — không tag thì không cho tạo resource
# + Budget alert khi chi tiêu chạm 80%
resource "azurerm_consumption_budget_resource_group" "main" {
  name              = "budget-shopvn"
  resource_group_id = azurerm_resource_group.main.id
  amount            = 500      # USD/tháng
  time_grain        = "Monthly"
  notification {
    threshold = 80
    operator  = "GreaterThan"
    contact_emails = ["devops@shopvn.example"]
  }
}
```
```bash
# Ước lượng chi phí NGAY trong PR (Infracost) trước khi apply
infracost breakdown --path .
# → thấy "AKS thêm node pool: +$140/tháng" trước khi merge
```

### 10c. Dọn dẹp (bài học FinOps cuối)
```bash
terraform destroy    # ← tắt hết để khỏi cháy ví sau khi tập
```

---

## ✅ Checklist "đã hiểu toàn bộ vòng đời"

Sau lab, tự trả lời được:
```
   [ ] Vì sao state Terraform phải để remote? (team + không mất)
   [ ] Private Endpoint khác gì firewall IP? (IP private trong VNet, không ra Internet)
   [ ] HPA vs Cluster Autoscaler khác gì? (scale POD vs scale NODE)
   [ ] OIDC trong CI/CD lợi gì so với lưu secret? (không secret dài hạn để rò)
   [ ] OTel Collector giúp gì? (đổi backend không sửa code app)
   [ ] Alert nên dựa trên gì? (SLO/burn-rate, không phải CPU thô)
   [ ] Trivy chặn build khi nào? (CVE HIGH/CRITICAL)
   [ ] 3 đòn FinOps rẻ nhất? (tắt cái không dùng, right-size, autoscale)
```

## 🔗 Nối tiếp
- Lý thuyết & "chọn tool nào": [DevOps-Azure-JD-DeepDive.md](./DevOps-Azure-JD-DeepDive.md)
- Muốn mình viết tiếp: **GitOps với ArgoCD**, **hybrid VPN nối on-prem cụ thể**, hay **kịch bản phỏng vấn AZ-400** — cứ nói.
