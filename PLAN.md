# Project Plan: Data-Driven Demand Forecasting and Inventory Optimization for Small Businesses

**WTM Tech Mentorship 2026 · Data Science · Group B**

**Final submission:** Friday 6 November 2026, 11:59 p.m. WAT (we aim to submit on the afternoon of Friday 6 November, leaving the evening as a buffer)

## Our question and answer

**Question:** A small shop: how much of each product will it sell in the coming weeks, and what should it buy more of?

**Our answer has two parts:**

| Part | It says | Who |
|---|---|---|
| **1. Forecast** | "This product will sell about **50 a week**." | Molemi |
| **2. Restock** | "So **buy more of it now**." (or "You have enough, don't buy.") | Everyone, using Molemi's forecast |

**Like a weather forecast:** the forecast says *"It will rain tomorrow."* The restock part says *"So take an umbrella."* The forecast tells us what will happen; the restock part tells the shop owner what to do about it.

## Team and roles

| Member | Role | Owns |
|---|---|---|
| Mugisha Elisa | Member 1: Data cleaning & EDA | Clean dataset, weekly sales table, EDA, data section of the report, report formatting |
| Muluka Zerga Lendino | Member 2: SQL & sales analysis | SQLite database, sales queries, ABC classification, query outputs for Power BI |
| Molemi Molokoane | Member 3: Demand forecasting & evaluation · **Team lead** | Baseline and ML forecasts, model evaluation, README, submission package, check-ins |
| Khadijat Ozohu Yusuf | Member 4: Power BI dashboard & slides | Dashboard pages, presentation slides, backup demo video |

Everyone works on the restock recommendations, writes their own report section, reviews the other parts, and presents their own contribution.

## How we check in

- **Daily WhatsApp update.** Post once a day, whenever suits you, in three lines:
    - **Done:** what you finished
    - **Next:** what you're doing next
    - **Blocked:** anything you're waiting on (or "nothing")
- If you're blocked, say so straight away in the group. Don't wait for the next update.

![Group B workflow diagram: who passes what to whom, and who does what each day](PLAN_flow.png)

### How it works, in simple terms

1. **Mugisha cleans the data** and shares the clean files with everyone.
2. **Muluka** takes the clean data and does the SQL analysis (Member 2): what sold, when, and which products matter most.
3. **Molemi** takes the same clean data and does the forecasting (Member 3): how many units of each top product will sell in the coming weeks.
4. **Khadijat** takes Muluka's results and Molemi's forecasts and builds the Power BI dashboard and the slides (Member 4).
5. **Together**, we turn Molemi's forecasts into **restock recommendations**: which products the shop should reorder, and how many.

**Why restock?** The forecast says how many units will *sell*. The restock step turns that into an *action* for the shop owner: "order 120 more of this product this week". It's the "inventory optimization" part of our title. The dataset has no real stock levels, so we assume them and state the assumptions clearly.

## Scope

Scope is the **fence around our project**: what we promise to deliver, and what we deliberately won't do. It stops us from trying to do too much in one week.

**In scope (what we will deliver):**

- Weekly forecasts, 4 weeks ahead, for the top 10 products (by quantity sold)
- One simple guess (the baseline) and one ML model, compared with each other
- Restock recommendations, based on assumptions we state clearly
- A Power BI dashboard

**Out of scope (what we won't do, and why):**

- **All ~4,000 products:** too many, and most of them barely sell.
- **Real stock or supplier data:** the dataset doesn't have any.

## Google Drive folder

**Link:** [WTM-Data-Science-Group-B](https://drive.google.com/drive/folders/1wUqEXejA86LHFEbBc9m0CmNRy1Mccc7V?usp=sharing)

Our data files live here. Our notebooks connect through this folder: one person's notebook saves a file here, and the next person's notebook opens it.

| Folder | What goes in it | Who puts it there |
|---|---|---|
| data → raw | The original Excel file (`Online Retail.xlsx`) | Molemi |
| data → processed | The clean data files | Mugisha |
| outputs | SQL results and forecasts | Muluka and Molemi |
| slides | Presentation slides | Khadijat |
| dashboard | Power BI file | Khadijat |

### Adding a shortcut

**Why:** a Colab notebook can only see files in **My Drive**. A folder someone shares with you sits in **"Shared with me"**, which Colab can't see. Adding a shortcut puts a link to the shared folder inside My Drive, so Colab can find it.

It's **not a copy**. It's the same folder, so we all still work on the same files. It's like putting an app icon on your phone's home screen: the app doesn't move, you just get a way to open it.

| Person | Needs the shortcut? | Why |
|---|---|---|
| Molemi | No | Created the folder, so it's already in My Drive |
| Mugisha | **Yes** | Opens the raw data and saves the clean files from Colab |
| Muluka | **Yes** | Opens the clean data from Colab |
| Khadijat | **Yes** | Opens the clean data, SQL results and forecasts for Power BI |

**How (once only):**

1. Open Google Drive.
2. Click **Shared with me** on the left.
3. Right-click **WTM-Data-Science-Group-B**.
4. Click **Organize → Add shortcut**.
5. Choose **My Drive**, then click **Add**.

## Rules for the repo

- Commit your own work from your own GitHub account. The commit history is our evidence for the teamwork score.
- Don't commit large raw data files. Raw data goes in the shared Google Drive folder.
- Never commit passwords, tokens or personal details.
