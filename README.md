# fnb-prediction-docs

This documentation is for the API we are building with Astra. The API builds AI models on restaurant sales data, and outputs predicted sales.

To contribute to the docs, please submit issues or pull requests, and our moderators will review.

A human-friendly site can be viewed at [here](https://bump.sh/chingjuiyoung/doc/fnb-prediction)

## Overview

- We build a prediction model for every "resource". A resource is a menu_item, category, or store.
- To get a sales prediction, we call a single model directly for its prediction. ie to get a prediction for "red wines" category, you call the single model for the red wines category.
- To get a prediction, you only need to provide the model_id. You can provide optional parameters, such as `days` (for # of days predicted), `major_events` for inputting known upcoming events, etc.
- A menu_item can only belong to 1 category, and a category can only belong to 1 store (strict hierarchy)
- Even if multiple stores have same or similar categories ("red wine"), for purposes of our models, each "red wine" resource will be tied to only 1 store.

## Thought process behind this structure

- Aggregating individual forecasts are less accurate than modeling aggregation directly.
- Example: Getting predictions for all red wine menu items individually (merlot, cabernet, pinot, etc.) and adding them all up, is less accurate than building 1 model for the "red wine" category directly

## Examples

### Case 1: Getting a prediction for a single menu item

Use case: *"I want to predict the unit sales of Corona beers for the next 14 days at my Revolucion store in Guangzhou"*

First, look up which models are built for "corona"

`GET /v1/models?name=corona`

Result

| Created At               | Model ID                | Name  | Restaurant           | Type       | Status | Output Type |
|--------------------------|-------------------------|-------|----------------------|------------|--------|-------------|
| 2023-05-19T10:00:30.000Z | m_3238434738438738934   | Corona| Revolucion GuangZhou | menu_item  | ready  | unit        |
| 2023-05-19T10:00:30.000Z | m_1234567890123456789   | Corona| Revolucion Xiamen    | menu_item  | ready  | unit        |
| 2023-05-19T10:00:30.000Z | m_2345678901234567890   | Corona| Bottega              | menu_item  | ready  | unit        |
| 2023-05-18T10:00:30.000Z | m_3456789012345678901   | Corona| Revolucion GuangZhou | menu_item  | failed | unit        |

Then, retrieve predictions from the model you want

`GET /v1/predictions/m_3238434738438738934?days=3`

| Date       | Model ID                | Name  | Type       | Restaurant           | Predicted Avg Sales | Buffered Prediction | Confidence | Manual Adjustment | Status | Created At               | Output Type |
|------------|-------------------------|-------|------------|----------------------|---------------------|---------------------|------------|------------------|--------|--------------------------|-------------|
| 2023-05-24 | m_3238434738438738934   | Corona| menu_item  | Revolucion GuangZhou | 23                  | 29                  | 0.95       | 3                | ready  | 2023-05-24T10:00:30.000Z | unit        |
| 2023-05-25 | m_3238434738438738934   | Corona| menu_item  | Revolucion GuangZhou | 25                  | 31                  | 0.94       | 4                | ready  | 2023-05-25T10:00:30.000Z | unit        |
| 2023-05-26 | m_3238434738438738934   | Corona| menu_item  | Revolucion GuangZhou | 27                  | 33                  | 0.92       | 2                | ready  | 2023-05-26T10:00:30.000Z | unit        |

### Case 2: Getting a prediction for a category of foods

Use case: "I want to predict the total red wine sales in my Guangzhou Revolucion store for next 7 days."

`GET /v1/models?name=red_wine`

| Created At               | Model ID                | Name  | Restaurant           | Type       | Status | Output Type |
|--------------------------|-------------------------|-------|----------------------|------------|--------|-------------|
| 2023-05-19T10:00:30.000Z | m_4872394239842394823   | red wine | Revolucion GuangZhou | category  | ready  | revenue        |
| 2023-05-19T10:00:30.000Z | m_2394832948239842384   | red wine-infused steak | Revolucion Xiamen    | menu_item  | ready  | unit        |
| 2023-05-19T10:00:30.000Z | m_4239842398439239842   | Pinot red wine | Bottega              | menu_item  | ready  | revenue        |
| 2023-05-18T10:00:30.000Z | m_2384923849238492384   | red wine | Revolucion GuangZhou | category  | failed | revenue        |

Then, retrieve predictions from the model you want

`GET /v1/predictions/m_4872394239842394823&days=3`

| Date       | Model ID                | Name  | Type       | Restaurant           | Predicted Avg Sales (RMB) | Buffered Prediction (RMB) | Confidence | Manual Adjustment | Output Type |
|------------|-------------------------|-------|------------|----------------------|---------------------------|---------------------------|------------|------------------|-------------|
| 2023-05-24 | m_4872394239842394823   | red wine | category  | Revolucion GuangZhou | 2300                    | 2900                      | 0.95       | 3                | revenue     |
| 2023-05-25 | m_4872394239842394823   | red wine | category  | Revolucion GuangZhou | 2500                    | 3100                      | 0.94       | 4                | revenue     |
| 2023-05-26 | m_4872394239842394823   | red wine | category  | Revolucion GuangZhou | 2700                    | 3300                      | 0.92       | 2                | revenue     |

### Case 3: Getting a prediction for all sales in a store

I want to predict the total sales in my Guangzhou Revolucion store for next 14 days.
`GET /v1/models?name=revolucion`

List of all models trained on revolucion
`GET /v1/predictions/m_407854834`
