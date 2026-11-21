# Agent guide

ChronoFrame is a photo management and gallery pnpm monorepo built with Nuxt 4,
Vue 3, TypeScript, Nitro and Tailwind CSS. Read the affected application's or
package's README before editing; this guide describes shared repository rules.

## Fork 上游同步规范（fivk-custom）

本节是**唯一且自包含的权威规则**，与所用工具无关，本仓库不维护第二份规则文件。
无论使用哪种 AI 或命令行，只要任务涉及「同步官方更新 / 合并上游 / 处理 fork
合并冲突」，都必须遵守，无需用户重复说明，也无需读取任何其他文件。无文件系统
访问的网页版 AI，直接把本节整段复制粘贴给它即可。

本仓库是 ChronoFrame 的个人 fork：`origin` = fac5/chronoframe，`upstream` =
HoshinoSuzumi/chronoframe（官方）。

- 分支约定：`main` 是 `upstream/main` 的纯镜像，禁止任何本地修改；
  `fivk-custom` 是长期维护的个性化分支（日常开发只在此分支）；`feat/*` 从 `main`
  单独开出用于向官方提 PR，禁止携带任何个性化改动。
- 同步流程（只 merge，不 rebase）：`git fetch upstream` → 在 `main` 上
  `git merge --ff-only upstream/main` 并推送 `origin/main` → 切到 `fivk-custom`
  执行 `git merge main`（ours = 我的定制，theirs = 官方）→ 解冲突 →
  `pnpm install --frozen-lockfile && pnpm lint && pnpm test`。
- 措辞消歧：用户说「同步 main 的最新版本」「更新 main」「拉最新」「同步一下」
  「把官方最新版合过来」等含糊说法时，**一律理解为上述完整单向同步**
  （`upstream/main → main → fivk-custom`）。禁止把 `fivk-custom` 合回 `main`、
  禁止只更新 `main` 就结束、禁止用 `origin/main` 代替 `upstream/main` 作为来源、
  禁止因为提到 main 就切到 `main` 上开发。只有用户明确说「只更新 main 镜像」
  时才允许仅执行第一步。
- 冲突规则一：个人信息/个性化内容（备案号、社交与项目链接、作者署名、Docker
  镜像名、fork 声明、术语偏好等）→ 保留我方（`--ours`）。
- 冲突规则二：功能/逻辑类冲突 → 优先采用官方实现（`--theirs`），再把必要的个性化
  配置贴回；同一文件两类改动混杂时逐处判断，禁止整文件覆盖。典型混合文件：
  `Footer.vue`、`package.json`、i18n 语言包、`AGENTS.md`（本文件自身）。
- 冲突规则三：无冲突但官方新增功能与本地自定义功能重复 → 删除本地重复实现、
  采用官方版本。已知待办：官方合并 `feat/photo-date-edit`（照片拍摄时间编辑）
  的 PR 后，必须删除 `fivk-custom` 里的本地实现（`9e7cf8e1`、`d5e7a8c9`）。
- 禁止 rebase：它会反转 ours/theirs 语义、重写已推送历史并制造重复提交。
- 安全边界：合并后先汇报「采用了哪些官方实现、保留了哪些个性化内容、有无遗留
  冲突」，并人工核对备案号、作者署名、Docker 镜像名是否还在；**未经用户明确
  确认不要 `git push`**。

## Repository map

### Application entries

- `apps/self-hosted`: complete Node/Docker application, composing Console and
  Gallery with onboarding, authentication, API, SQLite, storage and embedded-job
  adapters. Root development and build commands target this entry.
- `apps/cloud`: Cloudflare Workers marketing, authentication and platform portal.
- `apps/site`: independent Cloudflare Workers public gallery entry; extends
  Gallery, not Console or SaaS UI.
- `apps/worker`: standalone native photo processing node that claims tasks from
  Cloud, sends heartbeats and submits results. This is not an edge Worker.
- `apps/shared`: shared application adapters and platform wiring.
- `apps/admin`: separate Git submodule, excluded from the root pnpm workspace.
  Do not assume it is initialized or add workspace dependencies without checking
  its independent repository setup.

### Shared packages

- `packages/gallery`: public gallery Nuxt layer, viewer, composables, state and
  translations. Its `shared` directory includes compatibility presentation types.
- `packages/console`: reusable photo management dashboard Nuxt layer.
- `packages/saas-ui`: opt-in Cloud/Admin UI primitives, semantic tokens and shell;
  do not introduce its styling into Self-hosted or Site.
- `packages/contracts`: browser-compatible schemas, DTOs and named contract
  exports; no database models, Nuxt globals, Node globals or runtime services.
- `packages/core`: framework-independent business use cases. Applications inject
  repositories, trusted access context and infrastructure capabilities.
- `packages/database`: canonical Self-hosted SQLite schema, migrations, DTO
  projections and repositories.
- `packages/platform`: edge platform D1 persistence for identity, sessions and
  Workspace/Site directory data; separate from Self-hosted SQLite.
- `packages/jobs`: durable embedded job execution, claims, leases and retries.
- `packages/processing`: framework-independent photo processing orchestration.
- `packages/storage`, `packages/media`, `packages/geocoding`: storage providers,
  media capabilities and reverse geocoding.
- `packages/webgl-image`: standalone hardware-accelerated image viewer.

`tests` contains repository and integration tests; `scripts` contains launchers,
maintenance and architecture checks; `docs` contains English and Chinese
VitePress documentation. Root `.nuxt`, `.output`, `node_modules` and local `data`
are not canonical source locations.

## Development commands

Use Node.js 22 and the pnpm version pinned in `package.json`. Install with pnpm;
do not introduce another package manager or lockfile.

