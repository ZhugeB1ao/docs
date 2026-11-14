# Màn hình quản lý tài khoản và hồ sơ cá nhân — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking. Tuân thủ `front-end/AGENTS.md`: chỉ hướng dẫn và cung cấp diff/code mẫu cho `src/**`, trừ khi người dùng sau này cho phép sửa trực tiếp.

**Goal:** Admin quản lý tài khoản từ tạo, sửa đến vô hiệu hóa, kích hoạt lại và đặt lại mật khẩu; mọi người dùng xem hồ sơ của mình và đổi mật khẩu, với quyền và lỗi rõ ràng.

**Architecture:** Giữ route guard, Auth Context và Axios interceptor hiện có. Tạo feature `users`, dùng cùng API client cho MSW và backend; danh sách xử lý lọc/phân trang ở API. Dữ liệu tài khoản mock phải dùng chung với auth để tạo/sửa/khóa tài khoản thực sự ảnh hưởng đăng nhập và hồ sơ.

**Tech Stack:** TypeScript, React 19, React Router, Axios, Tailwind v4, MSW, Vitest, React Testing Library đã có trong `package.json`; không thêm dependency.

**Spec:** Yêu cầu ticket người dùng ngày 05/11/2026, được chép lại và cụ thể hóa ở phần 1–4 bên dưới; tham chiếu `../specs/2026-10-29-auth-role-scope.md` và `../../api/auth-contract.md`. Các quyết định bổ sung là đề xuất của người lập plan, chưa phải contract BE đã duyệt.

**Trạng thái:** Đã thực hiện, review trong preview rồi áp dụng vào source gốc sau khi người dùng xác nhận “Cho phép áp dụng trực tiếp”. Kiểm chứng tại front-end gốc: 64 test, lint/format/build đạt; browser acceptance desktop/mobile trên preview đạt. Xem `docs/implementation/account-management/README.md` và ledger cùng thư mục. Mọi đường dẫn trong tài liệu tính từ `front-end/`.

## Checklist skill — đọc trước khi thực hiện

Đây là checklist thực thi, không phải xác nhận các skill đã được áp dụng đầy đủ. Trước khi làm task, mở `SKILL.md` của các skill được gắn bên dưới từ danh sách skill của phiên hiện tại, thông báo skill đang dùng và thực hiện đúng hướng dẫn. Đọc lại khi bắt đầu phiên mới nếu chưa có nội dung skill trong ngữ cảnh; trong cùng phiên không đọc lại vô ích. Nếu skill không có sẵn, ghi rõ thiếu gì và ảnh hưởng, không tự nhận đã áp dụng.

Ưu tiên yêu cầu người dùng và `AGENTS.md`; hướng dẫn skill không cho phép tự vượt phạm vi sửa source, thêm dependency, commit/push/merge hoặc publish. GitNexus là công cụ/quy tắc dự án, không phải skill: vẫn phải chạy context/impact trước khi đề xuất diff như phần khảo sát bên dưới.

### A. Skill cần dùng trong luồng triển khai này

| Theo dõi | Skill                            | Thời điểm                                                   | Kết quả cần ghi nhận                                                                                                                                                                                |
| -------- | -------------------------------- | ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [x]      | `using-superpowers`              | Bắt đầu phiên, trước khi thực hiện task                     | Xác định skill áp dụng và đọc hướng dẫn trước hành động tương ứng                                                                                                                                   |
| [x]      | `ponytail`                       | Xuyên suốt task 1–8                                         | Tận dụng nền auth/MSW/component hiện có; không thêm abstraction/dependency không cần thiết                                                                                                          |
| [x]      | `brainstorming`                  | Trước triển khai; dùng lại khi thay đổi nghiệp vụ/phạm vi   | Review các giả định mục 2 và thiết kế với người dùng; bản plan này chưa thay thế việc duyệt thiết kế                                                                                                |
| [x]      | `writing-plans`                  | Khi hoàn thiện hoặc điều chỉnh plan                         | Đồng bộ contract, file, interface, task và test; giữ checklist theo dõi được                                                                                                                        |
| [x]      | `executing-plans`                | Khi bắt đầu thực thi trực tiếp, sau khi thiết kế được duyệt | Đọc plan/spec, theo thứ tự phụ thuộc; ghi tiến độ và quyết định thay đổi vào ledger; không làm lại task đã có bằng chứng hoàn thành                                                                 |
| [x]      | `test-driven-development`        | Trước code logic tính năng/bugfix ở task 1–7                | Với schema, quyền, trạng thái, request và form: test hành vi thất bại đúng lý do → triển khai tối thiểu → test pass; không tạo test chỉ lặp lại implementation hoặc cho thay đổi tài liệu đơn thuần |
| [x]      | `ui-ux-pro-max`                  | Trước thiết kế UI ở task 4; áp dụng tiếp task 5–7           | Chốt bố cục bảng/filter/form/profile và trạng thái loading/empty/error/disabled; kiểm tra responsive, focus, tương phản và khả năng thao tác                                                        |
| [x]      | `ui-styling`                     | Khi dựng giao diện task 4–7                                 | Tái sử dụng Tailwind, CSS và Button có sẵn; native dialog; không cài shadcn/Radix hay thư viện mới chỉ vì skill có ví dụ dùng chúng                                                                 |
| [x]      | `vercel-react-best-practices`    | Khi viết/review React, context và data fetching ở task 3–7  | Effect cleanup/dependencies đúng, tránh fetch nối tiếp không cần thiết và state trùng lặp; chỉ áp dụng quy tắc phù hợp React/Vite, không đưa Next.js/SWR vào dự án                                  |
| [x]      | `web-design-guidelines`          | Audit UI tại task 8                                         | Lấy bản guidelines mới theo SKILL.md lúc review; đối chiếu file thực tế, ghi finding bằng file/dòng và khắc phục vấn đề ảnh hưởng người dùng                                                        |
| [x]      | `requesting-code-review`         | Trước nghiệm thu/bàn giao toàn bộ thay đổi ở task 8         | Review diff so với ticket, contract, ma trận quyền và Review Focus; ghi finding và kết quả xử lý, không chỉ dựa vào việc test xanh                                                                  |
| [x]      | `verification-before-completion` | Trước đánh dấu từng task xong và trước báo hoàn thành       | Chạy lệnh kiểm tra tương ứng, đọc output/exit code; cuối luồng có `npm run check` và bằng chứng demo thực tế                                                                                        |

### B. Skill có điều kiện — kiểm tra điều kiện, không kích hoạt tất cả

