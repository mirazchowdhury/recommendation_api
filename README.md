# Retail Item Recommendation API

A Flask based retail item recommendation API that serves ranked product suggestions for a customer basket. The system uses an XGBoost ranking model, category rule artifacts, customer history, item co purchase signals, context features, category embeddings, and optional NGCF candidate scores to return top product recommendations.

## Project Summary

This repository provides a deployable API layer for an item recommendation system. The API receives a customer id, purchase time, optional time slot, and a list of basket items. It then builds a live recommendation context, creates a candidate item pool, scores candidates with a trained XGBoost ranker, applies rule based reranking, and returns a ranked list of recommended items.

The repository also includes several notebook files that appear to build the offline training and artifact generation pipeline. These notebooks cover category labeling, item catalog creation, stage one rule artifact creation, NGCF candidate model preparation, XGBoost ranker training, and runtime artifact creation.

## Main Use Case

This API is useful for grocery, retail, online shopping, and sales systems where the application needs to recommend related items after a customer adds products to a basket.

Example use cases:

1. Recommend cooking ingredients when a customer buys fish or meat.
2. Recommend breakfast items when a customer buys bread in the morning.
3. Recommend clothing related items from the same family group.
4. Recommend popular items based on season, time slot, week of month, and customer history.
5. Combine machine learning ranker output with business rules for safer recommendation results.

## Key Features

1. Flask API with app factory structure.
2. Swagger support through Flasgger.
3. Health checking endpoint.
4. Metrics endpoint for reading final model result summary.
5. Recommendation endpoint for live basket based prediction.
6. XGBoost ranker model loading from JSON.
7. Runtime feature alignment with saved training feature columns.
8. Candidate generation from multiple signals.
9. Rule based allowed category filtering.
10. Category family based recommendation diversity.
11. Context aware features using season, time slot, month part, and week of month.
12. Customer history and category affinity scoring.
13. Item co purchase scoring.
14. Category embedding similarity scoring.
15. Optional NGCF candidate score and rank support.
16. Stable reranking to avoid repeated items from the same category.
17. JSON based request and response format.

## Repository Structure

```text
recommendation_api
    app.py
    config.py
    requirements.txt
    README.md
    .gitignore

    app
        __init__.py
        extensions.py

        routes
            __init__.py
            recommendation_routes.py

        services
            __init__.py
            recommender_service.py

        utils
            __init__.py
            helpers.py

    (11)_updated retraining notebook.ipynb
    (2)_category_label_and_basket_embedding.ipynb.ipynb
    (3)_item_catalog_create.ipynb.ipynb
    (4)_Stage_1_item_catalog_and_category_rules_create.ipynb.ipynb
    (5)_ngcf_candidate_model_create.ipynb
    (6)_ranker_training_and_xgboost_model_create_dynamic_updated.ipynb
    (7)_stage1_runtime_artifacts_create.ipynb.ipynb
```

## Folder and File Details

### app.py

The main entry point of the Flask project. It imports `create_app`, creates the Flask app instance, and runs the server on host `0.0.0.0` and port `5000`.

### config.py

Stores project configuration, model artifact paths, data paths, and Swagger title information.

Important configuration values:

```python
BASE_DIR = Path(r"D:\recommendation_api")
MODEL_ASSETS_DIR = BASE_DIR / "model_assets"
MODEL_DIR = MODEL_ASSETS_DIR / "model_outputs_v2"
DATA_DIR = MODEL_ASSETS_DIR / "data"
STAGE1_ARTIFACT_DIR = MODEL_ASSETS_DIR / "stage1_runtime_artifacts"
```

The current code uses a Windows absolute path. Before running on another machine, update `BASE_DIR` to the local repository path or convert it to a relative path.

### app package

Contains the Flask application factory, Swagger extension, API routes, service layer, and helper functions.

### app routes

The `recommendation_routes.py` file defines the API routes.

Routes included:

1. `GET /api/health`
2. `GET /api/metrics`
3. `POST /api/recommend`

### app services

The `recommender_service.py` file contains the main recommendation logic. It loads the trained ranker model, item catalog, category rules, runtime counters, NGCF candidate scores, and performs candidate scoring.

### app utils

The `helpers.py` file contains utility functions for loading JSON, loading pickle files, cleaning text, inferring season, inferring time slot, computing cosine similarity, and mean pooling vectors.

## Recommendation Pipeline

The live recommendation flow works as follows:

1. The client sends customer id, basket items, purchase date and time, and optional time slot.
2. The API validates the request body.
3. The service extracts customer id and input item ids.
4. The date and time value is converted into context features.
5. The system infers season, time slot, month part, and week of month.
6. The input item ids are mapped to item categories.
7. Business rules select allowed target categories.
8. Candidate items are collected from co purchase history, context popularity, user history, NGCF candidates, and allowed category catalog.
9. Already selected basket items are removed from the candidate pool.
10. The service builds numeric and categorical live features for every candidate.
11. The trained XGBoost ranker predicts a score for each candidate.
12. Rule based reranking adds category rule weight, NGCF score, and category rank influence.
13. The API returns the top recommended items with display scores.

