# Workflow Skills — PO → BA → Design → Dev → Test

Bộ skill chuẩn để Claude Code / Codex / AI assistant tự động sinh tài liệu và code đúng cấu trúc của team. Mỗi skill là một folder chứa `SKILL.md` với frontmatter `name` + `description` để AI tự trigger khi user mô tả task phù hợp.

## Triết lý

> **Skill = bộ "luật chơi" cho AI.** Khi user yêu cầu một task, AI đọc description trong frontmatter để biết có nên dùng skill nào, sau đó đọc body skill để làm theo đúng quy chuẩn — không cần user phải nhắc đi nhắc lại "nhớ đặt file ở đâu", "frontmatter có gì", "format thế nào".

## Cách cài đặt

### Cho Claude Code
Đặt thư mục này vào `.claude/skills/` ở root repo:

```
<repo-root>/
└── .claude/
    ├── CLAUDE.md           # working agreement (xem WORKFLOW-STANDARD.md)
    └── skills/
        ├── feature-prd-author/SKILL.md
        ├── feature-srs-author/SKILL.md
        └── ...
```

Claude Code tự động phát hiện và liệt kê trong `available_skills`.

### Cho Codex CLI / OpenAI tools
Đặt vào `.codex/skills/` hoặc thư mục Codex chỉ định, kèm `AGENTS.md` ở root.

### Cho Claude.ai (web/desktop)
- Cá nhân: upload từng skill qua Settings → Capabilities → Skills.
- Team: admin upload vào workspace.

### Cho Cowork
Skill cùng cấu trúc, đặt vào project knowledge.

## Shopify platform skills (đặc thù nền tảng)

Bổ sung cho luồng SDLC chung — cover các mảng bắt buộc khi build app Shopify thật (dùng `shopify-dev-mcp` để tra API chuẩn 2025-07).

| Mảng | Skill | Khi nào dùng |
|---|---|---|
| Webhooks + GDPR | `feature-shopify-webhooks` | 3 compliance webhook bắt buộc, app/uninstalled, HMAC, idempotency |
| Billing | `feature-shopify-billing` | subscription/usage/trial; App Pricing vs Billing API; off-platform = cấm |
| Extensions | `feature-shopify-extension-impl` | theme app ext, checkout UI, admin action/block, Functions + deploy |
| App Store | `feature-shopify-appstore-checklist` | audit điều kiện submit (PASS/FAIL) trước khi lên store |

## Mapping vai trò → skill

| Vai trò | Skill chính | Output |
|---|---|---|
| **PO** | `feature-prd-author` | `docs/phases/<phase-slug>/00-prd.md` (1 PRD = 1 phase, multi-features) |
| **BA** | `feature-srs-author` | `01-srs.md` (user stories + Gherkin AC ở §11) |
| **Designer** | `feature-design-spec-author` | `02-design-spec.md` |
| **Tech Lead** | `feature-tech-design-author` | `03-tech-design.md` (Prisma schema §3, API contract §11) |
| **FE Dev** | `feature-frontend-impl` | Code FE trong `app/modules/<slug>/` |
| **BE Dev** | `feature-backend-impl` | Code BE trong `app/services/` + `app/api/v1/<slug>/` + migration |
| **QA** | `feature-test-plan-author` | `04-test-plan.md` |
| **Release Manager** | `feature-release-author` | `05-release-plan.md` |
| **Tech Lead / Doc Maintainer** | `feature-docs-sync` | sync `docs/system-architecture.md`, `codebase-summary.md`, ... sau khi feature ship |
| **Tech Lead / BA / QA** | `feature-doc-consistency-check` | audit SRS §5 ↔ Tech Design §3 ↔ Zod ↔ routes; output report `plans/reports/doc-consistency-*.md` |
| **Tech Lead / Architect** | `feature-adr-author` | `docs/_adr/<NNNN>-<slug>.md` (Michael Nygard format, immutable) |
| **SRE / On-call** | `feature-incident-postmortem` | `docs/_incidents/<YYYY-MM-DD>-<slug>.md` (blameless 5-whys + action items) |
| **Tech Lead / Doc Maintainer** | `feature-bugfix-doc-sync` | weekly catch-all: scan `fix:` commits → propose missed doc bumps |

