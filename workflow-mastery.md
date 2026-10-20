<div dir="rtl">

# تسلط بر Workflow — از صفر تا صد

## مقدمه

کلمهٔ Workflow در این پروژه دو معنا دارد...

markdown_content = r'''<div dir="rtl">

<style>
body, p, h1, h2, h3, h4, h5, h6, ul, ol, li, blockquote, table {
  direction: rtl;
  text-align: right;
}
code, pre {
  direction: ltr;
  text-align: left;
}
</style>

# تسلط بر Workflow — از صفر تا صد

> **هدف:** تسلط کامل بر مفهوم Workflow در معماری ایجنت — هم معنای مفهومی (L2 در طیف پیچیدگی) و هم معنای عملی (پایپ‌لاین end-to-end از داده تا تحویل).

---

## فهرست مطالب

- فصل ۱ — معنای مفهومی: چهار سطح معماری
- فصل ۲ — معنای عملی: پایپ‌لاین End-to-End
- فصل ۳ — ساخت Workflow از صفر: نقشهٔ ۶ مرحله‌ای
- فصل ۴ — چک‌لیست تسلط
- فصل ۵ — یافته‌های طلایی این پروژه
- جمع‌بندی

---

# مقدمه — دو معنای «Workflow»

کلمهٔ «Workflow» در این پروژه دو معنا دارد که خیلی مهم است فرقشان را بفهمی:

| معنا | کجا | چه چیزی |
|---|---|---|
| **معنای ۱ (مفهومی)** | اسلاید ۳، بخش ۱.۱ | یک **سطح معماری** (در برابر Agent و Multi-Agent) |
| **معنای ۲ (عملی)** | کل پروژه | یک **پایپ‌لاین end-to-end** از داده تا تحویل |

هر دو برای تسلط لازم‌اند.

---

# فصل ۱ — معنای مفهومی: چهار سطح معماری

## ۱.۱ — طیف پیچیدگی (طبق اسلاید ۳)

```
L0: Simple Call        مدل، یک بار، بدون ابزار، بدون حافظه
L2: Workflow           مدل + مراحل ثابت (کد مسیر را تعیین می‌کند)
L3: Agent              مدل + حلقه + ابزار (مدل مسیر را تصمیم می‌گیرد)
L4: Multi-Agent        چند Agent + Coordinator (تصمیم توزیع‌شده)
```

> **نکتهٔ حیاتی:** این طیف **مستقل از سطح اختیار انسان** است:
>
> - L0-L4 = پیچیدگی معماری (چقدر تصمیم پویا)
> - L1-L5 = اختیار انسان (چقدر انسان کنترل می‌کند)
>
> مثال: یک سیستم می‌تواند **L2** (Workflow ثابت) باشد ولی **L4 Approver** (تأیید انسانی). این دو را با هم اشتباه نگیر.

---

## ۱.۲ — L0: Simple Call (ساده‌ترین)

**مثال از پروژه (۱.۳):** «سامانه فقط یک رزومه را خلاصه می‌کند و هیچ ابزار یا حافظه‌ای ندارد.»

```python
# L0: یک فراخوانی مدل
def summarize(resume_text: str) -> str:
    return llm.complete(f"Summarize this resume: {resume_text}")
```

**مشخصات:**

- صفر ابزار، صفر حافظه، صفر حلقه
- ورودی → مدل → خروجی
- هیچ تصمیمی در زمان اجرا گرفته نمی‌شود

**چه وقت استفاده کن:**

- وقتی کار ساده است
- وقتی ورودی کافی است
- وقتی خروجی قابل اعتبارسنجی است

> **نکته از پروژه:** حتی این ساده‌ترین شکل هم باید اعتبارسنجی خروجی داشته باشد. در ۱.۸ بررسی شد که مدل ممکن است «توصیهٔ استخدام» تولید کند — که باید رد شود.

---

## ۱.۳ — L2: Workflow قطعی (این فصل اصلی است)

**مثال از پروژه (۹.۵):** «الگوی chaining استخراج ← schema gate ← گزارش.»

```python
def workflow(employee_id):
    # قدم ۱: استخراج
    extracted = extract_claims(employee_id)

    # قدم ۲: گیت
    gated = schema_gate(extracted)
    if not gated["passed"]:
        return {"reported": False, "reason": gated["reason"]}

    # قدم ۳: گزارش
    return {"reported": True, "report": build_report(gated)}
```

**سه ویژگی تعریف‌کنندهٔ L2:**

| ویژگی | یعنی |
|---|---|
| **مسیر از پیش معلوم** | مراحل ۱ ← ۲ ← ۳ ثابت است |
| **کد تصمیم می‌گیرد** | مدل فقط داخل هر مرحله اجرا می‌شود |
| **حلقه نیست** | یا تکرار ثابت، یا پایان |

- **فرق L0 با L2:** L0 یک فراخوانی است، L2 چند فراخوانی با ترتیب.
- **فرق L2 با L3:** در L3 مدل تصمیم می‌گیرد قدم بعدی چیست؛ در L2 کد تصمیم می‌گیرد.

---

### ۱.۳.۱ — چهار الگوی Workflow

این‌ها الگوهای استاندارد L2 هستند. در پروژه همهٔ چهار آمده‌اند.

#### الگوی ۱ — Prompt Chaining (زنجیره)

```
مرحله ۱ → (گیت) → مرحله ۲ → (گیت) → مرحله ۳
```

**مثال پروژه (۹.۵):** extract_claims → schema_gate → build_report

**کی استفاده کن:** وقتی هر مرحله به خروجی مرحلهٔ قبل وابسته است.

```python
def chain(employee_id):
    extracted = extract_claims(employee_id)
    if not valid_claims(extracted):
        return {"stopped": "invalid_input"}
    report = build_report(extracted)
    if not valid_report(report):
        return {"stopped": "invalid_report"}
    return finalize(report)
```

> **نکتهٔ کلیدی:** گیت یک تابع خالص است، نه prompt. تصمیم «ادامه یا توقف» در کد گرفته می‌شود.

**مثال اشتباه در برابر درست:**

```python
# ❌ گیت prompt-based (شکننده)
if llm.complete("Is this valid? yes/no").lower() == "yes":
    ...

# ✅ گیت code-based (deterministic)
if all(c.get("skill") and c.get("evidence_id") for c in claims):
    ...
```

---

#### الگوی ۲ — Routing (مسیریابی)

```
ورودی → [تصمیم] → شاخهٔ A
              ↓
              شاخهٔ B
              ↓
              شاخهٔ C
```

**مثال پروژه (۳.۷ و ۹.۱):** مسیریاب TF-IDF چهار مسیر دارد: profile / context / memory / unsupported

```python
def routing(request_text):
    route = router.predict([request_text])[0]
    if route == "profile":
        return handle_profile(request_text)
    elif route == "context":
        return handle_context(request_text)
    elif route == "memory":
        return handle_memory(request_text)
    else:
        return "clarify"
```

> **نکته از پروژه (۹.۴):** مسیریاب باید آستانهٔ confidence داشته باشد. با C=0.1، max_prob هرگز به ۰٫۵۵ نرسید → coverage=0. انتخاب C و آستانه باید با هم دیده شوند.

---

#### الگوی ۳ — Parallelization (موازی)

```
        ┌─→ worker A ─┐
input ──┤             ├─→ merge
        └─→ worker B ─┘
```

**مثال پروژه (۹.۶):** evidence worker و policy worker هم‌زمان.

```python
from concurrent.futures import ThreadPoolExecutor

def parallel(employee_id):
    with ThreadPoolExecutor(max_workers=2) as ex:
        f_ev = ex.submit(evidence_worker, employee_id)
        f_pol = ex.submit(policy_worker, "finance")
        ev, pol = f_ev.result(), f_pol.result()
    return merge(ev, pol)
```

> **نکتهٔ حیاتی از پروژه (۹.۱۲):**
>
> - کار I/O-bound (LLM، DB، شبکه) → موازی برنده است
> - کار CPU-bound (پردازش، Pydantic) → موازی کندتر است (GIL)
>
> در ۹.۱۲ ratio=7.88 بود — موازی ۷.۹ برابر کندتر.

---

#### الگوی ۴ — Orchestrator–Workers (پویا)

```
input → coordinator
         ↓
   چند worker لازم است؟
         ↓
   worker A + worker B + ...
         ↓
       merge
```

**مثال پروژه (۹.۷):** coordinator تعداد worker را بر اساس min_evidence تعیین می‌کند.

```python
def orchestrate(employee_id):
    workers = ["evidence", "policy"]
    verified = get_verified_count(employee_id)
    if verified < min_evidence:
        workers.append("reviewer_ambiguity")
    return run_workers(workers, employee_id)
```

**فرق با Parallelization:**

- Parallel: تعداد worker ثابت است
- Orchestrator: تعداد worker پویا است (بر اساس ورودی)

> **نتیجهٔ پروژه:** {2: 22, 3: 8} — از ۳۰ نفر، ۸ نفر (۲۷٪) به worker سوم نیاز داشتند.

---

### ۱.۳.۲ — کِی L2 کافی است، کِی L3 لازم است؟

| شرط | L2 کافی | L3 لازم |
|---|---|---|
| مراحل از پیش معلوم | ✅ | ❌ |
| ترتیب ثابت | ✅ | ❌ |
| تعداد قدم‌ها معلوم | ✅ | ❌ |
| تصمیم در زمان اجرا | ❌ | ✅ |
| نتیجهٔ ابزار قدم بعد را عوض می‌کند | ❌ | ✅ |

**مثال از پروژه:**

| سناریو | سطح |
|---|---|
| خلاصهٔ رزومه (۱.۳) | L0 |
| extract → gate → report (۹.۵) | L2 |
| route → specialist (۹.۱) | L2 |
| evidence ∥ policy (۹.۶) | L2 |
| search loop (۱.۲، ۷.۲) | L3 |
| coordinator + workers پویا (۹.۷) | L2/L3 مرزی |

> **درس طلایی:** همیشه از ساده‌ترین شروع کن. اگر Workflow جواب می‌دهد، Agent نساز.

---

## ۱.۴ — L3: Agent با حلقهٔ محدود

**مثال از پروژه (بخش ۷):** حلقهٔ ReAct با چهار کنترل.

```python
def agent_loop(principal, model, employee_id, config):
    trace = []
    for step in range(config.max_steps):
        if config.kill_switch:            return "killed"
        if deadline_hit(config):          return "deadline_exceeded"

        action = model.next_action(employee_id, trace)

        if action["kind"] == "final":     return "completed"
        if action["kind"] != "tool":      return "invalid_action"
        if repeated(action):              return "repeated_action"
        if cost + estimated > budget:     return "budget_exceeded"

        result = dispatch(principal, action["name"], action["arguments"])
        trace.append({"step": step+1, "kind": "tool", **action, "result": result})

    return "max_steps_reached"
```

**تفاوت‌های کلیدی با L2:**

| ویژگی | L2 Workflow | L3 Agent |
|---|---|---|
| مسیر | از پیش معلوم | پویا |
| تعداد قدم | معلوم | متغیر |
| کنترل | فقط بودجه/زمان | + تکرار + step cap + kill switch |
| پیچیدگی | ساده | نیاز به bounded loop |
| هزینه | قابل پیش‌بینی | متغیر |

---

## ۱.۵ — L4: Multi-Agent

**توجیه واقعی از پروژه (۳.۱۱ و ۹.۱۰):** فقط وقتی که نقش‌ها به context و مجوزهای متفاوت نیاز دارند.

**مثال از پروژه:**

- maker: get_employee + get_department_context + get_evidence
- checker: فقط get_employee + get_evidence
- هیچ‌کدام write_profile ندارند (least-privilege)

**توجیه اشتباه:**

- ❌ «چون اسمش جذاب‌تره»
- ❌ «چون baseline نداریم»
- ❌ «چون رایج است»

---

# فصل ۲ — معنای عملی: پایپ‌لاین End-to-End

```
۱. آماده‌سازی داده (بخش ۵)
   Load → Scope Filter → Budget → TF-IDF → Build Profile
                          ↓
۲. ابزارها (بخش ۶)
   MCP Server (stdio) → 3 tools + 1 resource + 1 prompt
                          ↓
۳. حلقه (بخش ۷)
   Router → Loop (bounded) → Tools → Final
                          ↓
۴. حافظه و تأیید (بخش ۸)
   Draft → Interrupt → Reviewer → Commit → Audit
                          ↓
۵. مسیریابی و چندنقشی (بخش ۹)
   Router → Chaining / Parallel / Orchestrator
                          ↓
۶. دفاع (بخش ۱۰)
   10 Threat Families → Structural Controls
                          ↓
۷. ارزیابی (بخش ۱۱)
   Bootstrap → P50/P95 → Cost → JS Divergence
                          ↓
۸. انتشار (بخش ۱۲)
   Gate → Canary → Shadow → Rollout → Handoff
```

---

## ۲.۱ — مرحله ۱: آماده‌سازی داده

```python
def prepare_data(tenant, department):
    # ۱. بارگذاری
    documents = load_documents()
    # ۲. Scope Filter (قبل از scoring!)
    allowed = [d for d in documents
               if d["tenant"] == tenant and d["department"] == department]
    # ۳. Budget Allocation
    selected = allocate_budget(WINDOW=1200, RESERVE=200)
    # ۴. TF-IDF روی اسناد مجاز
    vec = TfidfVectorizer().fit([d["text"] for d in allowed])
    # ۵. Redact + Dedupe
    clean = [redact_emails(d) for d in allowed]
    deduped = dedupe_by_canonical(clean)
    return {"docs": deduped, "vec": vec, "budget": selected}
```

**نکات حیاتی:**

| نکته | چرا |
|---|---|
| Scope قبل از scoring | اسناد غیرمجاز نباید در ranking ظاهر شوند |
| Budget قبل از allocation | ۲۰۰ توکن برای پاسخ رزرو می‌شود |
| Redact قبل از log | ایمیل نباید در context بیاید |
| Dedupe قبل از scoring | آزادسازی بودجه |

---

## ۲.۲ — مرحله ۲: ابزارها (MCP)

```python
mcp = MCPServer("day4-agent-lab")

@mcp.tool()
def get_employee(employee_id: str) -> dict:
    return dispatch(PRINCIPAL, "get_employee", {"employee_id": employee_id})
```

**سه اصل:**

1. ابزار محدود، نه عمومی: get_employee نه run_anything(sql)
2. Schema strict: extra=forbid, StrictStr, enum برای department
3. Scope از server: tenant از principal، نه از args

> ✗ «به مدل بگو tenant را عوض نکن» — مدل ممکن است فریب بخورد
> ✓ tenant در schema نباشد؛ server خودش از principal می‌گذارد

---

## ۲.۳ — مرحله ۳: حلقه (Bounded Loop)

```python
def agent_loop(principal, model, employee_id, config):
    trace = []
    cost = 0.0
    seen_fps = set()
    t0 = time.monotonic()

    for step in range(config.max_steps):
        if config.kill_switch:            return {"status": "killed"}
        if elapsed_ms(t0) > config.deadline: return {"status": "deadline"}

        action = model.next_action(employee_id, trace)

        if action["kind"] == "final":     return {"status": "completed"}
        if action["kind"] != "tool":      return {"status": "invalid"}

        fp = fingerprint(action)
        if fp in seen_fps:                return {"status": "repeated"}
        seen_fps.add(fp)

        if cost + estimated > config.budget: return {"status": "budget"}

        result = dispatch(principal, action["name"], action["arguments"])
        cost += estimated
        trace.append({"step": step+1, "kind": "tool", **action, "result": result})

    return {"status": "max_steps"}
```

> **قاعدهٔ طلایی:** همهٔ کنترل‌ها قبل از ابزار. کنترل بعد از ابزار = پول خرج شده، اثر انجام شده.

---

## ۲.۴ — مرحله ۴: حافظه و تأیید

```python
def build_graph(saver):
    g = StateGraph(GraphState)
    g.add_node("draft", build_draft)
    g.add_node("review", review_node)      # interrupt
    g.add_node("commit", commit_node)      # write
    g.add_edge(START, "draft")
    g.add_edge("draft", "review")
    g.add_edge("review", "commit")
    g.add_edge("commit", END)
    return g.compile(checkpointer=saver)

# ۱. اجرا تا interrupt
graph.invoke({"tenant": "org-a", "employee_id": "X-83c9..."}, config)

# ۲. ادامه با تأیید
graph.invoke(Command(resume={
    "approved": True,
    "reviewer": "hr_reviewer_01",
    "base_version": 0,
    "idempotency_key": "idem-first-commit",
}), config)
```

**سه binding در commit:**

```python
# ۱. payload_hash
if body_hash != approval.payload_hash:
    return "payload_changed_after_approval"

# ۲. base_version
if approval.base_version != current_version:
    return "stale_approval"

# ۳. idempotency
if existing_key and existing_hash == body_hash:
    return "replay_no_op"
if existing_key and existing_hash != body_hash:
    return "conflict"
```

> **نکتهٔ ظریف:** idempotency_key باید PRIMARY KEY یکتا باشد، نه composite با tenant.

---

## ۲.۵ — مرحله ۵: مسیریابی و چندنقشی

```python
route = router.predict([request_text])[0]

if route == "profile":
    return chain_profile(employee_id)
elif route == "context":
    return parallel_context(employee_id)
elif route == "memory":
    return orchestrate_memory(employee_id)
else:
    return "clarify"
```

**سه الگوی چندنقشی:**

| الگو | مثال | یادداشت |
|---|---|---|
| Chaining | ۹.۵ | ترتیبی با گیت |
| Parallel | ۹.۶ | موازی، latency کم |
| Orchestrator | ۹.۷ | تعداد پویا |

---

## ۲.۶ — مرحله ۶: دفاع

```python
DEFENSES = {
    "goal_hijack":         "skill ∈ required_skills + quote == source",
    "tool_misuse":         "name ∈ allowlist + extra=forbid",
    "privilege_abuse":     "tenant from principal, NOT from args",
    "supply_chain":        "SHA-256 manifest",
    "code_exec":           "exec NOT in registry",
    "memory_poisoning":    "text == source.text",
    "fake_replay":         "HMAC + nonce + exp",
    "cascading_failure":   "partial + escalate (no invented data)",
    "approval_deception":  "payload_hash + base_version binding",
    "rogue_agent":         "kill_switch + step_cap",
}
```

**اصل مشترک:** دفاع ساختاری (قابلیت وجود ندارد)، نه رفتاری (به مدل بگو).

---

## ۲.۷ — مرحله ۷: ارزیابی

```python
# ۱. جفت‌شده روی test
pivot = test.pivot_table(index="scenario_id", columns="variant", values="success")
diffs = pivot["candidate"] - pivot["baseline"]

# ۲. Bootstrap
boot_means = [mean(random.choices(diffs, k=len(diffs))) for _ in range(2000)]
ci_low, ci_high = percentile(boot_means, [2.5, 97.5])

# ۳. تصمیم
if ci_low <= 0 <= ci_high:
    print("NOT significant — از ادعای برتری صرف‌نظر کن")
```

> **یافتهٔ صادقانه از پروژه (۱۱.۴):** diff=+0.0625 ولی CI=[-0.025, +0.137] → شامل صفر → معنادار نیست.

---

## ۲.۸ — مرحله ۸: انتشار

```python
# ۱. Regression Gate (فقط dev!)
gate = {
    "quality":   succ_diff >= 0.03,
    "policy":    violation <= 0.02,
    "latency":   p95 <= 2300,
}
if not all(gate.values()):
    return "blocked"

# ۲. Canary (5% با hash پایدار)
def in_canary(request_id):
    h = sha256(request_id)
    return h % 100 < 5

# ۳. Shadow (بدون commit)
# ۴. Rollout کامل
```

> **نکتهٔ حیاتی (از ۱۲.۳):**
>
> - Shadow: بدون اثر واقعی (کمترین ریسک)
> - Canary: ۵٪ ترافیک واقعی (متوسط)
> - Full rollout: ۱۰۰٪ (بیشترین ریسک)

---

# فصل ۳ — ساخت Workflow از صفر: نقشهٔ ۶ مرحله‌ای

## مرحله ۱ — تعریف ورودی/خروجی

```python
Input = TypedDict("Input", {
    "employee_id": str,
    "department": str,
    "task": str,
})

Output = TypedDict("Output", {
    "decision": Literal["skills_inventory_only"],
    "claims": list[dict],
    "status": Literal["completed", "partial", "clarify"],
})
```

> **قاعده:** شکل ورودی/خروجی را قبل از هر کد تعیین کن.

---

## مرحله ۲ — طراحی مراحل

```python
STEPS = [
    ("extract",  extract_claims),
    ("gate",     schema_gate),
    ("report",   build_report),
]
```

> **قاعده:** هر مرحله یک تابع خالص است که ورودی می‌گیرد و خروجی می‌دهد. هیچ side-effect.

---

## مرحله ۳ — تعریف گیت‌ها

```python
def schema_gate(extracted):
    for c in extracted["claims"]:
        if not c.get("skill"):
            return {"passed": False, "reason": "missing_skill"}
        if not c.get("evidence_id"):
            return {"passed": False, "reason": "missing_evidence_id"}
    return {"passed": True, "valid": extracted["claims"]}
```

**گیت deterministic، نه prompt-based:**

```python
# ❌ شکننده
if llm.complete("Is this valid?").lower() == "yes":
    ...

# ✅ deterministic
if all(c.get("skill") and c.get("evidence_id") for c in claims):
    ...
```

---

## مرحله ۴ — اجرای Workflow

```python
def workflow(employee_id, department):
    trace = []

    extracted = extract_claims(employee_id)
    trace.append({"step": 1, "name": "extract", "ok": True})

    gated = schema_gate(extracted)
    if not gated["passed"]:
        return {"status": "partial", "reason": gated["reason"], "trace": trace}
    trace.append({"step": 2, "name": "gate", "ok": True})

    report = build_report(gated)
    trace.append({"step": 3, "name": "report", "ok": True})

    return {"status": "completed", "report": report, "trace": trace}
```

---

## مرحله ۵ — افزودن کنترل‌ها

```python
def workflow_with_controls(employee_id, department, config):
    t0 = time.monotonic()

    if config.max_tokens < 1200:
        return {"status": "budget_rejected"}

    if config.deadline_ms < 100:
        return {"status": "deadline_too_short"}

    result = workflow(employee_id, department)

    if not validate_output(result.get("report")):
        return {"status": "output_invalid", "reason": "validation_failed"}

    return result
```

> **قاعده:** هر کنترل قبل از اثر اعمال می‌شود.

---

## مرحله ۶ — آزمون

```python
def test_workflow():
    # آزمون ۱: ورودی معتبر
    r1 = workflow("X-83c9...", "finance")
    assert r1["status"] == "completed"

    # آزمون ۲: ورودی خراب
    r2 = workflow_with_broken_claim()
    assert r2["status"] == "partial"
    assert r2["reason"] == "missing_skill"

    # آزمون ۳: خروجی خارج از دامنه
    r3 = workflow_with_decision_hire()
    assert r3["status"] == "output_invalid"

    # آزمون ۴: مسیر معتبر
    assert validate_trajectory(r1["trace"], ALLOWED_STEPS)
```

**قاعده:** هر Workflow باید حداقل این چهار آزمون را داشته باشد.

---

# فصل ۴ — چک‌لیست تسلط

## ۴.۱ — قبل از ساخت

- [ ] ورودی/خروجی را دقیق تعریف کردم
- [ ] مراحل ثابت را لیست کردم
- [ ] گیت‌ها را تعریف کردم (deterministic)
- [ ] سطح L0/L2/L3 را انتخاب کردم (با دلیل)
- [ ] baseline (اگر L3) دارم

## ۴.۲ — در ساخت

- [ ] هر مرحله تابع خالص است
- [ ] گیت بین مراحل است
- [ ] trace هر قدم را ثبت می‌کند
- [ ] خروجی جدا از حلقه اعتبارسنجی می‌شود

## ۴.۳ — بعد از ساخت

- [ ] چهار آزمون (معتبر، خراب، خارج‌دامنه، trajectory)
- [ ] trajectory_gate روی مسیر عادی
- [ ] output_validator روی خروجی
- [ ] کنترل بودجه/زمان

## ۴.۴ — در مصاحبه/رزومه

- [ ] «چرا L2 و نه L3؟» جواب دارم
- [ ] «گیت چطور کار می‌کند؟» جواب دارم
- [ ] «اگر مرحله ۲ شکست بخورد چه می‌شود؟» جواب دارم
- [ ] «trajectory چطور معتبر می‌شود؟» جواب دارم

---

# فصل ۵ — یافته‌های طلایی این پروژه

## یافتهٔ ۱ — L2 همیشه کافی نیست، ولی همیشه اول امتحان کن

> «مسیر از پیش معلوم» یک قید قوی است. اگر قید را رد کردی، L3 لازم است.

## یافتهٔ ۲ — گیت باید کد باشد، نه prompt

> گیت prompt-based شکننده است. `if all(...)` deterministic است.

## یافتهٔ ۳ — trace باید final را هم داشته باشد

> بدون ثبت final در trace، trajectory_gate نمی‌تواند مسیر کامل را ببیند.

## یافتهٔ ۴ — خروجی completed ≠ خروجی معتبر

> completed یعنی «اجرا تمام شد»، نه «خروجی درست است». validate_output جدا لازم است.

## یافتهٔ ۵ — Workflow + Agent = معماری ترکیبی

> در ۹.۱، مسیریاب (L2) + حلقهٔ bounded (L3) با هم کار می‌کنند. مسیریاب تصمیم می‌گیرد، Agent اجرا می‌کند.
>
> ```
> [Router: L2] → [Route A: L2 Chaining]
>              → [Route B: L3 Agent Loop]
>              → [Route C: L4 Multi-Agent]
> ```
>
> این معماری واقعی است، نه انتخاب یکی از چهار.

---

# جمع‌بندی

> **Workflow (L2)** = مسیر ثابت + گیت‌های deterministic + خروجی معتبر + trace.
>
> **Agent (L3)** = Workflow + تصمیم پویا + حلقه + چهار کنترل.
>
> **Multi-Agent (L4)** = Agent + توزیع + coordinator + مجوز جدا.
>
> **همیشه از L2 شروع کن. L3 فقط با شواهد. L4 فقط با مرز امنیتی.**

---

## پیوست — واژه‌نامه

| اصطلاح | معنی |
|---|---|
| L0 | Simple Call — یک فراخوانی مدل بدون ابزار |
| L2 | Workflow — مراحل ثابت، تصمیم در کد |
| L3 | Agent — حلقه + ابزار، تصمیم در مدل |
| L4 | Multi-Agent — چند Agent + Coordinator |
| L1-L5 (اختیار) | Operator / Consultant / Approver / Observer — مستقل از L0-L4 |
| Bounded Loop | حلقه با max_steps + budget + deadline + kill_switch |
| Trajectory Gate | کنترل مسیر (ابزار + args + ترتیب) |
| Grounding | ادعا فقط از شاهد verified + quote دقیق |
| Idempotency Key | کلید یکتا برای نوشتن، replay=no-op، conflict=reject |
| Lethal Trifecta | داده خصوصی + محتوای نامطمئن + ارسال بیرونی |
| Bootstrap جفت‌شده | نمونه‌گیری روی جفت‌ها، نه نمونه‌های مستقل |
| Canary | ۵٪ ترافیک واقعی با برگشت‌پذیری |
| Shadow | اجرای موازی بدون اثر واقعی |
| Execution Scope | تفکیک واقعی / fixture / اجرانشده |

---

</div>
