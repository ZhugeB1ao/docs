# FE-2: Tìm kiếm tài khoản và hoàn tất checklist — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans (hoặc superpowers:subagent-driven-development) để thực thi từng task. Steps dùng checkbox (`- [ ]`). Tuân thủ `front-end/AGENTS.md`: TDD, Ponytail/YAGNI, 0 warning lint, không thêm dependency, không commit/push khi chưa được yêu cầu.

**Goal:** Hoàn tất các mục còn thiếu của ticket FE-2 trong checklist: ô tìm kiếm ở danh sách tài khoản, nối API thật thay cho mock, kiểm tra màn hình từ 1366px.

**Architecture:** Tìm kiếm là bộ lọc phía server như `departmentId`/`role`/`active`: thêm `keyword` vào `AccountQuery`, gửi qua `usersApi.list`, lọc trong mock MSW và (task riêng thuộc BE-2) trong `AccountService.list`. Ô nhập dùng `CrudToolbar` có sẵn (`onSearchChange`); debounce 300 ms bằng `setTimeout` trong `UsersPage`, không viết hook mới (chỉ một nơi dùng).

**Tech Stack:** TypeScript, React 19, Axios, MSW, Vitest + React Testing Library (đã có); Java/Spring Data JPA `Specification` ở account-service. Không thêm dependency.

**Spec:** Ticket FE-2 (ảnh checklist ngày 10/11/2026): "Danh sách tài khoản có phân trang, tìm kiếm, lọc theo đơn vị, role, trạng thái", "Nối API thật thay cho mock", "Kiểm tra hiển thị trên màn hình 1366 trở lên"; tiêu chí hoàn thành: admin quản lý đủ vòng đời tài khoản, menu/nút ẩn hoặc khóa đúng theo role. Phụ thuộc BE-2. Tên tham số `keyword` là quyết định của plan này, chưa phải contract BE đã duyệt.

## Global Constraints

- UI tiếng Anh; ô tìm kiếm có `aria-label="Search accounts"`, placeholder `Search...` (do `CrudToolbar` quy định).
- `keyword`: sau `trim()`, rỗng thì **không gửi**; tối đa 254 ký tự (bằng độ dài email tối đa).
- Khớp **chứa**, không phân biệt hoa thường, trên `username` **hoặc** `email`; kết hợp AND với bộ lọc khác.
- Đổi `keyword` luôn đưa `page` về 0; debounce 300 ms.
- Giữ nguyên quy ước API hiện có: `/accounts`, `page` bắt đầu từ 0, `size` 10/20/50, phản hồi `AccountPageResult`.
- `npm run check` phải đạt (lint `--max-warnings 0`, prettier, vitest, build).

## Review Focus

1. Keyword chỉ có khoảng trắng → coi như không lọc, không gửi tham số (test ở Task 2).
2. Gõ nhanh nhiều ký tự → chỉ kết quả của chuỗi cuối được hiển thị, không gọi mỗi phím (Task 2, đã có test).
3. Đang ở trang 2/3 rồi gõ tìm kiếm → quay về trang 1 (Task 2, đã có test).
4. Keyword chứa `%` hoặc `_` → khớp theo nghĩa đen, không thành ký tự đại diện SQL (Task 3, BE).
5. Không có kết quả → thông báo "No accounts match your filters." và "Clear filters" xóa cả nội dung ô tìm kiếm (Task 2, đã có test).

## File Structure

| File | Trách nhiệm |
|---|---|
| `src/features/users/types.ts` | Thêm `keyword?: string` vào `AccountQuery` |
| `src/features/users/api/usersApi.ts` | Không đổi (Axios tự bỏ `undefined`); test xác nhận gửi `keyword` |
| `src/mocks/handlers.ts` | `GET /accounts` lọc theo `keyword` |
| `src/features/users/pages/UsersPage.tsx` | Ô tìm kiếm, debounce, reset trang, Clear filters |
| `account-service/.../AccountController.java`, `AccountService.java` | Tham số `keyword` và điều kiện lọc (Task 3) |
| `docs/api/users-contract.md`, `docs/account-management-acceptance.md` | Cập nhật theo hợp đồng thật (Task 5) |

---

### Task 1: Mock và kiểu dữ liệu hỗ trợ `keyword`

**Files:**
- Modify: `src/features/users/types.ts` (interface `AccountQuery`)
- Modify: `src/mocks/handlers.ts` (handler `GET ${base}/accounts`, khoảng dòng 695–725)
- Test: `src/features/users/api/usersApi.test.ts`, `src/mocks/accounts.test.ts`

