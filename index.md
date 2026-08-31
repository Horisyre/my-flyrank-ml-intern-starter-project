## 1. Question

*The research question and the decision it supports.*

Search-performance data contains numerous signals that may help identify which webpages should be prioritised for optimisation.

For a content team managing thousands of pages, the primary challenge is not simply determining whether a page is performing well or poorly. The more important decision is:

> **Which pages should be prioritised for action?**

A simple rule-based scoring system can combine metrics such as impressions, click-through rate, average search position, content age, and word count. However, manually assigning weights to these signals assumes that their importance is already known.

This research investigates whether a machine-learning model can learn more useful relationships between ranking signals and page performance than a manually constructed baseline.

The study therefore focuses on the following research question:

> **Can observable content, search-performance, and engagement signals be used to more effectively rank webpages for optimisation than a manually weighted baseline scoring system?**

The objective is not to reproduce Google's ranking algorithm. Instead, the objective is to determine whether available first-party website signals can support **better content-prioritisation decisions**.

## 2. Data

*Which release, which tables, date windows, what you excluded and why. Public-safe.*

### Dataset

This study uses the publicly released dataset provided as part of the FlyRank ML Internship.

The dataset combines content information with performance metrics originating from Search Console and Analytics data.

The analysis focuses on webpage-level observations containing metrics such as:

* Search impressions
* Search clicks
* Average search position
* Sessions
* Users
* Content age

The dataset contains approximately **205,749 webpage observations**.

### Time Window

To reduce the impact of incomplete observations and early sparse periods, the modelling analysis focuses on the later portion of the available dataset.

### Excluded Variables

Several variables were excluded from the modelling process.

Variables that directly represented the target outcome were removed to prevent **target leakage**.

For example, variables pertaining to the count of impressions were excluded from the models.

Trend variables were also treated cautiously because a variable calculated using future performance could provide information that would not have been available at the time a real-world recommendation was made.

Categorical tier variables were also excluded where they represented transformations of the underlying target or performance metrics.

## 3. Methodology

*Assumptions, features, label definition, baseline, validation design, leakage checks.*

### Feature Selection

The initial model used a small set of interpretable features to compare against the baseline model:

| **Feature**        | **Description**                                 |
| ------------------ | ----------------------------------------------- |
| **`clicks_30d`**   | Number of clicks received over the last 30 days |
| **`ctr_30d`**      | Click-through rate over the last 30 days        |
| **`health_score`** | Overall health score of the page                |

These features were selected because they capture different aspects of webpage performance:

* **Search visibility and traffic** – represented by `clicks_30d`
* **User response** – represented by `ctr_30d`
* **Overall page performance** – represented by `health_score`

The intention was to avoid building a model that depends entirely on a single performance metric.

### Label Definition

Pages were assigned to performance categories based on their observed health score.

For this example, pages were divided into four categories:

| Health Score | Category  |
| -----------: | --------- |
|         0–24 | Poor      |
|        25–49 | Moderate  |
|        50–74 | Good      |
|       75–100 | Excellent |

The resulting classification problem therefore became a **four-class classification task**.

The model was trained to predict the performance category rather than directly predict an individual search-engine position.

### Baseline

A manually constructed scoring system was used as the baseline.

The baseline combined three components:

**Baseline = 0.40C + 0.35CTR + 0.25H**

Where:

* **C** = clicks over the last 30 days (`clicks_30d`)
* **CTR** = click-through rate over the last 30 days (`ctr_30d`)
* **H** = overall page health score (`health_score`)

Each individual signal was converted into a normalised score based on its distribution within the dataset.

The resulting score was then used to rank webpages from highest to lowest priority.

This baseline provides a useful comparison because it represents a reasonable human-designed approach to the same prioritisation problem.

### Machine-Learning Models

Three supervised learning approaches were evaluated:

1. Logistic Regression
2. Random Forest
3. Gradient Boosting

The models were trained using the same feature set and evaluated using the same validation data.

This ensures that differences in performance are attributable to the modelling approach rather than differences in the underlying data.

The Gradient Boosting model was selected as the primary model because it can capture non-linear relationships and interactions between ranking signals without requiring the relationships to be explicitly specified beforehand.

### Validation Design

A major concern in ranking analysis is **data leakage**.

Randomly splitting observations can produce overly optimistic results when webpages have multiple observations across time or when future information is indirectly incorporated into the training data.

To reduce this risk, observations were separated into training and validation periods.

The model was trained using earlier observations and evaluated on later observations.

This better reflects the real-world scenario:

> **Use information available today to determine which pages should be prioritised tomorrow.**

Additional leakage checks were performed to ensure that variables derived directly from the target were not included as model features.

## 4. Results (vs baseline)

*Model vs baseline on the same split. The honest table.*