## Workflow tổng thể

```
PO     → feature-prd-author       → docs/phases/<phase-slug>/00-prd.md  (multi-features)
   ↓
BA     → feature-srs-author       → 01-srs.md  (user stories + Gherkin AC merged ở §11)
   ↓
Design → feature-design-spec-author → 02-design-spec.md
   ↓
TL     → feature-tech-design-author → 03-tech-design.md  (API contract = §11 summary table)
   ↓
FE Dev → feature-frontend-impl    → code FE
BE Dev → feature-backend-impl     → code BE + migration
   ↓
QA     → feature-test-plan-author → 04-test-plan.md
   ↓
RM     → feature-release-author   → 05-release-plan.md
   ↓ (sau khi feature ship)
TL     → feature-docs-sync        → cập nhật system-architecture / codebase-summary / roadmap / code-standards
```

## Cách dùng — ví dụ thực tế

### Ví dụ 1: PO bắt đầu một phase mới
```
User: "Tôi muốn mở phase 2 — example feature group, gồm product-page,
       cart-page, popup placements, giúp merchant cross-sell"

→ Claude tự trigger skill feature-prd-author
→ Hỏi phase_slug (vd phase-2-example-b), KPI, persona, list sub-features
→ Tạo docs/phases/phase-2-example-b/00-prd.md với 8 mục chuẩn
   (Section 8 liệt kê sub-features: example-feature-a, ...)
```

### Ví dụ 2: BA viết SRS cho 1 sub-feature trong phase
```
User: "PRD phase-2 đã approved, viết SRS cho example-feature-b"

→ Claude tự trigger skill feature-srs-author
→ Resolve <phase-slug> bằng grep slug trong docs/phases/*/00-prd.md
→ Đọc docs/phases/phase-2-example-b/00-prd.md, xác nhận open questions
→ Sinh docs/phases/phase-2-example-b/features/example-feature-b/01-srs.md
   (11 mục, user stories + Gherkin AC merged inline ở Section 11)
```

### Ví dụ 3: FE Dev triển khai
```
User: "Code component CampaignForm theo docs example-feature-b"

→ Claude tự trigger skill feature-frontend-impl
→ Đọc 00-prd.md, 01-srs.md, 02-design-spec.md, 03-tech-design.md (§11 endpoints), 04-test-plan.md (§4 TC-IDs)
→ Dùng Figma MCP đọc frame "example-feature-b / S-02 - Create / Default"
→ Code component theo file structure ở Tech Design
→ Viết test song song
→ Tạo MR description đầy đủ
```

### Ví dụ 4: QA sinh test plan
```
User: "Sinh test cases từ SRS này, cần import được vào Jira Xray"

→ Claude tự trigger skill feature-test-plan-author
→ Đọc 01-srs.md (§3 business rules, §5 data model, §6 state, §7 runtime, §8 QA, §11 user stories Gherkin), 02-design-spec.md, 03-tech-design.md (§3 schema, §11 endpoints)
→ Build coverage matrix
→ Sinh 04-test-plan.md với 50-150 test cases
→ Convert sang CSV format Jira Xray
```

## Quan hệ giữa các skill

```
feature-prd-author          (1 phase, multi sub-features)
    ↓ (output làm input cho)
feature-srs-author          (per-feature; resolve <phase-slug> qua grep PRD)
    ↓
feature-design-spec-author  (đọc SRS §3/§5/§11 + Figma)
    ↓
feature-tech-design-author  (đọc SRS §5 → schema; PRD §scope; output §11 endpoints)
    ↓
feature-frontend-impl ──┐
feature-backend-impl ───┘   (đọc SRS + Tech Design + Test Plan §4 TC-IDs)
    ↓
feature-test-plan-author    (đọc SRS + Design Spec + Tech Design)
    ↓
feature-release-author      (đọc Test Plan §9 exit criteria)
    ↓ (sau khi feature ship)
feature-docs-sync           (cập nhật project-level docs)

Cross-cutting / hygiene (chạy on-demand):
- feature-doc-consistency-check  — audit drift trước khi bump status
- feature-adr-author             — record cross-feature decisions
- feature-incident-postmortem    — sau SEV-1/2/3 production incidents
- feature-bugfix-doc-sync        — weekly cron: catch fix: commits chưa bump doc
```