| Theo dõi | Skill                            | Khi nào dùng                                                                                  | Giới hạn cho ticket này                                                                                                                      |
| -------- | -------------------------------- | --------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [ ]      | `using-git-worktrees`            | Trước triển khai source khi đã được phép sửa/tích hợp                                         | Kiểm tra workspace hiện tại đã cách ly chưa; giữ thay đổi AppRouter có sẵn; không tạo worktree cho lần chỉ cập nhật tài liệu này             |
| [x]      | `systematic-debugging`           | Có bug, test fail hoặc hành vi khác dự kiến ở bất kỳ task nào                                 | Tái hiện, xác định nguyên nhân rồi sửa; không sửa assertion/fixture chỉ để test xanh                                                         |
| [x]      | `receiving-code-review`          | Khi nhận finding/góp ý code                                                                   | Kiểm chứng với source, contract và test trước khi áp dụng; ghi lý do nếu không áp dụng                                                       |
| [x]      | `playwright`                     | Nghiệm thu trình duyệt thật qua terminal ở task 8                                             | Kiểm tra dialog, focus, viewport và network; đọc skill trước khi dùng CLI, ưu tiên môi trường hiện có, không cài thêm npm dependency tự phát |
| [ ]      | `computer-use:computer-use`      | Cần kiểm tra trực tiếp browser/app bằng UI và chọn công cụ computer use                       | Là phương án theo nhu cầu thực tế; không chạy cả computer use và Playwright cho cùng một kiểm tra nếu không có lý do                         |
| [ ]      | `vercel-composition-patterns`    | Component thực sự phát sinh quá nhiều boolean props hoặc cần API composition                  | Không refactor thành compound components/context chỉ để dùng skill                                                                           |
| [ ]      | `design-system`                  | Phát hiện thiếu token/spec dùng chung cản trở tính nhất quán UI                               | Mặc định dùng CSS/token hiện có; chỉ bổ sung phần thực sự thiếu, không tạo design system mới cho ticket                                      |
| [ ]      | `subagent-driven-development`    | Chọn phương thức triển khai bằng subagent theo quyền và hướng dẫn phiên                       | Là phương án thay cho `executing-plans`, không vận hành cả hai luồng cho cùng task; không tự spawn trong lần cập nhật plan này               |
| [ ]      | `dispatching-parallel-agents`    | Có ít nhất hai công việc độc lập, không ghi chung file, và luồng thực thi cho phép delegation | Chuỗi task 1–8 có phụ thuộc nên mặc định tuần tự; không chia song song các mutation auth/mock/router phụ thuộc nhau                          |
| [ ]      | `finishing-a-development-branch` | Source đã được tích hợp, test/review đạt và chuẩn bị kết thúc nhánh                           | Kiểm tra trạng thái nhánh và trình bày phương án bàn giao phù hợp quyền hiện có; không coi skill là quyền tự merge/push                      |

Không cần các skill Canva, Sites, Unity, imagegen, tài liệu Office hoặc branding để hoàn thành ticket hiện tại. Chỉ bổ sung khi yêu cầu thực tế thay đổi; mục tiêu là đủ skill phù hợp, không phải dùng hết danh sách skill.

### C. Cách tránh bỏ sót khi chuyển task hoặc đổi phiên

- [ ] Trước Task 1: đọc `AGENTS.md`, kiểm tra Git status, đọc các giả định và Review Focus; hoàn tất bước thiết kế theo `brainstorming` trước khi viết code.
- [ ] Khi bắt đầu thực thi: tạo ledger theo `executing-plans`, ghi rõ định danh plan và task hiện tại. Chỉ cập nhật checklist, không tự chạy triển khai khi yêu cầu mới chỉ là lập/sửa plan.
- [ ] Đầu mỗi task: đọc dòng **Skills trước khi làm**, kiểm tra các điều kiện ở bảng B và ghi skill được dùng. Task gặp lỗi chuyển sang `systematic-debugging`; có góp ý chuyển sang `receiving-code-review`.
- [ ] Cuối mỗi task: ghi theo mẫu `Task N | skill đã áp dụng | bằng chứng (test/finding/quyết định) | kết quả hoặc lý do không áp dụng`; tick skill sau khi có hành động/bằng chứng, không tick chỉ vì đã đọc tên.
- [ ] Khi có compaction/phiên mới: đọc plan và ledger, đối chiếu source/Git; tiếp tục task chưa xong, không mặc định những lần đọc skill hoặc test trong phiên trước vẫn là bằng chứng cho thay đổi mới.
- [ ] Trước bàn giao: hoàn tất UI audit, code review, kiểm chứng; mỗi skill có điều kiện ghi “đã dùng” hoặc “không áp dụng — lý do”. Đính kèm lệnh/kết quả thật và giới hạn chưa kiểm tra.

**Quyền tích hợp đã xác nhận:** Người dùng đã trả lời “Cho phép áp dụng trực tiếp” sau khi xem kết quả preview/patch. Quyền này cho phép sửa `src/**` trong phạm vi ticket hiện tại, thay thế chế độ chỉ cung cấp diff nêu trong bản plan ban đầu. Giữ nguyên branch/staging và phần sửa có sẵn; không tự commit/push/merge.

## Global Constraints

- **Quy ước mới từ người dùng:** Toàn bộ UI dùng tiếng Anh, gồm nhãn, thông báo, validation, aria-label và dữ liệu demo. Quy ước này thay thế các câu chữ tiếng Việt trong ví dụ/lịch sử plan bên dưới. Nội dung thực do người dùng nhập và tài liệu nguồn không bị dịch tự động.

- Người dùng đã cho phép tích hợp trực tiếp `src/**` cho ticket này; vẫn giữ các quy tắc khác của `AGENTS.md`, không ghi đè thay đổi ngoài phạm vi.
- Dùng `.ts`/`.tsx` theo source thực tế; single quote, không semicolon, ESLint 0 warning.
- Một tài khoản có một role trong `ADMIN | MANAGER | EMPLOYEE`, lưu bằng mảng `roles` tương thích auth hiện có.
- UI gọi API client; MSW thực thi mock. Không đặt fixture hoặc nhánh `if (mock)` trong component.
- Dùng `FormData`, native validation, `AbortController`, `<dialog>`; tái sử dụng `Button` và CSS sẵn có.
- Protected request 401 dùng refresh/replay tối đa một lần như hiện tại; 403/409 không refresh.
- Không lưu password hoặc access token vào localStorage/sessionStorage; mật khẩu mock chỉ sống trong bộ nhớ và reset khi reload.
- Phải giữ nguyên thay đổi chưa commit có sẵn trong `src/app/router/AppRouter.tsx`; không ghi đè bằng bản cũ.

## Review Focus