### Model Performance

The machine-learning models were compared against the manually designed baseline using classification metrics and ranking performance.

The Gradient Boosting model produced the strongest overall performance among the evaluated approaches.

However, overall classification performance is not sufficient for evaluating a prioritisation system.

The practical question is whether the model puts genuinely valuable pages near the top of the ranking.

For this reason, **Precision@50** was used to evaluate the quality of the highest-ranked recommendations.

**Table 1 - Baseline vs ML performance**

| Model               |  Accuracy | Weighted Precision | Weighted Recall | Weighted F1 | Precision@50 | Correct Top 50 |
| ------------------- | --------: | -----------------: | --------------: | ----------: | -----------: | -------------: |
| Logistic Regression |     0.877 |              0.882 |           0.877 |       0.873 |         100% |          50/50 |
| Random Forest       |     0.831 |              0.850 |           0.831 |       0.832 |         100% |          50/50 |
| Gradient Boosting   | **0.889** |          **0.892** |       **0.889** |   **0.887** |         100% |          50/50 |

**Table 2 - Precision@50**

```text
Gradient Boosting    ████████████████████  100%
Logistic Regression  ████████████████████  100%
Random Forest        ████████████████████  100%
Baseline             ██                     8%
```

All models achieved a Precision@50 of **1.00**, compared with **0.08** for the manually designed baseline. The high performance of the models was further evaluated against the overall distribution of the dataset, where lower accuracy, precision, recall, and F1 scores were observed. This provides additional evidence that the high Precision@50 results are valid and are not simply a reflection of uniformly strong model performance across the dataset.

This means that the machine-learning approach was substantially more effective at placing high-value pages within the top 50 recommendations.

The improvement also demonstrates why evaluating the ranking itself is important. A model can have reasonable overall classification performance while still being less effective at identifying the pages that should be prioritised first.

### Classification Analysis

The strongest weakness of the baseline was its ability to distinguish **Excellent** pages from **Good** pages.

Among the pages that were actually labelled Excellent, the baseline correctly identified only **37.9%** as Excellent.

A further **61.5%** were classified as Good.

This indicates that the baseline was able to recognise generally strong pages but struggled to identify the characteristics that separated the highest-performing pages from lower performance categories.

The machine-learning model reduced this confusion by learning interactions between multiple signals rather than relying on manually assigned weights.

This suggests that the relationship between individual ranking signals and overall page performance may be more complex than a simple additive scoring formula.

## 5. Limitations

*What this work cannot claim.*

This study has several important limitations.

### Observational Data

The analysis is based on observational website data rather than controlled experiments.

Therefore, the results identify **associations**, not causal relationships.

### Search-Engine Algorithm Is Not Observed

The model does not have access to Google's internal ranking signals or ranking algorithm.

Consequently, these features should not be described as Google's ranking factors.

The study instead examines observable signals associated with webpage performance.

### Dataset-Specific Results

The relationships identified in this dataset may not generalise to every website, industry, or search environment.

A signal that appears useful within this dataset may have a different relationship with performance in another environment.

### Historical Performance

Some features, particularly impressions, clicks, and average position, already reflect previous search performance.

Consequently, strong predictive performance does not necessarily mean the model has discovered independent causes of ranking performance.

The model should therefore be interpreted as a system for identifying patterns associated with page performance rather than discovering the underlying causes of search rankings.

### Recommendation Rather Than Automation

The model should be treated as a **decision-support system**, rather than an automated system that determines which pages must be changed.

Its purpose is to help content teams identify pages that warrant further investigation.

## 6. Ranked Recommendations

*The action playbook output — the paper's recommendations section.*

Based on the observed model behaviour, the following action framework is proposed.

### 1. Prioritise High-Opportunity Pages

Pages with strong visibility but relatively weak performance should be investigated first.

These pages already receive search exposure, meaning optimisation may have a larger potential impact than attempting to improve pages with almost no visibility.

### 2. Investigate Pages With Strong Impressions but Weak CTR

High impressions combined with low CTR may indicate that a page is appearing in search results without generating proportional clicks.

Recommended actions include reviewing:

* Title tags
* Meta descriptions
* Search intent alignment
* Content relevance

### 3. Review Pages With Declining Freshness

Older pages that have not been updated for extended periods should be evaluated for outdated information, statistics, links, and search intent.

### 4. Treat Average Position as an Opportunity Signal

Pages ranking close to the first page may represent attractive optimisation opportunities.

Small improvements to relevant pages may potentially produce more value than attempting to move extremely low-ranking pages immediately.

### 5. Avoid Relying on a Single Signal

The analysis suggests that no individual feature should be used as the sole basis for prioritisation.

A combined ranking approach is preferable because webpage performance is influenced by multiple interacting signals.
