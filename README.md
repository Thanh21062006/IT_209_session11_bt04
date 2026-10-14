# Báo cáo Thực hành DevOps - Session 11: Bài 4

## Khai báo GitHub Secrets và cấu hình Đăng nhập Container Registry

### 1. Giới thiệu tổng quan
Bài thực hành hướng dẫn thiết lập cơ chế bảo mật cho luồng CI/CD thông qua tính năng GitHub Repository Secrets. Sinh viên khởi tạo Personal Access Token (classic) với quyền `write:packages`, cấu hình biến bí mật `REGISTRY_TOKEN` trong Settings của Repository và tích hợp bước xác thực tự động vào GitHub Container Registry (`ghcr.io`) bằng action `docker/login-action@v3`.

### 2. Cấu trúc thư mục bài nộp
```text
SS11_HW04_DaoTrongTri_PTIT062/
├── .github/
│   └── workflows/
│       └── ci.yml             # Kịch bản CI có bước docker/login-action@v3
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── src/
│   ├── main/
│   │   ├── java/com/ptit/devops_ss11/Application.java
│   │   └── resources/application.properties
│   └── test/
│       └── java/com/ptit/devops_ss11/ApplicationTests.java
├── .gitignore
├── build.gradle
├── gradlew
├── gradlew.bat
├── settings.gradle
├── README.md                  # Hướng dẫn tổng quan
└── REPORT.md                  # Báo cáo kỹ thuật chi tiết về GitHub Secrets và Registry Login
```

### 3. Cấu hình bước Đăng nhập Registry trong Workflow
```yaml
      - name: Đăng nhập vào GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

Chi tiết hướng dẫn sinh token, cấu hình bí mật trên GitHub và phân tích bảo mật xem tại [REPORT.md](REPORT.md).
