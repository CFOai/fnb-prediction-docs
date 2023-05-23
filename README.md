# fnb-prediction-docs

This documentation is for the API we are building with Astra. The API builds AI models on restaurant sales data, and outputs predicted sales.

To contribute to the docs, please submit issues or pull requests, and our moderators will review.

A human-friendly site can be viewed at [here](https://bump.sh/chingjuiyoung/doc/fnb-prediction)

## Overview

- We build a prediction model for every "resource". A resource is a menu_item, category, or store.
- To get a sales prediction, we call a single model directly for its prediction. ie to get a prediction for "red wines" category, you call the single model for the red wines category.
- To get a prediction, you only need to provide the model's `_id`. You can provide optional parameters, such as `days` (for # of days predicted), `major_events` for inputting known upcoming events, etc.
- A menu_item can only belong to 1 category, and a category can only belong to 1 store (strict hierarchy)
- Even if multiple stores have same or similar categories ("red wine"), for purposes of our models, each "red wine" resource will be tied to only 1 store.

## Thought process behind this structure

- Aggregating individual forecasts are less accurate than modeling aggregation directly.
- Example: Getting predictions for all red wine menu items individually (merlot, cabernet, pinot, etc.) and adding them all up, is less accurate than building 1 model for the "red wine" category directly

## Examples

### Case 1: Getting a prediction for a single menu item

Use case: *"I want to predict the unit sales of Corona beers for the next 14 days at my Revolucion store in Guangzhou"*

First, look up which models are built for "corona"

`GET /v1/models?resource_name=corona`

Result

| created_at               | _id                | resource_name  | resource_id | store           | resource_type       | status | output_type |
|--------------------------|-------------------------|-------|---|----------------------|------------|--------|-------------|
| 2023-05-19T10:00:30.000Z | 646c39f586b0765ba0a4f0d6   | Corona| corona_2750rf | Viva La Rev GuangZhou | menu_item  | ready  | unit        |
| 2023-05-19T10:00:30.000Z | 646c39f586b0765ba0a4f0e3   | Corona| corona_8947rf | Viva La Rev Xiamen    | menu_item  | ready  | unit        |
| 2023-05-19T10:00:30.000Z | 646c39f586b0765ba0a4f0f1   | Corona| corona_1457rf | Bacha              | menu_item  | ready  | unit        |
| 2023-05-18T10:00:30.000Z | 646c39f586b0765ba0a4f1a2   | Corona| corona_2750rf | Viva La Rev GuangZhou | menu_item  | failed | unit        |

Then, retrieve predictions from the model you want

`GET /v1/predictions/646c39f586b0765ba0a4f0d6?days=3`

| predicted_date       | _id                | resource_name  | resource_type       | store           | predicted_avg_sales | buffered_prediction | confidence | manual_adjustment | status | created_at               | output_type |
|------------|-------------------------|-------|------------|----------------------|---------------------|---------------------|------------|------------------|--------|--------------------------|-------------|
| 2023-05-24 | 646c39f586b0765ba0a4f0d6   | Corona| menu_item  | Viva La Rev GuangZhou | 23                  | 29                  | 0.95       | 3                | ready  | 2023-05-24T10:00:30.000Z | unit        |
| 2023-05-25 | 646c39f586b0765ba0a4f0d6   | Corona| menu_item  | Viva La Rev GuangZhou | 25                  | 31                  | 0.94       | 4                | ready  | 2023-05-25T10:00:30.000Z | unit        |
| 2023-05-26 | 646c39f586b0765ba0a4f0d6   | Corona| menu_item  | Viva La Rev GuangZhou | 27                  | 33                  | 0.92       | 2                | ready  | 2023-05-26T10:00:30.000Z | unit        |

### Case 2: Getting a prediction for a category of foods

Use case: "I want to predict the total red wine sales in my Guangzhou Viva La Rev store for next 7 days."

`GET /v1/models?resource_name=red_wine`

| created_at               | _id                | resource_name  | resource_id | store           | resource_type       | status | output_type |
|--------------------------|-------------------------|-------|---|----------------------|------------|--------|-------------|
| 2023-05-19T10:00:30.000Z | 646c39f586b0765ba0a4f0d6   | red wine | red_wine_2875dw | Viva La Rev GuangZhou | category  | ready  | revenue        |
| 2023-05-19T10:00:30.000Z | 646c39f586b0765ba0a4f0f1   | red wine-infused steak | red_wine-infused_steak_5729ps | Viva La Rev Xiamen    | menu_item  | ready  | unit        |
| 2023-05-19T10:00:30.000Z | 646c39c086b0765ba0a4f0d4   | Pinot red wine | pinot_red_wine_0174bi | Bacha              | menu_item  | ready  | revenue        |
| 2023-05-18T10:00:30.000Z | 646c39c08620765ba0a3f0d4   | red wine | red_wine_2875dw | Viva La Rev GuangZhou | category  | failed | revenue        |

Then, retrieve predictions from the model you want

`GET /v1/predictions/646c39f586b0765ba0a4f0d6&days=3`

| predicted_date       | _id                | resource_name  | resource_type       | store           | predicted_avg_sales | buffered_prediction | confidence | manual_adjustment | output_type |
|------------|-------------------------|-------|------------|----------------------|---------------------------|---------------------------|------------|------------------|-------------|
| 2023-05-24 | 646c39f586b0765ba0a4f0d6   | red wine | category  | Viva La Rev GuangZhou | 2300                    | 2900                      | 0.95       | 3                | revenue     |
| 2023-05-25 | 646c39f586b0765ba0a4f0d6   | red wine | category  | Viva La Rev GuangZhou | 2500                    | 3100                      | 0.94       | 4                | revenue     |
| 2023-05-26 | 646c39f586b0765ba0a4f0d6   | red wine | category  | Viva La Rev GuangZhou | 2700                    | 3300                      | 0.92       | 2                | revenue     |

### Case 3: Getting a prediction for all sales in a store

I want to predict the total sales in my Guangzhou Viva La Rev store for next 14 days.
`GET /v1/models?resource_name=Viva La Rev`

List of all models trained on Viva La Rev
`GET /v1/predictions/646c39f586b0295ba0a4f0d6`