## Git hooks (enforce)

`scripts/install-git-hooks.sh` cài 1 commit-msg hook:
- Block commit nếu staged files match `app/services/`, `app/api/`, `app/modules/`, `app/schemas/`, `prisma/schema*` mà không stage doc kèm hoặc thiếu trailer `Docs-bumped:` / `Docs-impact: none — <lý do>`.
- Bypass: `DOCS_BUMP_SKIP=1 git commit ...` hoặc `git commit --no-verify` (CI / emergency).
- Cần chạy 1 lần sau clone: `bash scripts/install-git-hooks.sh`.

## Quy tắc viết description (cho việc tạo skill mới)

Description trong frontmatter có 2 nhiệm vụ:
1. **WHAT**: Skill làm gì (1 câu đầu)
2. **WHEN to trigger**: Liệt kê các phrase user thường dùng để AI nhận ra

**Pushy** ở đây có nghĩa là liệt kê đa dạng phrase, kể cả khi user không nói chính xác tên skill. Ví dụ: skill viết PRD nên trigger khi user nói "describe a new feature" hoặc "ghi lại yêu cầu" chứ không chỉ khi nói "viết PRD".

## Customization cho team của bạn

Mỗi team có conventions riêng. Để adapt:

1. **Đổi file structure trong tech-design-author** — nếu repo của bạn dùng layered architecture khác (Clean Architecture, Hexagonal, MVC, ...).
2. **Đổi enum status** — nếu Jira workflow của bạn có status khác.
3. **Đổi naming convention Figma** — nếu team đã có convention khác.
4. **Đổi tech stack default** — sửa skill `feature-frontend-impl` / `feature-backend-impl` để khớp stack thực tế (Next.js / Vue / Laravel / Spring Boot...).
5. **Thêm skill mới** — ví dụ `feature-i18n-author`, `feature-analytics-spec-author`, `feature-security-review`...

## Đo hiệu quả

Sau 2 sprint dùng skill này, đo:

- **Thời gian từ PRD → SRS approved**: kỳ vọng giảm 40-60%
- **Số lần Dev hỏi lại spec**: kỳ vọng giảm xuống gần 0
- **Số bug do "spec không rõ"**: kỳ vọng giảm 70%+
- **Onboarding time cho member mới**: kỳ vọng giảm 50%

## Câu hỏi thường gặp

**Q: Skill có override CLAUDE.md không?**
A: Không. CLAUDE.md là working agreement chung. Skill là playbook cho từng loại task cụ thể. Cả hai đều được AI consume.

**Q: Nếu task không khớp skill nào?**
A: AI làm bình thường theo CLAUDE.md. Skill chỉ trigger khi description khớp.

**Q: Có thể chạy 2 skill cùng lúc không?**
A: Có. Ví dụ task "viết SRS và test plan luôn" → AI có thể trigger cả `feature-srs-author` rồi `feature-test-plan-author`.

**Q: Skill có version không?**
A: Nên thêm vào commit message khi update SKILL.md, hoặc dùng tag `skill/feature-srs-author@v1.2`.

**Q: Làm sao biết skill có trigger đúng không?**
A: Trong Claude Code, gõ `/skills` để xem skill nào available. Trong Claude.ai, AI sẽ nói rõ "I'll use the X skill to ...".

## Bảo trì

- Owner mỗi skill nên là 1 senior trong vai trò tương ứng (ví dụ skill BA do BA Lead maintain).
- Review skill mỗi quarter để cập nhật theo conventions mới.
- Khi feature pilot xong → retrospective → cập nhật skill.

## Xem thêm

- `docs/_conventions/WORKFLOW-STANDARD.md` — quy chuẩn workflow tổng.
- `docs/_conventions/naming.md` — naming convention chi tiết.
- `docs/_conventions/frontmatter.md` — schema frontmatter chuẩn.
- `docs/_ai-prompts/` — prompt mẫu cho từng giai đoạn (bổ sung cho skill).

---

**Bộ skill này là living document.** Mỗi lần team gặp tình huống mới mà skill chưa cover → bổ sung vào skill. AI càng có nhiều context → output càng bám sát mong đợi.
