# bow-skill-review

Bộ skill **Reviewer** cho [bow-agent](https://github.com/Bow-T/bow-agent). Đăng ký như một
*stack* trong registry và nạp **tự động** khi bow-agent chạy ở **Reviewer Mode**
(`npm run ui:review:share` / `BOW_REVIEWER_MODE=true`).

Reviewer Mode là read-only với code nhưng cho chạy `git`/`gh` (đọc PR + `gh pr comment`/`gh pr
review`) + test/analyze + Jira đọc, nên skill ở đây review được PR và (khi duyệt) ghi review lên
GitHub.

## Skill

| Skill | Việc | Ghi GitHub? |
|-------|------|-------------|
| `pr-review` | Đọc PR (GitHub) hoặc diff branch local, nhận xét có cấu trúc (correctness / bảo mật / reuse / test coverage), đối chiếu ticket Jira; khi duyệt thì comment / approve / request-changes qua `gh pr`. | Có — nhưng LUÔN hỏi duyệt trước. Không merge/push/sửa code. |

## Cấu trúc

```
bow-skill.json          # manifest: id=review, skillsDir=skills, danh sách skill
skills/
  pr-review/SKILL.md
```

## Ghim phiên bản

bow-agent chỉ tải theo **tag đã ghim** trong registry (`~/.bow-agent/registry.json`). Đổi nội
dung → tạo tag mới (vd `v1.1.0`) rồi cập nhật `ref`; bản đã duyệt là bất biến.
