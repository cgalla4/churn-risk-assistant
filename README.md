# Customer Churn Risk Assistant

Predicts which telecom customers are likely to cancel, then combines business rules and generative AI to give account managers a ranked call list with a reason, an offer, and an opening line for each at-risk customer.

## The Problem
About 26.5% of customers in this dataset left. Retention teams can't call everyone, so they need to know which customers are most at risk, why, and what to offer them.

## Data
IBM's public Telco Customer Churn sample dataset (7,043 customers, 21 columns), loaded directly from IBM's GitHub. The same approach would apply to live CRM data, such as Salesforce account and opportunity records.

## What I Did
1. **Cleaned the data.** TotalCharges was stored as text because 11 customers had blank values. All 11 had zero months of tenure (they hadn't been billed yet), so I filled them with $0, their true value, instead of an average.
2. **Compared two models.** Logistic regression outperformed a random forest (AUC 0.84 vs. 0.83), and it's easier to explain to business users.
3. **Tuned for the business goal.** I balanced the model to catch more churners, raising recall from 56% to 79%. That creates more false alarms, but a check-in call costs far less than losing a customer.
4. **Identified churn drivers.** Fiber internet and electronic check payments raise risk the most. One- and two-year contracts, online security, and tech support lower it. These are patterns, not proof of cause.
5. **Generated AI retention notes.** Business rules assign each at-risk customer an offer (customers in their first 3 months → free tech support, bills of $95+ → 1-year contract discount, others → autopay discount). Gemini then writes an internal note on why they're at risk and an opening line for the call.

## Results
| Model | Recall (churners caught) | Precision | AUC |
|---|---|---|---|
| Logistic regression | 0.56 | 0.66 | 0.842 |
| Random forest | 0.52 | 0.64 | 0.827 |
| **Balanced logistic regression** | **0.79** | 0.51 | 0.842 |

## Example AI Note
| | |
|---|---|
| **Customer** | 6861-XWTWQ (93% churn risk) |
| **Why at risk** | The customer has been with the company for 7 months on a month-to-month contract with fiber internet and electronic check payments, lacking both a stable term agreement and tech support. |
| **Offer (set by business rules)** | 1-year contract with 10% off |
| **Opening line** | "Hello, I'm following up to see how you're enjoying your fiber internet and to share a great way to lock in some extra savings." |

## Key Insight
All 10 highest-risk customers share the same profile: new customers (1–7 months) on fiber internet, month-to-month contracts, and electronic check payments, with no tech support. Rather than 10 individual calls, a retention team would get more value from **one targeted onboarding campaign for this segment**, for example offering a contract discount and free tech support in a customer's first few months.

## Making the AI Reliable
- **Rules decide, AI writes:** offers come from simple business rules, so they're predictable and easy to audit. The AI only writes the explanation and call script, and it can't change the offer or invent prices.
- **Structured output:** the AI returns JSON, so the notes go straight into a table.
- **Efficient:** all 10 customers go in one request instead of 10, which keeps the project within the free tier.
- **Fault-tolerant:** the free Gemini models were often overloaded, so the code uses a time limit, automatic retries, and a backup model. Notes are saved so re-running doesn't call the AI again.

## Limitations & Next Steps
- **Offers were first chosen by the AI, which just rotated them.** Because the top 10 customers are nearly identical, it couldn't tell them apart. I moved offer selection into simple business rules and kept the AI for writing, which is more predictable and easier to audit. Next step: test which offer actually reduces churn.
- **Correlation, not causation.** Testing offers with a control group would show which ones actually reduce churn.
- **Sample data.** The dataset is a public IBM sample. Real CRM data would need ongoing retraining as customer behavior changes.

## Tools
Python, pandas, scikit-learn, Google Gemini API, Google Colab

## Run It Yourself
Open `churn_analysis.ipynb` in Google Colab and click **Runtime → Run all**. The dataset loads automatically. To run the AI step, add a free Gemini API key in Colab Secrets as `GEMINI_API_KEY`.
