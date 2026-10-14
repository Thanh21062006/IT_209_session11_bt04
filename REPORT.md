# Báo cáo Kỹ thuật: Khai báo GitHub Secrets và cấu hình Đăng nhập Container Registry

## 1. Mục tiêu bài thực hành
- Thiết lập quy trình quản lý thông tin định danh nhạy cảm (Credentials / Tokens) an toàn theo tiêu chuẩn DevSecOps trên GitHub Actions bằng GitHub Secrets.
- Khởi tạo Personal Access Token (PAT classic) được phân quyền tối thiểu với scope `write:packages` và `read:packages`.
- Cấu hình biến môi trường bảo mật `REGISTRY_TOKEN` trong phạm vi Repository.
- Tích hợp và kiểm chứng bước đăng nhập tự động vào GitHub Container Registry (`ghcr.io`) thông qua action chuẩn `docker/login-action@v3`.

---

## 2. Kịch bản GitHub Actions Workflow (`.github/workflows/ci.yml`)

```yaml
name: Java Spring Boot CI (Gradle)

on:
  push:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar

      - name: Lưu trữ thành phẩm JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: build/libs/*.jar

      - name: Đăng nhập vào GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

---

## 3. Quy trình Thiết lập GitHub Personal Access Token và Secrets

### 3.1. Khởi tạo Personal Access Token (classic)
1. Trên giao diện GitHub cá nhân, bấm vào ảnh đại diện góc phải trên cùng -> Chọn **Settings**.
2. Cuộn xuống thanh điều hướng bên trái -> Chọn **Developer settings** -> **Personal access tokens** -> **Tokens (classic)**.
3. Bấm nút **Generate new token** -> **Generate new token (classic)**.
4. Đặt tên gợi nhớ tại trường **Note**: `GHCR_PUSH_TOKEN`.
5. Đặt thời gian hết hạn (**Expiration**): Ví dụ 30 ngày hoặc 90 ngày.
6. Tích chọn các quyền (Scopes) bắt buộc:
   - `write:packages`: Cho phép tải lên (upload / push) container images lên GitHub Packages / GHCR.
   - `read:packages`: Cho phép tải về (pull / download) container images.
7. Bấm **Generate token** và sao chép chuỗi token (dạng `ghp_...`).

### 3.2. Khai báo Repository Secret `REGISTRY_TOKEN`
#### Cách 1: Thiết lập qua Giao diện Web GitHub
1. Mở Repository `https://github.com/ChongChi-hub/IT209_Session_11_Bai_04`.
2. Truy cập tab **Settings** của repository -> Chọn **Secrets and variables** -> **Actions**.
3. Bấm nút **New repository secret**.
4. Nhập:
   - **Name:** `REGISTRY_TOKEN`
   - **Secret:** Dán chuỗi token vừa tạo ở bước trên.
5. Bấm **Add secret** để lưu.

#### Cách 2: Thiết lập nhanh bằng GitHub CLI (`gh`)
```bash
gh secret set REGISTRY_TOKEN --body "<YOUR_TOKEN>" -R ChongChi-hub/IT209_Session_11_Bai_04
```

---

## 4. Phân tích Bảo mật DevSecOps

```mermaid
graph TD
    A["GitHub Repository Secret: REGISTRY_TOKEN"] -->|Mã hóa bất đối xứng NaCL Box| B["Lưu trữ an toàn trên GitHub KMS"]
    B -->|Tiêm bí mật tại runtime| C["GitHub Runner: ubuntu-latest"]
    C -->|Biến ${{ secrets.REGISTRY_TOKEN }}| D["docker/login-action@v3"]
    D -->|Xác thực HTTP Basic Auth| E["ghcr.io (GitHub Container Registry)"]
    E -->|Xác thực hợp lệ| F["Login Succeeded"]
    C -.->|Tự động lọc mặt nạ ***| G["Runner Log Output (Masked)"]
```

### Các nguyên tắc an toàn được áp dụng:
1. **Secret Masking:** GitHub Actions tự động quét toàn bộ chuỗi ký tự trùng với giá trị bí mật và thay thế bằng `***` trên giao diện console log, triệt tiêu nguy cơ lộ lọt qua log công khai.
2. **Không lưu mã cứng (Hardcoded Credentials):** Không bao giờ ghi trực tiếp mật khẩu hay token vào tệp tin YAML trong git.
3. **Phân quyền tối thiểu (Least Privilege):** Token chỉ cấp quyền thao tác package (`write:packages`), không cấp quyền admin hay can thiệp vào mã nguồn ngoài ý muốn.
4. **Bảo vệ nhánh (Branch Protection):** Chỉ các workflow chạy trên nhánh được bảo vệ hoặc được ủy quyền mới có thể truy xuất Secrets nhạy cảm.

---

## 5. Nhật ký thực thi trên GitHub Actions Runner

Trích xuất nhật ký thực tế tại bước `Đăng nhập vào GitHub Container Registry`:

```text
Run docker/login-action@v3
  with:
    registry: ghcr.io
    username: ChongChi-hub
    password: ***
Logging into ghcr.io...
Login Succeeded
```

Dòng thông báo `Login Succeeded` khẳng định runner đã kết nối và xác thực thành công vào hạ tầng GitHub Container Registry với tư cách tài khoản `ChongChi-hub`.

---

## 6. Hướng dẫn kiểm tra và minh chứng trên GitHub Actions

### Các bước kiểm tra:
1. Truy cập vào kho lưu trữ: `https://github.com/ChongChi-hub/IT209_Session_11_Bai_04`.
2. Vào **Settings** -> **Secrets and variables** -> **Actions**, xác nhận mục **Repository secrets** hiển thị biến `REGISTRY_TOKEN`.
3. Vào tab **Actions** -> Chọn workflow run mới nhất -> Mở job **Run Unit Tests**.
4. Bấm mở rộng bước **Đăng nhập vào GitHub Container Registry**, xác nhận dòng thông báo **Login Succeeded**.

### Ảnh chụp màn hình minh chứng:
> *(Dán ảnh chụp màn hình trang Repository Secrets hiển thị biến REGISTRY_TOKEN và ảnh chụp log Login Succeeded vào vị trí dưới đây)*

`ImgEx`
