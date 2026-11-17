# Customer Churn Risk Assistant

Predicts which telecom customers are likely to cancel, then uses generative AI to write a short retention plan for each at-risk customer so an account manager knows who to call and what to offer.

## The Problem
About 26.5% of customers in this dataset left. Retention teams can't call everyone, so they need to know which customers are most at risk, why, and what to offer them.

## Data
IBM's public Telco Customer Churn sample dataset (7,043 customers, 21 columns), loaded directly from IBM's GitHub. The same approach would apply to live CRM data, such as Salesforce account and opportunity records.

## What I Did
1. **Cleaned the data.** TotalCharges was stored as text because 11 customers had blank values. All 11 had zero months of tenure (they hadn't been billed yet), so I filled them with $0, their true value, instead of an average.
2. **Compared two models.** Logistic regression outperformed a random forest (AUC 0.84 vs. 0.83), and it's easier to explain to business users.
3. **Tuned for the business goal.** I balanced the model to catch more churners, raising recall from 56% to 79%. That creates more false alarms, but a check-in call costs far less than losing a customer.
4. **Identified churn drivers.** Fiber internet and electronic check payments raise risk the most. One- and two-year contracts, online security, and tech support lower it. These are patterns, not proof of cause.
5. **Generated AI retention notes.** For the 10 highest-risk customers, Gemini writes why they're at risk, an offer, and an opening line for the call.

## Results
| Model | Recall (churners caught) | Precision | AUC |
|---|---|---|---|
| Logistic regression | 0.56 | 0.66 | 0.842 |
| Random forest | 0.52 | 0.64 | 0.827 |
| **Balanced logistic regression** | **0.79** | 0.51 | 0.842 |

## Example AI Note
| | |
|---|---|
| **Customer** | 5178-LMXOP (94% churn risk) |
| **Why at risk** | Fiber internet, electronic check payments, and a month-to-month contract without tech support. |
| **Offer** | 1-year contract with 10% off |
| **Opening line** | "We noticed you are on a month-to-month plan and want to help you save on your fiber internet." |

## Key Insight
All 10 highest-risk customers share the same profile: new customers (1–7 months) on fiber internet, month-to-month contracts, and electronic check payments, with no tech support. Rather than 10 individual calls, a retention team would get more value from **one targeted onboarding campaign for this segment**, for example offering a contract discount and free tech support in a customer's first few months.

## Making the AI Reliable
- **No made-up offers:** the first version invented discounts that didn't exist, so I limited the AI to a fixed list of approved offers.
- **Structured output:** the AI returns JSON, so the notes go straight into a table.
- **Efficient:** all 10 customers go in one request instead of 10, which keeps the project within the free tier.
- **Fault-tolerant:** the free Gemini models were often overloaded, so the code uses a time limit, automatic retries, and a backup model. Notes are saved so re-running doesn't call the AI again.

## Limitations & Next Steps
- **Offers aren't truly personalized.** Because the top 10 customers are nearly identical, the AI rotated through the three offers instead of tailoring them. Next step: give it more distinguishing details (tenure, services used, support history) or match offers to specific risk factors with rules.
- **Correlation, not causation.** Testing offers with a control group would show which ones actually reduce churn.
- **Sample data.** The dataset is a public IBM sample. Real CRM data would need ongoing retraining as customer behavior changes.

## Tools
Python, pandas, scikit-learn, Google Gemini API, Google Colab

## Run It Yourself
Open the notebook in Google Colab and click **Runtime → Run all**. The dataset loads automatically. To run the AI step, add a free Gemini API key in Colab Secrets as `GEMINI_API_KEY`.
