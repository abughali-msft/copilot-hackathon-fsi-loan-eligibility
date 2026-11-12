# Loan eligibility API

**GitHub Copilot Hackathon: Build Smart, Spend Smart** · FSI breakout · Challenge B: Workflow app

Decide personal-loan applications with clear eligibility rules, and give the reason for every decision.

> [!IMPORTANT]
> Synthetic data only. Never bring real customer data, credentials or production systems into this repo.

## Start here

1. Select **Use this template** > **Create a new repository** to create your team's copy. Make it public, or give the judges access.
2. Clone it and open it in your IDE with GitHub Copilot Chat.
3. Read this brief and the [data dictionary](data/README.md).
4. **Before you build**, plan with Copilot Plan mode and estimate your AI Credits per phase in [docs/ai-credit-budget.md](docs/ai-credit-budget.md).
5. Build, test, review and document with Copilot. Commit often: your commit history is evidence.
6. Before your demo, complete [Your solution](#your-solution) at the end of this README.

## The day at a glance

The day has two goals:

1. Use GitHub Copilot beyond autocomplete: plan, agent, instructions, prompt files and review.
2. Build the habit of planning AI Credit consumption before you start prompting.

| | |
|---|---|
| **Team** | 3–5 developers |
| **Build time** | 4 hours |
| **Usage checkpoints** | Your facilitator records each seat's AI Credit usage before the build starts, at the mid-point check (2:00) and at the end. |
| **Demo** | 5 minutes, then Q&A |
| **Scoring** | 100 points. See the [scoring rubric](#scoring-rubric). |

## The challenge

**Who it's for:** a credit officer in a bank's retail lending team, and the applicants they serve.

**The problem:** eligibility checks are manual and inconsistent. Applicants deserve a clear reason for every decision, and auditors need to see how each decision was made.

**What to build:** an API that accepts loan applications and runs three eligibility rules. It approves or rejects each application with reasons and keeps an audit trail. A UI is optional.

### Workflow

An application has three states: `SUBMITTED`, then `APPROVED` or `REJECTED`. Invalid input is refused at submission with a validation error. That isn't a decision, so it never becomes `REJECTED`.

**Suggested API**

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/applications` | Submit and validate an application. Returns its ID with status `SUBMITTED`. |
| `POST` | `/applications/{id}/decision` | Run the rules and record the decision. |
| `GET` | `/applications/{id}` | The application, its status and the decision reasons. |
| `GET` | `/applications/{id}/audit` | The audit trail. |

The API shape is a suggestion. Agree your own API standards as a team and put them in `.github/copilot-instructions.md`.

### Eligibility rules

| Rule ID | Passes when | Suggested default |
|---|---|---|
| `MIN_INCOME` | The applicant earns enough. | `monthly_income_kwd` ≥ **500.000 KWD** |
| `DEBT_TO_INCOME` | Monthly debt, including the new loan, is affordable. | (`existing_monthly_debt_kwd` + new instalment) ÷ `monthly_income_kwd` ≤ **40%** |
| `AGE` | The applicant is old enough now and not too old when the loan ends. | **21 or older** on the application date and **65 or younger** on the last instalment date |

Approve an application only if every rule passes. A rejection lists every rule that failed, not just the first.

Use these definitions:

- **Monthly instalment** = P × r ÷ (1 − (1 + r)<sup>−n</sup>), where P = `loan_amount_kwd`, r = `annual_rate_pct` ÷ 1200 and n = `term_months`. If the rate is 0, the instalment is P ÷ n. Round the instalment to 3 decimal places (half up) before you compute the ratio.
- **Age** is in completed years.
- **The application date** is the date part of `submitted_at`, in Kuwait time (UTC+03:00).
- **The last instalment date** is the application date plus `term_months` calendar months. If that day doesn't exist in the target month, use the month's last day. For example, 31 October + 4 months is 28 February, or 29 February in a leap year.

The defaults are a starting point for illustration, not regulatory limits. You can change them if you record the change in your plan and test it. Keep limits in configuration, not in code.

Decide these in your plan and cover them with tests:

- **Explainability:** what the applicant and the credit officer see for each rule, for example `DEBT_TO_INCOME: 46.2% is above the 40% limit`.
- **Validation:** what makes an application invalid rather than rejected, and the error the caller gets.
- **Audit trail:** who did what and when, the inputs, each rule's result and the rule configuration used. The trail is append-only: entries are never updated or deleted.
- **Boundaries:** test values exactly at the limits, such as 500.000 KWD, a ratio of exactly 40%, a 21st birthday on the application date and a last instalment on the applicant's 66th birthday.
- **Messy data:** the data has deliberate quality issues. See [Things to know](data/README.md#things-to-know).

### What to build

**Must have**

- [ ] Load [`data/loan_applications.csv`](data/loan_applications.csv) through your API, for example with an import script.
- [ ] Endpoints to submit and decide an application, and to read the application, its decision and its audit trail.
- [ ] The three eligibility rules, with configurable limits.
- [ ] Every decision lists each rule's result with the actual values.
- [ ] An append-only audit trail for each application.
- [ ] Automated tests on the rules and the API, including boundaries and invalid input.

**Stretch goals (optional)**

- A `REFERRED` outcome for borderline cases, for example a ratio between 35% and 40%, so a credit officer approves or rejects the application with a note.
- A lower ratio limit for retired applicants (`employment_status` is `RETIRED`), for example 30%.
- An OpenAPI description of your API.
- Idempotent submission, so a resubmitted application isn't decided twice.
- Roles for the applicant, the credit officer and the auditor.

Use any language, framework or database, as long as it runs locally from your README. No machine learning is needed: plain business rules only.

## Starter data

| File | Rows | Contents |
|---|---:|---|
| [`loan_applications.csv`](data/loan_applications.csv) | 402 | Personal-loan applications submitted in October 2026 |

All data is synthetic. Applicants are anonymised IDs, with no names or civil ID numbers. The [data dictionary](data/README.md) explains every column and the data-quality issues.

## Copilot focus for this challenge

Every team should use the practices the rubric scores: Plan mode, agent mode, a `.github/copilot-instructions.md` file with your team's conventions, reusable prompt files and Copilot Chat review. This challenge puts extra weight on three of them:

| Practice | How to apply it here |
|---|---|
| **Instructions file for API standards** | Put your API conventions in `.github/copilot-instructions.md`: resource names, status codes, the error format, validation, money as decimals and audit fields. Then every endpoint that agent mode writes follows them. |
| **Agent scaffolding** | Use agent mode to scaffold the API, the data model, the audit trail and an import script for the CSV. |
| **Prompt files for tests** | Save reusable prompts in `.github/prompts/` for rule tests and API tests. Reuse them for each new rule or endpoint. |

<details>
<summary>Example prompts</summary>

**Instructions file**

```text
Draft .github/copilot-instructions.md for our loan eligibility API. Cover resource names,
status codes, one error format for validation errors, money as decimals with 3 places and
the fields every audit entry must have. Keep it short enough to read in two minutes.
```

**Rule tests** (for example, saved as `.github/prompts/rule-tests.prompt.md`)

```text
Write unit tests for the eligibility rule in the current file. Cover a pass, a fail, the
values exactly at the limit and the data-quality issues described in data/README.md.
Check that each reason includes the actual values. Follow our test conventions.
```

**Plan mode**

```text
Read README.md and data/README.md. Plan a loan eligibility API with the three rules,
input validation and an append-only audit trail. For each rule and endpoint, list
acceptance criteria and edge cases. For each phase, recommend a model tier and estimate
the AI Credits it will use.
```

</details>

## AI Credit budget

Estimate AI Credits per phase before you build, then track what you actually use in [docs/ai-credit-budget.md](docs/ai-credit-budget.md). The example below is from the kickoff and assumes a team of 3–5 (950 credits is about USD 9.50). Make your own estimate: the point is the habit of estimating, not hitting an exact number.

| Phase | Copilot feature | Model tier | Budget (AI Credits) |
|---|---|---|---:|
| Plan | Plan mode / Ask | Frontier | 150 |
| Scaffold | Agent mode + instructions file | Low-cost | 100 |
| Build features | Agent / Edit mode | Mixed | 500 |
| Tests + review | Prompt file for tests, chat review | Low-cost | 150 |
| Docs + demo | Ask, README generation | Low-cost | 50 |
| **Total** | | | **950** |

Tips to spend smart:

- Match the model to the task. Save frontier models for planning and tricky logic, and use low-cost models for scaffolding, tests and docs.
- Keep context scoped. Attach only the files Copilot needs.
- Reuse prompt files instead of retyping long prompts.

## Scoring rubric

The rubric awards 100 points across the lifecycle. It's the same in all three sectors, so winners are comparable.

| Stage | Points |
|---|---:|
| Plan | 15 |
| Build & quality with Copilot | 40 |
| Token efficiency | 25 |
| Impact & demo | 20 |
| **Total** | **100** |

### How scoring works

- Judges pick one level per stage: **Solid**, **Strong** or **Outstanding**. Each level is worth the points shown.
- Your score is the sum of the four stage scores. A stage with no evidence scores 0.
- Build & quality plus Token efficiency carry 65 points: they're the two goals of the day.
- Judges score from evidence, not claims: your repo, budget sheet, usage export and demo.
- Tie-break: the team with the higher Token efficiency score wins.
- Solid is the expected baseline. Most teams should reach Strong in four hours. Outstanding should stand out.

### Levels

| Stage | Solid | Strong | Outstanding |
|---|---|---|---|
| **Plan**<br>15 points | **5 pts**<br>Plan written with Copilot Plan mode; credit budget estimated per phase. | **10 pts**<br>Plan has acceptance criteria and a model choice per phase, with the reasoning explained. | **15 pts**<br>Plan drove the build, and the team re-planned at the mid-point check using actual usage. |
| **Build & quality with Copilot**<br>40 points | **16 pts**<br>Agent mode built the main features; `copilot-instructions.md` sets team conventions; runs end to end with tests on the core logic. | **28 pts**<br>Adds reusable prompt files for repeat tasks (tests, docs); edge cases tested; Copilot asked in chat to review the code and findings fixed. | **40 pts**<br>Copilot used across the whole loop (plan, build, test, review, docs); team shows where it saved the most time and where they had to correct it; another team could run it from the README. |
| **Token efficiency**<br>25 points | **10 pts**<br>Usage tracked; actual within ±30% of budget. | **18 pts**<br>Within ±15% of budget; models matched to tasks; context kept scoped. | **25 pts**<br>A concrete before/after where a prompt, context or model change cut credits without hurting the output. |
| **Impact & demo**<br>20 points | **8 pts**<br>Clear user and problem; working demo in 5 minutes. | **14 pts**<br>Rules or workflow are realistic for the sector; Q&A handled well. | **20 pts**<br>Credible path to a real pilot; judges can picture it being used. |

### What judges look at

| Stage | Evidence |
|---|---|
| Plan | `docs/plan.md` from Plan mode, and the budget and mid-point re-plan in `docs/ai-credit-budget.md` |
| Build & quality | `.github/copilot-instructions.md`, `.github/prompts/*.prompt.md`, tests, commit history and this README |
| Token efficiency | Budget versus actual usage in `docs/ai-credit-budget.md`, the usage export, and your before/after example |
| Impact & demo | Your 5-minute demo and Q&A |

### Clarifications

- **Token efficiency isn't about using the fewest credits.** It's judged against your own budget plus proof of optimization, so a team that barely used Copilot can't win it by spending little. Judges reward value delivered per credit. The ±30% and ±15% bands apply to overspending and underspending.
- **IDE features are enough.** Every level can be reached with Copilot Chat features in the IDE: Plan mode, agent mode, instructions, prompt files and chat review. If your organization also has Copilot code review on pull requests, MCP or the Copilot coding agent, they count as extra evidence for Outstanding, but they're never required.
- **Sector judgement counts.** Sector judges apply domain judgement in Impact & demo. For this challenge, strong teams show traceability from each rule to a test, and an audit trail an auditor could follow.

## Your solution

> [!NOTE]
> Complete this section before your demo. Judges, and other teams, will run your app from it.

- **Team name:**
- **Team members:**
- **Stack:**

### Run it

```bash
# Replace with the exact steps to install and run your app.
```

### Test it

```bash
# Replace with the exact command that runs your tests.
```

### Rule-to-test traceability

| Rule | Implemented in | Tested in |
|---|---|---|
| `MIN_INCOME` | | |
| `DEBT_TO_INCOME` | | |
| `AGE` | | |
| Input validation | | |

### Copilot retrospective

- **Where Copilot saved us the most time:**
- **Where we had to correct it:**

### Evidence checklist

- [ ] Plan: `docs/plan.md`
- [ ] Budget, actual usage, mid-point re-plan and before/after example: [`docs/ai-credit-budget.md`](docs/ai-credit-budget.md)
- [ ] Usage export: `docs/usage/`
- [ ] Team conventions: `.github/copilot-instructions.md`
- [ ] Prompt files: `.github/prompts/`