## Required Model Assets

The API needs several files before it can run successfully.

Expected model output files:

```text
model_assets
    model_outputs_v2
        xgboost_ranker_model.json
        ranker_feature_columns.json
        ranker_training_summary.json
        final_result_summary.csv
```

Expected data files:

```text
model_assets
    data
        main_data.csv
        item_catalog.csv
        category_rule_artifacts.json
        category_embedding_lookup.csv
        ngcf_top_candidates.csv
```

Expected stage one runtime artifact files:

```text
model_assets
    stage1_runtime_artifacts
        item_pair_counter.pkl
        context_item_counter.pkl
        user_item_counter.pkl
        user_category_counter.pkl
        category_popularity_counter.pkl
        pair_to_related_items.pkl
        context_to_items.pkl
        user_to_items.pkl
        customer_default_timeslot.pkl
        item_to_category.pkl
        item_to_name.pkl
        category_to_vector.pkl
        category_family_map.pkl
```

The `category_family_map.pkl` file is optional. If it is missing, the service tries to use `category_family_map` from the category rule artifact JSON.

## Technology Stack

1. Python
2. Flask
3. Flasgger
4. XGBoost
5. Pandas
6. NumPy
7. Scikit learn
8. JSON and Pickle based runtime artifacts

## Installation

Clone the repository.

```bash
git clone https://github.com/mirazchowdhury/recommendation_api.git
cd recommendation_api
```

Create a virtual environment.

```bash
python -m venv .venv
```

Activate the virtual environment on Windows.

```bash
.venv\Scripts\activate
```

Activate the virtual environment on Linux or macOS.

```bash
source .venv/bin/activate
```

Upgrade pip.

```bash
python -m pip install --upgrade pip
```

Install dependencies.

```bash
pip install -r requirements.txt
```

## Dependency List

The repository requirements file contains:

```text
Flask
flasgger
xgboost
pandas
numpy
scikit-learn
```

## Configuration Before Running

Open `config.py` and update the base path.

Current style:

```python
BASE_DIR = Path(r"D:\recommendation_api")
```

Recommended local style:

```python
BASE_DIR = Path(__file__).resolve().parent
```

Then keep the artifact folders inside the repository root.

Expected local structure:

```text
recommendation_api
    model_assets
        data
        model_outputs_v2
        stage1_runtime_artifacts
```

## How to Run

Run the Flask API from the repository root.

```bash
python app.py
```

The server runs at:

```text
http://127.0.0.1:5000
```

For network access inside a local area network, use:

```text
http://YOUR_LOCAL_IP:5000
```

## Swagger Documentation

The API uses Flasgger. After running the server, open:

```text
http://127.0.0.1:5000/apidocs
```

Swagger can be used to test the recommendation endpoint directly from the browser.

## API Endpoints

## 1. Health Check

### Endpoint

```text
GET /api/health
```

### Purpose

Checks whether the API is running.

### Response Example

```json
{
    "success": true,
    "message": "API is running"
}
```

## 2. Model Metrics

### Endpoint

```text
GET /api/metrics
```

### Purpose

Reads `final_result_summary.csv` and returns the saved model result summary.

### Success Response

```json
{
    "success": true,
    "metrics": [
        {
            "metric_name": "example",
            "value": 0.95
        }
    ]
}
```

### Error Response

```json
{
    "success": false,
    "message": "final_result_summary.csv not found"
}
```

## 3. Get Recommendations

### Endpoint

```text
POST /api/recommend
```

### Purpose

Returns top recommended items for a customer basket.

### Request Body

```json
{
    "customerid": 23412,
    "date and time": "2026-04-14 05:00:00",
    "timeSlot": "Afternoon",
    "items": [
        {
            "itemid": 13989,
            "quantity": 1
        }
    ]
}
```

### Required Fields

1. `customerid`
2. `date and time`
3. `items`

### Optional Fields

1. `timeSlot`

If `timeSlot` is not provided, the API infers the time slot from the hour in `date and time`.

### Response Example

```json
{
    "input_item_names": [
        "Example Input Item"
    ],
    "recommendations": [
        {
            "category": "Beverage-Hot",
            "item_name": "Example Recommended Item",
            "itemid": 1001,
            "score": 1.0
        }
    ]
}
```

## Request Validation

The API checks the following conditions:

1. Request body must be valid JSON.
2. `customerid`, `date and time`, and `items` must exist.
3. `items` must be a non empty list.
4. Every input item should contain an item id and quantity.
5. Date and time must be parseable by Pandas.

## Feature Engineering Used at Runtime

The service builds the following feature groups:

### Basket Features

1. Basket size.
2. Input item ids.
3. Input item categories.
4. Input category family count.
5. Mixed basket flag.

### Customer Features

1. Customer item history score.
2. Customer category affinity score.
3. NGCF score.
4. NGCF rank.

### Item Relationship Features

1. Item co purchase score.
2. Pair related item score.
3. Context item popularity score.

