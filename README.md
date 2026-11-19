# B2B Account Prioritisation + AI Account-Brief Assistant

**Question:** Which named accounts should marketing and sales focus on next, and what should the team know before reaching out?

**Data:** Maven Analytics, *CRM Sales Opportunities*. A fictional B2B hardware company with about 8,800 opportunities across 4 tables: accounts, products, sales teams and pipeline.

## What I built
1. **Data preparation and quality checks.** Joined the 4 CRM tables and fixed mismatched product keys (`GTXPro` vs `GTX Pro`). I also found that many Prospecting deals have no account, and documented this. Leakage-prone fields (`close_value`, `close_date`) were left out of the features.
2. **Win-probability model.** Logistic Regression vs Gradient Boosting vs a smoothed account win rate, trained on closed deals and validated on a **time-based hold-out** (trained on older deals, tested on the newest 20%).
3. **Account prioritisation.** Scored open deals as expected value = P(win) × list price, then rolled them up to rank accounts into tiers A/B/C. The output is `account_priority.csv`, which feeds the dashboard.
4. **AI account briefs with guardrails:**
   - The LLM (Gemini) sees only a JSON of verified account facts.
   - An automatic **fact-check** flags any number in a brief that isn't in the facts.
   - A **human reviewer** approves, rejects or edits each brief.
   - Every decision is logged in `brief_review_log.csv` as an audit trail.
   - API calls retry and fall back to other models if the service is overloaded.

## Results
| Metric | Value |
|---|---|
| Closed deals modelled | 6,711 |
| Open deals with no account assigned | 1,088 of 1,589 (68%), worth $2.6M at list price |
| Best ML time hold-out AUC | 0.52 (near chance), so I used a smoothed account win rate instead |
| Accounts ranked / open deals scored | 85 / 501 |
| Share of expected pipeline in top 10 accounts | 28% |
| AI brief review | The fact-checker flagged a brief with a misleading "data gap" claim; it was rejected in human review |

**Honest limitation:** deal-level CRM attributes carry only modest predictive signal (see AUC). Most of the value comes from expected-value ranking and a repeatable, reviewable briefing workflow, not from model accuracy alone.

## Run it
Open `B2B_Account_Prioritisation.ipynb` in Google Colab, upload the 4 CSVs, add `GEMINI_API_KEY` to Colab Secrets, then Run all.

**Stack:** Python (pandas, scikit-learn, matplotlib), Gemini API, Power BI
