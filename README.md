# Bài tập 3   : môn Phát triển ứng dụng trên nền web
# Lường Văn Hạnh - K225480106013
### Giảng viên  : Đỗ Duy Cốp
### Lớp học phần: 58KTPM
### Ngày giao   : 2025-10-24 13:50
### Hạn nộp     : 2025-11-05 00:00
--------------------------------------------------
# Yêu cầu     : LẬP TRÌNH ỨNG DỤNG WEB trên nền linux
## 1. Cài đặt môi trường linux: SV chọn 1 trong các phương án
 - enable wsl: cài đặt docker desktop
 - enable wsl: cài đặt ubuntu
 - sử dụng Hyper-V: cài đặt ubuntu
 - sử dụng VMware : cài đặt ubuntu
 - sử dụng Virtual Box: cài đặt ubuntu
   
Em chọn cài đặt docker desktop:

Đàu tiên tải file này về:
<img width="1902" height="909" alt="image" src="https://github.com/user-attachments/assets/fbac6922-3fd5-4e65-8481-116aec7df6aa" />

Sau đó tắt dịch vụ chiếm các cổng cần thiết như IIS và bật WSL lên. Sau đó chạy lệnh wsl --list --verbose.
<img width="1208" height="327" alt="image" src="https://github.com/user-attachments/assets/84ab7c29-cb2a-49bd-8c27-4d82ed21066f" />

## 2. Cài đặt Docker (nếu dùng docker desktop trên windows thì nó có ngay)
Thiết lập cấu trúc thư mục:
<img width="1919" height="652" alt="image" src="https://github.com/user-attachments/assets/b8fa4716-6b55-4656-b73d-916e6d80a1e2" />

## 3. Sử dụng 1 file docker-compose.yml để cài đặt các docker container sau: 
   mariadb (3306), phpmyadmin (8080), nodered/node-red (1880), influxdb (8086), grafana/grafana (3000), nginx (80,443)

   Cấu hình file docker-compose.yml:
```   
   version: '3.8'

services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: appdb
      MYSQL_USER: user
      MYSQL_PASSWORD: user123
    volumes:
      - ./data/mariadb:/var/lib/mysql
    ports:
      - "3306:3306"
  
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    restart: always
    environment:
      PMA_HOST: mariadb
      PMA_USER: root
      PMA_PASSWORD: root
    ports:
      - "8080:80"
    depends_on:
      - mariadb

  nodered:
    image: nodered/node-red:latest
    container_name: nodered
    restart: always
    ports:
      - "1880:1880"
    volumes:
      - ./data/nodered:/data

  influxdb:
    image: influxdb:latest
    container_name: influxdb
    restart: always
    ports:
      - "8086:8086"
    volumes:
      - ./data/influxdb:/var/lib/influxdb

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: always
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ALLOW_EMBEDDING=true
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Viewer
    volumes:
      - ./data/grafana:/var/lib/grafana
      - ./data/grafana/grafana.ini:/etc/grafana/grafana.ini   # ✅ Mount file cấu hình mới
    depends_on:
      - influxdb

  nginx:
    image: nginx:latest
    container_name: nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./web:/usr/share/nginx/html:ro
    depends_on:
      - nodered
      - grafana
 ```   
<img width="1919" height="1006" alt="image" src="https://github.com/user-attachments/assets/81f67983-14a0-4514-a44b-56131d551c07" />

