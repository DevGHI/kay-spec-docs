<!--
NOTE FOR AI AGENTS: This file is a Myanmar-language translation intended for HUMAN developers only.
AI agents should read the canonical English guide at docs/subagents.md instead. Do not use this file as a source of truth.
-->

# Kay Platform — Cursor Subagents သုံးနည်း

> Kay — Kachin Handloom Weaving platform အတွက် `.cursor/agents/` ထဲမှာ သတ်မှတ်ထားတဲ့ custom subagent တွေကို Cursor chat မှာ ဘယ်လို ခေါ်သုံးရမလဲ။

---

## အကျဉ်းချုပ်

Subagent ဆိုတာ တစ်ခုချင်းစီအတွက် သီးခြား instruction ရှိတဲ့ AI assistant တွေဖြစ်ပါတယ် — frontend ရေးခြင်း၊ backend ရေးခြင်း၊ verify လုပ်ခြင်း၊ audit လုပ်ခြင်း၊ test စစ်ခြင်း၊ spec sync လုပ်ခြင်း စသဖြင့်။ Main chat ကို ရှင်းရှင်းလင်းလင်း ထားနိုင်ဖို့ isolated context မှာ run ပါတယ်။

**Agent definition file တွေရှိတဲ့နေရာ:**

```
.cursor/agents/
├── frontend-developer.md
├── backend-developer.md
├── verifier.md
├── auditor.md
├── test-reviewer.md
└── spec-maintainer.md
```

**Canonical spec (agent တွေ ဖတ်ရမယ့် file):** `docs/feature-spec.md`

---

## ဘယ်လို ခေါ်သုံးမလဲ

Cursor chat မှာ subagent နာမည်ကို ထည့်ပြောပါ။ Natural language သို့မဟုတ် slash prefix သုံးနိုင်ပါတယ် —

```
Use the auditor subagent to review the recent auth changes
```

```
/frontend-developer add dark mode to kay-dashboard
```

```
Use the test-reviewer subagent to verify order status flow against the spec
```

```
Use the verifier subagent to review order status flow against the spec
```

**ပိုကောင်းအောင် prompt ရေးနည်း:**

- **App** (`kay-web`, `kay-dashboard`, `kay-api`) နဲ့ **module/feature** ကို ရှင်းရှင်းပြောပါ။
- လ最近 လုပ်ထားတဲ့ work ကို ညွှန်ပြပါ — "checkout refactor ပြီးနောက်", "order status history feature" စသည်။
- ဘာလိုချင်တယ်ဆိုတာ ပြောပါ — "audit only", "implement and build", "sync spec and report gaps"။

---

## အကြံပြု workflow

Feature အကြီးတစ်ခု ပြီးသွားတိုင်း ဒီ အစဉ်လိုက် သုံးရင် quality ကောင်းပါတယ် —

| အဆင့် | Subagent | အကြောင်းရင်း |
|-------|----------|----------------|
| 1 | **frontend-developer** သို့မဟုတ် **backend-developer** | Feature implement / ပြင်ဆင်ခြင်း |
| 2 | **test-reviewer** | Spec နဲ့ ကိုက်ညီမှု စစ်ခြင်း; လိုအပ်တဲ့ test သာ ထည့်ခြင်း |
| 3 | **verifier** | Read-only: correctness, completeness, spec alignment, code quality |
| 4 | **auditor** | Merge/release မတိုင်ခင် read-only production risk review |
| 5 | **spec-maintainer** | `docs/feature-spec.md` ကို implementation နဲ့ sync လုပ်ခြင်း |

မလိုအပ်တဲ့ step တွေကို ကျော်နိုင်ပါတယ် (ဥပမာ docs-only change → frontend-developer မလို)။

---

## Subagent တစ်ခုချင်းစီ

### 1. `frontend-developer`

**ရည်ရွယ်ချက်:** **kay-web** နဲ့ **kay-dashboard** မှာ UI, page, component, form, data fetching, auth, i18n, theme ရေးခြင်း/ပြင်ခြင်း။

**ဘယ်အချိန် သုံးမလဲ:** storefront/admin page အသစ်၊ design system, dark mode, React Query, MM/EN translation စသည်။

**လုပ်ပေးမယ်:** Frontend code ရေးခြင်း/refactor; affected app မှာ `npm run build` run ခြင်း။

**မလုပ်ပါ:** Task မှာ မပြောရင် `kay-api` မပြင်ပါ။

**Example prompts:**

```
Use the frontend-developer subagent to add a product filter sidebar on kay-web ProductsPage
```

```
/frontend-developer refactor kay-dashboard orders list to use shared ErrorState and dark mode
```

---

### 2. `backend-developer`

**ရည်ရွယ်ချက်:** **kay-api** မှာ REST endpoint, Prisma model/migration, auth, validation, middleware, integration ရေးခြင်း။

**ဘယ်အချိန် သုံးမလဲ:** API route အသစ်၊ schema ပြောင်းခြင်း၊ auth, Zod validation, S3/Spaces upload, Telegram စသည်။

**လုပ်ပေးမယ်:** Backend code implement; `npm run typecheck` နဲ့ `npm run build` run ခြင်း။

**မလုပ်ပါ:** Frontend ကို မပြောရင် မပြင်ပါ။

**Example prompts:**

```
Use the backend-developer subagent to add order status history endpoint with note field
```

```
/backend-developer implement PATCH /api/admin/products/:id/stock per feature spec
```

---

### 3. `verifier`

**ရည်ရွယ်ချက်:** **Read-only** implementation review — spec နဲ့ project requirements နဲ့ **ကိုက်ညီမှု**, missing functionality, incorrect behavior, unnecessary complexity, coding standards, maintainability, codebase consistency စစ်ခြင်း။

