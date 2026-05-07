# PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ
# BÀI TẬP 2: DJANGO
# Sinh viên: Đậu Văn Khánh
# MSSV: K225480106099
# Lớp: K58KTP


# ĐỀ BÀI:
## TỔ CHỨC CSDL CHO HỆ THỐNG QUẢN LÝ TIỆM CẦM ĐỒ: viết tay ra giấy, lấy điện thoại chụp lại, upload ảnh lên github (đã nói về các nghiệp vụ trên lớp, ghi bảng)
## SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ:
- Mariadb : chứa csdl của hệ thống này
- Phpmyadmin: để soi được csdl (chỉ để xem, ko cần tạo bảng từ đây, django sẽ làm hết)
- Django: build 1 docker container (dùng Dockerfile): trên nền python, sử dụng django, nhớ mount thư mục để dễ edit, edit dùng: sudo nano ten_file
- Sau khi có 3 service này trong file docker-compose.yml :
  + Run nó, cấu hình để Django nhận csdl mariadb (sửa file settings.py), cấu hình user login ban đầu, mô tả các bảng trong models.py, .... (đc phép sử dụng AI để làm) => KQ được trang admin, y/c đăng nhập, vào trang admin: cho phép thêm sửa xoá dữ liệu các bảng. các trường là khoá ngoại chỉ việc chọn text (mặc dù là csdl tại trường FK đó lưu ID của PK mà nó tham chiếu : sử dụng phpmyadmin để kiểm chứng).
  + Chú ý kết hợp ssh để chạy lệnh tác động vào django và sudo nano để edit file.
  + Sử dụng template (file html, sử dụng cú pháp jinja2), lấy context từ 1 view home_page, để tạo trang liệt kê các con nợ đến hạn mà chưa trả tiền!
  + Sử dụng cloudflare tunnel để public kết quả lên 1 sub-domain => chụp kết quả
- Hướng dẫn:
  + Tạo thư mục để chứa image tự buid cho django
  + Vào thư mục đó tạo file tên: Dockerfile (nội dung hỏi AI xem file này cần có nội dung gì, full comment cho từng dòng lệnh trong file này => mục tiêu kép: để hiểu và để hệ thống chạy được)
  + AI sẽ nói cần thêm file requirements.txt để cài các thư viện cho python (cài qua lệnh pip) => tạo file requirements.txt với nội dung tưng ứng, trong file này cũng comment được => comment xem thư viện nào dùng để làm gì.
  + Sau mỗi lần sửa đỏi có thể phải chạy lệnh dạng : docker compose exec TÊN_SERVICE_DJANGO_CỦA_BẠN python manage.py migrate để tác động vào django (còn nhiều lệnh khác chứ ko luôn như này), để django thay đổi csdl hoặc thay đổi cấu hình.

 # BÀI LÀM
 ## 1. Tổ chức CSDL cho hệ thống quản lý tiệm cầm đồ

 ## 2. Cài đặt Ubuntu
 ### Cập nhật Ubuntu
 - Chạy lệnh:
```
sudo apt update
sudo apt upgrade -y
```
### Cài Docker
- Chạy lệnh: ```sudo apt install docker.io -y```
- Kiểm tra: ```docker --version```
<img width="656" height="52" alt="image" src="https://github.com/user-attachments/assets/99e6e45e-6596-4323-805c-8f9cd1d7d34a" />

### Cài Docker compose
- Kiểm tra: ```docker-compose version```
<img width="572" height="141" alt="image" src="https://github.com/user-attachments/assets/4dc10b7b-84b8-4b64-9850-8fb438312940" />

## 3. Tạo thư mục project
### Tạo thư mục
```
mkdir pawnshop_project
cd pawnshop_project
```

### Tạo thư mục Django
```mkdir django_app```

- Cấu trúc thư mục:
```
pawnshop_project/
│
├── docker-compose.yml
│
└── django_app/
    ├── Dockerfile
    └── requirements.txt
```