1. Tìm kiếm A trả chậm hơn B: chỉ B được hiển thị; test tại task 4.
2. Thao tác làm bản ghi rời bộ lọc hoặc trang cuối: refetch đúng bộ lọc, lùi trang hợp lệ, cập nhật tổng số; test tại task 4 và 6.
3. Tài khoản bị khóa nhưng còn metadata phiên: login/me/refresh không cho tiếp tục phiên; kích hoạt lại không phục hồi token cũ; test tại task 2.
4. Đơn vị/chức vụ hiện tại đã ngưng dùng: form sửa vẫn hiển thị nhãn cũ; chỉ cho giữ nguyên hoặc đổi sang lựa chọn hợp lệ; test tại task 5.
5. Logout/chuyển tài khoản khi request hồ sơ hoặc reloadUser chưa xong: không phục hồi user cũ hay hiển thị dữ liệu cũ; test tại task 3 và 7.

## 1. Hiện trạng và phạm vi

Đã có `/admin/users` chỉ cho ADMIN nhưng render `Workspace`; `/change-password` đã hoạt động qua `AccountPage`. Sidebar dùng `routes` và `canAccess`; `Guard` chặn truy cập URL trực tiếp. `http.ts` đã xử lý refresh dùng chung và loại phản hồi thuộc phiên cũ. Chưa có trang hồ sơ, CRUD user, trạng thái tài khoản, dropdown đơn vị/chức vụ cho tài khoản.

`categoryApi.ts` đang dùng Axios riêng đến cổng 8080 và token demo dành cho category-service. Không sao chép cơ chế đó sang user API. Dropdown của ticket phải có mock độc lập, dùng client `api` chung; khi nối thật mới ánh xạ sang nguồn dữ liệu được BE thống nhất.

Phạm vi hoàn thành là năm mục checklist người dùng: danh sách có phân trang/tìm kiếm/lọc; tạo/sửa; ba thao tác có xác nhận; hồ sơ và đổi mật khẩu; thông báo 401/403/409. Không bao gồm xóa vĩnh viễn, CRUD đơn vị/chức vụ, tự đăng ký nghiệp vụ, bulk action, avatar upload hoặc gửi email thật.

### Khảo sát GitNexus ngày 05/11/2026

| Symbol                    | Caller/vùng ảnh hưởng đã thấy                            | Process                                        | Kết quả graph                                |
| ------------------------- | -------------------------------------------------------- | ---------------------------------------------- | -------------------------------------------- |
| `canAccess`               | `Guard`, `safeDestination`, `Sidebar`, test app          | AppRouter → CanAccess; AccountPage → CanAccess | LOW, 7 upstream                              |
| `AppRouter`               | `App` → `main`                                           | Điều hướng toàn app                            | LOW, 2 upstream                              |
| `AccountPage`             | `AppRouter` → `App` → `main`                             | AppRouter → CanAccess/UseAuth                  | LOW, 3 upstream                              |
| `AuthProvider`, `useAuth` | `App`; Guard/AccountPage/AppLayout/Sidebar               | Khôi phục phiên, quyền, layout                 | LOW, lần lượt 2 và 6 upstream                |
| `userFor`                 | handler auth/demo/categories → browser → main            | Xác thực request mock                          | LOW, 3 upstream; graph không liệt kê process |
| `authApi`                 | Đối chiếu source: `AuthProvider`, `AccountPage`          | Login/me/logout/đổi mật khẩu                   | UNKNOWN, graph không resolve property calls  |
| `resetMock`               | Đối chiếu source: App.test, handlers.test, contract.test | Reset fixture giữa các test                    | UNKNOWN, graph không resolve caller          |

Đã chạy query/context/impact upstream. Index ngày 01/11 vẫn trỏ tới JS/JSX; source hiện tại là TS/TSX. Query cũng bị hạn chế vì FTS không khả dụng. Vì vậy LOW chỉ phản ánh snapshot cũ; UNKNOWN còn cần xác minh khi triển khai, không coi là an toàn. Trước khi đề xuất diff thực tế, cập nhật index nếu môi trường cho phép và chạy lại impact. Không thay thuật toán `canAccess`/refresh để làm ticket này.

## 2. Các quyết định đề xuất cho bản mock

| Nội dung            | Mặc định dùng để lập plan                                                                                                                 |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Trạng thái          | `ACTIVE`, `INACTIVE`; tạo mới là ACTIVE; vô hiệu hóa giữ dữ liệu                                                                          |
| Hồ sơ cá nhân       | Trang chỉ đọc thông tin của người đang đăng nhập, có link đổi mật khẩu; tự sửa hồ sơ chưa nằm trong ticket                                |
| Định danh           | Email là tên đăng nhập; không thêm username; email được trim/lowercase để kiểm tra trùng                                                  |
| Form tài khoản      | Họ tên, email, một role, đơn vị, chức vụ; tạo mới thêm mật khẩu ban đầu và xác nhận                                                       |
| Đơn vị/chức vụ      | MANAGER/EMPLOYEE bắt buộc có đơn vị; ADMIN có thể không thuộc đơn vị; chức vụ chọn được và tạm cho phép để trống                          |
| Role và chức vụ     | Hai thuộc tính riêng; chức vụ không tự cấp quyền và chưa có quy tắc phụ thuộc đơn vị                                                      |
| Reset mật khẩu      | ADMIN nhập mật khẩu mới và xác nhận trong dialog; không cần mật khẩu cũ; không gửi email và không trả password trong response             |
| Chính sách mật khẩu | 8–128 ký tự như contract auth hiện có; không trim password                                                                                |
| Tự quản trị         | Không tự vô hiệu hóa hoặc tự đổi role; nút reset chính mình khóa và dẫn sang đổi mật khẩu. Vẫn được sửa tên/email/đơn vị/chức vụ của mình |
| Phiên sau thay đổi  | Đổi mật khẩu cá nhân thu hồi phiên như hiện tại. Reset bởi ADMIN thu hồi phiên của user đích; kích hoạt lại phải đăng nhập mới            |
| Dữ liệu mock        | Có ít nhất 25 tài khoản, đủ 3 role, nhiều đơn vị và cả 2 trạng thái; giữ 4 tài khoản demo cũ để test hồi quy                              |
| Lưu trữ             | CRUD giữ trong bộ nhớ trong vòng đời trang; reload khôi phục fixture, không tuyên bố lưu bền vững                                         |

Các cách cân nhắc: (a) state/fixture ngay trong UI: nhanh nhưng không kiểm tra được contract/401/403 và phải viết lại khi nối thật; (b) MSW + client `api` hiện có: đề xuất vì chạy qua đúng HTTP và auth; (c) gọi ngay các service thật: chưa phù hợp với yêu cầu mock trước và API user chưa có contract trong repo.

Các điểm có thể đổi sau review: hồ sơ có cần tự sửa hay không; reset qua email hay nhập mật khẩu mới; chức vụ có bắt buộc/phụ thuộc đơn vị; chính sách bảo vệ tài khoản Admin. Nếu thay đổi, cập nhật contract và test trước khi viết UI.