**Interfaces:**
- Produces: `AccountQuery.keyword?: string`; `GET /api/accounts?keyword=<text>` lọc `username`/`email` chứa `<text>` không phân biệt hoa thường (sau trim).

- [x] **Step 1: Viết test thất bại.** Trong `usersApi.test.ts` thêm `it('sends keyword and omits it when undefined')`: gọi `usersApi.list({ page: 0, size: 10, keyword: 'staff' })` và khẳng định `query.keyword === 'staff'`; gọi lại không có `keyword` và khẳng định `'keyword' in query === false`. Trong `accounts.test.ts` thêm `it('filters accounts by keyword on username or email, ignoring case')`: đăng nhập admin, `GET /accounts?keyword=STAFF0` trả `totalElements` 9 (staff01–staff09 theo seed, kiểm tra lại số khi chạy) và `keyword=other@demo` trả đúng 1 tài khoản `other`.
- [x] **Step 2: Chạy `npx vitest run src/features/users/api src/mocks/accounts.test.ts`**, kỳ vọng FAIL (TypeScript không có `keyword`, mock chưa lọc).
- [x] **Step 3: Thêm `keyword?: string` vào `AccountQuery`; trong handler mock đọc `query.get('keyword')?.trim().toLowerCase()` và thêm điều kiện `!keyword || a.username.toLowerCase().includes(keyword) || a.email.toLowerCase().includes(keyword)` vào `matches`.**
- [x] **Step 4: Chạy lại hai file test**, kỳ vọng PASS.

---

### Task 2: Ô tìm kiếm trên `UsersPage`

**Files:**
- Modify: `src/features/users/pages/UsersPage.tsx`
- Test: `src/features/users/pages/UsersPage.test.tsx`

**Interfaces:**
- Consumes: `CrudToolbar` props `search`, `onSearchChange`, `searchAriaLabel` (đã có); `AccountQuery.keyword` (Task 1).
- Produces: state `searchText` trong `UsersPage`; `query.keyword` cập nhật sau 300 ms kể từ lần gõ cuối.

- [x] **Step 1: Viết các test thất bại** trong `UsersPage.test.tsx` (dùng `renderAccountRoute`, `userEvent`):
  - `it('searches by username or email and returns to the first page')`: sang trang 2 (`Page 2 / 3`), gõ `staff0` vào `getByLabelText('Search accounts')`, `findByText('9 accounts')` và `Page 1 / 1`.
  - `it('does not call the API for each keystroke')`: ghi đếm request `GET /accounts` bằng `featureServer.use`; gõ `staff01` bằng một lượt `user.type`; sau khi hiện `1 account`, số request có `keyword` đúng bằng 1 và giá trị là `staff01`.
  - `it('treats a whitespace-only keyword as no filter')`: gõ `   `, đợi 400 ms, khẳng định không có request nào mang `keyword` và vẫn `28 accounts`.
  - `it('clears the search box with Clear filters and shows an empty message when nothing matches')`: gõ `zzzz`, `findByText('No accounts match your filters.')`; bấm `Clear filters`, ô tìm kiếm rỗng (`toHaveValue('')`) và `28 accounts`.
- [x] **Step 2: Chạy `npx vitest run src/features/users/pages/UsersPage.test.tsx`**, kỳ vọng 4 test mới FAIL (không có ô tìm kiếm).
- [x] **Step 3: Cài đặt trong `UsersPage.tsx`:** thêm `const [searchText, setSearchText] = useState('')`; một `useEffect` phụ thuộc `[searchText]` đặt `setTimeout` 300 ms gọi `setQuery(prev => prev.keyword === keyword ? prev : { ...prev, keyword, page: 0 })` với `keyword = searchText.trim().slice(0, 254) || undefined`, và `clearTimeout` ở cleanup (trả về `prev` khi không đổi để mount không gây tải lại). Truyền `search={searchText}`, `onSearchChange={setSearchText}`, `searchAriaLabel="Search accounts"` cho `CrudToolbar`. Nút `Clear filters` gọi thêm `setSearchText('')`. Điều kiện thông báo rỗng thêm `query.keyword`.
- [x] **Step 4: Chạy lại file test và `npx vitest run src/features/users`**, kỳ vọng toàn bộ PASS (các test cũ không bị ảnh hưởng).
- [x] **Step 5: Chạy `npm run check`**, kỳ vọng exit 0.
- [ ] **Step 6: Commit** (chỉ khi người dùng yêu cầu): `feat(front-end): add keyword search to account list`.

---