## 4. Tạo Dockerfile
### Vào thư mục: 
```cd django_app```
### Tạo file
```sudo nano Dockerfile```
- Nội dung Dockerfile
```
# Sử dụng Python 3.12 chính thức
FROM python:3.12

# Không tạo file pyc
ENV PYTHONDONTWRITEBYTECODE=1

# Hiển thị log realtime
ENV PYTHONUNBUFFERED=1

# Thư mục làm việc trong container
WORKDIR /app

# Copy requirements vào container
COPY requirements.txt .

# Cài thư viện Python
RUN pip install --no-cache-dir -r requirements.txt

# Copy toàn bộ source code
COPY . .

# Mở cổng Django
EXPOSE 8000

# Chạy Django server
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```
<img width="1470" height="751" alt="image" src="https://github.com/user-attachments/assets/d4c538bc-b712-4b80-9333-7aa5023efda4" />

- Giải thích:
```
| Dòng    | Ý nghĩa           |
| ------- | ----------------- |
| FROM    | dùng image Python |
| ENV     | tối ưu Python     |
| WORKDIR | thư mục làm việc  |
| COPY    | copy file         |
| RUN     | chạy lệnh         |
| EXPOSE  | mở port           |
| CMD     | lệnh khởi động    |
```

## 5. Tạo requirements.txt
### Tạo file: ```sudo nano requirements.txt```
### Nội dung:
```
# Framework web Django
django

# Driver kết nối MariaDB/MySQL
mysqlclient

# Thư viện hỗ trợ MySQL
PyMySQL
```

## 6. Tạo docker-compose.yml
- Tạo file: ```sudo nano docker-compose.yml```
- Nội dung file:
```
services:

  mariadb:
    image: mariadb:11
    container_name: pawnshop_mariadb

    restart: always

    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: pawnshop_db
      MYSQL_USER: pawn_user
      MYSQL_PASSWORD: pawn123

    ports:
      - "3307:3306"

    volumes:
      - mariadb_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin

    container_name: pawnshop_phpmyadmin

    restart: always

    ports:
      - "8080:80"

    environment:
      PMA_HOST: mariadb
      MYSQL_ROOT_PASSWORD: root123

    depends_on:
      - mariadb

  django:
    build: ./django_app

    container_name: pawnshop_django

    restart: always

    ports:
      - "8000:8000"

    volumes:
      - ./django_app:/app

    depends_on:
      - mariadb

volumes:
  mariadb_data:
```
<img width="1479" height="756" alt="image" src="https://github.com/user-attachments/assets/4e0d5879-eb64-4df7-8d70-d6220d110744" />

## 7. Build và chạy lại container
- Chạy lệnh: ```docker-compose up --build -d```
<img width="1477" height="751" alt="image" src="https://github.com/user-attachments/assets/9cec4054-a5fd-484c-99e4-f946c9b6c4d2" />

- Kiểm tra: ```docker ps```
<img width="1478" height="247" alt="image" src="https://github.com/user-attachments/assets/d7d2139f-f054-4f98-8213-7020e4bffffe" />

## 8. Tạo Django project
- Vào container Django: ```docker-compose exec django bash```
- Tạo project: ```django-admin startproject pawnshop .```
- Tạo app: ```python manage.py startapp management```

## 9. Cầu hình Django
- Mở settings.py: ```nano pawnshop/settings.py```
- Sửa ALLOWED_HOSTS
  + Tìm dòng: ALLOWED_HOSTS = []
  + Sửa thành: ALLOWED_HOSTS = ['*']