## 3. Ma trận quyền và trạng thái giao diện

| Chức năng                              | ADMIN                     | MANAGER                 | EMPLOYEE                | Chưa đăng nhập |
| -------------------------------------- | ------------------------- | ----------------------- | ----------------------- | -------------- |
| `/admin/users`: xem/tìm/lọc/phân trang | Có                        | Ẩn menu, URL → 403      | Ẩn menu, URL → 403      | → login        |
| Tạo/sửa/khóa/kích hoạt/reset           | Có theo điều kiện bản ghi | Không có nút; API → 403 | Không có nút; API → 403 | API → 401      |
| `/profile`: xem hồ sơ chính mình       | Có                        | Có                      | Có                      | → login        |
| `/change-password`                     | Có                        | Có                      | Có                      | → login        |

- ACTIVE: hiện vô hiệu hóa; INACTIVE: hiện kích hoạt lại; không hiện đồng thời hai hành động.
- Reset chỉ khả dụng cho ACTIVE và không phải chính mình; sửa được cả tài khoản INACTIVE.
- Khóa nút khi đang gửi request; nêu lý do cho trường hợp tự khóa/tự đổi role/reset không hợp lệ.
- Client chặn để hướng dẫn người dùng; mock/BE vẫn kiểm tra actor, role và trạng thái trên từng request, không tin role trong payload.

## 4. Contract và interface dự kiến

### Types trong `src/features/users/types.ts`

- `AccountStatus = 'ACTIVE' | 'INACTIVE'`.
- `ManagedAccount extends User`: bắt buộc `email: string`, `displayName: string`, `roles: [Role]`, `departmentId: string | null`; thêm `positionId: number | null`, `status: AccountStatus`.
- `AccountWrite`: `email`, `displayName`, `roles: [Role]`, `departmentId`, `positionId`; không nhận `id` hoặc `status`.
- `CreateAccountInput = AccountWrite & { initialPassword: string }`; confirm chỉ ở UI.
- `AccountQuery`: `page: number`, `pageSize: 10 | 20 | 50`, `q?: string`, `departmentId?: string`, `role?: Role`, `status?: AccountStatus`.
- `AccountPageResult`: `{ items: ManagedAccount[], page: number, pageSize: number, total: number }`.
- `DepartmentOption`: `{ id: string, name: string, isActive: boolean }`; `PositionOption`: `{ id: number, name: string, isActive: boolean }`. Giữ ID chức vụ dạng number tương thích CategoryItem.
- `Profile = ManagedAccount & { departmentName: string | null, positionName: string | null }`; không trả password/hash/token.
- Lỗi dùng shape hiện có `{ code: string, message: string, fieldErrors?: Record<string, string> }`.

### API proposal v0.1 — base URL lấy từ `env.apiBaseUrl`

| Method/path                      | Client/interface                                                                             | Thành công                                  |
| -------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------- |
| GET `/users`                     | `usersApi.list(query: AccountQuery, signal?: AbortSignal): Promise<AccountPageResult>`       | 200, lọc trước rồi mới phân trang           |
| POST `/users`                    | `usersApi.create(input: CreateAccountInput, signal?): Promise<ManagedAccount>`               | 201                                         |
| PUT `/users/:id`                 | `usersApi.update(id: string, input: AccountWrite, signal?): Promise<ManagedAccount>`         | 200                                         |
| PATCH `/users/:id/deactivate`    | `usersApi.deactivate(id: string, signal?): Promise<ManagedAccount>`                          | 200                                         |
| PATCH `/users/:id/activate`      | `usersApi.activate(id: string, signal?): Promise<ManagedAccount>`                            | 200                                         |
| POST `/users/:id/reset-password` | `usersApi.resetPassword(id: string, input: { newPassword: string }, signal?): Promise<void>` | 204, không đọc JSON                         |
| GET `/departments`               | `accountOptionsApi.departments(signal?): Promise<DepartmentOption[]>`                        | 200, đọc danh mục cho ADMIN                 |
| GET `/categories?type=POSITION`  | `accountOptionsApi.positions(signal?): Promise<PositionOption[]>`                            | 200, lấy cả inactive để hiển thị giá trị cũ |
| GET `/users/me`                  | `usersApi.me(signal?): Promise<Profile>`                                                     | 200, actor từ token, cho cả 3 role          |
| POST `/auth/change-password`     | Giữ `authApi.change(values, signal?)`                                                        | 204, logout sau thành công                  |

`signal?` trong bảng đều là `AbortSignal`; mọi method HTTP protected dùng `api` chung. Đăng ký `/users/me` trước mọi route động `/users/:id` nếu có bổ sung route đọc chi tiết. `/auth/me` giữ response tương thích cũ; `/users/me` dành cho thông tin hồ sơ mở rộng. Contract user viết riêng, không thêm path vào OpenAPI auth đang có test cố định 6 path.

Quy ước list: page bắt đầu từ 1, mặc định 1/10; tìm không phân biệt hoa thường trên tên/email, debounce 300 ms; nhiều filter kết hợp AND, đổi filter/pageSize đưa về page 1; sort ổn định theo email rồi id. Giới hạn q tối đa 254 ký tự; khoảng trắng đầu/cuối bị bỏ. Dữ liệu ngoài page trả items rỗng, UI tính trang cuối và refetch khi cần.

| Lỗi                           | Hành vi/Thông báo                                                                                                                      |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| 401 sau refresh thất bại      | Xóa phiên, về login; “Phiên đăng nhập đã hết hạn. Vui lòng đăng nhập lại.” Hiển thị thông báo trên trang đích để không mất khi unmount |
| 403                           | Route quản trị sai role → trang 403; lỗi API thao tác → “Bạn không có quyền thực hiện thao tác này.”; giữ form, không retry tự động    |
| 409 `EMAIL_EXISTS`            | “Email đã được sử dụng.”; lỗi ngay trường email, focus trường đó, giữ dữ liệu form                                                     |
| 409 `SELF_ACTION_NOT_ALLOWED` | “Bạn không thể tự vô hiệu hóa hoặc tự thay đổi quyền quản trị.”; không cập nhật UI giả thành công                                      |
| 409 `INVALID_ACCOUNT_STATE`   | “Trạng thái tài khoản đã thay đổi. Vui lòng kiểm tra lại.”; refetch danh sách                                                          |
| 422                           | Hiển thị fieldErrors cho input sai, kể cả đơn vị/chức vụ không hợp lệ và sai mật khẩu hiện tại                                         |
| 404 / mạng / 5xx              | Báo không tìm thấy hoặc không thể kết nối; có thử lại; giữ input, không hiện thành công                                                |

## 5. Bản đồ file