## 4. Lập trình web frontend+backend:
 ### 4.1 Web thương mại điện tử
 - Tạo web dạng Single Page Application (SPA), chỉ gồm 1 file index.html, toàn bộ giao diện do javascript sinh động.
 - Có tính năng login, lưu phiên đăng nhập vào cookie và session
   <img width="839" height="427" alt="image" src="https://github.com/user-attachments/assets/4a44a47e-43b1-4503-b3fd-db58c7e3ca88" />

   Thông tin login lưu trong cơ sở dữ liệu của mariadb, được dev quản trị bằng phpmyadmin, yêu cầu sử dụng mã hoá khi gửi login.
   
  Chưa có mã hoá khi gửi login:
   <img width="843" height="433" alt="image" src="https://github.com/user-attachments/assets/7fdb99d9-9d07-470f-b856-15fe081b23bf" />

   Chỉ cần login 1 lần, bao giờ logout thì mới phải login lại.
   Mỗi khi reload lại trang thì sẽ request api/check với username+cookie để kiểm trả tình trạng đăng nhập cảu user đó:
   <img width="846" height="429" alt="image" src="https://github.com/user-attachments/assets/86540403-d658-4d5d-93e6-f2d3f491b02d" />

 - Có tính năng liệt kê các sản phẩm bán chạy ra trang chủ
 - Có tính năng liệt kê các nhóm sản phẩm
   
   Em chưa kịp cho ảnh sản phẩm, trông hơi tệ:
   <img width="1919" height="888" alt="image" src="https://github.com/user-attachments/assets/3c55a205-a718-4ce6-9617-77e5e9589a91" />
   
 - Có tính năng liệt kê sản phẩm theo nhóm
   <img width="1919" height="958" alt="image" src="https://github.com/user-attachments/assets/8318fc26-aeb9-46d2-8484-9fc982c94337" />

 - Có tính năng tìm kiếm sản phẩm
   <img width="1919" height="956" alt="image" src="https://github.com/user-attachments/assets/562f7472-bca4-49db-8e62-2907d4f9ac1c" />

 - Có tính năng chọn sản phẩm (đưa sản phẩm vào giỏ hàng, thay đổi số lượng sản phẩm trong giỏ, cập nhật tổng tiền)
   <img width="1919" height="813" alt="image" src="https://github.com/user-attachments/assets/23ccbca3-4ad3-4e7f-a5d2-b866ce1d1eff" />
   <img width="1919" height="838" alt="image" src="https://github.com/user-attachments/assets/302f0eda-5c91-4234-8f46-96b925f2de8f" />
   
 - Có tính năng đặt hàng, nhập thông tin giao hàng => được 1 đơn hàng.
   <img width="1919" height="735" alt="image" src="https://github.com/user-attachments/assets/06680d59-48f5-465e-be38-b82594e5ac44" />

 - Có tính năng dành cho admin: Thống kê xem có bao nhiêu đơn hàng, call để xác nhận và cập nhật thông tin đơn hàng. chuyển cho bộ phận đóng gói, gửi bưu điện, cập nhật mã COD, tình trạng giao hàng, huỷ hàng,...
 - 
   Phần này em chưa kịp làm.
   
 - Có tính năng dành cho admin: biểu đồ thống kê số lượng mặt hàng bán được trong từng ngày. (sử dụng grafana)
   <img width="1919" height="1046" alt="image" src="https://github.com/user-attachments/assets/4459dff4-3480-4d7b-be9c-fe8c1f8cb827" />
 - backend: sử dụng nodered xử lý request gửi lên từ javascript, phản hồi về json
  <img width="1919" height="966" alt="image" src="https://github.com/user-attachments/assets/7b7548a6-7eab-4218-837b-dc01aed05250" />
  
  <img width="1919" height="1035" alt="image" src="https://github.com/user-attachments/assets/452c0eef-4d13-470d-a4f2-d2ef57948fee" />

## 5. Nginx làm web-server
 - Cấu hình nginx để chạy được website qua url http://fullname.com  (thay fullname bằng chuỗi ko dấu viết liền tên của bạn)
 - Cấu hình nginx để http://fullname.com/nodered truy cập vào nodered qua cổng 80, (dù nodered đang chạy ở port 1880)
 - Cấu hình nginx để http://fullname.com/grafana truy cập vào grafana qua cổng 80, (dù grafana đang chạy ở port 3000)
```
   events {}

http {
  server {
    listen 80;
    server_name luongvanhanh.com;

    # Website chính (index.html)
    location / {
      root /usr/share/nginx/html;
      index index.html;
    }

    # Proxy tới Node-RED (backend)
    location /nodered/ {
      proxy_pass http://nodered:1880/;
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_cache_bypass $http_upgrade;
    }

    # Proxy tới Grafana (admin dashboard)
    location /grafana/ {
      proxy_pass http://grafana:3000/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;

      # ✅ Cho phép iframe embed (rất quan trọng)
      proxy_hide_header X-Frame-Options;
      add_header X-Frame-Options "ALLOWALL";
      add_header Content-Security-Policy "frame-ancestors *";
    }
  }
}
```
## Yêu cầu sinh viên lưu code trên github
có file readme.md có hình ảnh + text: ghi lại nhật ký quá trình làm bài.

## CÁCH ĐÁNH GIÁ:
1. Cài đặt được môi trường: 1đ
2. Cài đặt được các docker container với cấu hình phù hợp: 1đ
3. Web chạy được, giao diện phù hợp, chạy trên web sever nginx: 2đ
4. nodered api trả về json, test được: 2đ
5. front-end có js gọi được api nodered, nhận về json, hiển thị được kết quả từ json này. 2đ
6. Bài làm có dấu ấn, giải thích rõ ràng, hiểu vấn đề: 2đ
