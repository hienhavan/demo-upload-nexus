# Hướng Dẫn Publish Maven & Gradle Lên Nexus  

## Cài Đặt & Chạy Nexus  

### Chạy Nexus bằng Docker  
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
```
### Chạy Nexus bằng lệnh:
```sh
docker-compose up -d
```

### Truy cập http://localhost:8081 để vào giao diện Nexus.
## Lấy mật khẩu Admin mặc định
```sh
docker exec -it nexus cat /nexus-data/admin.password
```
## Cấu Hình Maven
### Cấu hình settings.xml
Mở file /settings.xml và thêm thông tin:
```xml
<servers>
    <server>
        <id>nexus</id>
        <username>admin</username>
        <password>your-password</password>
    </server>
</servers>
```
## Cấu hình thêm đoạn mã vào file pom.xml
```xml
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
```
## Build & Upload
```sh
mvn deploy
```

## Cấu hình build.gradle
Thêm nội dung sau:
```
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
```
## Upload Lên Nexus
Chạy lệnh sau:

```sh
gradle publish
```
## Lưu Ý Quan Trọng
Dùng đúng loại repository:
maven-snapshots chỉ dành cho SNAPSHOT (1.0.0-SNAPSHOT).
gradle-snapshots cũng chỉ dành cho SNAPSHOT.