| File tạo mới                                                                         | Trách nhiệm                                                   |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------- |
| `docs/api/users-contract.md`, `docs/api/openapi-users.proposal.yaml`                 | Contract đề xuất, endpoint, schema, lỗi, giới hạn mock        |
| `src/features/users/types.ts`                                                        | Types user quản trị, query, profile, options                  |
| `src/features/users/api/usersApi.ts`, `accountOptionsApi.ts` cùng thư mục            | Gọi endpoint, trả dữ liệu typed; mapping nguồn options        |
| `src/features/users/pages/UsersPage.tsx`                                             | Query, fetch, phân trang, điều phối form/action               |
| `src/features/users/components/UserFormDialog.tsx`                                   | Tạo/sửa, validation và dropdown                               |
| `src/features/users/components/UserActionDialog.tsx`                                 | Xác nhận deactivate/activate/reset; reset có password/confirm |
| `src/features/users/pages/ProfilePage.tsx`                                           | Hồ sơ chỉ đọc và link đổi mật khẩu                            |
| `src/mocks/accountStore.ts`                                                          | Fixture mutable dùng chung auth/users, password mock, reset   |
| `src/mocks/users.test.ts`, `usersContract.test.ts`                                   | Luồng API, quyền/trạng thái và schema                         |
| `src/features/users/pages/UsersPage.test.tsx`, `ProfilePage.test.tsx`                | List/race conditions, hồ sơ                                   |
| `src/features/users/components/UserFormDialog.test.tsx`, `UserActionDialog.test.tsx` | Form và xác nhận hành động                                    |
| `src/features/users/api/usersApi.test.ts`                                            | Mapping query/payload/204, dùng MSW                           |
| `src/features/auth/context/AuthProvider.test.tsx`                                    | reloadUser và phản hồi cũ sau logout                          |
| `docs/account-management-acceptance.md`                                              | Kịch bản nghiệm thu mock và checklist nối thật                |

File dự kiến chỉnh: `src/mocks/handlers.ts` và `handlers.test.ts`; `src/app/router/access.ts`, `AppRouter.tsx`, `src/app/App.test.tsx`; `src/features/auth/api/authApi.ts`, `context/AuthProvider.tsx`, `context/useAuth.ts`, `pages/AccountPage.tsx`; `src/shared/styles/styles.css`; `docs/api/auth-contract.md` và `README.md` để cập nhật giới hạn fixture. `src/shared/api/http.ts` giữ nguyên thuật toán; test hồi quy bằng `http.test.ts`. Không sửa `Sidebar.tsx` chỉ để thêm menu vì nó đã đọc config routes.

## 6. Các task triển khai theo thứ tự

### Task 1 — Chốt contract và mô hình dữ liệu

**Skills trước khi làm:** `brainstorming` (review thiết kế/giả định trước triển khai), `writing-plans`, `executing-plans`, `ponytail`, `test-driven-development`; dùng `verification-before-completion` trước khi đánh dấu xong. Đọc `using-superpowers` ở đầu phiên.

**Files:** Tạo `types.ts`, hai tài liệu contract và `src/mocks/usersContract.test.ts` như bản đồ.

**Interfaces:** Xuất toàn bộ type ở mục 4; OpenAPI user độc lập, schema có `$ref` nội bộ phù hợp Ajv hiện có.

- [ ] Ghi rõ các đề xuất ở mục 2, đặc biệt reset/khởi tạo password, trường bắt buộc và trang hồ sơ chỉ đọc.
- [ ] Viết test schema `accepts_account_page_and_profile_without_secrets`: chấp nhận list/profile; từ chối role lạ, status lạ, pageSize không thuộc 10/20/50 và thuộc tính password/hash trong response.
- [ ] Kiểm tra test thất bại trước khi bổ sung schema; hoàn thiện OpenAPI/type theo đúng tên ở mục 4. Thêm schema errors 401/403/409/422.
- [ ] Chạy `npm test -- src/mocks/usersContract.test.ts`; đạt khi các input hợp lệ/không hợp lệ được phân biệt đúng. Commit riêng tài liệu/type/test nếu luồng tích hợp cho phép.

**Đầu ra:** Contract đủ để FE chạy mock, chưa cần đợi API thật; không đổi contract auth cũ.

### Task 2 — Mock vòng đời tài khoản dùng chung với auth

**Skills trước khi làm:** `executing-plans`, `ponytail`, `test-driven-development`, `verification-before-completion`; nếu mock/auth/test sai dự kiến, kích hoạt `systematic-debugging` trước khi sửa.

**Files:** Tạo `src/mocks/accountStore.ts`, `users.test.ts`; chỉnh `handlers.ts`, `handlers.test.ts`, nối response vào `usersContract.test.ts`.

**Interfaces:** `accountStore.accounts: ManagedAccount[]`, `accountStore.passwords: Record<string, string>`; `resetAccounts(): void` reset cả hai. `resetMock()` gọi resetAccounts đồng thời reset phiên/categories như hiện tại. Mật khẩu nằm riêng, không được spread vào response tài khoản.

- [ ] Viết test API trước: ADMIN tạo → đăng nhập user mới → ADMIN sửa → user thấy hồ sơ mới → vô hiệu hóa → login bị chặn → kích hoạt → login được; reset password làm mật khẩu cũ thất bại, mật khẩu mới thành công. Test từng bước không dựa vào fixture trùng nhau.
- [ ] Thêm assertions: list 25+ tài khoản lọc AND đúng tổng số/trang; email trùng khác hoa thường trả 409/EMAIL_EXISTS; role/đơn vị/chức vụ sai trả 422; actor MANAGER/EMPLOYEE gọi trực tiếp mọi mutation trả 403 và dữ liệu không đổi; chưa đăng nhập trả 401.
- [ ] Chuyển bốn fixture gốc và password map sang accountStore; login, userFor, me, refresh, change-password và CRUD đọc cùng một store. Không tạo danh sách user độc lập với auth.
- [ ] Tạo đủ endpoint mục 4, kiểm tra actor trước khi đọc/ghi. Status ACTIVE mới được login/me/refresh; thao tác self và trạng thái không hợp lệ trả mã đã định nghĩa.
- [ ] Dùng phiên bản phiên theo từng user hoặc epoch tương đương: deactivate/reset làm token/refresh metadata cũ mất hiệu lực; activate không hợp thức hóa metadata đó. `resetMock` reset epoch để test độc lập. Test bằng metadata phiên lưu trước hành động rồi phục hồi để gọi me/refresh, vì mock hiện chỉ có một phiên active, không giả vờ hỗ trợ nhiều người đăng nhập đồng thời.
- [ ] Test dropdown inactive và test `resetMock` khôi phục cả user/password/options. Đảm bảo GET categories POSITION không gọi cổng 8080 khi demo ticket này.
- [ ] Chạy `npm test -- src/mocks/users.test.ts src/mocks/handlers.test.ts src/mocks/contract.test.ts src/mocks/usersContract.test.ts`; tất cả pass, gồm auth/category hồi quy. Commit riêng mock nếu được phép tích hợp.

