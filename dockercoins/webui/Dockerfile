# FROM node:4-slim
# RUN npm install express
# RUN npm install redis@3
# COPY files/ /files/
# COPY webui.js /
# CMD ["node", "webui.js"]
# EXPOSE 80

FROM node:22-slim

# Tạo thư mục làm việc trong container
WORKDIR /app

# Cài đặt các thư viện cần thiết
RUN npm install express redis@3

# Sao chép các tệp cần thiết vào container
COPY files/ /app/files/
COPY webui.js /app/

# Mở cổng 80
EXPOSE 80

# Khởi chạy ứng dụng
CMD ["node", "webui.js"]