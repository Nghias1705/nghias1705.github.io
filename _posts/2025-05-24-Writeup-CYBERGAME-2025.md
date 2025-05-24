---
title: Writeup Giải CyberGame 2025
date: 2025-05-24 22:30:00 +0700
categories: [web]
tags: [writeup]     # TAG names should always be lowercase
---
# CYBERGAME 2025
Đây là một giải CTF diễn gia trong thời gian khá là dài nên mình có nhiều thời gian để giải các thử thách được đặt ra và theo cảm nhận của mình thì giải này có nhiều thử thách khá là khó và thú vị. Mục đích mình viết bài này là để tổng hợp những kiến thức mình đã học được thông qua các thử thách của giải trên và cũng như chia sẻ đến với mọi người cùng tham khảo 


## Web Exploitation and Binary Exploitation
> Ở mảng này thì gồm có 3 thử thách lớn và các thử thác nhỏ trong mỗi thử thách lớn. Đến hiện tại mình đã giải gần xong hai thử thách lớn nên sẽ viết trước Writeup về những phần này. Những thử thách còn lại mình sẽ cập nhật sau (nếu minh giải được)


### I. Equestria - Độ khó [★★☆]
---
#### 1. Equestria - Door To The Stable 
Ở thử thách này thì đề bài cho chúng ta một file config của sever: [nginx.conf](/assets/2025-05-24-Writeup%20CYBERGAME%202025/nginx.conf) và thử thách được đặt tại http://exp.cybergame.sk:7000/.
Khi ta truy cập vào trang web trên thì hiển thị trước mắt chúng ta là một trang web liên quan đến chủ để hoạt hình và các bài viết không có gì đặc biệt.
Quay lại với file config thì khi mở file config ta được nội dung như sau:

```plain text
events {
    worker_connections 1024;
}

http {
    include mime.types;

    server {
        listen 80;
        server_name localhost;

        root /app/src/html/;
        index index.html;


        location /images {
            alias /app/src/images/;
            autoindex on;
        }

        location /ponies/ {
            alias /app/src/ponies/;
        }

        location /resources/ {
            alias /app/src/resources/;
        }

        location /secretbackend/ {
            proxy_pass http://secretbackend:3000/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

Để ý ở đoạn code trên có hai phần khá đặc biệt. Thứ nhất là phần `autoindex on;` tại vị trí `/images` và sau vài đường google thì mình biết được tham số trên sẽ liệt kê toàn bộ thư mục và file nằm bên trong nó. Điểm đặc biệt thứ hai đó là vị trí `/secretbackend/` nơi đây được cấu hình một reverse proxy. Thử truy cập vào http://exp.cybergame.sk:7000/secretbackend/ ta nhận được kết quả là một trang Basic Authentication như bên dưới.

``![Form yêu cầu đăng nhập](/assets/2025-05-24-Writeup%20CYBERGAME%202025/Basic%20Auth.png)
_form đăng nhập_

mình thử đăng một số username và passwd như admin:admin, root:root thì đều không nhận được kết quả nên mình đã chuyển sang hướng khai thác khác đó là chú trọng vào thư mục `images`. Thử truy cập vào http://exp.cybergame.sk:7000/images/ thì nhận được kết quả như sau:

![danh sách các tệp hình ảnh hiển thị trong trang web](/assets/2025-05-24-Writeup%20CYBERGAME%202025/list.png)
_các tệp hình ảnh của trang web_

nhận thấy đây rất có thể liên quan đến lỗ hổng bảo mật Path Traversal. Nếu việc cấu hình không an toàn khiến ta có thể truy cập vượt ra khỏi thư mục hiện tại và liệt kê các thư mục khác. Nếu như ý tưởng trên thì khi ta thêm `../` vào phầm `/image/` thì tức là ta có thể đọc được toàn bộ file và thư mục tại thư mục cha của `images`. Thử request http://exp.cybergame.sk:7000/images../ thì đúng như mình đoán là nó đã hiển thị toàn bộ các file và thư mục trong đó có thư mục `secretbackend`

![Thư mục cha](/assets/2025-05-24-Writeup%20CYBERGAME%202025/payload.png)
_thư mục cha_

khi truy cập vào `secretbackend` ta có các file như sau

![source code](/assets/2025-05-24-Writeup%20CYBERGAME%202025/secret.png)
_thư mục secretbackend_

trong đó phần `public` chính là trang hiện lên khi chúng ta truy cập vào http://exp.cybergame.sk:7000/secretbackend/ và `index.js` là file chứa cấu hình chính của sever reverse proxy này. Trong đó có một chuỗi base64encode để xác thực Basic Authentication đó là `cHIxbmNlc3M6U0stQ0VSVHswZmZfYnlfNF9zMW5nbGVfc2w0c2hfZjgzNmE4YjF9`
giải mã ra ta được `pr1ncess:SK-CERT{0ff_by_4_s1ngle_sl4sh_f836a8b1}` và `SK-CERT{0ff_by_4_s1ngle_sl4sh_f836a8b1}` chính là cờ của thử thách con đầu tiên. Tiếp tục đến với thử thác con thứ hai nào!!!
> FLAG: SK-CERT{0ff_by_4_s1ngle_sl4sh_f836a8b1}
-----


#### 2. Equestria - Shadow Realm
> Sắp cập nhật :v