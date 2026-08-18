# Phản Ánh Lab NB1–NB8: Top 5 Anti-Pattern Lakehouse

## Cái Bẫy Dễ Vướng Nhất Cho Team

**Vấn Đề Small-File (NB2, NB6)** — nếu quên compaction, nó sẽ giết ta.

### Tại Sao?

Streaming ingestion (Kafka→Lakehouse, trigger mỗi 5 giây) tự nhiên tạo micro-batch. Mỗi commit là ACID hoàn hảo. Sau 24 giờ, 17.000 commit nhỏ → 17.000 file. Queries bỗng chốc chậm 60×, hóa đơn S3 nổ tung — **không phải vì dữ liệu tăng, mà vì số request tăng**.

Lab đo được rõ: 200 file → 40 phút downtime không lên kế hoạch/tháng/TB, ở $0.0004/request GET. Một job cron quên mất = $100/ngày chỉ tính phí request.

### Top 5 Anti-Pattern Hay Gặp

1. **File nhỏ** (NB2, NB6) — quên `OPTIMIZE` + `Z-ORDER`
2. **Không enforce schema** (NB1) — dữ liệu xấu im lặng hỏng bảng
3. **Orphan từ job crash** (NB6) — vô hình, tốn tiền, không ai lấy
4. **Metadata phình** (NB6) — chạy `expire_snapshots` mà không `remove_orphan_files`
5. **Không có medallion tier** (NB4) — trộn raw + processed = mất audit trail

### Cam Kết 

**Tự động hóa maintenance** (4 job chạy theo lịch, checksum hàng giờ) và **mặc định deny schema changes** (không wildcard column, migration job explicit). Thuật toán tìm orphan của NB6 sẽ sống trong cron, không phải notebook.