### Task 3: Backend `keyword` (thuộc BE-2, cần chủ service duyệt)

**Files:**
- Modify: `account-service/src/main/java/com/example/accountservice/controller/AccountController.java` (method `list`)
- Modify: `account-service/src/main/java/com/example/accountservice/service/AccountService.java:34` (method `list`)
- Modify: `account-service/openapi.yaml` (mô tả tham số)
- Test: `account-service/src/test/java/com/example/accountservice/AccountServiceTest.java`

**Interfaces:**
- Produces: `PageResponse<AccountResponse> AccountService.list(Long departmentId, UserRole role, Boolean active, String keyword, int page, int size)`; `GET /api/accounts?keyword=`.

- [ ] **Step 1: Viết test thất bại** `list_keywordMatchesUsernameOrEmailIgnoringCaseAndEscapesWildcards` trong `AccountServiceTest` (hoặc test tích hợp H2 như `AuthFlowIntegrationTest` bên auth-service nếu `Specification` cần DB thật): với `keyword = "50%"` không khớp `"500"`; `keyword = "ALICE"` khớp `alice@hsu.edu.vn`.
- [ ] **Step 2: Chạy `./mvnw -q -pl . test -Dtest=AccountServiceTest`** trong `account-service`, kỳ vọng FAIL.
- [ ] **Step 3: Thêm `@RequestParam(required = false) @Size(max = 254) String keyword`; trong `Specification` thêm `cb.or(cb.like(cb.lower(username), pattern, '\\'), cb.like(cb.lower(email), pattern, '\\'))` với `pattern = "%" + escape(keyword.trim().toLowerCase()) + "%"`, `escape` thay `\`, `%`, `_` bằng dạng có `\` phía trước; bỏ qua khi `keyword` rỗng.**
- [ ] **Step 4: Chạy lại test**, kỳ vọng PASS; chạy toàn bộ `./mvnw -q test` của account-service.

---

### Task 4: Nối API thật và kiểm tra 1366px

**Files:** không sửa code trừ khi phát hiện lỗi; ghi kết quả vào `docs/account-management-acceptance.md`.

- [ ] **Step 1:** Chạy backend theo `HOW_TO_RUN.md` (docker-compose DB, auth-service, account-service, department-service, category-service, api-gateway), chạy frontend với `VITE_ENABLE_MOCKS` không bật và `VITE_API_BASE_URL` trỏ gateway.
- [ ] **Step 2:** Đăng nhập admin và đi qua kịch bản: danh sách (phân trang, lọc đơn vị/role/trạng thái, tìm kiếm), tạo, sửa, vô hiệu hóa, kích hoạt lại, đặt lại mật khẩu, đăng nhập bằng tài khoản mới, trang cá nhân, đổi mật khẩu. Mỗi bước ghi đạt/lỗi.
- [ ] **Step 3:** Với Manager và Employee: không thấy menu Users; vào thẳng `/admin/users` nhận trang 403; gọi API trực tiếp nhận 403.
- [ ] **Step 4:** Đặt cửa sổ 1366×768 (và 1920×1080): bảng 6 cột, thanh bộ lọc (tìm kiếm, phòng ban, role, trạng thái, Clear filters), hộp thoại tạo/sửa không tràn, không cuộn ngang trang. Chụp ảnh lưu vào `output/playwright/`.
- [ ] **Step 5:** Cập nhật `docs/api/users-contract.md` (endpoint `/accounts`, `page`/`size` từ 0, `active`, `keyword`) và ghi kết quả thật vào `account-management-acceptance.md`; bỏ các câu "chưa kiểm chứng" chỉ khi đã chạy được.

---

## Self-Review

- **Spec coverage:** tìm kiếm → Task 1–3; nối API thật → Task 4 (bước 1–3); 1366px → Task 4 bước 4; menu/nút theo role → Task 4 bước 3 (code đã có, chỉ xác minh). Các mục đã tick (form, xác nhận, hồ sơ, lỗi 401/403/409) không đụng tới.
- **Type consistency:** `keyword` dùng thống nhất ở `AccountQuery`, query string, mock và backend.
- **Proportion:** các task dùng chữ thay cho thân hàm; chỉ nêu chữ ký, giá trị và điều kiện kiểm chứng.
- **Rủi ro đã biết:** Task 3 nằm ngoài `front-end/` (BE-2). Nếu chưa làm Task 3, backend thật sẽ **bỏ qua** `keyword` (Spring không báo lỗi tham số lạ), nên ô tìm kiếm trông như không hoạt động khi chạy với API thật.
