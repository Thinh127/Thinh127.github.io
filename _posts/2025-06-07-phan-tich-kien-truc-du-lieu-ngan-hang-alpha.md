---
layout: post

title: Phân tích chi tiết kiến trúc dữ liệu và data pipeline tại ngân hàng – Trường hợp Ngân hàng Alpha

date: 2025-06-07

categories: [Data Architecture, Banking]

tags: [case study, databricks, lakehouse]

---


# Phân tích chi tiết kiến trúc dữ liệu và data pipeline tại ngân hàng – Trường hợp Ngân hàng Alpha

![Sơ đồ tổng quan hệ thống dữ liệu ngân hàng](https://files.chatgpt.com/file_00000000688861f9a39caef4cf71af19)

---

## 1. Kiến trúc tổng quan ngành ngân hàng tại Việt Nam

Các ngân hàng hiện đại tại Việt Nam đang áp dụng kiến trúc hybrid – kết hợp hệ thống giao dịch truyền thống (core banking) với hạ tầng cloud-native, gồm:

- **OLTP Systems**: Oracle Database, SQL Server dùng cho xử lý giao dịch realtime.
- **Integration Layer**: Dùng các middleware như Apache Kafka, MuleSoft hoặc Azure Event Hub để đẩy dữ liệu giao dịch theo thời gian thực về hệ thống trung tâm.
- **Data Lake**: Sử dụng Amazon S3 hoặc Azure Data Lake Gen2 để lưu trữ dữ liệu thô (raw zone).
- **Data Warehouse**: Redshift, Snowflake, hoặc Databricks Lakehouse (Delta Lake) cho xử lý phân tích.
- **BI Layer**: Power BI, Tableau phục vụ báo cáo và dashboard cho từng phòng ban.

---

## 2. Kiến trúc Lakehouse tại Ngân hàng Alpha

Ngân hàng Alpha triển khai kiến trúc **Lakehouse trên Databricks**, cho phép:

- **Ingest dữ liệu từ >50 hệ thống** (tài chính, tín dụng, CRM, giao dịch).
- Dùng **Apache Spark Structured Streaming** để ingest gần realtime dữ liệu từ Kafka vào Bronze Layer.
- Áp dụng chuẩn **multi-hop architecture**:
  - **Bronze Layer**: lưu dữ liệu thô gốc từ các hệ thống source.
  - **Silver Layer**: xử lý, làm sạch, chuẩn hóa, enrich thêm metadata.
  - **Gold Layer**: tổng hợp theo logic nghiệp vụ (ví dụ: daily transaction summary per branch).

---

## 3. Data Pipeline phục vụ phòng Tài chính

### Mục tiêu:
- Tính toán P&L theo ngày, chi nhánh, sản phẩm.
- Đối chiếu giữa hệ thống kế toán và hệ thống giao dịch.

### Kỹ thuật:
1. **Source**: Truy vấn giao dịch từ core banking (Oracle OLTP), accounting system (SAP hoặc tương đương).
2. **Ingestion**:
   - CDC bằng Oracle GoldenGate hoặc Debezium → Kafka.
   - Push vào Bronze table dạng Delta bằng Databricks Autoloader.
3. **Processing**:
   - Silver: chuẩn hóa schema, join với bảng master data như customer, branch, GL code.
   - Gold: tổng hợp theo chiều time, region, product.
4. **Serving**:
   - Publish bảng Gold sang Power BI hoặc expose qua Databricks SQL Endpoint cho phòng tài chính.

---

## 4. Kiến trúc xử lý giao dịch và đồng bộ

### OLTP Layer:
- Xử lý trực tiếp trên Oracle RAC Cluster.
- Transaction được ghi log (redo log, archive log) → phục vụ CDC.

### Ingestion Layer:
- CDC log được đọc bởi GoldenGate → Kafka Topic.
- Mỗi Kafka topic tương ứng một entity (transaction, account, customer).
- Dữ liệu được phân phối đến Databricks hoặc Snowflake.

---

## 5. Quản lý đồng bộ dữ liệu và chống inconsistency

### Các vấn đề thường gặp:
- Giao dịch rollback nhưng đã ingest.
- Dữ liệu phân tán → late arriving data.
- ETL job chạy trước khi upstream job hoàn thành.

### Cách khắc phục:
- Áp dụng **idempotent writes** và **merge (upsert)** trên Delta Lake.
- Dùng **watermark + window** trong Spark Structured Streaming.
- Cơ chế **exactly-once delivery** với checkpointing và schema enforcement.

---

## 6. POS, ATM và ứng dụng ngân hàng số

### POS / ATM:
- Hệ thống ATM/POS chạy phần mềm C++ hoặc Java trên hệ điều hành nhúng.
- Giao dịch gửi về core banking qua **ISO 8583 protocol** (thường dùng TCP socket hoặc web service).
- Nếu offline: POS ghi giao dịch local, sync lại khi online.

### Ngân hàng số:
- Ứng dụng Ngân hàng Alpha dùng backend dựa trên microservice, triển khai trên Kubernetes.
- Các request được quản lý qua API Gateway (AWS API Gateway hoặc Kong).
- Toàn bộ event lưu lại bằng Kafka hoặc EventHub để phục vụ audit và phân tích hành vi người dùng.

---

## 7. Giao dịch online và offline – xử lý kỹ thuật

| Tình huống         | Cách xử lý kỹ thuật                       |
|--------------------|-------------------------------------------|
| POS mất mạng       | Local cache + checksum, sync khi kết nối lại |
| Giao dịch bị treo  | Retry + Dead-letter queue                 |
| Trùng lặp giao dịch| Transaction ID + Idempotent key           |
| ATM ngắt điện      | Ghi vào flash + batch recover khi khởi động lại |

---

## 8. Gắn dữ liệu với nghiệp vụ và báo cáo

Ví dụ: báo cáo doanh thu tín dụng theo vùng miền
- Gold Layer chứa bảng: `loan_summary_gold`
- Partition theo `region` và `transaction_date`
- Power BI truy vấn theo time-series: rolling 30-day avg, YoY growth
- Các chỉ số KPI tính trực tiếp trên Databricks SQL: DPD, NPL ratio

---

## Kết luận

Ngân hàng Alpha đang áp dụng một mô hình dữ liệu tiên tiến, kết hợp kiến trúc Lakehouse, streaming data ingestion và phân tích theo chiều nghiệp vụ. Điều này giúp họ vừa đáp ứng giao dịch realtime, vừa phục vụ phân tích chiến lược, kiểm soát rủi ro, và trải nghiệm khách hàng tốt hơn.

