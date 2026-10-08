# n8n GitHub Weekly Digest

Workflow tự động tìm kiếm repository GitHub mới theo từng chủ đề công nghệ, loại trùng hai lớp, sử dụng OpenAI (`gpt-4o-mini`) biên tập bản tin và gửi email HTML qua Gmail.

Repository chính thức: [github.com/ananh31300/n8n-github-weekly-digest](https://github.com/ananh31300/n8n-github-weekly-digest)

## Yêu cầu

- Một workspace n8n Cloud đang hoạt động (hoặc n8n self-hosted).
- GitHub Personal Access Token (classic hoặc fine-grained).
- OpenAI API Key (model `gpt-4o-mini`).
- Google OAuth2 credential có quyền gửi Gmail (`https://mail.google.com/`).
- Workflow timezone: `Asia/Ho_Chi_Minh`.

Tài liệu này chỉ hướng dẫn cấu hình và vận hành workflow trên n8n Cloud. Việc đăng ký tài khoản, cài đặt n8n hoặc quản trị hạ tầng máy chủ không nằm trong phạm vi tài liệu.

## Các chủ đề theo dõi (6 truy vấn)

Workflow phân tách thành 6 truy vấn độc lập để tránh lỗi cú pháp GitHub HTTP 422:

1. **AI Agents**: `topic:ai-agent` (stars > 10)
2. **MCP**: `topic:mcp` (stars > 5)
3. **MCP Protocol**: `topic:model-context-protocol` (stars > 5)
4. **RAG & Vector Search**: `topic:rag language:python` (stars > 10)
5. **Modern Data Stack (DuckDB)**: `topic:duckdb` (stars > 10)
6. **Modern Data Stack (Iceberg)**: `topic:apache-iceberg` (stars > 10)

Tất cả các truy vấn đều tự động lọc repository tạo trong 7 ngày gần nhất, loại bỏ fork (`fork:false`) và repository lưu trữ (`archived:false`).

## Cấu hình workflow trên n8n Cloud

1. Đăng nhập n8n Cloud (hoặc self-hosted phiên bản `n8n >= 1.117.0`), tạo workflow mới và import [`n8n-github-weekly-digest-workflow.json`](./n8n-github-weekly-digest-workflow.json).
2. Lấy **GitHub Token**: Vào GitHub profile $\rightarrow$ **Settings** $\rightarrow$ **Developer settings** $\rightarrow$ **Personal access tokens** $\rightarrow$ Tạo **Fine-grained token** cấp quyền `Public Repositories (read-only)`.
3. Tạo **Header Auth** credential trong n8n:
   - Vào **Credentials** $\rightarrow$ **Add Credential** $\rightarrow$ chọn **Header Auth**.
   - **Name:** `Authorization`
   - **Value:** `Bearer <GITHUB_TOKEN>` (Gõ từ `Bearer`, một dấu cách trắng, rồi dán token).
   - Bấm **Save**.
4. Gắn credential vào node HTTP Request:
   - Mở node `Tìm kiếm trên GitHub API`.
   - Đổi **Authentication** từ `None` sang **Generic Credential Type**.
   - Chọn **Generic Auth Type:** `Header Auth`.
   - Chọn credential `Authorization` vừa tạo.
5. Chọn OpenAI credential trong node `AI Phân tích & Biên tập` (chọn model `gpt-4o-mini`).
6. Chọn Gmail credential trong node `Gửi Email qua Gmail`.
7. Thay `your_email@gmail.com` bằng địa chỉ email người nhận trong node `Tạo giao diện Email HTML`.
8. Kiểm tra timezone `Asia/Ho_Chi_Minh` trong Workflow Settings.
9. Chạy Manual Trigger để kiểm tra từng node.
10. Lưu và **Publish** workflow để Schedule Trigger và static data hoạt động tự động.

## Cơ chế chống gửi trùng hai lớp & Transactional Commit

Workflow áp dụng cơ chế lọc và ghi nhận an toàn:

1. **Lớp 1 (Nội bộ execution)**: Dùng `Set` loại trùng các repository xuất hiện ở nhiều chủ đề khác nhau trong cùng một lần chạy.
2. **Lớp 2 (Xuyên suốt các tuần)**: Đọc `$getWorkflowStaticData('global').daGuiRepoIds` trong node lọc để loại bỏ các repository đã gửi ở các lần chạy trước.
3. **Commit sau khi gửi thành công (Transactional Safety)**: Node lọc **chỉ đọc** và truyền danh sách `topRepoIds`. Việc ghi đè vào `staticData.daGuiRepoIds` chỉ diễn ra tại node `Ghi nhận repo đã gửi thành công` **sau khi** node Gmail đã gửi email thành công. Nếu AI hoặc Gmail gặp sự cố, workflow sẽ dừng và ID repo không bị đánh dấu nhầm, đảm bảo không bỏ sót repo ở các lần chạy sau.

*Lưu ý:* `staticData` chỉ được n8n lưu lại khi chạy Production (thông qua Schedule Trigger hoặc Webhook). Manual Execution trong trình soạn thảo sẽ không lưu trạng thái này.

## Quy trình kiểm thử

- **GitHub API**: Xác nhận tất cả 6 request trả mã HTTP 200.
- **Lọc & Xếp hạng**: Output trả về tối đa 15 repository có stars cao nhất, không trùng lặp ID.
- **OpenAI Node**: Sử dụng model `gpt-4o-mini`, temperature `0.3`, trả về HTML fragment chuẩn không kèm markdown block.
- **Gmail Node**: Gửi email HTML thành công đến hòm thư nhận.
- **Static Data**: Node cuối ghi nhận danh sách ID mới vào `daGuiRepoIds`.
