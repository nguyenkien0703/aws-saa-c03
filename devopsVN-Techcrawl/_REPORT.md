# Báo cáo Crawl — devopsvn.tech

- **Nguồn liệt kê page:** https://devopsvn.tech/sitemap.xml
- **Tổng số page tìm thấy (sitemap):** 219
- **Crawl thành công:** 187
- **Thất bại (HTTP 404 vĩnh viễn, đã thử lại):** 32
- **Page phát hiện qua internal link nhưng KHÔNG có trong sitemap:** 3 (đều là asset/hệ thống, không phải nội dung)

## Kết luận kiểm tra
- Đã đối chiếu toàn bộ internal link trong các page đã crawl với sitemap: **không còn page nội dung nào chưa được crawl**.
- Toàn bộ 32 URL thất bại đều trả về **HTTP 404** (đã retry, vẫn 404) — là các entry cũ/đã gỡ trong sitemap hoặc slug bị lỗi bỏ dấu tiếng Việt. Phần lớn là URL alias (`/new/*`, `/top-post/*`, `/chuong-trinh-mentor/*`) trỏ tới bài viết đã tồn tại ở URL chuẩn (đã crawl thành công), nên **không mất nội dung**.

## Danh sách 3 link ngoài sitemap (không phải page nội dung)
- https://devopsvn.tech/878c6843d92e45a3af8e2153fd6bb103
- https://devopsvn.tech/_next/image
- https://devopsvn.tech/cdn-cgi/l/email-protection

## Page CRAWL THÀNH CÔNG

