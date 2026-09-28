# Thông Tin Deploy — Checkpoint 5 (Railway)

> Service đã được triển khai trên Railway. `pytest tests/test_cp5.py` đọc file
> này để gọi trực tiếp các endpoint công khai.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo công khai không được chứa khóa bí mật.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Phạm Đức Anh |
| Mã học viên | 2A202602994 |
| Repo | https://github.com/phamducanh552004/K4-L3A-DAY12-PhamDucAnh-2A202602994-https-github.com-VinUni-AI20k-K4-L3A-CloudServicesAndDeploy |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-l3a-day12-phamducanh-2a202602994-https-github-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 28/09/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Tham chiếu đến Redis được provision trong cùng project Railway |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Tái Kiểm Tra

```bash
curl -i https://k4-l3a-day12-phamducanh-2a202602994-https-github-production.up.railway.app/health
curl -i https://k4-l3a-day12-phamducanh-2a202602994-https-github-production.up.railway.app/ready
curl -i -X POST https://k4-l3a-day12-phamducanh-2a202602994-https-github-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
```

## Kết Quả Chạy Thật

```text
GET /health  -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready   -> 200 {"status":"ready","redis":true}
POST /ask không gửi API key -> 401
```

Kết quả xác nhận service sống, đã kết nối Redis, và bảo vệ endpoint `/ask`
khi request không có API key.

## Ảnh Chụp Màn Hình

Railway dashboard hiển thị deployment **Active**. Public URL ở đầu tài liệu là
domain Railway đã tạo cho service này.