**Đầu ra:** Vòng đời hoạt động thực qua HTTP mock và ảnh hưởng đăng nhập; response không lộ password.

### Task 3 — Client API, đồng bộ user và điều hướng theo quyền

**Skills trước khi làm:** `executing-plans`, `ponytail`, `test-driven-development`, `vercel-react-best-practices`, `verification-before-completion`. Chạy lại GitNexus impact cho symbol auth/router trước khi đề xuất diff.

**Files:** Tạo hai API client và `usersApi.test.ts`, `AuthProvider.test.tsx`; chỉnh `access.ts`, `AppRouter.tsx`, `App.test.tsx`, `authApi.ts`, `AuthProvider.tsx`, `useAuth.ts`.

**Interfaces:** Các method API mục 4. Bổ sung `authApi.me(signal?: AbortSignal): Promise<User>` giữ `skipRefresh` như hiện tại. Bổ sung `AuthContextValue.reloadUser(signal?: AbortSignal): Promise<void>`: lấy lại `/auth/me`, chỉ cập nhật state nếu revision/user vẫn là phiên khởi tạo request. Bổ sung route `/profile` cho cả 3 role, dùng canAccess sẵn có.

- [ ] Viết test API gửi query/filter và body đúng, truyền AbortSignal, response 204 không parse JSON. Viết test reloadUser cập nhật displayName sau sửa chính mình và không đưa user cũ trở lại sau logout/login tài khoản khác.
- [ ] Viết test tích hợp route: ADMIN vào users; MANAGER/EMPLOYEE không thấy Users và truy cập URL nhận 403; ba role vào profile/change-password; guest về login, không gọi API user trước khi có phiên.
- [ ] Cài client và reloadUser tối thiểu; không chuyển sang categoryHttp hoặc viết interceptor mới.
- [ ] Thêm route bằng diff nhỏ trên AppRouter hiện tại, giữ OCR/chat/categories và thay đổi chưa commit. Sidebar tự xuất menu profile qua config; không thêm menu thứ hai ở header.
- [ ] Chạy `npm test -- src/features/users/api/usersApi.test.ts src/features/auth/context/AuthProvider.test.tsx src/app/App.test.tsx src/shared/api/http.test.ts` sau khi trang ở task 4/7 có thể render. Trong task này có thể dùng stub riêng trong test, không tạo trang giả trong source.

**Đầu ra:** Interface client/context sẵn dùng. Xác nhận route end-to-end phụ thuộc trang thật ở task 4/7; không đánh dấu hoàn thành tích hợp route trước mốc đó.

### Task 4 — Danh sách, tìm kiếm, bộ lọc và phân trang

**Skills trước khi làm:** `ui-ux-pro-max` trước thiết kế bố cục/trạng thái; `ui-styling`, `vercel-react-best-practices`, `test-driven-development` khi xây page; duy trì `executing-plans`, `ponytail`, `verification-before-completion`.

**Files:** Tạo `UsersPage.tsx`, `UsersPage.test.tsx`; chỉnh `styles.css`, nối route users trong AppRouter khi page đã có.

**Interfaces:** `UsersPage(): ReactElement`; dùng `usersApi.list`, `accountOptionsApi.departments`, `AccountQuery` của task 1/3.

- [ ] Viết test `combines_search_department_role_status_and_resets_page`: 4 điều kiện cùng gửi tới API, đổi bộ lọc/pageSize về page 1; xóa lọc trở về mặc định. Test debounce bằng fake timer hoặc handler đếm request có thời gian kiểm soát.
- [ ] Render bảng họ tên/email/đơn vị/chức vụ/role/trạng thái/thao tác; thanh tìm kiếm, 3 bộ lọc, nút xóa lọc; tổng số, trang hiện tại, previous/next, chọn 10/20/50.
- [ ] Tách loading ban đầu, đang tải lại, không có tài khoản, không có kết quả lọc và lỗi có retry. Dữ liệu cũ không bị gắn nhầm vào bộ lọc mới.
- [ ] Abort fetch cũ khi query thay đổi/unmount; thêm revision để bỏ response trễ. Test A chậm/B nhanh bảo đảm chỉ B hiển thị.
- [ ] Chuẩn bị callback refresh sau mutation: dùng query hiện tại; nếu page vượt trang cuối, tính `max(1, ceil(total/pageSize))` rồi fetch lại; không reset toàn bộ bộ lọc sau mỗi thao tác.
- [ ] Chạy `npm test -- src/features/users/pages/UsersPage.test.tsx src/app/App.test.tsx`; đạt khi navigation và danh sách đúng; kiểm tra keyboard/label bảng, viewport 375 px không làm tràn cả trang.

**Đầu ra:** Admin duyệt và lọc được dữ liệu mock nhiều trang, kể cả request chạy đua và kết quả rỗng.

### Task 5 — Form tạo và sửa tài khoản

**Skills trước khi làm:** `ui-ux-pro-max`, `ui-styling`, `vercel-react-best-practices`, `test-driven-development`; duy trì `executing-plans`, `ponytail`, `verification-before-completion`. Đặc biệt kiểm tra validation, focus dialog và giữ input khi lỗi.

**Files:** Tạo `UserFormDialog.tsx`, `UserFormDialog.test.tsx`; nối vào `UsersPage.tsx`, bổ sung CSS.

**Interfaces:** `UserFormDialog({ account, onClose, onSaved })`; `account: ManagedAccount | null` (null là tạo), `onClose(): void`, `onSaved(account: ManagedAccount): Promise<void> | void`. Dialog được mount khi mở và unmount khi đóng để không giữ password của lần trước.

