# n8n GitHub Weekly Digest

Workflow n8n tìm các repository GitHub mới theo chủ đề, lọc kết quả, dùng AI để tóm tắt và gửi bản tin HTML qua Gmail.

## Các bước chính

1. Kích hoạt thủ công hoặc theo lịch hằng tuần.
2. Tạo danh sách truy vấn GitHub theo chủ đề.
3. Gọi GitHub Search API.
4. Loại bản fork, repository thiếu mô tả và mục đã gửi trước đó.
5. Dùng mô hình AI để phân nhóm và tóm tắt.
6. Tạo email HTML và gửi qua Gmail.

## Cách import

1. Tải file [`n8n-github-weekly-digest-workflow.json`](./n8n-github-weekly-digest-workflow.json).
2. Mở n8n và chọn **Import from File**.
3. Chọn file JSON vừa tải.
4. Cấu hình credential cho OpenAI và Gmail.
5. Thay `your_email@gmail.com` bằng địa chỉ nhận bản tin.
6. Chạy thử bằng node Manual Trigger trước khi bật lịch tự động.

## Lưu ý

- File không chứa API key, access token hoặc credential n8n.
- GitHub API có giới hạn số request. Có thể thêm token GitHub vào HTTP Request node nếu cần hạn mức cao hơn.
- Kiểm tra múi giờ của n8n trước khi bật Schedule Trigger.

## Tệp workflow

- [Tải workflow JSON](./n8n-github-weekly-digest-workflow.json)
