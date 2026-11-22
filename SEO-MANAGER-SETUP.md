# SEO Manager AI Pro – Anh Văn Giao Tiếp Biên Hòa

Bộ bổ sung SEO dành cho website HTML tĩnh hiện tại. Không thay framework và không sửa nội dung học liệu trong `index.html`.

## Thành phần đã thêm

- `seo-admin/`: Dashboard quản trị, noindex.
- `seo/seo-config.json`: cấu hình SEO khởi tạo.
- `functions/api/seo/`: API đăng nhập, lưu cấu hình vào D1 và gọi Workers AI.
- `functions/_middleware.js`: bảo vệ API bằng token HMAC 8 giờ.
- `migrations/0001_seo_settings.sql`: bảng D1.
- `scripts/build.mjs`: tạo `dist/` và áp metadata trang chủ.
- `scripts/audit.mjs`: audit SEO cơ bản trên HTML gốc.
- `wrangler.toml`: mẫu cấu hình Cloudflare Pages, D1 và Workers AI.

## Cảnh báo trước khi triển khai

1. ZIP hiện có một `index.html` rất lớn và sitemap liệt kê 4 URL. Cần xác minh 3 URL ngoài trang chủ có thật và trả HTTP 200. Không tự động tạo trang giả hoặc xóa URL cũ.
2. Chủ website đã xác nhận phường Long Bình. ZIP cập nhật Schema và README theo phường Long Bình; vẫn cần đối chiếu số nhà, tên đường và ghim bản đồ với hồ sơ Google Business Profile trước khi công bố.
3. `seo-config.json` chỉ có dữ liệu trang chủ. Sitemap được rút về trang chủ vì ZIP chỉ chứa một tệp HTML trang chủ; chỉ thêm URL khác sau khi tạo trang HTML riêng và kiểm tra HTTP 200, nội dung riêng, canonical riêng.
4. AI chỉ tạo bản nháp. Không xuất bản tự động thông tin địa chỉ, học phí, cam kết, thành tích hoặc đánh giá chưa được xác minh.
5. Không đưa GitHub token, mật khẩu, hoặc API key vào HTML/JavaScript public hay commit vào Git.

## Cấu hình Cloudflare

### A. Build

Yêu cầu Node.js 20+.

```bash
npm run seo:audit
npm run build
```

Output tĩnh ở `dist/`. Build hiện áp SEO Title, Meta Description, Open Graph title/description và canonical cho trang chủ từ `seo/seo-config.json`.

### B. Tạo D1

Tạo database tên `seo-manager` trong Cloudflare Dashboard > Workers & Pages > D1, hoặc dùng Wrangler:

```bash
npx wrangler d1 create seo-manager
```

Copy `database_id` vào `wrangler.toml` thay `REPLACE_WITH_YOUR_D1_DATABASE_ID`, rồi chạy migration:

```bash
npx wrangler d1 migrations apply seo-manager --remote
```

Trong Cloudflare Pages > Settings > Functions > D1 database bindings, tạo binding:
- Variable name: `DB`
- Database: `seo-manager`

### C. Bật Workers AI

Cloudflare Dashboard > Workers & Pages > AI > Workers AI, cấp quyền sử dụng Workers AI cho tài khoản. Trong Pages > Settings > Bindings, tạo Workers AI binding (nếu giao diện tài khoản hiển thị mục này). Cloudflare Pages yêu cầu cấu hình binding trong Dashboard; không dựa vào `[ai]` trong `wrangler.toml`. Tạo binding:
- Variable name: `AI`

Nếu tài khoản không hỗ trợ AI binding cho Pages Functions trong cấu hình hiện tại, cần chuyển endpoint AI sang Worker riêng rồi gọi Worker đó từ Pages Function.

### D. Đặt secrets

Cloudflare Pages > Settings > Variables and Secrets > Add:

- `SEO_ADMIN_PASSWORD`: mật khẩu quản trị mạnh, duy nhất.
- `SEO_SESSION_SECRET`: chuỗi ngẫu nhiên tối thiểu 32 bytes.

Có thể tạo chuỗi bí mật bằng:

```bash
openssl rand -base64 48
```

Không đặt hai giá trị này trong `wrangler.toml`, GitHub repository hoặc JavaScript.

### E. Build & deploy

Nếu project Cloudflare Pages đang kết nối GitHub:
- Build command: `npm run build`
- Build output directory: `dist`
- Root directory: `/`

Commit các thay đổi vào nhánh đang deploy. Trước tiên kiểm tra Preview deployment; chỉ promote khi trang chủ và các tính năng hiện tại hoạt động bình thường.

## Cách dùng Dashboard

1. Mở `https://anhvangiaotiepbienhoa.pages.dev/seo-admin/`.
2. Đăng nhập bằng `SEO_ADMIN_PASSWORD`.
3. Dashboard đọc/ghi cấu hình qua API và D1.
4. Chọn URL để chỉnh metadata; thêm URL chỉ khi URL đó đã tồn tại.
5. Dùng AI Assistant để tạo đề xuất; kiểm chứng rồi mới sử dụng.
6. Nhấn **Xuất JSON** để lưu cấu hình D1 và tải `seo-config.json`.
7. Thay file `seo/seo-config.json` trong repository bằng file vừa tải, commit và deploy để metadata trang chủ được đưa vào HTML build.

## Giới hạn phiên bản đầu

- Dashboard lưu cấu hình vào D1; build tĩnh đọc file JSON trong Git. Đây là hai nơi lưu riêng, nên sau khi chỉnh trên Dashboard phải xuất JSON và commit file vào repository để thay đổi xuất hiện trên website.
- Chức năng build hiện áp metadata cho trang chủ. Các trang URL khác cần có file HTML riêng hoặc cơ chế routing/render tương ứng trước khi tự động áp metadata.
- Phân tích hiện tại là kiểm tra on-page cơ bản, không phải công cụ crawler toàn bộ website.
- Chưa kết nối Google Search Console/Analytics. Không có API key hoặc OAuth được tự ý tạo.
- Điểm SEO là điểm nội bộ để ưu tiên công việc, không phải điểm Google sử dụng để xếp hạng.

## Local SEO sau khi xác minh thông tin

- Kiểm tra tên doanh nghiệp, số điện thoại, email, số nhà/đường/phường và ghim bản đồ khớp Google Business Profile.
- Chỉ bổ sung giờ mở cửa, tọa độ, logo và ảnh khi có dữ liệu thật đã xác minh.
- Không tự tạo đánh giá, AggregateRating, học phí, chứng nhận hoặc số liệu học viên.
- Dùng cùng một tên, địa chỉ, số điện thoại trên website và hồ sơ doanh nghiệp.
- Sau khi cập nhật, kiểm tra JSON-LD bằng Rich Results Test và Schema Markup Validator.

## Checklist sau khi deploy

- [ ] Trang chủ HTTP 200 và HTTPS.
- [ ] `/seo-admin/` mở được, không xuất hiện trên kết quả tìm kiếm.
- [ ] API không cho truy cập nếu thiếu token.
- [ ] Đăng nhập sai bị từ chối.
- [ ] D1 lưu và đọc cấu hình.
- [ ] AI endpoint trả kết quả (nếu đã bật AI binding).
- [ ] View Source có title, description, canonical đúng.
- [ ] Sitemap chỉ chứa URL canonical, tồn tại, trả HTTP 200.
- [ ] Schema hợp lệ và địa chỉ đã xác minh.
- [ ] Gửi sitemap qua Google Search Console.

## Tham khảo chính thức

- Cloudflare Pages Functions: https://developers.cloudflare.com/pages/functions/
- D1: https://developers.cloudflare.com/d1/
- Workers AI: https://developers.cloudflare.com/workers-ai/
- Google Search Central: https://developers.google.com/search/docs