- [ ] Viết test create/edit: dữ liệu mặc định đúng, edit không gửi initialPassword; thiếu tên/email/role/đơn vị theo role và password không khớp chặn submit; field đầu tiên sai được focus. Native validity kiểm tra cả input và select.
- [ ] Dùng `<dialog>` có heading/label và trả focus về nút mở. Tạo mới có initialPassword/confirm; sửa không hiện password; status chỉ thay qua action riêng.
- [ ] Load dropdown qua accountOptionsApi, hiển thị trạng thái tải/lỗi và retry; chưa tải đủ thì không cho submit dữ liệu không xác định. Chọn role không được tự đổi đơn vị/chức vụ âm thầm.
- [ ] Option inactive hiện có vẫn được hiển thị và giữ nguyên khi edit; không chọn inactive mới. Nếu chuyển role sang MANAGER/EMPLOYEE thì bắt buộc đơn vị active hoặc giá trị cũ hợp lệ theo contract.
- [ ] Xử lý 409 EMAIL_EXISTS tại trường email, không đóng form/mất dữ liệu; disable khi pending, chặn submit hai lần; abort khi unmount và không refetch sau request đã bị hủy.
- [ ] Sau save, gọi onSaved để refetch list; nếu sửa chính mình, gọi auth.reloadUser trước khi báo hoàn tất đồng bộ. Nếu reload lỗi, thông báo đã lưu nhưng chưa tải lại thông tin phiên, không gửi mutation lần hai.
- [ ] Chạy `npm test -- src/features/users/components/UserFormDialog.test.tsx src/features/users/pages/UsersPage.test.tsx`; test tạo thành công rồi dữ liệu không khớp filter phải có thông báo thành công dù hàng không hiện trên trang hiện tại.

**Đầu ra:** Tạo/sửa đầy đủ, dropdown không phụ thuộc server thật và xử lý xung đột không mất input.

### Task 6 — Vô hiệu hóa, kích hoạt và reset có xác nhận

**Skills trước khi làm:** `ui-ux-pro-max`, `ui-styling`, `vercel-react-best-practices`, `test-driven-development`; duy trì `executing-plans`, `ponytail`, `verification-before-completion`. Nếu quyền/trạng thái hoặc mutation sai, dùng `systematic-debugging`.

**Files:** Tạo `UserActionDialog.tsx`, `UserActionDialog.test.tsx`; nối hành động vào UsersPage và test list.

**Interfaces:** `UserAction = 'deactivate' | 'activate' | 'resetPassword'`; `UserActionDialog({ action, account, onClose, onDone })`, với `onDone(): Promise<void> | void`; `action` và `account` bắt buộc khi dialog được mount.

- [ ] Viết test Cancel không gọi API; xác nhận chỉ gửi một request; mỗi dialog nêu đúng tên/email và hậu quả; Escape/đóng trả focus về trigger.
- [ ] Deactivate/activate có nút hủy/xác nhận; reset thêm newPassword/confirm và mô tả user phải đăng nhập lại. Không hiển thị password ở bảng/thông báo thành công.
- [ ] Ẩn/khóa hành động đúng mục 3, gồm self action; mock vẫn từ chối khi gọi trực tiếp. Khóa đóng/gửi lại trong lúc mutation pending hoặc abort và bỏ phản hồi nếu dialog thực sự unmount.
- [ ] Thành công mới cập nhật/refetch list, giữ query hiện tại. Test vô hiệu hóa dưới filter ACTIVE làm hàng biến mất, total giảm và trang cuối tự lùi; lỗi 403/409/5xx không giả thành công.
- [ ] Xóa password khi đóng/thành công/unmount; gọi resetPassword không đưa confirmPassword vào body. Test ACTIVE/INACTIVE không có nút đối nghịch sai trạng thái.
- [ ] Chạy `npm test -- src/features/users/components/UserActionDialog.test.tsx src/features/users/pages/UsersPage.test.tsx src/mocks/users.test.ts`.

**Đầu ra:** Admin hoàn thành vòng đời tài khoản qua UI với xác nhận và kết quả phản ánh đúng server mock.

### Task 7 — Hồ sơ cá nhân và hoàn thiện đổi mật khẩu

**Skills trước khi làm:** `ui-ux-pro-max`, `ui-styling`, `vercel-react-best-practices`, `test-driven-development`; duy trì `executing-plans`, `ponytail`, `verification-before-completion`. Review riêng rò dữ liệu user cũ và thông báo sau khi phiên kết thúc.

**Files:** Tạo `ProfilePage.tsx`, `ProfilePage.test.tsx`; chỉnh AccountPage/AppRouter/App.test để hoàn thiện thông báo.

**Interfaces:** `ProfilePage(): ReactElement` dùng `usersApi.me(signal)`; không nhận id user từ URL. Giữ `AccountPage mode="change"` và `authApi.change`.

- [ ] Viết test mỗi role thấy đúng email/họ tên/role/đơn vị/chức vụ/trạng thái của mình; Admin không đơn vị hiển thị “Chưa có”; request không gửi id tùy ý.
- [ ] Render hồ sơ chỉ đọc, loading/error/retry, link `/change-password`; không dùng danh sách quản trị để tìm hồ sơ của user thường.
- [ ] Abort request khi rời trang; test response cũ đến sau logout/chuyển user không lộ hồ sơ trước.
- [ ] Bổ sung test UI đổi mật khẩu: sai mật khẩu hiện tại, confirm lệch, giới hạn 8–128; thành công thu hồi phiên và về login với “Đổi mật khẩu thành công. Vui lòng đăng nhập lại.”; password không còn trong DOM.
- [ ] Hoàn thiện thông báo 401 trên trang đích bằng route state trong Guard/AccountPage, chỉ cho trạng thái phiên đã hết hạn; guest mới truy cập login không bị báo hết phiên. Không thêm refresh riêng ở page.
- [ ] Chạy `npm test -- src/features/users/pages/ProfilePage.test.tsx src/app/App.test.tsx src/features/auth/context/AuthProvider.test.tsx src/shared/api/http.test.ts`. Chạy lại route test còn phụ thuộc trang thật từ task 3.

**Đầu ra:** Ba role truy cập hồ sơ của chính mình, đổi mật khẩu và hiểu vì sao cần đăng nhập lại.

### Task 8 — Nghiệm thu mock và chuẩn bị nối API thật

**Skills trước khi làm:** `web-design-guidelines`, `requesting-code-review`, `verification-before-completion`; tiếp tục `executing-plans`, `ponytail`. Dùng `playwright` hoặc `computer-use:computer-use` tùy cách kiểm tra browser; `receiving-code-review` khi nhận finding, `systematic-debugging` khi có lỗi. Chỉ dùng `finishing-a-development-branch` nếu đã có nhánh triển khai cần bàn giao.

**Files:** Tạo acceptance notes; cập nhật README/auth-contract về fixture mở rộng, cách demo và giới hạn bộ nhớ. Chỉ chỉnh API mapping/config nếu đã có endpoint BE thật và yêu cầu triển khai tương ứng.