| File | URL | Title | Ảnh | Code | Bảng | Link nội bộ | Link ngoài |
|---|---|---|---|---|---|---|---|
| 001.md | https://devopsvn.tech/ | DevOps VN | 125 | 0 | 0 | 76 | 1 |
| 002.md | https://devopsvn.tech/aws-practice | AWS Practice | 78 | 0 | 0 | 38 | 0 |
| 003.md | https://devopsvn.tech/aws-practice/aws-cicd-security | Lỗi bảo mật khi làm CI/CD trên AWS mà các bạn không để ý | 19 | 0 | 0 | 7 | 1 |
| 004.md | https://devopsvn.tech/aws-practice/cac-loai-co-so-du-lieu-tren-aws | Các loại cơ sở dữ liệu trên AWS | 20 | 0 | 0 | 3 | 1 |
| 005.md | https://devopsvn.tech/aws-practice/cach-xin-khoang-5k-dollar-credit-tu-aws-cho-doanh-nghiep | Cách xin khoảng $5k credit từ AWS cho doanh nghiệp | 11 | 0 | 0 | 5 | 4 |
| 006.md | https://devopsvn.tech/aws-practice/improve-network-performance-of-application-load-balancer-with-cloudfront | Improve Network Performance of Application Load Balancer with CloudFront | 22 | 0 | 0 | 7 | 0 |
| 007.md | https://devopsvn.tech/aws-practice/pipecd-gitops-on-aws | PipeCD: GitOps on AWS | 30 | 13 | 0 | 8 | 6 |
| 008.md | https://devopsvn.tech/aws-practice/tai-sao-lai-su-dung-aws-s3 | Tại sao lại sử dụng AWS S3? | 24 | 12 | 0 | 7 | 2 |
| 009.md | https://devopsvn.tech/aws-practice/thuc-hanh-aws-ma-khong-can-tao-tai-khoan | Thực hành AWS mà không cần tạo tài khoản | 18 | 6 | 0 | 7 | 4 |
| 010.md | https://devopsvn.tech/azure | Azure cơ bản | 22 | 0 | 0 | 12 | 0 |
| 011.md | https://devopsvn.tech/azure/azure-directory-services | Azure Directory Services | 1 | 0 | 0 | 0 | 0 |
| 012.md | https://devopsvn.tech/azure/azure-resources | Azure Resources | 15 | 0 | 0 | 4 | 1 |
| 013.md | https://devopsvn.tech/azure/azure-storage-accounts | Azure Storage Accounts | 1 | 0 | 0 | 0 | 0 |
| 014.md | https://devopsvn.tech/azure/azure-virtual-machine-scale-sets | Azure Virtual Machine Scale Sets | 18 | 0 | 0 | 8 | 1 |
| 015.md | https://devopsvn.tech/azure/azure-virtual-machine-storage | Azure Virtual Machine Storage | 19 | 0 | 1 | 8 | 2 |
| 016.md | https://devopsvn.tech/azure/azure-virtual-machines | Azure Virtual Machines | 11 | 0 | 2 | 4 | 12 |
| 017.md | https://devopsvn.tech/azure/exercise-attach-a-data-disk-to-a-vm | Exercise - Attach a Managed Data Disk to a Azure VM | 20 | 9 | 1 | 8 | 2 |
| 018.md | https://devopsvn.tech/azure/exercise-create-a-azure-vm | Exercise - Create a Azure VM | 21 | 3 | 1 | 9 | 2 |
| 019.md | https://devopsvn.tech/azure/exercise-create-azure-virtual-machine-scale-sets | Exercise - Create Azure Virtual Machine Scale Sets | 17 | 9 | 0 | 8 | 2 |
| 020.md | https://devopsvn.tech/azure/exercise-create-azure-vm-image-to-share-your-vm | Exercise - Create Azure VM Image to share your VM | 19 | 7 | 0 | 8 | 2 |
| 021.md | https://devopsvn.tech/azure/gioi-thieu-microsoft-azure | Giới thiệu Microsoft Azure | 14 | 0 | 0 | 4 | 4 |
| 022.md | https://devopsvn.tech/azure/regions-and-availability-zones | Azure Regions and Availability Zones | 11 | 0 | 0 | 4 | 2 |
| 023.md | https://devopsvn.tech/banking-infrastructure-on-cloud | Vikki - Banking Infrastructure on Cloud | 12 | 0 | 0 | 5 | 2 |
| 024.md | https://devopsvn.tech/banking-infrastructure-on-cloud/aws-account-management | AWS Account Management | 7 | 0 | 0 | 5 | 4 |
| 025.md | https://devopsvn.tech/banking-infrastructure-on-cloud/kubernetes-cross-cluster-comunication | Kubernetes Cross Cluster Communication | 12 | 2 | 0 | 5 | 1 |
| 026.md | https://devopsvn.tech/banking-infrastructure-on-cloud/kubernetes-infrastructure-for-scale | Kubernetes Infrastructure for Scale | 7 | 1 | 0 | 5 | 3 |
| 027.md | https://devopsvn.tech/banking-infrastructure-on-cloud/networking-for-multi-aws-accounts | Networking for Multi AWS Accounts | 13 | 0 | 0 | 5 | 3 |
| 028.md | https://devopsvn.tech/banking-infrastructure-on-cloud/provisioning-infrastructure-for-multi-aws-accounts | Provisioning Infrastructure for Multi AWS Accounts | 6 | 8 | 0 | 5 | 1 |
| 029.md | https://devopsvn.tech/cdk | CDK | 12 | 0 | 0 | 6 | 0 |
| 030.md | https://devopsvn.tech/cdk/bai-0-iac-va-aws-cloud-development-kit | Chinh phục AWS CDK - Bài 0 - IaC và AWS Cloud Development Kit | 18 | 17 | 0 | 7 | 3 |
| 031.md | https://devopsvn.tech/cdk/bai-1-cac-buoc-khoi-tao-ung-dung-va-viet-cau-hinh-cho-du-an | Chinh phục AWS CDK - Bài 1 - Các bước khởi tạo ứng dụng và viết cấu hình cho dự án | 16 | 23 | 0 | 7 | 2 |
| 032.md | https://devopsvn.tech/cdk/bai-2-cac-thanh-phan-co-ban-cua-cdk | Chinh phục AWS CDK - Bài 2 - Các thành phần cơ bản của CDK | 19 | 15 | 0 | 7 | 1 |
| 033.md | https://devopsvn.tech/cdk/bai-4-construct-layer | Chinh phục AWS CDK - Bài 4 - Construct Layer | 19 | 4 | 0 | 7 | 2 |
| 034.md | https://devopsvn.tech/cdk/bai-5-stacks | Chinh phục AWS CDK - Bài 5 - Stacks | 16 | 20 | 0 | 7 | 1 |
| 035.md | https://devopsvn.tech/cdk/thiet-ke-va-xay-dung-ha-tang-cho-ung-dung-q-and-a | Chinh phục AWS CDK - Thực hành: thiết kế và xây dựng hạ tầng cho ứng dụng Q&A | 27 | 30 | 0 | 7 | 1 |
| 036.md | https://devopsvn.tech/chia-se-hanh-trinh-tro-thanh-cloud-engineer | Chia sẻ về hành trình trở thành Cloud Engineer | 3 | 0 | 0 | 0 | 1 |
| 037.md | https://devopsvn.tech/chia-se-tu-chuyen-gia | Chia sẻ từ chuyên gia | 7 | 0 | 0 | 2 | 2 |
| 038.md | https://devopsvn.tech/chia-se-tu-chuyen-gia/con-duong-thanh-devops-va-co-hoi-nghe-nghiep | Hình ảnh buổi hội thảo: “Con đường thành DevOps và cơ hội nghề nghiệp” | 36 | 0 | 0 | 0 | 0 |
| 039.md | https://devopsvn.tech/chia-se-tu-chuyen-gia/hnh-trnh-tr-thnh-cloud-engineer | Hành trình trở thành Cloud Engineer | 1 | 0 | 0 | 1 | 0 |
| 040.md | https://devopsvn.tech/chia-se-tu-chuyen-gia/kh-khn-trong-vic-chuyn-i-mi-trng-on-prem-ln-cloud-c-hi-thc-tp-cloud-fpt | Khó khăn trong việc chuyển đổi môi trường On-prem lên Cloud? Cơ hội thực tập Cloud ở FPT? | 1 | 0 | 0 | 0 | 1 |
| 041.md | https://devopsvn.tech/chia-se-tu-chuyen-gia/ngi-mi-nn-hc-aws-nh-th-no-lm-th-no-tr-thnh-aws-solutions-architect | Người mới nên học AWS như thế nào? Làm thế nào để trở thành AWS Solutions Architect? | 1 | 0 | 0 | 0 | 1 |
| 046.md | https://devopsvn.tech/cloud-computing | Cloud Computing | 6 | 0 | 0 | 3 | 0 |
| 047.md | https://devopsvn.tech/cloud-computing/bai-0-khai-niem-cloud-computing-cloud-la-gi | Cloud Computing - Bài 0 - Khái niệm Cloud Computing: Cloud là gì? | 18 | 0 | 0 | 7 | 1 |
| 048.md | https://devopsvn.tech/cloud-computing/bai-1-cac-thanh-phan-va-dac-tinh-cua-cloud | Cloud Computing - Bài 1 - Các thành phần và đặc tính của Cloud | 14 | 0 | 0 | 4 | 1 |
| 049.md | https://devopsvn.tech/cloud-computing/bai-2-cac-to-chuc-xay-dung-tieu-chuan-cho-cloud | Cloud Computing - Bài 2 - Các tổ chức xây dựng tiêu chuẩn cho Cloud | 10 | 0 | 0 | 3 | 6 |
| 050.md | https://devopsvn.tech/devops | DevOps | 14 | 0 | 0 | 7 | 0 |
| 051.md | https://devopsvn.tech/devops-practice | DevOps Practice | 86 | 0 | 0 | 47 | 0 |
| 052.md | https://devopsvn.tech/devops-vn | Giới thiệu DevOps VN | 5 | 0 | 0 | 0 | 2 |
| 053.md | https://devopsvn.tech/devops/bat-nhac-lenh-cho-kubectl-tren-linux | Bật nhắc lệnh cho kubectl trên Linux | 68 | 11 | 0 | 31 | 1 |
| 054.md | https://devopsvn.tech/devops/cai-dat-docker-len-linux-voi-mot-cau-lenh | Cài đặt Docker lên Linux với một câu lệnh | 4 | 3 | 0 | 0 | 1 |
| 055.md | https://devopsvn.tech/devops/cicd-la-gi-cong-cu-continuous-integration | CI/CD là gì - Công cụ Continuous Integration | 71 | 0 | 0 | 31 | 1 |
| 056.md | https://devopsvn.tech/devops/common-network-problem | Common Network Problem | 4 | 0 | 0 | 0 | 1 |
| 057.md | https://devopsvn.tech/devops/copy-elasticsearch-data-to-remote-server | Copy Elasticsearch Data to Remote Server | 18 | 6 | 0 | 8 | 1 |
| 058.md | https://devopsvn.tech/devops/cost-compare-for-30-million-requests | Tính toán chi phí Cloud CDN và Storage cho 30 triệu request trên tháng | 18 | 0 | 5 | 7 | 1 |
| 059.md | https://devopsvn.tech/devops/cpu-memory-10k-rps | Tính toán CPU và Memory cho 10K Requests per Second | 18 | 5 | 1 | 7 | 1 |
| 060.md | https://devopsvn.tech/devops/di-chuyen-du-lieu-tu-on-premises-len-aws | Di chuyển dữ liệu từ on-premises lên Cloud | 10 | 2 | 0 | 3 | 1 |
| 062.md | https://devopsvn.tech/devops/gitops-la-gi | GitOps là gì? | 5 | 0 | 0 | 5 | 10 |
| 063.md | https://devopsvn.tech/devops/khi-nao-thi-ta-khong-nen-su-dung-kubernetes | Khi nào thì ta không nên sử dụng Kubernetes? | 66 | 0 | 0 | 31 | 1 |
| 064.md | https://devopsvn.tech/devops/kubernetes-lam-viec-voi-container-nhu-the-nao | Kubernetes làm việc với Container như thế nào? | 74 | 0 | 0 | 31 | 2 |
| 065.md | https://devopsvn.tech/devops/kubernetes-logging-fluent-bit-grafana-loki | Kubernetes Logging: Fluent Bit to Grafana Loki | 22 | 11 | 0 | 7 | 3 |
| 066.md | https://devopsvn.tech/devops/lam-the-nao-de-tranh-o-dia-bi-day-khi-xai-docker | Làm thế nào để tránh ổ đĩa bị đầy khi xài Docker? | 66 | 17 | 0 | 31 | 2 |
| 067.md | https://devopsvn.tech/devops/lam-the-nao-de-tro-thanh-devops-engineer | Làm thế nào để trở thành DevOps Engineer? | 66 | 0 | 0 | 32 | 19 |
| 068.md | https://devopsvn.tech/devops/liet-ke-requests-limits-trong-kubernetes-mot-cach-de-dang-voi-kube-capacity-cli | Liệt kê tài nguyên requests, limits trong Kubernetes một cách dễ dàng với kube-capacity CLI | 66 | 16 | 0 | 31 | 3 |
| 069.md | https://devopsvn.tech/devops/linux-namespaces-va-cgroups-container-duoc-xay-dung-tu-gi | Linux Namespaces và Cgroups: Container được xây dựng từ gì? | 66 | 13 | 0 | 31 | 2 |
| 070.md | https://devopsvn.tech/devops/nginx-subdirectory-as-root-for-react | Nginx subdirectory as root for React | 4 | 4 | 0 | 0 | 1 |
| 071.md | https://devopsvn.tech/devops/nhung-cuon-sach-nen-doc-de-hoc-kubernetes-cho-nguoi-moi-bat-dau | Những cuốn sách nên đọc để học Kubernetes cho người mới bắt đầu | 73 | 1 | 0 | 31 | 9 |
| 072.md | https://devopsvn.tech/devops/nhung-cuon-sach-nen-doc-tro-thanh-devops-cho-nguoi-moi-bat-dau | Những cuốn sách nên đọc để trở thành DevOps cho người mới bắt đầu | 31 | 0 | 0 | 7 | 16 |
| 073.md | https://devopsvn.tech/devops/nhung-loi-imagepullbackoff-thuong-gap-trong-kubernetes | Những lỗi ImagePullBackOff thường gặp trong Kubernetes | 66 | 29 | 0 | 31 | 1 |
| 074.md | https://devopsvn.tech/devops/nomad-cong-cu-thay-the-kubernetes | Nomad - Công cụ thay thế Kubernetes | 68 | 0 | 0 | 31 | 4 |
| 075.md | https://devopsvn.tech/devops/podman-cong-cu-thay-the-docker | Podman - Công cụ thay thế Docker | 18 | 26 | 0 | 7 | 2 |
| 077.md | https://devopsvn.tech/devops/ssl-hoat-dong-nhu-the-nao | SSL hoạt động như thế nào? | 71 | 0 | 0 | 31 | 2 |
| 078.md | https://devopsvn.tech/devops/su-dung-chatgpt-cho-devops | Sử dụng ChatGPT cho DevOps | 76 | 0 | 0 | 31 | 2 |
| 079.md | https://devopsvn.tech/devops/su-dung-ssh-port-forwarding-de-ket-noi-toi-tai-nguyen-bi-chan | Sử dụng SSH Port Forwarding để kết nối tới tài nguyên bị chặn | 67 | 5 | 0 | 32 | 2 |
| 080.md | https://devopsvn.tech/devops/tim-hieu-sau-hon-ve-container-container-runtime-la-gi | Tìm hiểu sâu hơn về Container - Container Runtime là gì? | 70 | 12 | 0 | 31 | 2 |
| 081.md | https://devopsvn.tech/devops/tu-dong-dong-bo-kubernetes-configmaps-va-secrets-qua-cac-namespaces-khac-voi-reflector | Tự động đồng bộ Kubernetes ConfigMaps và Secrets qua các Namespaces khác với Reflector | 66 | 9 | 0 | 31 | 2 |
| 082.md | https://devopsvn.tech/devops/tu-xay-dung-container-voi-go | Tự xây dựng Container với Go | 66 | 31 | 0 | 31 | 1 |
| 083.md | https://devopsvn.tech/devops/vai-tro-cua-devops-trong-cong-ty | Vai trò của DevOps trong công ty | 18 | 0 | 0 | 7 | 1 |
| 084.md | https://devopsvn.tech/devops/viet-lai-linux-shell-voi-go | Viết lại Linux Shell với Go | 66 | 21 | 0 | 31 | 2 |
| 085.md | https://devopsvn.tech/devops/xay-dung-load-balancer-don-gian-voi-go | Xây dựng Load Balancer đơn giản với Go | 67 | 15 | 0 | 31 | 2 |
| 096.md | https://devopsvn.tech/kubernetes | Kubernetes | 70 | 0 | 0 | 34 | 0 |
| 097.md | https://devopsvn.tech/kubernetes-practice | Kubernetes | 12 | 0 | 0 | 6 | 0 |
| 098.md | https://devopsvn.tech/kubernetes-practice/ab-testing-deployment | A/B Testing Deployment | 19 | 13 | 0 | 6 | 1 |
| 099.md | https://devopsvn.tech/kubernetes-practice/blue-green-deployment-with-argo-rollouts | Automating Blue/Green Deployment with Argo Rollouts | 36 | 21 | 0 | 11 | 2 |
| 100.md | https://devopsvn.tech/kubernetes-practice/cau-hinh-postgresql-master-slave-replication-tren-kubernetes | Cấu hình PostgreSQL Master Slave Replication trên Kubernetes | 28 | 27 | 0 | 11 | 3 |
| 101.md | https://devopsvn.tech/kubernetes-practice/debug-with-ephemeral-container | Kubernetes Practice - Debug with Ephemeral Container | 26 | 19 | 0 | 11 | 2 |
| 102.md | https://devopsvn.tech/kubernetes-practice/gateway-api | Kubernetes Gateway API | 17 | 19 | 0 | 6 | 2 |
| 103.md | https://devopsvn.tech/kubernetes-practice/kubernetes-based-event-driven-autoscaler | Kubernetes based Event Driven Autoscaler | 28 | 24 | 0 | 12 | 8 |
| 104.md | https://devopsvn.tech/kubernetes-practice/kubernetes-logging-voi-logstash-va-fluentd | Kubernetes Logging với Logstash và FluentD | 19 | 12 | 0 | 6 | 2 |
| 105.md | https://devopsvn.tech/kubernetes-practice/manual-bluegreen-deployment | Kubernetes Manual Blue/Green Deployment | 30 | 14 | 0 | 11 | 2 |
| 106.md | https://devopsvn.tech/kubernetes-practice/quan-ly-user-va-phan-quyen-tren-argocd | Quản lý user và phân quyền trên ArgoCD | 27 | 13 | 0 | 11 | 3 |
| 107.md | https://devopsvn.tech/kubernetes-practice/trien-khai-elasticsearch-len-tren-kubernetes-cloud | Triển khai Elasticsearch lên trên Kubernetes Cloud | 27 | 29 | 0 | 11 | 5 |
| 108.md | https://devopsvn.tech/kubernetes-practice/trien-khai-he-thong-microservices-len-tren-kubernetes | Triển khai hệ thống Microservices lên trên Kubernetes | 27 | 36 | 0 | 11 | 3 |
| 109.md | https://devopsvn.tech/kubernetes/argocd-architecture | ArgoCD Architecture | 8 | 0 | 0 | 10 | 1 |
| 110.md | https://devopsvn.tech/kubernetes/argocd-core-concepts | ArgoCD Core Concepts | 10 | 7 | 0 | 10 | 2 |
| 111.md | https://devopsvn.tech/kubernetes/argocd-getting-started | ArgoCD Getting Started | 16 | 7 | 0 | 11 | 3 |
| 112.md | https://devopsvn.tech/kubernetes/argocd-with-helm | ArgoCD with Helm | 7 | 9 | 0 | 11 | 2 |
| 113.md | https://devopsvn.tech/kubernetes/argocd-with-private-repo | ArgoCD with Private Repo | 12 | 5 | 0 | 10 | 4 |
| 114.md | https://devopsvn.tech/kubernetes/dam-bao-so-luong-pod-voi-replicationcontrollers | Đảm bảo số lượng Pod với Replication Controllers | 23 | 7 | 0 | 6 | 1 |
| 115.md | https://devopsvn.tech/kubernetes/kubernetes-la-gi | Bài 1 - Kubernetes là gì? | 17 | 0 | 0 | 6 | 2 |
| 116.md | https://devopsvn.tech/kubernetes/patterns | Patterns | 12 | 0 | 0 | 6 | 0 |
| 117.md | https://devopsvn.tech/kubernetes/patterns/behavioral-patterns-batch-job-periodic-job | Kubernetes Behavioral Patterns: Batch Job và Periodic Job | 7 | 6 | 0 | 0 | 1 |
| 118.md | https://devopsvn.tech/kubernetes/patterns/behavioral-patterns-service-discovery | Kubernetes Behavioral Patterns: Service Discovery | 20 | 1 | 0 | 6 | 1 |
| 119.md | https://devopsvn.tech/kubernetes/patterns/behavioral-patterns-singleton-service | Kubernetes Behavioral Patterns: Singleton Service | 24 | 0 | 0 | 6 | 1 |
| 120.md | https://devopsvn.tech/kubernetes/patterns/structural-patterns-adapter-va-ambassador-container | Kubernetes Structural Patterns: Adapter và Ambassador Container | 18 | 1 | 0 | 6 | 1 |
| 121.md | https://devopsvn.tech/kubernetes/patterns/structural-patterns-init-containers | Kubernetes Structural Patterns: Init Containers | 17 | 1 | 0 | 6 | 1 |
| 122.md | https://devopsvn.tech/kubernetes/patterns/structural-patterns-sidecar-containers | Kubernetes Structural Patterns: Sidecar Containers | 18 | 2 | 0 | 6 | 1 |
| 123.md | https://devopsvn.tech/kubernetes/pod-la-gi | Bài 2 - Pod là gì? | 18 | 17 | 0 | 6 | 2 |
| 124.md | https://devopsvn.tech/kubernetes/quan-ly-pod-voi-labels | Bài 3 - Quản lý Pod với Labels | 16 | 22 | 0 | 6 | 1 |
| 125.md | https://devopsvn.tech/kubernetes/replicasets-daemonset | Bài 5 - ReplicaSets và DaemonSet | 16 | 10 | 0 | 6 | 2 |
| 126.md | https://devopsvn.tech/kubernetes/tips | Tips | 6 | 0 | 0 | 3 | 0 |
| 127.md | https://devopsvn.tech/kubernetes/tips/backup-va-restore-cho-argocd | Backup và Restore cho ArgoCD | 10 | 4 | 0 | 3 | 1 |
| 128.md | https://devopsvn.tech/kubernetes/tips/giam-thoi-gian-dns-resolution-cua-10000-pod-tren-eks | Kubernetes Tips - Giảm thời gian DNS Resolution của 10000 Pod trên EKS | 10 | 1 | 0 | 3 | 3 |
| 129.md | https://devopsvn.tech/kubernetes/tips/tao-va-phan-quyen-nguoi-dung-tren-kubernetes | Tạo và phân quyền người dùng trên Kubernetes | 10 | 12 | 0 | 4 | 1 |
| 134.md | https://devopsvn.tech/linux-tip | Linux tip | 11 | 0 | 0 | 10 | 0 |
| 135.md | https://devopsvn.tech/linux-tip/cap-nhat-current-time-cho-may-chu | Cập nhật current time cho máy chủ | 3 | 2 | 0 | 0 | 1 |
| 136.md | https://devopsvn.tech/linux-tip/giam-thoi-gian-tim-kiem-cua-cau-lenh-find-voi-quit | Giảm thời gian tìm kiếm của câu lệnh find với -quit | 12 | 0 | 0 | 10 | 0 |
| 137.md | https://devopsvn.tech/linux-tip/lay-ngay-trong-nam-voi-date | Lấy ngày trong năm với date | 3 | 0 | 0 | 0 | 0 |
| 138.md | https://devopsvn.tech/linux-tip/liet-ke-tep-tin-theo-chieu-doc | Liệt kê tệp tin theo chiều dọc | 12 | 0 | 0 | 10 | 0 |
| 139.md | https://devopsvn.tech/linux-tip/linux-echo-and-rm | Linux echo and rm | 2 | 1 | 0 | 0 | 1 |
| 140.md | https://devopsvn.tech/linux-tip/nhom-tep-tin-theo-extension-voi-lx | Nhóm tệp tin theo extension với -lX | 3 | 0 | 0 | 0 | 0 |
| 141.md | https://devopsvn.tech/linux-tip/xem-thong-tin-file-voi-getfacl | Xem thông tin file với getfacl | 12 | 0 | 0 | 10 | 0 |
| 142.md | https://devopsvn.tech/linux-tip/xoa-co-xac-nhan | Xóa có xác nhận | 12 | 0 | 0 | 10 | 0 |
| 143.md | https://devopsvn.tech/linux-tip/xoa-dong-trong-trong-tep-tin-voi-grep | Xóa dòng trống trong tệp tin với grep | 3 | 0 | 0 | 0 | 0 |
| 144.md | https://devopsvn.tech/linux-tip/xoa-toan-bo-container-dang-o-trang-thai-exited | Xóa toàn bộ container ở trạng thái exited | 2 | 1 | 0 | 0 | 0 |
| 145.md | https://devopsvn.tech/networking-for-devops | Networking for DevOps | 10 | 0 | 0 | 8 | 0 |
| 146.md | https://devopsvn.tech/networking-for-devops/dns | DNS | 11 | 0 | 0 | 8 | 2 |
| 147.md | https://devopsvn.tech/networking-for-devops/linux-networking-tools | Linux Networking Tools | 2 | 8 | 0 | 8 | 1 |
| 148.md | https://devopsvn.tech/networking-for-devops/osi-model | OSI Model | 4 | 0 | 0 | 8 | 2 |
| 149.md | https://devopsvn.tech/networking-for-devops/ports | Ports | 4 | 0 | 0 | 8 | 1 |
| 150.md | https://devopsvn.tech/networking-for-devops/protocols-tcpudpip | Protocols : TCP/UDP/IP | 3 | 0 | 0 | 8 | 1 |
| 151.md | https://devopsvn.tech/networking-for-devops/routing | Routing | 3 | 1 | 0 | 8 | 4 |
| 152.md | https://devopsvn.tech/networking-for-devops/subnetting | Subnetting | 5 | 0 | 0 | 8 | 3 |
| 153.md | https://devopsvn.tech/networking-for-devops/virtual-private-network | Virtual Private Network | 5 | 0 | 0 | 8 | 1 |
| 154.md | https://devopsvn.tech/new | Featured Posts | 6 | 0 | 0 | 6 | 0 |
| 155.md | https://devopsvn.tech/new/chinh-phc-terraform-bi-3-cch-lp-trnh-trong-terraform | Triển khai hệ thống Microservices lên trên Kubernetes | 1 | 0 | 0 | 1 | 0 |
| 156.md | https://devopsvn.tech/new/khi-no-th-ta-khng-nn-s-dng-kubernetes | Làm thế nào để trở thành DevOps Engineer? | 1 | 0 | 0 | 1 | 0 |
| 158.md | https://devopsvn.tech/new/lm-th-no-trnh-a-b-y-khi-xi-docker | Xây dựng hạ tầng phục vụ hàng triệu người dùng trên AWS - Bài 0 - Chuẩn bị | 1 | 0 | 0 | 1 | 0 |
| 159.md | https://devopsvn.tech/new/nhng-cun-sch-nn-c-hc-kubernetes-cho-ngi-mi-bt-u | Thực hành AWS mà không cần tạo tài khoản | 1 | 0 | 0 | 1 | 0 |
| 161.md | https://devopsvn.tech/new/service-mesh-on-kubernetes-bi-1-ci-t-istio-vo-kubernetes | Kubernetes Tips - Giảm thời gian DNS Resolution của 10000 Pod trên EKS | 1 | 0 | 0 | 1 | 0 |
| 164.md | https://devopsvn.tech/new/t-xy-dng-container-vi-go | Giới thiệu Microsoft Azure | 1 | 0 | 0 | 1 | 0 |
| 165.md | https://devopsvn.tech/prometheus-series | Prometheus Series | 13 | 0 | 0 | 6 | 0 |
| 166.md | https://devopsvn.tech/prometheus-series/prometheus | Prometheus | 12 | 0 | 0 | 6 | 0 |
| 167.md | https://devopsvn.tech/prometheus-series/prometheus/bai-0-monitoring-la-gi | Chinh phục Prometheus - Bài 0 - Monitoring là gì? | 17 | 0 | 0 | 7 | 1 |
| 168.md | https://devopsvn.tech/prometheus-series/prometheus/bai-1-cai-dat-prometheus | Chinh phục Prometheus - Bài 1 - Cài đặt Prometheus | 21 | 17 | 0 | 7 | 2 |
| 169.md | https://devopsvn.tech/prometheus-series/prometheus/bai-2-giam-sat-may-chu-voi-node-exporter | Chinh phục Prometheus - Bài 2 - Giám sát máy chủ với Node Exporter | 17 | 16 | 0 | 7 | 2 |
| 170.md | https://devopsvn.tech/prometheus-series/prometheus/bai-3-cong-thuc-tinh-toan-chi-so-cpu | Chinh phục Prometheus - Bài 3 - Công thức tính toán chỉ số CPU | 21 | 9 | 0 | 7 | 1 |
| 171.md | https://devopsvn.tech/prometheus-series/prometheus/bai-4-cong-thuc-tinh-toan-chi-so-memory | Chinh phục Prometheus - Bài 4 - Công thức tính toán chỉ số Memory | 21 | 6 | 0 | 7 | 1 |
| 172.md | https://devopsvn.tech/prometheus-series/prometheus/bai-5-cong-thuc-du-doan-o-dia-day | Chinh phục Prometheus - Bài 5 - Công thức dự đoán ổ đĩa đầy | 17 | 4 | 0 | 7 | 1 |
| 173.md | https://devopsvn.tech/quan-huynh | Quan Huynh | 5 | 0 | 0 | 0 | 4 |
| 174.md | https://devopsvn.tech/service-mesh-on-kubernetes | Service Mesh on Kubernetes | 8 | 0 | 0 | 4 | 0 |
| 175.md | https://devopsvn.tech/service-mesh-on-kubernetes/bai-1-cai-dat-istio-vao-kubernetes | Service Mesh on Kubernetes - Bài 1 - Cài đặt Istio vào Kubernetes | 14 | 15 | 0 | 5 | 3 |
| 176.md | https://devopsvn.tech/service-mesh-on-kubernetes/bai-2-ung-dung-dau-tien-voi-istio | Service Mesh on Kubernetes - Bài 2 - Ứng dụng đầu tiên với Istio | 17 | 20 | 0 | 5 | 2 |
| 177.md | https://devopsvn.tech/service-mesh-on-kubernetes/bai-3-nhung-tinh-nang-chinh-cua-istio | Service Mesh on Kubernetes - Bài 3 - Những tính năng chính của Istio | 20 | 10 | 0 | 5 | 1 |
| 178.md | https://devopsvn.tech/service-mesh-on-kubernetes/gioi-thieu-istio-istio-la-gi | Giới thiệu Istio - Istio là gì | 19 | 0 | 0 | 5 | 1 |
| 180.md | https://devopsvn.tech/terraform-series | Terraform Series | 39 | 0 | 0 | 19 | 0 |
| 181.md | https://devopsvn.tech/terraform-series/terraform | Terraform | 38 | 0 | 0 | 19 | 0 |
| 182.md | https://devopsvn.tech/terraform-series/terraform/bai-0-infrastructure-as-code-va-terraform | Chinh phục Terraform - Bài 0 - Infrastructure as Code và Terraform | 52 | 9 | 0 | 19 | 4 |
| 183.md | https://devopsvn.tech/terraform-series/terraform/bai-1-cac-buoc-khoi-tao-va-viet-cau-hinh-terraform-cho-du-an | Chinh phục Terraform - Bài 1 - Các bước khởi tạo và viết cấu hình Terraform cho dự án | 44 | 19 | 0 | 19 | 1 |
| 184.md | https://devopsvn.tech/terraform-series/terraform/bai-10-cicd-voi-terraform-cloud-va-trien-khai-zero-downtime | Chinh phục Terraform - Bài 10 - CI/CD với Terraform Cloud và triển khai Zero-downtime | 62 | 8 | 0 | 19 | 1 |
| 185.md | https://devopsvn.tech/terraform-series/terraform/bai-11-terraform-bluegreen-deployment | Chinh phục Terraform - Bài 11 - Terraform Blue/Green Deployment | 47 | 7 | 0 | 19 | 1 |
| 186.md | https://devopsvn.tech/terraform-series/terraform/bai-12-terraform-ab-testing-deployment | Chinh phục Terraform - Bài 12 - Terraform A/B Testing Deployment | 52 | 27 | 0 | 19 | 2 |
| 187.md | https://devopsvn.tech/terraform-series/terraform/bai-13-ansible-with-terraform | Chinh phục Terraform - Bài 13 - Ansible with Terraform | 46 | 19 | 0 | 19 | 1 |
| 188.md | https://devopsvn.tech/terraform-series/terraform/bai-14-xay-dung-cicd-cho-terraform-voi-gitlab-ci | Chinh phục Terraform - Bài 14 - Xây dựng CI/CD cho Terraform với Gitlab CI | 51 | 24 | 0 | 19 | 5 |
| 189.md | https://devopsvn.tech/terraform-series/terraform/bai-16-multi-cloud | Chinh phục Terraform - Bài 16 - Multi-cloud | 50 | 11 | 0 | 19 | 1 |
| 190.md | https://devopsvn.tech/terraform-series/terraform/bai-17-securing-logs-and-securing-state-file | Chinh phục Terraform - Bài 17 - Securing Logs and Securing State File | 44 | 11 | 0 | 19 | 1 |
| 191.md | https://devopsvn.tech/terraform-series/terraform/bai-18-xu-ly-khac-biet-giua-terraform-state-va-ha-tang-thuc-te | Chinh phục Terraform - Xử lý khác biệt giữa Terraform State và hạ tầng thực tế | 43 | 27 | 0 | 19 | 1 |
| 192.md | https://devopsvn.tech/terraform-series/terraform/bai-2-vong-doi-cua-mot-resource-trong-terraform | Chinh phục Terraform - Bài 2 - Vòng đời của một resource trong Terraform | 59 | 19 | 0 | 19 | 1 |
| 193.md | https://devopsvn.tech/terraform-series/terraform/bai-3-cach-lap-trinh-trong-terraform | Chinh phục Terraform - Bài 3 - Cách lập trình trong Terraform | 44 | 20 | 0 | 19 | 1 |
| 194.md | https://devopsvn.tech/terraform-series/terraform/bai-4-dung-terraform-de-trien-khai-trang-web-len-s3 | Chinh phục Terraform - Bài 4 - Dùng Terraform để triển khai trang web lên S3 | 45 | 12 | 0 | 19 | 2 |
| 195.md | https://devopsvn.tech/terraform-series/terraform/bai-5-tao-aws-virtual-private-cloud-voi-terraform-module | Chinh phục Terraform - Bài 5 - Tạo AWS Virtual Private Cloud với Terraform Module | 55 | 25 | 0 | 19 | 4 |
| 196.md | https://devopsvn.tech/terraform-series/terraform/bai-6-xay-dung-ha-tang-cho-mot-ung-dung-thuc-te-voi-terraform-module | Chinh phục Terraform - Bài 6 - Xây dựng hạ tầng cho một ứng dụng thực tế với Terraform Module | 52 | 30 | 0 | 19 | 1 |
| 197.md | https://devopsvn.tech/terraform-series/terraform/bai-7-terraform-backend-backend-trong-terraform-la-gi | Chinh phục Terraform - Bài 7 - Terraform Backend: Backend trong Terraform là gì? | 50 | 2 | 0 | 19 | 1 |
| 198.md | https://devopsvn.tech/terraform-series/terraform/bai-8-su-dung-s3-standard-backend-vao-du-an | Chinh phục Terraform - Bài 8 - Sử dụng S3 Standard Backend vào dự án | 47 | 11 | 0 | 19 | 3 |
| 199.md | https://devopsvn.tech/terraform-series/terraform/bai-9-tim-hieu-terraform-cloud-remote-backend | Chinh phục Terraform - Bài 9 - Tìm hiểu Terraform Cloud: Remote Backend | 62 | 11 | 0 | 19 | 2 |
| 200.md | https://devopsvn.tech/terraform-series/terraform/xay-dung-cicd-voi-jenkins | Chinh phục Terraform - Bài 15 - Xây dựng CI/CD với Jenkins | 55 | 8 | 0 | 19 | 5 |
| 201.md | https://devopsvn.tech/top-post | New Posts | 6 | 0 | 0 | 6 | 0 |
| 202.md | https://devopsvn.tech/top-post/backup-v-restore-cho-argocd | Cài đặt Docker lên Linux với một câu lệnh | 1 | 0 | 0 | 1 | 0 |
| 203.md | https://devopsvn.tech/top-post/cc-loi-c-s-d-liu-trn-aws | ArgoCD Getting Started | 1 | 0 | 0 | 1 | 0 |
| 204.md | https://devopsvn.tech/top-post/chinh-phc-prometheus-bi-0-monitoring-l-g | Cách xin $5000 credit từ AWS cho doanh nghiệp | 1 | 0 | 0 | 1 | 0 |
| 205.md | https://devopsvn.tech/top-post/chinh-phc-terraform-bi-13-ansible-with-terraform | Kubernetes cơ bản - Kubernetes là gì? | 1 | 0 | 0 | 1 | 0 |
| 210.md | https://devopsvn.tech/top-post/kubernetes-structural-patterns-adapter-v-ambassador-container | Tính toán chi phí Cloud CDN và Storage cho 30 triệu request trên tháng | 1 | 0 | 0 | 1 | 0 |
| 211.md | https://devopsvn.tech/top-post/linux-namespaces-v-cgroups-container-c-xy-dng-t-g | Tạo và phân quyền người dùng trên Kubernetes | 1 | 0 | 0 | 1 | 0 |
| 215.md | https://devopsvn.tech/tu-van-va-trien-khai-ha-tang-aws | Tư vấn và triển khai hạ tầng AWS | 2 | 0 | 0 | 0 | 1 |
| 216.md | https://devopsvn.tech/xay-dung-ha-tang-phuc-vu-hang-trieu-nguoi-dung-tren-aws | AWS | 6 | 0 | 0 | 3 | 0 |
| 217.md | https://devopsvn.tech/xay-dung-ha-tang-phuc-vu-hang-trieu-nguoi-dung-tren-aws/bai-0-chuan-bi | Xây dựng hạ tầng phục vụ hàng triệu người dùng trên AWS - Bài 0 - Chuẩn bị | 27 | 24 | 0 | 5 | 7 |
| 218.md | https://devopsvn.tech/xay-dung-ha-tang-phuc-vu-hang-trieu-nguoi-dung-tren-aws/bai-1-1k-nguoi-dung | Xây dựng hạ tầng phục vụ hàng triệu người dùng trên AWS - Bài 1 - 1k người dùng | 29 | 3 | 0 | 6 | 3 |
| 219.md | https://devopsvn.tech/xay-dung-ha-tang-phuc-vu-hang-trieu-nguoi-dung-tren-aws/bai-2-10k-nguoi-dung | Xây dựng hạ tầng phục vụ hàng triệu người dùng trên AWS - Bài 2 - 10k người dùng | 22 | 6 | 0 | 6 | 4 |

