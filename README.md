# AI Analytics Projects

Two small projects applying LLMs (Claude API) to quick-commerce operations problems. Built in Python by Vedant Shingari.

| Project | North star metric | Result |
| --- | --- | --- |
| 1. LLM Support Ticket Triage Classifier | % of tickets auto-routed to the correct queue with no human touch | 97.6% auto-routed at 95.1% precision |
| 2. Text-to-SQL Self-Serve Analytics Assistant | % of business questions answered correctly without an analyst | 29/30 (97%) correct |

---

## Project 1: LLM Support Ticket Triage Classifier

Notebook: [`project1_ticket_classifier.ipynb`](project1_ticket_classifier.ipynb)

**North star metric:** % of tickets auto-routed to the correct queue with no human touch.

**Problem:** Quick-commerce support teams route customer tickets by hand; misrouted tickets add handling time and agent cost.

**Approach:**
- **Data:** [Bitext customer-support dataset](https://huggingface.co/datasets/bitext/Bitext-customer-support-llm-chatbot-training-dataset) (26,872 messages, 27 intents). Stratified 500-message held-out test set.
- **Baseline:** hand-written keyword rules (e.g. "refund" → `get_refund`).
- **LLM:** Claude Haiku 5.5 with a few-shot prompt (27 intent definitions + 2 solved examples per intent, none overlapping the test set) and structured JSON output (intent, urgency, confidence).
- **Human hand-off:** tickets below a confidence threshold go to an agent instead of being auto-routed.

**Results:**

| Metric | Result |
| --- | --- |
| Keyword baseline accuracy | 44.0% |
| Claude accuracy | **93.6%** (+49.6 pts) |
| Auto-routed at the 0.70 confidence threshold | **97.6% of tickets, at 95.1% precision** |
| Cost per 1,000 tickets | about Rs 24 |

**Coverage vs precision trade-off:**

| Confidence threshold | Auto-routed (%) | Precision (%) |
| --- | --- | --- |
| 0.50 | 99.8 | 93.8 |
| 0.60 | 99.2 | 94.4 |
| 0.70 | 97.6 | 95.1 |
| 0.80 | 96.8 | 95.5 |
| 0.90 | 86.4 | 98.1 |
| 0.95 | 37.4 | 100.0 |

The 0.70 threshold is the lowest that clears 95% precision. A business that can't tolerate misroutes could use 0.90 (98.1% precision) and send 13.6% of tickets to agents.

**Next steps:** test on real ticket data with messier language (Hinglish, typos), measure urgency against human labels, and compare cost with a small trained classifier at high volume.

---

## Project 2: Text-to-SQL Self-Serve Analytics Assistant

Notebook: [`project2_text_to_sql.ipynb`](project2_text_to_sql.ipynb)

**North star metric:** % of business questions answered correctly without an analyst.

**Problem:** Ops managers depend on analysts for routine data pulls, so simple questions wait in a queue.

**Approach:**
- **Data:** mock quick-commerce SQLite database (45 stores across 5 cities, 300 riders, 6,000 orders), generated with a fixed random seed so results are reproducible.
- **Test set:** 30 business questions (10 easy, 10 medium, 10 hard: joins, late-delivery logic, window functions, subqueries), each with analyst-written correct SQL.
- **Scoring:** execution accuracy. Claude's query result is compared with the correct result, ignoring column names and row order.
- **Guardrails:** read-only database connection; only SELECT queries are run.

**Results:**

| Version | What changed | Score |
| --- | --- | --- |
| A | Table and column names only | 26/30 (87%) |
| B | + column definitions and allowed values, business rules (GMV, late delivery, % formats), 3 examples, one retry on SQL errors | **29/30 (97%)** |

**What still fails and why:** the one remaining miss (Q18, orders by day of week) was a false negative in my evaluator, not a wrong answer. Claude returned day numbers as integers while the answer key held them as text, so the counts matched but the types didn't. Next version: normalise data types on both sides before comparing.

---

## How to run

1. `pip install anthropic datasets pandas python-dotenv ipykernel`
2. Create a `.env` file in this folder containing `ANTHROPIC_API_KEY=your-key`
3. Open each notebook in VS Code or Jupyter and run all cells in order

Project 1 makes about 500 API calls (about 10 minutes, a few US cents). Project 2 builds its own database in Cell 2.