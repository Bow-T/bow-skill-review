---
name: pr-review
version: "1.0.0"
description: Review một PR GitHub (theo số/URL) hoặc diff branch local. Đọc thay đổi qua gh pr diff / git diff, nhận xét có cấu trúc (correctness, bảo mật, reuse/đơn giản hoá, test coverage), đối chiếu ticket Jira nếu PR nhắc key, rồi khi người dùng duyệt thì comment / approve / request-changes qua gh pr. Dùng trong Reviewer Mode (read-only code + git/gh). KHÔNG sửa code, merge hay push.
argument-hint: "<PR#-hoặc-URL, hoặc 'local' để review diff branch hiện tại>"
allowed-tools: Read, Glob, Bash, mcp__jira__jira_get_issue, mcp__jira__jira_get_comments, mcp__jira__jira_search_issues, AskUserQuestion
user-invocable: true
---

# /pr-review — Reviewer đọc PR/diff, nhận xét, comment/approve

Review một pull request (GitHub) hoặc diff branch local, đưa nhận xét có cấu trúc, rồi — khi
người dùng duyệt — ghi review lên PR (comment / approve / request-changes). Dành cho **Reviewer
Mode**: đọc code + chạy `git`/`gh` (đọc + `gh pr comment`/`gh pr review`), KHÔNG sửa code /
merge / push. Nếu chỉ muốn xem nhận xét mà không ghi lên PR, cứ dừng ở Bước 3.

## Đầu vào

- Số/URL PR: `42`, `#42`, hoặc `https://github.com/org/repo/pull/42`. Bóc lấy số PR.
- Hoặc `local` / "diff hiện tại": review thay đổi của branch đang checkout so với nhánh gốc.
- Không rõ → hỏi lại đúng 1 câu (PR nào? hay review diff local?).

## Bước 1 — Lấy thay đổi

- **PR GitHub**: `gh pr view <n> --json title,body,author,baseRefName,headRefName,files,url` để
  lấy metadata; `gh pr diff <n>` để lấy diff. Cần trạng thái CI: `gh pr checks <n>`.
- **Diff local**: `git diff <base>...HEAD` (đoán `<base>` = `main`/`master`/`develop` qua
  `git log`/`git status`; nếu mơ hồ, hỏi nhánh gốc). `git diff --stat` để nắm phạm vi trước.
- Đọc file liên quan bằng `Read`/`Glob` khi diff chưa đủ ngữ cảnh (hàm gọi tới, test tương ứng).
- Nếu tiêu đề/nhánh/body PR nhắc key Jira (`[A-Z][A-Z0-9]+-\d+`), `mcp__jira__jira_get_issue` để
  đối chiếu PR có làm ĐÚNG yêu cầu ticket không.

## Bước 2 — Nhận xét có cấu trúc

Soi diff theo các trục (chỉ nêu điều THẬT thấy trong diff, trích dòng/file cụ thể — không phán chung):

- **Correctness**: lỗi logic, off-by-one, null/なし, race, xử lý lỗi thiếu, edge case bỏ sót.
- **Bảo mật**: input chưa kiểm, secret lộ, injection, quyền/authz, dữ liệu nhạy cảm log ra.
- **Reuse / đơn giản hoá**: code lặp lại thứ đã có, chỗ có thể gọn hơn, trừu tượng sai chỗ.
- **Test coverage**: thay đổi có test kèm không; nhánh nào chưa được phủ; test có thật sự kiểm hành vi.
- **Khớp convention repo**: đặt tên, cấu trúc, pattern của dự án (đọc CLAUDE.md/quy ước nếu có).

In báo cáo gọn:

```
🔍 PR #<n> — <title>   (hoặc: diff local <base>...HEAD)
Phạm vi     : <x file, +A/-B dòng>  | CI: <pass/fail/none>
Ticket      : <KEY nếu có> — PR khớp yêu cầu? <có/lệch chỗ nào>
Nhận xét    :
  • [correctness] <file:line> — <vấn đề + vì sao>
  • [bảo mật]     …
  • [test]        …
Kết luận    : ✅ Approve | 💬 Comment (góp ý) | 🔧 Request changes (có blocker)
```

Bám bằng chứng: mỗi gạch đầu dòng trích `file:line` hoặc đoạn diff thật. Thiếu ngữ cảnh để chắc →
nói rõ "chưa chắc, cần xem thêm X", đừng phán bừa.

## Bước 3 — HỎI DUYỆT trước khi ghi lên PR

**Không tự ý ghi lên GitHub.** Dùng `AskUserQuestion` (hoặc hỏi thẳng) cho người dùng chọn:

- **Không ghi** — chỉ giữ nhận xét tại chỗ (mặc định nếu là diff local, hoặc khi chưa chắc).
- **Comment góp ý** — `gh pr comment <n> --body "…"` (không đổi trạng thái review).
- **Approve** — `gh pr review <n> --approve --body "…"` (chỉ khi kết luận ✅ và bạn thật sự chắc).
- **Request changes** — `gh pr review <n> --request-changes --body "…"` (khi có blocker rõ).

Diff local KHÔNG có PR để ghi → chỉ in nhận xét, không có bước ghi.

## Bước 4 — GHI review (khi đã duyệt)

- Soạn body review từ báo cáo Bước 2 (tiếng Việt, gọn, có `file:line`). Nêu rõ đây là review qua
  bow-agent Reviewer Mode.
- Chạy đúng một lệnh `gh pr comment`/`gh pr review` theo lựa chọn. Body dài để trong 1 tham số
  `--body "…"` (KHÔNG nối lệnh, KHÔNG dùng `$(...)` — Reviewer Mode chặn command-chaining).
- Nếu lệnh lỗi (quyền, PR đã đóng…), báo rõ lỗi, KHÔNG thử lại mù — để người dùng quyết.

## Nguyên tắc

- **Read-only code**: không bao giờ sửa file, `git commit/push`, `gh pr merge`. Chỉ đọc + review.
- **Duyệt trước khi ghi**: approve/request-changes là quyết định nặng — luôn để người dùng chốt.
- **Bám bằng chứng**: mọi nhận xét trích từ diff/file thật.
- **Tối thiểu & trung thực**: ưu tiên comment góp ý; chỉ Request changes khi có blocker rõ ràng;
  thiếu chắc chắn thì nói thẳng thay vì Approve/Reject bừa.
