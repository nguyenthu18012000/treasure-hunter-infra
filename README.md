# treasure-hunter-infra

Hạ tầng chạy local cho Treasure Hunter bằng Docker. Hiện có:

| Service  | Địa chỉ                 | Dùng để                                 |
|----------|-------------------------|-----------------------------------------|
| Kafka    | `localhost:9092`        | Các service Spring gửi/nhận event       |
| Kafka UI | http://localhost:8090   | Xem topic, message, consumer group      |

## Yêu cầu

Chỉ cần **Docker Desktop**, mở lên và đợi báo "Engine running".
Kiểm tra (phải thấy cả Client và Server):

```bash
docker version
```

## Chạy

Mở PowerShell trong thư mục này rồi chạy:

```bash
docker compose up -d
```

Lần đầu Docker sẽ tự tải image về, mất vài phút. Kiểm tra:

```bash
docker compose ps
```

Kafka sẵn sàng khi cột STATUS ghi `healthy`. Lúc đó mở http://localhost:8090 sẽ thấy cluster `local`.

## Các lệnh hay dùng

| Việc                                   | Lệnh                              |
|----------------------------------------|-----------------------------------|
| Bật tất cả                             | `docker compose up -d`            |
| Xem trạng thái                         | `docker compose ps`               |
| Xem log Kafka                          | `docker compose logs -f kafka`    |
| Tắt (giữ dữ liệu)                      | `docker compose down`             |
| Tắt và **xoá hết dữ liệu**             | `docker compose down -v`          |
| Khởi động lại Kafka                    | `docker compose restart kafka`    |

Các container có `restart: unless-stopped` nên sẽ tự bật lại khi Docker khởi động.
Muốn Docker tự chạy khi mở máy: Docker Desktop → Settings → General → bật
"Start Docker Desktop when you sign in".

## Thử Kafka bằng dòng lệnh

Tạo topic và xem danh sách:

```bash
docker exec -it kafka /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test --partitions 3
```

```bash
docker exec -it kafka /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

Mở 2 cửa sổ PowerShell. Cửa sổ 1 là producer (gõ chữ rồi Enter):

```bash
docker exec -it kafka /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test
```

Cửa sổ 2 là consumer:

```bash
docker exec -it kafka /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```

## Kết nối từ service Spring

`pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

`application.yml`:

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
```

## Lỗi hay gặp

| Lỗi                                              | Cách xử lý                                                        |
|--------------------------------------------------|-------------------------------------------------------------------|
| `cannot find ... dockerDesktopLinuxEngine`       | Docker Desktop chưa bật hoặc chưa khởi động xong                   |
| `port is already allocated`                      | Cổng 9092/8090 đang bị dùng, ví dụ còn container `kafka` cũ: `docker rm -f kafka` |
| `the input device is not a TTY`                  | Đang chạy trong Git Bash, chuyển sang PowerShell                   |
| Spring báo `Connection to node -1 could not be established` | Kafka chưa chạy hoặc `bootstrap-servers` sai              |
