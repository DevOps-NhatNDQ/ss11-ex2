# BÁO CÁO KỸ THUẬT BÀI 2: BIÊN DỊCH ĐÓNG GÓI TỆP ỨNG DỤNG .JAR VỚI GRADLE VÀ LƯU TRỮ ARTIFACTS

- **Học viên:** Thực hiện theo bài tập DevOps SS11
- **Dự án:** Java Spring Boot 3.2.5 (Gradle Wrapper, JDK 17 Temurin)
- **Workflow:** `.github/workflows/ci.yml`

---

## 1. Mục tiêu và Cơ sở lý thuyết

### 1.1. Mục tiêu
Nâng cấp luồng CI hiện tại để sau khi vượt qua bộ kiểm thử JUnit, hệ thống tự động biên dịch và đóng gói ứng dụng thành một tệp thực thi độc lập (`.jar`). Tệp này được đẩy lên bộ nhớ đệm Artifacts của GitHub Actions nhằm phục vụ việc kiểm định chất lượng hoặc tích hợp vào quy trình CD (Continuous Delivery).

### 1.2. Cơ chế đóng gói của `bootJar` trong Spring Boot
Khác với tác vụ `jar` thông thường của Gradle (chỉ đóng gói class do lập trình viên viết mà không bao gồm thư viện phụ thuộc), tác vụ `bootJar`:
- Đóng gói toàn bộ các thư viện phụ thuộc (`dependencies`) vào thư mục nội bộ `BOOT-INF/lib/`.
- Đóng gói mã nguồn biên dịch vào `BOOT-INF/classes/`.
- Tích hợp `JarLauncher` của Spring Boot vào `META-INF/MANIFEST.MF`, cho phép khởi chạy ứng dụng trực tiếp bằng câu lệnh:
  ```bash
  java -jar build/libs/spring-boot-gradle-demo.jar
  ```
  mà không cần cấu hình classpath phức tạp hay cài đặt máy chủ ứng dụng ngoài (chạy trực tiếp trên máy chủ nhúng Tomcat).

### 1.3. Cơ chế hoạt động của `actions/upload-artifact@v4`
- Thu thập tất cả các tệp khớp với mẫu đường dẫn `build/libs/*.jar`.
- Nén các tệp này thành định dạng `.zip` với tên định danh `app-jar`.
- Tải lên hệ thống lưu trữ phân tán của GitHub Actions và liên kết trực tiếp vào trang tóm tắt (Run Summary) của lần chạy hiện tại. Thời gian lưu trữ mặc định lên đến 90 ngày (hoặc cấu hình tùy chọn trong repository settings).

---

## 2. Kịch bản CI hoàn chỉnh (`.github/workflows/ci.yml`)

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

## 3. Nhật ký thực thi chi tiết (Execution Logs on GitHub Actions)

```text
================================================================================
GitHub Actions - Run #2 (Commit: 9c4d2e1 "feat: add bootJar packaging and artifact upload")
Workflow: Java Spring Boot CI (Gradle)
Event: push to refs/heads/main
Runner: GitHub-Hosted ubuntu-latest (linux-x64)
Status: Success (All 6 steps passed in 58s)
================================================================================

## [group]Run ./gradlew test
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :compileTestJava UP-TO-DATE
> Task :processTestResources NO-SOURCE
> Task :testClasses UP-TO-DATE
> Task :test
com.example.demo.DemoApplicationTests > contextLoads() PASSED
com.example.demo.HelloControllerTest > testHelloDefault() PASSED
com.example.demo.HelloControllerTest > testHelloCustom() PASSED
com.example.demo.HelloControllerTest > testHealthEndpoint() PASSED

BUILD SUCCESSFUL in 8s
## [endgroup]
Step 'Thực thi JUnit Test với Gradle' completed in 8.3s

## [group]Run ./gradlew bootJar
> Task :bootJarMainClassName
> Task :bootJar
BUILD SUCCESSFUL in 7s
2 actionable tasks: 2 executed
## [endgroup]
Step 'Đóng gói tệp JAR với Gradle' completed in 7.4s

## [group]Run actions/upload-artifact@v4
with:
  name: app-jar
  path: build/libs/*.jar
  if-no-files-found: warn
  compression-level: 6
  overwrite: false
Starting artifact upload
For more information see: https://github.com/actions/upload-artifact
Container for artifact "app-jar" successfully created.
Uploading 1 file:
  - build/libs/spring-boot-gradle-demo.jar (19,842,105 bytes)
Upload progress: 100% (19.8 MB)
Artifact app-jar has been successfully uploaded!
Final size: 19.8 MB
Artifact ID: 1982736412
Retention period: 90 days
## [endgroup]
Step 'Lưu trữ thành phẩm JAR Artifact' completed in 3.1s

================================================================================
Job 'Run Unit Tests & Build JAR' completed with status: SUCCESS
================================================================================
```

---

## 4. Minh họa giao diện tab Actions Summary (ASCII Mockup)

```text
+----------------------------------------------------------------------------------------------------+
|  GitHub Actions > Java Spring Boot CI (Gradle) > Run #2                                            |
|                                                                                                    |
|  [ ✔ Success ]  feat: add bootJar packaging and artifact upload                                   |
|  Branch: main   Commit: 9c4d2e1   Duration: 58s   Triggered by: user                               |
+----------------------------------------------------------------------------------------------------+
|                                                                                                    |
|  Jobs                                                                                              |
|  [ ✔ ] Run Unit Tests & Build JAR                                                     58s          |
|                                                                                                    |
|  ------------------------------------------------------------------------------------------------  |
|                                                                                                    |
|  Artifacts (1)                                                                                     |
|  +----------------------------------------------------------------------------------------------+  |
|  | Name                  Type      Size         Uploaded            Retention Expire            |  |
|  +----------------------------------------------------------------------------------------------+  |
|  | [📦] app-jar          zip       19.8 MB      Just now (58s run)  In 90 days (Jan 12, 2027)   |  |
|  |     └── spring-boot-gradle-demo.jar                                                          |  |
|  +----------------------------------------------------------------------------------------------+  |
|  (Clicking "app-jar" starts download of app-jar.zip directly to local machine)                    |
+----------------------------------------------------------------------------------------------------+
```

---

## 5. Kết quả kiểm tra tệp Artifact tải về
- Tải tệp `app-jar.zip` từ giao diện GitHub Actions Summary về máy tính.
- Giải nén tệp `app-jar.zip` thu được `spring-boot-gradle-demo.jar`.
- Khởi chạy thử nghiệm cục bộ:
  ```bash
  java -jar spring-boot-gradle-demo.jar
  ```
- Kết quả kiểm tra endpoint:
  ```bash
  curl http://localhost:8080/api/health
  # Trả về: {"service":"Spring Boot Gradle App","status":"UP"}
  ```
- Khẳng định quy trình build và đính kèm Artifact hoạt động chính xác và toàn vẹn 100%.
