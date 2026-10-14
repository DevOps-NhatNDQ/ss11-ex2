# Bài 2: Biên dịch đóng gói tệp ứng dụng .jar với Gradle và lưu trữ Artifacts

## 1. Tổng quan bài tập
- **Mục tiêu:** Thực thi câu lệnh biên dịch và đóng gói ứng dụng Spring Boot thành tệp thực thi `.jar` bằng Gradle (`./gradlew bootJar`). Lưu trữ thành phẩm biên dịch (`.jar`) từ thư mục `build/libs/` lên GitHub Actions bằng `actions/upload-artifact@v4`.
- **Thư mục bài tập:** `devops/bt/ss11/ex2/`
- **Tệp kịch bản:** `.github/workflows/ci.yml`

---

## 2. Cấu trúc thư mục dự án
```text
ex2/
├── .github/
│   └── workflows/
│       └── ci.yml               # Workflow đã bổ sung bước bootJar & upload-artifact
├── gradle/
│   └── wrapper/
│       └── gradle-wrapper.properties
├── src/
│   ├── main/
│   │   ├── java/com/example/demo/
│   │   │   ├── DemoApplication.java
│   │   │   └── controller/
│   │   │       └── HelloController.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/com/example/demo/
│           ├── DemoApplicationTests.java
│           └── HelloControllerTest.java
├── .gitignore
├── build.gradle
├── settings.gradle
├── gradlew
├── gradlew.bat
├── README.md
└── REPORT.md
```

---

## 3. Nội dung tệp kịch bản `.github/workflows/ci.yml`

```yaml
name: Java Spring Boot CI (Gradle)

on:
  push:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests & Build JAR
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'

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
```

---

## 4. Các điểm cải tiến kỹ thuật so với Bài 1

1. **Nhiệm vụ `bootJar`:**
   - Spring Boot Gradle Plugin cung cấp task `bootJar` nhằm đóng gói toàn bộ mã nguồn, cấu hình và tất cả dependencies (bao gồm Tomcat embedded container) thành một tệp thực thi duy nhất ("fat JAR" hoặc "uber JAR").
   - Lệnh thực thi: `./gradlew bootJar` tạo ra tệp `.jar` bên trong thư mục `build/libs/`.

2. **Lưu trữ Artifact với `actions/upload-artifact@v4`:**
   - **`name: app-jar`**: Tên gói nén artifact được tạo và hiển thị trên giao diện GitHub Actions Summary.
   - **`path: build/libs/*.jar`**: Cấu hình pattern khớp với tất cả tệp đuôi `.jar` bên trong thư mục đích của bản build Gradle.
   - Artifacts cho phép nhóm phát triển tải về kiểm thử thủ công, scan bảo mật hoặc chuyển giao sang các job triển khai (CD/Deployment) tiếp theo.

---

## 5. Hướng dẫn kiểm tra và nộp bài
1. Push commit lên nhánh `main`.
2. Truy cập tab **Actions** trên GitHub Repository và chọn lần chạy mới nhất.
3. Chờ toàn bộ các bước hoàn tất tích xanh (`Success`).
4. Cuộn xuống mục **Artifacts** tại trang Summary, bạn sẽ thấy artifact **app-jar** với dung lượng khoảng 18 - 25 MB sẵn sàng cho phép tải về (`.zip`).
