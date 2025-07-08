# 🐬 MySQL Replication Helm Chart (Master-Slave) with ArgoCD

Triển khai hệ thống MySQL Master-Slave Replication trên Kubernetes bằng Helm Chart kết hợp với ArgoCD để dễ dàng quản lý, đồng bộ và phát triển.

---

## 📁 Cấu trúc thư mục

```bash
argocd-mysql-helmchart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── configmap-master-init.yaml
    ├── configmap-slave-init.yaml
    ├── configmap-mysql-master-conf.yaml
    ├── configmap-mysql-slave-conf.yaml
    ├── mysql-master-deployment.yaml
    ├── mysql-slave-deployment.yaml
    ├── mysql-master-service.yaml
    ├── mysql-slave-service.yaml
    ├── mysql-master-pvc.yaml
    └── secrets.yaml
📜 Giải thích từng file
1. Chart.yaml
Định nghĩa metadata cho Helm chart: tên chart, version, mô tả.

Là file bắt buộc trong mọi Helm project.

2. values.yaml
Cung cấp các biến cấu hình mặc định cho chart.

Bao gồm các thông tin như:

image MySQL

thông tin database

mật khẩu mặc định

📂 templates/: Các file template để Helm render tài nguyên
🔐 secrets.yaml
Lưu các bí mật như:

MYSQL_ROOT_PASSWORD

MYSQL_REPLICATION_USER

MYSQL_REPLICATION_PASSWORD

Được mount vào Pod qua biến môi trường.

📦 configmap-master-init.yaml
Script init cho container master MySQL.

Tạo user repl, cấp quyền REPLICATION, bật log_bin, thiết lập gtid (nếu cần).

📦 configmap-slave-init.yaml
Script init cho slave để cấu hình kết nối với master.

Chạy lệnh CHANGE MASTER TO ... khi container khởi động lần đầu.

⚙️ configmap-mysql-master-conf.yaml & configmap-mysql-slave-conf.yaml
File .cnf cấu hình cụ thể cho MySQL:

Bật server-id, log_bin, read_only, v.v.

Gắn vào /etc/mysql/conf.d để cấu hình runtime.

📦 mysql-master-pvc.yaml
Khai báo PersistentVolumeClaim để lưu trữ dữ liệu master.

Giúp giữ dữ liệu khi Pod bị xóa/restart.

🚀 Triển khai
🛠 mysql-master-deployment.yaml
Khai báo Deployment của Pod Master:

Mount ConfigMap init & conf

Mount Secret

Mount PVC /var/lib/mysql

Set MYSQL_SERVER_ID = 1

Dùng init-container nếu có cần chuẩn bị file initdb.

🛠 mysql-slave-deployment.yaml
Khai báo Deployment của Pod Slave:

Mount ConfigMap init & conf

Mount Secret

Mount PVC nếu cần mở rộng

MYSQL_SERVER_ID đặt khác master (ví dụ: "2")

🔌 mysql-master-service.yaml
Tạo Service kiểu ClusterIP cho Pod Master.

Được dùng để các Pod slave kết nối tới (mysql-master:3306).

🔌 mysql-slave-service.yaml
Service dành cho nhóm Pod Slave.

Dùng cho việc truy cập hoặc test đọc dữ liệu từ các slave.

⚙️ ArgoCD
ArgoCD sẽ đọc manifest YAML từ repo GitHub và apply chúng lên cụm Kubernetes.

Mỗi lần thay đổi cấu hình, chỉ cần commit & push:

bash
Copy
Edit
git commit -am "Update config"
git push origin main
Sau đó vào UI ArgoCD để Sync lại ứng dụng.

✅ Kiểm tra hoạt động replication
Truy cập Pod master:

bash
Copy
Edit
kubectl exec -it <master-pod> -n tri-mysql -- bash
mysql -uroot -p
SHOW MASTER STATUS\G
Truy cập Pod slave:

bash
Copy
Edit
kubectl exec -it <slave-pod> -n tri-mysql -- bash
mysql -uroot -p
SHOW SLAVE STATUS\G
Kiểm tra Slave_IO_Running: Yes và Slave_SQL_Running: Yes.