- Thêm App management
  + Tìm dòng: INSTALLED_APPS = [
  + Thêm: 'management',
<img width="491" height="272" alt="image" src="https://github.com/user-attachments/assets/4c2a6be0-3ca2-45e8-aff3-a44e832717ed" />

- Sửa Databases
  + Tìm đoạn: DATABASES = {
  + Xóa toàn bộ block đó và thay bằng:
```DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',

        'NAME': 'pawnshop_db',

        'USER': 'pawn_user',

        'PASSWORD': 'pawn123',

        'HOST': 'mariadb',

        'PORT': '3306',
    }
}
```
<img width="1469" height="744" alt="image" src="https://github.com/user-attachments/assets/d05b950a-1310-4918-9954-a63bb80a43d9" />

## 10. Tạo Model
- Mở file: ```nano management/models.py```
- Nội dung:
```
from django.db import models


class Customer(models.Model):

    full_name = models.CharField(max_length=100)

    phone = models.CharField(max_length=20)

    address = models.TextField()

    citizen_id = models.CharField(max_length=20)

    def __str__(self):
        return self.full_name


class PawnItem(models.Model):

    customer = models.ForeignKey(
        Customer,
        on_delete=models.CASCADE
    )

    item_name = models.CharField(max_length=200)

    description = models.TextField()

    estimated_value = models.IntegerField()

    image = models.CharField(
        max_length=255,
        blank=True
    )

    def __str__(self):
        return self.item_name


class LoanContract(models.Model):

    customer = models.ForeignKey(
        Customer,
        on_delete=models.CASCADE
    )

    pawn_item = models.ForeignKey(
        PawnItem,
        on_delete=models.CASCADE
    )

    loan_amount = models.IntegerField()

    interest_rate = models.FloatField()

    start_date = models.DateField()

    due_date = models.DateField()

    status = models.CharField(
        max_length=50,
        default='Đang vay'
    )

    def __str__(self):
        return f"HD-{self.id}"


class Payment(models.Model):

    contract = models.ForeignKey(
        LoanContract,
        on_delete=models.CASCADE
    )

    payment_date = models.DateField()

    amount = models.IntegerField()

    note = models.TextField(blank=True)

    def __str__(self):
        return f"Payment-{self.id}"
```

<img width="1477" height="752" alt="image" src="https://github.com/user-attachments/assets/88353d33-32ef-4686-92e1-c2d4155ad72b" />

## 11. Tạo bảng Database
- Tạo Migration: ```python manage.py makemigrations```
<img width="795" height="164" alt="image" src="https://github.com/user-attachments/assets/ce37ccd0-e0f8-445a-b14e-6beb5c28b09c" />

- Tạo Migrate: ```python manage.py migrate```
<img width="892" height="571" alt="image" src="https://github.com/user-attachments/assets/c97180e2-b1cf-442d-8398-9ff43c037105" />

## 12. Tạo Admin Django
- Mở file: ```nano management/admin.py```
- Nội dung:
```
from django.contrib import admin

from .models import *

admin.site.register(Customer)
admin.site.register(PawnItem)
admin.site.register(LoanContract)
admin.site.register(Payment)
```
<img width="1470" height="747" alt="image" src="https://github.com/user-attachments/assets/9fcd7d17-1889-427d-9680-9e9f75321963" />

## 13. Tạo tài khoản Admin
- Chạy lệnh: ```python manage.py createsuperuser```
- Nhập thông tin:
  + Username: admin
  + Email: Tùy ý
  + Password: Tùy ý
<img width="819" height="259" alt="image" src="https://github.com/user-attachments/assets/837ee924-bd20-40b3-aa16-a28f9488dd15" />

## 14. Mở Django Admin
- Trên trình duyệt, truy cập: http://192.168.91.154:8000/admin
- Giao diện đăng nhập tài khoản Admin:
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/3d7c862f-63cb-4ee5-a908-3b6c26af22d5" />

- Giao diện sau khi đăng nhập:
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/1ebed5d8-6830-42cc-a670-e109690fb337" />

- Thêm khách hàng
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/18d11930-1a02-44dd-aa31-0d970e363950" />

- Thêm hợp đồng vay
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/16e195fe-4fc2-4a8b-870c-3c4efbacab2a" />

## 15. Kiểm tra phpMyAdmin
- Truy cập: http://192.168.91.154:8080/
- Đăng nhập tài khoản:
```
| Field    | Value   |
| -------- | ------- |
| server   | mariadb |
| user     | root    |
| password | root123 |
```

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/2d6a8d13-9c16-452d-b1bd-3c56bd8a1418" />

- Giao diện sau khi đăng nhập
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/49685a76-876a-46e3-a6b1-01bfd9aafd27" />

- Kiểm tra các bảng
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/fd758455-75ec-47ac-a341-1f6d7174b64b" />

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/22ee5cf6-0938-47d8-924d-03f969106eb4" />

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/2c3b7f95-5f0e-4ae3-8e33-a96544bb52a2" />

## 16. Tạo Template HTML
- Vào thư mục project: ```cd ~/pawnshop_project/django_app```
- Tạo thư mục: ```mkdir templates```
- Tạo file: ```nano templates/home.html```
- Nội dung file:
```
<!DOCTYPE html>
<html>

<head>
    <title>Danh sách nợ quá hạn</title>
</head>

<body>

<h1>Danh sách khách nợ quá hạn</h1>

<table border="1">

<tr>
    <th>Khách hàng</th>
    <th>Tài sản</th>
    <th>Số tiền vay</th>
    <th>Ngày đến hạn</th>
</tr>

{% for contract in contracts %}

<tr>

    <td>{{ contract.customer.full_name }}</td>

    <td>{{ contract.pawn_item.item_name }}</td>

    <td>{{ contract.loan_amount }}</td>

    <td>{{ contract.due_date }}</td>

</tr>

{% endfor %}

</table>

</body>
</html>
```

## 17. Tạo view
- Mở file: ```nano management/views.py```
- Nội dung:
```
from django.shortcuts import render

from datetime import date

from .models import LoanContract


def home_page(request):

    contracts = LoanContract.objects.filter(
        due_date__lt=date.today(),
        status='Đang nợ'
    )

    return render(
        request,
        'home.html',
        {
            'contracts': contracts
        }
    )
```
<img width="1476" height="745" alt="image" src="https://github.com/user-attachments/assets/08c1e779-09ce-4c9b-93da-793c6277b3b8" />

## 18. Tạo URL
- Mở file: ```nano pawnshop/urls.py```
- Nội dung:
```
from django.contrib import admin
from django.urls import path

from management.views import home_page

urlpatterns = [

    path('admin/', admin.site.urls),

    path('', home_page),

]
```
<img width="1468" height="755" alt="image" src="https://github.com/user-attachments/assets/d2b0dac4-f509-40c2-a628-e7c00bf6c9eb" />

## Restart lại
- Chạy lệnh: ```cd ~/pawnshop_project```
- ```docker-compose restart django```
<img width="819" height="84" alt="image" src="https://github.com/user-attachments/assets/6ef84ee8-93a4-4b54-bde7-753625e08c86" />

- Truy cập: http://192.168.91.154:8000/
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/a913bd26-6e76-4d7e-9cfb-f5d97cdb2300" />

## 19. Public bằng Cloudflare Tunnel
### Cài Cloudflare trên Ubuntu
- Download về:
```wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb```
<img width="1472" height="652" alt="image" src="https://github.com/user-attachments/assets/8a20330a-6380-49d5-b0a9-54debe033d2f" />

- Cài: ```sudo dpkg -i cloudflared-linux-amd64.deb```
- Kiểm tra: ```cloudflared --version```
<img width="935" height="272" alt="image" src="https://github.com/user-attachments/assets/7bdcfdac-d769-4262-a287-9be265936fe0" />

### Đăng nhập Cloudflare
- Chạy: cloudflared tunnel login
<img width="1475" height="192" alt="image" src="https://github.com/user-attachments/assets/dcb38270-9e7f-4d0e-986c-0a233a3d0ee0" />

- Ubuntu hiện link: https://dash.cloudflare.com/argotunnel?aud=&callback=https%3A%2F%2Flogin.cloudflareaccess.org%2FZqlA8KB8WO_UVqOi8H7vXaZRhgoQO5uusbxzSIkdIn8%3D
- Copy link và mở trình duyệt chạy.
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/c5168f33-fc6d-4494-a2fc-e45824e5e727" />

- Cloudflare sẽ hỏi: Authorize Cloudflared -> chọn domain -> nhấn Authorize
- Trong Terminal Ubuntu sẽ hiện: You have successfully logged in.
<img width="1473" height="330" alt="image" src="https://github.com/user-attachments/assets/ec4b45aa-408e-4a92-9100-b3525de5eab5" />

### Tạo Tunel
- Chạy lệnh: ```cloudflared tunnel create pawnshop```
- Kết quả hiển thị
<img width="1476" height="185" alt="image" src="https://github.com/user-attachments/assets/cf5d441c-52d5-4199-aef8-64d5fa676afe" />

### Tạo file config
- Tạo file: ```mkdir -p ~/.cloudflared```
- Chạy lệnh: ```nano ~/.cloudflared/config.yml```
- Nội dung file:
```
tunnel: 28543b52-e6a6-49a9-8907-cbf26cdb024d

credentials-file: /home/khanh/.cloudflared/28543b52-e6a6-49a9-8907-cbf26cdb024d.json

ingress:
  - hostname: khanh123.id.vn
    service: http://localhost:8000

  - service: http_status:404
```

- Kiểm tra file: ```cat ~/.cloudflared/config.yml```
<img width="1043" height="265" alt="image" src="https://github.com/user-attachments/assets/da47b7f5-190d-463d-a01f-fe4590112258" />

### Tạo DNS
- Chạy lệnh: ```cloudflared tunnel route dns pawnshop camdo.khanh123.id.vn```
<img width="1472" height="103" alt="image" src="https://github.com/user-attachments/assets/7be59d68-cd08-45bf-99c5-14797b6cd437" />

- Kiểm tra DNS trên Cloudflare
<img width="1917" height="1152" alt="image" src="https://github.com/user-attachments/assets/2cacfe9a-2885-4a7d-b3bd-0525b4b23355" />

### Chạy Tunnel
- Chạy tunnel: ```cloudflared tunnel run pawnshop```
<img width="1476" height="752" alt="image" src="https://github.com/user-attachments/assets/4303757b-652f-425f-9102-18adb8567cb2" />

- Lưu ý:
  + Không được tắt Terminal.
  + Nếu tắt -> Web sẽ off

## 20. Truy cập website
- Truy cập: https://camdo.khanh123.id.vn/
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/04251410-4c0f-4f34-a5de-2e5a274d1b70" />

- Truy cập Admin Django: https://camdo.khanh123.id.vn/admin
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/706bb53f-d47b-4a0f-9635-ff548b247c61" />

- Khi đăng nhập vào Admin Django sẽ bị lỗi hệ thống
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/e0cab5c5-4995-4333-90b7-320230e469ab" />

- Em cần bổ sung trong file settings.py
  + Gõ lệnh: ```sudo nano django_app/pawnshop/settings.py```
  + Thêm CSRF_TRUSTED_ORIGINS
```
CSRF_TRUSTED_ORIGINS = [
    "https://camdo.khanh123.id.vn",
]
```
<img width="1477" height="753" alt="image" src="https://github.com/user-attachments/assets/3b139f30-c304-4e4b-ad8f-c0c4f3235e2a" />

- Restart lại: ```docker-compose restart django```

- Trước khi truy cập lại trang web, cần khởi chạy lại Cloudflare Tunnel: ```cloudflared tunnel run pawnshop```
  + Nếu không thực hiện bước này, hệ thống sẽ không tạo kết nối ra Internet, vì vậy website sẽ không thể truy cập từ trình duyệt bên ngoài.

- Truy cập lại: https://camdo.khanh123.id.vn/admin và đăng nhập
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/c35a62fc-5c8d-4d39-ac32-bd8bf478a810" />

# KẾT LUẬN
- Sau khi hoàn thành bài tập, hệ thống quản lý tiệm cầm đồ đã được xây dựng thành công dựa trên Django, kết hợp với MariaDB làm hệ quản trị cơ sở dữ liệu và phpMyAdmin để hỗ trợ quan sát dữ liệu. Toàn bộ hệ thống được triển khai trong môi trường Docker trên Ubuntu, giúp việc cài đặt, vận hành và quản lý trở nên thống nhất và dễ dàng hơn.
- Trong quá trình thực hiện, em đã xây dựng được các thành phần chính của một ứng dụng web hoàn chỉnh như: thiết kế cơ sở dữ liệu, định nghĩa models trong Django, tạo giao diện quản trị (Django Admin) để thực hiện các thao tác thêm, sửa, xóa dữ liệu, đồng thời xây dựng giao diện người dùng bằng template HTML để hiển thị danh sách các khách hàng/con nợ đến hạn.
- Bên cạnh đó, hệ thống còn được tích hợp Cloudflare Tunnel để public website ra Internet thông qua domain riêng, giúp có thể truy cập từ bên ngoài một cách thuận tiện. Việc triển khai này giúp em hiểu rõ hơn về cách đưa một ứng dụng web từ môi trường local lên môi trường internet thực tế.
- Qua bài tập này, em đã nắm vững hơn các kiến thức về Django, Docker, cơ sở dữ liệu quan hệ và quy trình triển khai hệ thống web hoàn chỉnh. Đồng thời, em cũng rèn luyện được kỹ năng xử lý lỗi, cấu hình hệ thống và làm việc với nhiều công nghệ tích hợp trong một dự án thực tế.