### Category Rule Features

1. Allowed category rank.
2. Candidate category allowed flag.
3. Same category as context flag.
4. Category rule weight.
5. Candidate family in input flag.

### Context Features

1. Season.
2. Season label.
3. Time slot.
4. Time slot label.
5. Holiday flag.
6. Festival flag.
7. Month part label.
8. Week of month.

### Embedding Features

1. Category embedding similarity score.
2. Mean pooled context category vector.
3. Candidate category vector similarity.

## Time Slot Logic

The helper function maps hour values to time slots.

```text
6 to 11       Morning
12 to 13      Noon
14 to 16      Afternoon
17 to 20      Evening
Other hours   Night
```

## Season Logic

The helper function maps month values to seasons.

```text
November, December, January, February    Winter
March, April, May, June                  Summer
Other months                             Rainy
```

## Candidate Generation Sources

The candidate pool is built from:

1. Co purchased items related to the input basket.
2. Context based popular items.
3. Customer purchase history.
4. NGCF candidate items.
5. Items from allowed business rule categories.

## Reranking Logic

After the XGBoost ranker predicts candidate scores, the service applies additional reranking.

It adds influence from:

1. NGCF score.
2. Category rule weight.
3. Allowed category rank.
4. Category diversity.
5. Category family quota.

This helps the final recommendation list remain useful, business rule aware, and less repetitive.

## Example Python Client

```python
import requests

url = "http://127.0.0.1:5000/api/recommend"

payload = {
    "customerid": 23412,
    "date and time": "2026-04-14 05:00:00",
    "timeSlot": "Afternoon",
    "items": [
        {
            "itemid": 13989,
            "quantity": 1
        }
    ]
}

response = requests.post(url, json=payload)
print(response.json())
```

## Common Errors and Fixes

### final_result_summary.csv not found

Cause:

The metrics endpoint cannot find the model result summary file.

Fix:

Place `final_result_summary.csv` inside:

```text
model_assets/model_outputs_v2
```

### ranker_feature_columns.json has no final feature columns

Cause:

The feature file is empty or not in the expected JSON format.

Fix:

Regenerate the ranker feature columns file from the training notebook.

### date parse failed

Cause:

The `date and time` value is not readable by Pandas.

Fix:

Use this format:

```text
2026-04-14 05:00:00
```

### Model file not found

Cause:

The XGBoost model file is missing.

Fix:

Place `xgboost_ranker_model.json` inside:

```text
model_assets/model_outputs_v2
```

### Wrong BASE_DIR path

Cause:

`config.py` points to a local Windows path that does not exist on the current machine.

Fix:

Update `BASE_DIR` to the correct local path.

## Notebook Workflow

The repository includes notebook files for preparing the recommendation system artifacts.

Suggested notebook order:

1. Category label and basket embedding notebook.
2. Item catalog creation notebook.
3. Stage one item catalog and category rule artifact creation notebook.
4. NGCF candidate model creation notebook.
5. Ranker training and XGBoost model creation notebook.
6. Stage one runtime artifact creation notebook.
7. Updated retraining notebook.

After running the notebooks, copy the generated files into the expected `model_assets` folders.

## Suggested Local Setup Checklist

1. Clone the repository.
2. Create and activate a virtual environment.
3. Install dependencies.
4. Generate or copy model assets.
5. Update `BASE_DIR` in `config.py`.
6. Run `python app.py`.
7. Test `GET /api/health`.
8. Open Swagger at `/apidocs`.
9. Test `POST /api/recommend`.
10. Check `GET /api/metrics` if result CSV exists.

## Current Limitations

1. The base directory is hardcoded to a local Windows path.
2. Model assets are ignored by Git, so they must be prepared separately.
3. There is no authentication layer for API access.
4. There is no database connection in the current API layer.
5. Request validation is basic.
6. There is no Dockerfile in the repository.
7. There are no automated API tests.
8. Error messages are returned directly from exceptions in some cases.

## Suggested Improvements

1. Replace hardcoded `BASE_DIR` with environment variable based configuration.
2. Add `.env.example`.
3. Add Docker support.
4. Add API token based authentication.
5. Add request schema validation.
6. Add unit tests and integration tests.
7. Add logging for model loading, candidate generation, and prediction latency.
8. Add sample model asset folder structure with dummy files.
9. Add a small sample item catalog for demo testing.
10. Add production server instructions using Gunicorn or Waitress.

## Production Notes

For production use, consider:

1. Running the API with a production server instead of Flask debug server.
2. Loading model artifacts once during application startup.
3. Adding API access control.
4. Adding request logging.
5. Adding response time monitoring.
6. Adding a health check that verifies model asset availability.
7. Versioning the model and feature files.
8. Keeping training artifacts and serving artifacts clearly separated.

## License

No explicit license file was visible in the repository at the time this README was written. Add a license file before public or commercial use.

## Author

Miraz Uddin Chowdhury

Repository:

```text
https://github.com/mirazchowdhury/recommendation_api
```