**ဘယ်အချိန် သုံးမလဲ:** feature/refactor ပြီးနောက် merge မတိုင်ခင်; test သို့ production risk မဟုတ်ပဲ correctness စစ်ချင်တဲ့အခါ။

**လုပ်ပေးမယ်:** Code analyze; prioritized findings + recommended fixes ပါတဲ့ concise report return ခြင်း။

**မလုပ်ပါ:** Code fix, test ထည့်ခြင်း, spec edit, audit report save — လုံးဝ မလုပ်ပါ။

**Example prompts:**

```
Use the verifier subagent to review guest checkout against docs/feature-spec.md
```

```
/verifier verify order status transitions and stock side effects in kay-api
```

---

### 4. `auditor`

**ရည်ရွယ်ချက်:** **Read-only** production-readiness review — security, performance, reliability, scalability။

**ဘယ်အချိန် သုံးမလဲ:** feature work / refactor ပြီးနောက်၊ release မတိုင်ခင်၊ auth/upload လို sensitive area စစ်ချင်တဲ့အခါ။

**လုပ်ပေးမယ်:** Code/config analyze; report ကို `audit/reports/auditYYYYMMDD-HHMMSS.md` မှာ save ခြင်း။

**မလုပ်ပါ:** Code fix, test ထည့်ခြင်း, spec edit — လုံးဝ မလုပ်ပါ။

**Example prompts:**

```
Use the auditor subagent to review the recent auth changes
```

```
Use the auditor subagent to audit kay-api order module for production risks
```

```
Use the auditor subagent to review DigitalOcean Spaces upload flow and JWT handling
```

---

### 5. `test-reviewer`

**ရည်ရွယ်ချက်:** Spec နဲ့ implementation ကိုက်ညီမှု စစ်ခြင်း; **လိုအပ်တဲ့ test သာ** ထည့်ခြင်း; **full test suite** run ခြင်း။

**ဘယ်အချိန် သုံးမလဲ:** feature/bug fix ပြီးနောက်၊ release မတိုင်ခင် spec compliance စစ်ချင်တဲ့အခါ။

**လုပ်ပေးမယ်:** Test file သာ ပြင်ခြင်း; app တိုင်းရဲ့ test run; spec mismatch report။

**မလုပ်ပါ:** Production code fix, spec update — မလုပ်ပါ။

**Example prompts:**

```
Use the test-reviewer subagent to verify order status transitions match the spec
```

```
Use the test-reviewer subagent to add regression tests for guest checkout and run the full suite
```

---

### 6. `spec-maintainer`

**ရည်ရွယ်ချက်:** **`docs/feature-spec.md`** ကို လက်ရှိ implementation နဲ့ sync ထားခြင်း။

**ဘယ်အချိန် သုံးမလဲ:** feature/API ပြောင်းပြီးနောက်၊ spec outdated ဖြစ်နိုင်တဲ့အခါ။

**လုပ်ပေးမယ်:** Spec file သာ update; intentional deviation တွေကို implementation note နဲ့ မှတ်ခြင်း။

**မလုပ်ပါ:** Application code မပြင်ပါ; `feature-spec-mm.md` ကို မပြောရင် မပြင်ပါ။

**Example prompts:**

```
Use the spec-maintainer subagent to sync docs/feature-spec.md with the current order status flow
```

```
Use the spec-maintainer subagent to compare kay-api endpoints against the spec and report gaps
```

---

## Quick reference

| Subagent | Apps | Code ပြင်မလား? | Output |
|----------|------|----------------|--------|
| **frontend-developer** | kay-web, kay-dashboard | Frontend ဟုတ် | UI + build pass |
| **backend-developer** | kay-api | Backend ဟုတ် | API + typecheck/build |
| **verifier** | အားလုံး | **မပြင်ပါ** (read-only) | Correctness + spec report |
| **auditor** | အားလုံး | **မပြင်ပါ** (read-only) | `audit/reports/*.md` |
| **test-reviewer** | အားလုံး | Test file သာ | Spec + test report |
| **spec-maintainer** | docs | Spec file သာ | Updated `feature-spec.md` |

---

## Subagent တွေ ဆက်တိုက် သုံးနည်း

Turn တစ်ခုချင်းစီ ခွဲပြီး ခေါ်သုံးပါ —

1. **Build:** `Use the backend-developer subagent to add status history to orders`
2. **Test:** `Use the test-reviewer subagent to verify order status history against the spec`
3. **Verify:** `Use the verifier subagent to review order status and stock behavior for spec compliance`
4. **Audit:** `Use the auditor subagent to audit kay-api orders module for production risks`
5. **Document:** `Use the spec-maintainer subagent to sync the order status section in the spec`

**verifier**, **auditor** နဲ့ **test-reviewer** ကို production code fix လုပ်ခိုင်းမထားပါ — reviewer တွေဖြစ်လို့ **frontend-developer** / **backend-developer** နဲ့ fix လုပ်ပါ။

---

## Project layout

| Repo | Role |
|------|------|
| `kay-web` | Customer storefront |
| `kay-dashboard` | Admin panel |
| `kay-api` | Backend API |
| `docs` | Feature spec & documentation |

Subagent definitions (`.cursor/agents/`) က Kay repo အားလုံးပါတဲ့ **workspace root** မှာ ရှိပါတယ် — parent folder ကို Cursor မှာ open ထားရင် Kay app အားလုံးအတွက် share ဖြစ်ပါတယ်။

_ဤ file သည် developer (လူ) များ ဖတ်ရန်အတွက်သာ ဖြစ်ပါသည်။ မူရင်း (source of truth) မှာ `docs/subagents.md` (English) ဖြစ်သည်။_