## Page THẤT BẠI (HTTP 404 — đã thử lại)

| # | URL | Trạng thái |
|---|---|---|
| 042 | https://devopsvn.tech/chuong-trinh-mentor | HTTP 404 |
| 043 | https://devopsvn.tech/chuong-trinh-mentor/bi-ngc-anh-huy | HTTP 404 |
| 044 | https://devopsvn.tech/chuong-trinh-mentor/nguyn-huy-hong | HTTP 404 |
| 045 | https://devopsvn.tech/chuong-trinh-mentor/nguyn-vn-mnh | HTTP 404 |
| 061 | https://devopsvn.tech/devops/giai-phap-cong-nghe-cho-cac-ung-dung-lon | HTTP 404 |
| 076 | https://devopsvn.tech/devops/redis-mat-du-lieu-khi-restart | HTTP 404 |
| 086 | https://devopsvn.tech/dockerfile | HTTP 404 |
| 087 | https://devopsvn.tech/dockerfile-for-nodejs | HTTP 404 |
| 088 | https://devopsvn.tech/dockerfile/dockerfile-for-go | HTTP 404 |
| 089 | https://devopsvn.tech/dockerfile/dockerfile-for-java-springboot-and-quarkus | HTTP 404 |
| 090 | https://devopsvn.tech/dockerfile/dockerfile-for-net-core | HTTP 404 |
| 091 | https://devopsvn.tech/dockerfile/dockerfile-for-php-laravel | HTTP 404 |
| 092 | https://devopsvn.tech/dockerfile/dockerfile-for-python | HTTP 404 |
| 093 | https://devopsvn.tech/dockerfile/dockerfile-for-react | HTTP 404 |
| 094 | https://devopsvn.tech/dockerfile/dockerfile-for-ruby-on-rails | HTTP 404 |
| 095 | https://devopsvn.tech/dockerfile/dockerfile-for-rust | HTTP 404 |
| 130 | https://devopsvn.tech/learning-path | HTTP 404 |
| 131 | https://devopsvn.tech/learning-path/devops-learning-path | HTTP 404 |
| 132 | https://devopsvn.tech/learning-path/docker-learning-path | HTTP 404 |
| 133 | https://devopsvn.tech/learning-path/linux-learning-path | HTTP 404 |
| 157 | https://devopsvn.tech/new/lit-k-requests-limits-trong-kubernetes-mt-cch-d-dng-vi-kube-capacity-cli | HTTP 404 |
| 160 | https://devopsvn.tech/new/s-dng-ssh-port-forwarding-kt-ni-ti-ti-nguyn-b-chn | HTTP 404 |
| 162 | https://devopsvn.tech/new/service-mesh-on-kubernetes-gii-thiu-istio-istio-l-g | HTTP 404 |
| 163 | https://devopsvn.tech/new/t-ng-ng-b-kubernetes-configmaps-v-secrets-qua-cc-namespaces-khc-vi-reflector | HTTP 404 |
| 179 | https://devopsvn.tech/tai-sao-ban-hoc-english-khong-hieu-qua | HTTP 404 |
| 206 | https://devopsvn.tech/top-post/cicd-l-g-cng-c-continuous-integration | HTTP 404 |
| 207 | https://devopsvn.tech/top-post/cu-hnh-postgresql-master-slave-replication-trn-kubernetes | HTTP 404 |
| 208 | https://devopsvn.tech/top-post/khi-no-th-ta-khng-nn-s-dng-kubernetes | HTTP 404 |
| 209 | https://devopsvn.tech/top-post/kubernetes-lm-vic-vi-container-nh-th-no | HTTP 404 |
| 212 | https://devopsvn.tech/top-post/nhng-cun-sch-nn-c-tr-thnh-devops-cho-ngi-mi-bt-u | HTTP 404 |
| 213 | https://devopsvn.tech/top-post/t-xy-dng-container-vi-go | HTTP 404 |
| 214 | https://devopsvn.tech/top-post/trin-khai-elasticsearch-ln-trn-kubernetes-cloud | HTTP 404 |

## Ghi chú bảo mật
- File `113.md` (ArgoCD with Private Repo) chứa GitHub Personal Access Token thật trong ví dụ gốc của bài viết. Token đã được thay bằng `[REDACTED-GITHUB-TOKEN]` để tuân thủ GitHub push protection (không đẩy secret lên repo). Cấu trúc nội dung còn lại giữ nguyên.