- [ ] Chạy toàn bộ `npm run check` tại front-end; yêu cầu lint 0 warning, format, Vitest và production build đều thành công. Docs đang nằm trong `.prettierignore`: kiểm tra riêng plan/contract khi cần bằng config phù hợp, không sửa ignore chỉ để làm xanh check.
- [ ] Chạy ứng dụng với MSW mà không bật category-service; demo tạo → sửa → khóa → kích hoạt → reset → login mật khẩu mới → xem hồ sơ → đổi mật khẩu. Đăng xuất/đăng nhập giữa các actor vì mock có một phiên active.
- [ ] Nghiệm thu đủ 3 role: menu desktop/mobile, URL trực tiếp, nút, gọi API trái quyền; keyboard Tab/Enter/Escape, focus dialog/error, loading/empty/retry.
- [ ] Chủ động tạo 401 (refresh thành công và thất bại), 403 (từng action), 409 (trùng email/trạng thái đã đổi), mạng lỗi; ghi kết quả thực tế và giới hạn chưa kiểm tra.
- [ ] Dùng `web-design-guidelines` audit các page/dialog thực tế; ghi finding theo file/dòng, xử lý vấn đề accessibility và hành vi trước khi nghiệm thu.
- [ ] Dùng `requesting-code-review` review toàn bộ diff và Review Focus; xử lý góp ý qua `receiving-code-review`, test hồi quy các sửa đổi bằng `test-driven-development`. Ghi rõ người/công cụ review và phạm vi, không đánh dấu review khi chỉ có checklist dự kiến.
- [ ] Rà bảng skill A/B và ledger: skill bắt buộc có bằng chứng; skill có điều kiện đã được đánh giá và ghi lý do nếu không áp dụng. Không bổ sung tính năng ngoài ticket chỉ để kích hoạt một skill.
- [ ] Khi BE sẵn sàng: đối chiếu path/method/shape list, page 0 hay 1, ID đơn vị/chức vụ, enum trạng thái, lỗi, policy mật khẩu và cơ chế reset. Map ở usersApi/accountOptionsApi/authApi; sửa test contract theo contract đã thống nhất, giữ UI interface ổn định.
- [ ] Chạy staging với `VITE_ENABLE_MOCKS=false` và `VITE_API_BASE_URL` đúng; nếu options nằm ở service khác thì cấu hình mapping tập trung, không mang token giả `Bearer ADMIN` sang API thật. Kiểm tra refresh cookie/CORS/credentials/CSRF và thu hồi phiên bằng backend thật.
- [ ] Phân biệt mốc “FE hoàn thành trên mock” và “đã tích hợp BE”; chỉ đánh dấu mốc thứ hai khi smoke test thật đạt. Trước commit tích hợp chạy `gitnexus detect-changes -r document-management-frontend` và review phạm vi thay đổi.

**Đầu ra:** Bằng chứng nghiệm thu mock, hướng dẫn chạy và danh sách kiểm tra tích hợp thật; không tuyên bố BE đã chạy khi chưa có API.

## 7. Thứ tự phụ thuộc và tiêu chí hoàn thành

Task 1 → task 2 → task 3 → task 4 → task 5 → task 6 → task 7 → task 8. Route users chỉ nối khi task 4 có page; route profile chỉ nối khi task 7 có page; test tích hợp task 3 được hoàn tất ở các mốc này để tránh import trang chưa tồn tại.

| Checklist gốc                                      | Task chịu trách nhiệm | Bằng chứng nghiệm thu                                                          |
| -------------------------------------------------- | --------------------- | ------------------------------------------------------------------------------ |
| List + pagination/search/filter đơn vị/role/status | 1–4                   | Query kết hợp đúng, total/page đúng, không bị response cũ ghi đè               |
| Tạo/sửa, dropdown đơn vị/chức vụ                   | 1–3, 5                | Lưu thành công, validation/409 rõ, user mới login được                         |
| Vô hiệu hóa/kích hoạt/reset có xác nhận            | 2, 6                  | Cancel không gửi, confirm cập nhật thật, mật khẩu cũ bị từ chối                |
| Hồ sơ cá nhân/đổi mật khẩu                         | 3, 7                  | Cả 3 role thấy đúng hồ sơ mình; đổi xong đăng nhập lại                         |
| Thông báo 401/403/409                              | 2–7                   | Refresh không lặp; không mất thông báo khi chuyển login; giữ form khi conflict |
| Menu/nút đúng role, hoàn thành vòng đời            | 2–8                   | UI + URL guard + kiểm tra quyền API, check toàn bộ pass                        |

Ước lượng tham khảo cho một developer quen dự án: contract/mock 1–1,5 ngày; client/list 1 ngày; form/actions 1–1,5 ngày; hồ sơ và hồi quy/nghiệm thu 1 ngày. Tổng khoảng 4–5 ngày làm việc cho FE mock, đã gồm test; chưa gồm API BE, thay đổi nghiệp vụ và thời gian review của team. Ước lượng được cập nhật sau khi review các đề xuất mục 2.

## 8. Tự kiểm tra kế hoạch

- [x] Năm mục checklist gốc và yêu cầu phân quyền đều có task và bằng chứng nghiệm thu.
- [x] Interfaces/types/payload giữa client, UI và mock dùng chung tên; password/confirm không lọt vào response.
- [x] Năm trường hợp Review Focus đều gắn test ở task sở hữu logic.
- [x] Tận dụng route guard, HTTP interceptor, MSW, form/dialog native; không thêm dependency.
- [x] Ghi rõ graph cũ/UNKNOWN, source có thay đổi sẵn, quy tắc chỉ cung cấp diff và các giả định cần review.
- [x] Không coi kế hoạch, test dự kiến hoặc mock thành công là bằng chứng chức năng đã triển khai/API thật đã tích hợp.
- [x] Có checklist skill xuyên suốt, skill có điều kiện, lời nhắc ở từng task và yêu cầu ghi bằng chứng khi thực thi; các skill đã áp dụng trong preview được tick; skill có điều kiện không áp dụng được giải thích trong ledger.

## 9. Kết quả thực thi

Đã thực hiện task 1–8 cho phạm vi mock trong `output/account-management-preview` tại repo root, bàn giao patch và bằng chứng trong `front-end/docs/implementation/account-management`. Có 64 test đạt cùng lint/format/build, kiểm tra desktop/mobile và một lượt review độc lập; lỗi token mock được sửa và kiểm tra hồi quy. Các checkbox bước còn để trống không được hiểu là đã áp dụng vào source gốc hoặc đã tích hợp BE. Chưa có branch/commit/merge/push; worktree/computer-use/design-system/composition/subagent implementation không áp dụng cho luồng bàn giao này.

### Tích hợp đã được cho phép và hoàn tất

Người dùng đã xác nhận áp dụng trực tiếp. Patch 35 file được kiểm tra và áp dụng thành công vào `front-end/`, giữ sửa AppRouter có sẵn. `npm run check` tại checkout gốc: 12 file test / 64 test đạt, lint 0 warning, format và production build đạt. Không tạo nhánh hay commit/push. GitNexus detect-changes đã thử nhưng lỗi do đăng ký index trỏ checkout cũ; không ghi nhận bước này là đã đạt. Các bước API thật vẫn chờ backend theo phạm vi ticket.