Run these commands from the repository root:

| Command                                | Purpose                                                            |
| -------------------------------------- | ------------------------------------------------------------------ |
| `pnpm install --frozen-lockfile`       | Install workspace dependencies                                     |
| `pnpm dev`                             | Prepare Nuxt and run WebGL development build alongside Self-hosted |
| `pnpm dev:only`                        | Run Self-hosted alone after dependencies are built                 |
| `pnpm build:deps`                      | Build Storage and WebGL dependencies                               |
| `pnpm build`                           | Build Self-hosted                                                  |
| `pnpm preview`                         | Preview the Self-hosted build                                      |
| `pnpm dev:cloud` / `pnpm dev:site`     | Develop the corresponding edge entry                               |
| `pnpm build:cloud` / `pnpm build:site` | Build the corresponding edge entry                                 |
| `pnpm lint` / `pnpm fmt:check`         | Oxlint and Oxfmt checks                                            |
| `pnpm test`                            | Repository tests and Gallery utility tests                         |
| `pnpm check:packages`                  | UI/package boundary checks and package type checks                 |
| `pnpm check:ui`                        | Shared UI boundary checks                                          |
| `pnpm check:entries`                   | Application entry boundary checks                                  |
| `pnpm smoke:cloud` / `pnpm smoke:site` | Run built edge entries locally                                     |
| `pnpm db:generate`                     | Generate migrations after schema changes                           |
| `pnpm db:migrate`                      | Apply Self-hosted migrations manually                              |
| `pnpm docs:dev` / `pnpm docs:build`    | Develop/build documentation                                        |

For local UI verification, check whether port 3000 is already serving the app
before starting another development server. Use local development by default;
Docker verification and remote deployment are not routine requirements.

Self-hosted can start directly without creating or configuring a `.env` file.
Use the setup wizard to create an administrator and configure storage. Runtime
data paths remain repository-relative.
Cloud and Site have independent launchers and generated output; do not load the
Self-hosted environment or start its runtime services from an edge entry.

## Architecture and implementation rules

- Keep HTTP validation, authentication, cookies and runtime effects in application
  adapters. Shared UI layers must not acquire server implementations.
- Put shared domain contracts in `packages/contracts`, not database types or
  application-local aliases. Prefer named package exports and existing helpers.
- Keep Core independent of Nuxt, filesystem access and database connections.
  Supply infrastructure through its existing interfaces.
- Keep native SQLite, Sharp, EXIF tooling and native processing out of edge
  entries. Native work belongs in Self-hosted adapters or the processing node.
- Follow Vue Composition API, existing Nuxt auto-import conventions, strict
  TypeScript and the repository's formatting. Avoid `any`, unsafe casts and
  unnecessary abstractions.
- Preserve existing authorization, localization and edition-specific UX. Check
  every affected entry when changing shared packages; do not implement a feature
  in one entry by accidentally exposing it in another.
- Treat trusted Site/access context as an explicit boundary. Never infer tenant
  authorization from client input or a Self-hosted administrator session.
- Public DTOs must not expose passwords, password hashes, credentials or internal
  storage configuration. Preserve protected-media URL handling and redaction.
- Surface failures through established error/logging/notification patterns.
  Do not replace failures with silent defaults or success-shaped responses.

## Persistence and processing

- In Self-hosted server adapters, use the existing `useDB()` utility in
  `apps/self-hosted/server/utils/db.ts` and its exported tables/query helpers.
  Reuse package repositories for domain operations rather than introducing
  parallel database access paths.
- Schema and migrations are owned by `packages/database`. Historical server
  migration paths are compatibility paths, not a second source of truth.
- Generate and review SQL for schema changes; never edit released migrations.
  Check both a fresh database and an upgrade from existing data.
- Keep SQLite transactions synchronous and short. Never hold transactions across
  storage I/O, EXIF tooling or other asynchronous processing.
- Preserve explicit Site scope, source generations, immutable storage connection
  revisions and transactional resource/job publication.
- Preserve job leases and attempt identity on claims, heartbeats, stages and
  completion. Do not reset live workers or let stale attempts publish results.
- Reuse the shared photo processor for HEIC conversion, metadata, WebP/ThumbHash,
  EXIF, geocoding and Live/Motion Photo handling. Keep long-running native work
  outside HTTP request execution.
- Access storage through existing provider abstractions; do not hard-code a
  provider or bypass protected-media access controls.
- Keep platform D1 migrations separate from Self-hosted SQLite migrations.

## Verification and change hygiene

- Make focused changes and preserve unrelated working-tree edits. Do not commit,
  push, deploy or perform destructive Git operations without explicit direction.
- Start with the smallest existing test or check covering the change, then run
  affected package/type/boundary checks. Add regression coverage for changed
  behavior where practical.
- For application changes, build dependencies and the affected application, and
  verify the affected behavior. Edge changes also require entry checks and the
  corresponding built-entry smoke checks.
- For documentation changes, update both language versions when applicable.
  Build VitePress when changing documentation pages; standalone agent guidance
  does not require an application build.
- Do not change dependencies merely to run unrelated checks. Keep formatting
  changes scoped to files you intentionally modify.
- Never commit `.env` secrets, local databases, photos, logs or generated output.
  Do not copy production data into source directories or Docker build contexts.
- Preserve license notices. The repository default is AGPL-3.0-only, with MIT
  exceptions for Contracts and WebGL; consult `LICENSING.md`.

Use `CONTRIBUTING.md`, package READMEs and the development documentation for
details. Verify current code when documentation describes an earlier extraction
phase; do not assume a documented future capability is implemented.
