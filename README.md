# Tự triển khai n8n có SSL trên máy chủ Linux

Hướng dẫn này giới thiệu các bước chi tiết để tự triển khai [n8n](https://n8n.io), công cụ tự động hóa workflow miễn phí và mã nguồn mở, trên máy chủ Linux bằng Docker, Nginx và Certbot để cài SSL với tên miền cá nhân.

## Bước 1: Cài đặt Docker

1. **Cập nhật danh sách gói:**
   ```bash
   sudo apt update
   ```
2. **Cài đặt Docker:**
   ```bash
   sudo apt install docker.io
   ```
3. **Khởi động Docker:**
   ```bash
   sudo systemctl start docker
   ```
4. **Kích hoạt Docker khởi chạy cùng hệ thống:**
   ```bash
   sudo systemctl enable docker
   ```

## Bước 2: Khởi chạy n8n trong Docker

Chạy lệnh sau để khởi chạy n8n trong Docker. Thay `your-domain.com` bằng tên miền thực tế của bạn:

```bash
sudo docker run -d --restart unless-stopped -it \
  --name n8n \
  -p 5678:5678 \
  -e N8N_HOST="your-domain.com" \
  -e WEBHOOK_TUNNEL_URL="https://your-domain.com/" \
  -e WEBHOOK_URL="https://your-domain.com/" \
  -v ~/.n8n:/root/.n8n \
  n8nio/n8n
```

Nếu sử dụng subdomain, lệnh sẽ như sau:

```bash
sudo docker run -d --restart unless-stopped -it \
  --name n8n \
  -p 5678:5678 \
  -e N8N_HOST="subdomain.your-domain.com" \
  -e WEBHOOK_TUNNEL_URL="https://subdomain.your-domain.com/" \
  -e WEBHOOK_URL="https://subdomain.your-domain.com/" \
  -v ~/.n8n:/root/.n8n \
  n8nio/n8n
```

Lệnh trên thực hiện các công việc:

- Tải và chạy image Docker của n8n.
- Mở cổng 5678 cho n8n.
- Thiết lập các biến môi trường cho host và webhook.
- Gắn thư mục dữ liệu n8n để lưu trữ lâu dài.
- Sau khi chạy xong, bạn có thể truy cập n8n tại `your-domain.com:5678`.

## Bước 3: Cài đặt Nginx

Nginx được dùng làm reverse proxy chuyển tiếp request đến n8n và xử lý SSL.

1. **Cài đặt Nginx:**
   ```bash
   sudo apt install nginx
   ```

## Bước 4: Cấu hình Nginx

Cấu hình Nginx để reverse proxy giao diện web n8n:

1. **Tạo file cấu hình mới:**
   ```bash
   sudo nano /etc/nginx/sites-available/n8n.conf
   ```
2. **Dán cấu hình sau:**
   ```bash
   server {
       listen 80;
       server_name your-domain.com; # hoặc subdomain.your-domain.com nếu dùng subdomain

       location / {
           proxy_pass http://localhost:5678;
           proxy_http_version 1.1;
           chunked_transfer_encoding off;
           proxy_buffering off;
           proxy_cache off;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
       }
   }
   ```
   Thay `your-domain.com` bằng tên miền thực tế.

3. **Kích hoạt cấu hình:**
   ```bash
   sudo ln -s /etc/nginx/sites-available/n8n.conf /etc/nginx/sites-enabled/
   ```
4. **Kiểm tra và khởi động lại Nginx:**
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

## Bước 5: Cài SSL với Certbot

Certbot sẽ xin và cài chứng chỉ SSL từ Let's Encrypt.

1. **Cài Certbot và plugin Nginx:**
   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```
2. **Yêu cầu chứng chỉ SSL:**
   ```bash
   sudo certbot --nginx -d your-domain.com
   # Nếu dùng subdomain thì thay bằng subdomain.your-domain.com
   ```

Làm theo hướng dẫn trên màn hình để hoàn tất cài đặt SSL. Sau khi xong, n8n sẽ truy cập được qua HTTPS tại `your-domain.com`.

**LƯU Ý:** Thực hiện các bước trên theo đúng thứ tự. Bước 5 sẽ thay đổi file `/etc/nginx/sites-available/n8n.conf` thành cấu hình tương tự như hình dưới đây:

![image](/ssl.png)

Bằng cách dùng Nginx và Certbot, bạn đảm bảo n8n của mình được truy cập an toàn trên Internet qua HTTPS.
