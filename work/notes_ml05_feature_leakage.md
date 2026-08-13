# Takeaway Findings: Feature Engineering and Leakage Audit (ML-05)

## 1. Ground Truth Problem Scale (29.92% Base Rate)
- Across 331,436 real content items analyzed in March 2026, 29.92% experienced a severe (>15%) organic traffic decline in April.
- Takeaway: Content decline is a massive, recurring operational challenge. Because 70% of content remains stable or growing, editorial teams cannot manually inspect everything—a prioritized ranking queue is strictly necessary.

## 2. Pre-Decision Signals Hold Strong Predictive Power (0.8479 Grouped ROC-AUC)
- The honest 13-feature vector (momentum velocity ratios, Google rank volatility, CTR, and missingness flags) achieved 0.8479 ROC-AUC on unseen clients.
- Takeaway: Pre-decision signals provide genuine predictive power. Intramonth acceleration (comparing days 16–31 vs days 1–15) and daily ranking turbulence provide early-warning signals before traffic falls off a cliff.

## 3. The Leakage Test Harness Proved Effective (0.9967 vs 0.8637)
- When a future metric (`april_impressions`) was deliberately injected, the model's ROC-AUC artificially increased to 0.9967.
- When purged, the honest score settled at 0.8637.
- Takeaway: The test harness successfully detects data leakage. The verified feature pipeline is clean and free of target peeking or legacy product score contamination.

## 4. Client Memorization Bias is Real (+0.0137 Gap)
- Random 5-Fold Cross-Validation: 0.8616 ROC-AUC
- Client-Grouped 5-Fold Cross-Validation (`GroupKFold`): 0.8479 ROC-AUC
- Memorization Gap: +0.0137 (+1.37%)
- Takeaway: Random splitting inflates scores slightly because pages on the same client domain share domain authority and topic structure. Grouped cross-validation by client is mandatory to ensure the model generalizes to new client accounts.

## 5. Missingness as a Feature Prevents Panel Distortion
- Adding boolean indicators (`has_search_data`, `has_ga4_data`, `has_word_count`) allowed the model to distinguish between poor engagement and missing analytics integration.
- Takeaway: `has_search_data` proved to be the single most correlated volume filter (r = 0.6113), ensuring the model separates active content from dead URLs before ranking.
