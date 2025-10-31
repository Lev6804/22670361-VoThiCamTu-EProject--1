# README.md

## 1. Hệ thống giải quyết vấn đề gì

Hệ thống được thiết kế theo mô hình **Microservices**, giải quyết các vấn đề trong quản lý bán hàng trực tuyến, bao gồm:

* Quản lý người dùng (đăng ký, đăng nhập, xác thực JWT).
* Quản lý sản phẩm (thêm, xem, mua hàng).
* Quản lý đơn hàng (xử lý bất đồng bộ thông qua RabbitMQ).
* Đảm bảo khả năng mở rộng, dễ bảo trì và tách biệt chức năng.

![Sơ đồ hệ thống tổng quan](public/image/system-overview.png)

---

## 2. Các dịch vụ trong hệ thống

Dự án bao gồm 6 dịch vụ chính (được định nghĩa trong `docker-compose.yml`):

| Dịch vụ         | Chức năng                                         | Port  |
| --------------- | ------------------------------------------------- | ----- |
| **api-gateway** | Cổng vào chính, điều phối request tới các service | 3003  |
| **auth**        | Quản lý xác thực, JWT, và thông tin người dùng    | 3000  |
| **product**     | Quản lý sản phẩm, gửi thông tin đặt hàng          | 3001  |
| **order**       | Tiếp nhận và xử lý đơn hàng từ RabbitMQ           | 3002  |
| **mongo**       | Cơ sở dữ liệu MongoDB                             | 27017 |
| **rabbitmq**    | Hàng đợi message, kết nối các service             | 5672  |

![Kiến trúc dịch vụ](public/image/services-architecture.png)

---

## 3. Ý nghĩa từng dịch vụ

### **API Gateway**

* Là cầu nối giữa client và các service nội bộ.
* Định tuyến request theo URL.
* Giúp tập trung xử lý bảo mật và logging.

![API Gateway](public/image/api-gateway.png)

### **Auth Service**

* Cung cấp API đăng ký, đăng nhập, xác thực JWT.
* Lưu thông tin người dùng trong MongoDB.

### **Product Service**

* Cung cấp API để thêm, xem danh sách sản phẩm, và mua hàng.
* Khi người dùng mua hàng, service sẽ gửi message vào RabbitMQ.

### **Order Service**

* Nhận message từ RabbitMQ, lưu đơn hàng vào MongoDB.
* Đảm bảo quy trình xử lý đơn hàng diễn ra độc lập và ổn định.

![Order Flow](public/image/order-flow.png)

---

## 4. Các mẫu thiết kế được sử dụng

* **Microservices Architecture:** Mỗi chức năng là một service riêng biệt.
* **API Gateway Pattern:** Làm nhiệm vụ định tuyến và kiểm soát truy cập.
* **MVC Pattern:** Tách lớp controller, service, repository trong từng service.
* **Repository Pattern:** Truy cập dữ liệu tập trung qua repository.
* **Event-driven Architecture:** Giao tiếp bất đồng bộ thông qua RabbitMQ.
* **JWT Authentication:** Xác thực người dùng bằng token.
* **Docker Compose:** Dựng môi trường phát triển toàn bộ hệ thống.

![Mẫu thiết kế hệ thống](public/image/design-patterns.png)

---

## 5. Ví dụ thao tác từng dịch vụ

### 🔹 **Auth Service**

**Đăng ký người dùng:**

```bash
POST /auth/register
{
  "username": "user1",
  "email": "user1@example.com",
  "password": "123456"
}
```

**Đăng nhập:**

```bash
POST /auth/login
{
  "email": "user1@example.com",
  "password": "123456"
}
```

Kết quả trả về JWT token dùng cho các service khác.

![Auth thao tác](public/image/auth-api.png)

---

### 🔹 **Product Service**

**Thêm sản phẩm mới:**

```bash
POST /products/api/products
Authorization: Bearer <JWT_TOKEN>
{
  "name": "Silver Ring",
  "price": 200,
  "description": "Handmade silver ring."
}
```

**Xem danh sách sản phẩm:**

```bash
GET /products/api/products
```

**Mua hàng:**

```bash
POST /products/api/products/buy
Authorization: Bearer <JWT_TOKEN>
{
  "productId": "<id_sản_phẩm>",
  "quantity": 1
}
```

![Product thao tác](public/image/product-api.png)

---

### 🔹 **Order Service**

**Nhận và lưu đơn hàng (tự động từ RabbitMQ):**

```json
{
  "user": "user1",
  "product": "Silver Ring",
  "quantity": 1,
  "totalPrice": 200
}
```

Order được lưu vào MongoDB.

![Order thao tác](public/image/order-queue.png)

---

## 6. Hướng dẫn chạy hệ thống

1. **Clone project:**

```bash
git clone <repo_url>
cd project_folder
```

2. **Khởi chạy hệ thống:**

```bash
docker-compose up --build
```

3. **Truy cập API Gateway:**

```
http://localhost:3003
```

4. **Kiểm tra RabbitMQ Dashboard:**

```
http://localhost:15672 (user: guest / pass: guest)
```

![Chạy hệ thống](public/image/run-system.png)

---

## 7. Kết luận

Hệ thống minh họa cho kiến trúc **Microservices** gồm các service Auth – Product – Order được điều phối qua **API Gateway**, giao tiếp bằng **RabbitMQ**, và quản lý bởi **Docker Compose**. Cấu trúc này đảm bảo tính mở rộng, linh hoạt và dễ bảo trì cho ứng dụng thương mại điện tử.
