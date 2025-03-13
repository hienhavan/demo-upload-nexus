# 🚀 Hướng Dẫn Publish Maven & Gradle Lên Nexus  

## 📌 1. Cài Đặt & Chạy Nexus  

### 1️⃣ Chạy Nexus bằng Docker  
Tạo file `docker-compose.yml` với nội dung sau:  

```yaml
version: '3.8'

services:
  nexus:
    image: sonatype/nexus3:latest
    container_name: nexus
    restart: always
    ports:
      - "8081:8081"  # Cổng truy cập UI Nexus
    volumes:
      - nexus-data:/nexus-data  # Lưu trữ dữ liệu Nexus
    environment:
      - INSTALL4J_ADD_VM_PARAMS=-Xms512m -Xmx2g -XX:MaxDirectMemorySize=2g -Djava.util.prefs.userRoot=/nexus-data/javaprefs

volumes:
  nexus-data:
    driver: local
Chạy Nexus bằng lệnh:

sh
Sao chép
Chỉnh sửa
docker-compose up -d
Truy cập http://localhost:8081 để vào giao diện Nexus.

2️⃣ Lấy mật khẩu Admin mặc định
Chạy lệnh sau để lấy mật khẩu:

sh
Sao chép
Chỉnh sửa
docker exec -it nexus cat /nexus-data/admin.password
Đăng nhập với tài khoản:

Username: admin
Password: (lấy từ lệnh trên)
📌 2. Cấu Hình Nexus Repository
🔹 Tạo repository cho Maven
Vào "Repositories" > "Create repository"
Chọn "maven2 (hosted)"
Nhập thông tin:
Name: maven-snapshots
Version policy: Snapshot
Deployment policy: Allow redeploy
Nhấn "Create repository"
🔹 Tạo repository cho Gradle
Vào "Repositories" > "Create repository"
Chọn "maven2 (hosted)"
Nhập thông tin:
Name: gradle-snapshots
Version policy: Snapshot
Deployment policy: Allow redeploy
Nhấn "Create repository"
📌 3. Cấu Hình Maven
1️⃣ Cấu hình settings.xml
Mở file ~/.m2/settings.xml (hoặc C:\Users\<username>\.m2\settings.xml trên Windows) và thêm thông tin:

xml
Sao chép
Chỉnh sửa
<servers>
    <server>
        <id>nexus</id>
        <username>admin</username>
        <password>your-password</password>
    </server>
</servers>
2️⃣ Cấu hình pom.xml
Thêm đoạn sau vào pom.xml:

xml
Sao chép
Chỉnh sửa
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-library</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <distributionManagement>
        <snapshotRepository>
            <id>nexus</id>
            <url>http://localhost:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
</project>
3️⃣ Build & Upload
Chạy lệnh sau để đẩy lên Nexus:

sh
Sao chép
Chỉnh sửa
mvn deploy
📌 4. Cấu Hình Gradle
1️⃣ Cấu hình build.gradle
Thêm nội dung sau:

gradle
Sao chép
Chỉnh sửa
plugins {
    id 'java-library'
    id 'maven-publish'
}

group = 'com.example'
version = '1.0.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'com.google.guava:guava:32.0.1-jre'
}

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
    repositories {
        maven {
            name = "nexus"
            url = uri("http://localhost:8081/repository/gradle-snapshots/")
            allowInsecureProtocol = true
            credentials {
                username = "admin"
                password = "your-password"
            }
        }
    }
}
2️⃣ Upload Lên Nexus
Chạy lệnh sau:

sh
Sao chép
Chỉnh sửa
gradle publish
🎯 Lưu Ý Quan Trọng
Dùng đúng loại repository:
maven-snapshots chỉ dành cho SNAPSHOT (1.0.0-SNAPSHOT).
gradle-snapshots cũng chỉ dành cho SNAPSHOT.
