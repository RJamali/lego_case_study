# LEGO Recommendation System - Case Study

Author: Ruhollah Jamali
Date: November 2025

## Overview

This project presents a complete recommendation system for an e-commerce platform. I developed three complementary approaches to handle different use cases:

1. Top-Seller Products - identifies best-performing items based on purchase frequency
2. Frequently Bought Together - finds items commonly purchased in the same session
3. Real-Time Personalized Recommendations - provides tailored suggestions using collaborative filtering

The solution uses the Otto RecSys Challenge dataset from Kaggle, which contains approximately 12 million e-commerce sessions with click, cart, and purchase events.

## Why Three Models

Each model serves a specific business purpose:

- Top-sellers work for homepage recommendations and users with no browsing history
- Frequently bought together enables cross-selling on product detail pages
- Collaborative filtering delivers personalized recommendations for returning users

Together, they provide comprehensive coverage of different recommendation scenarios.

## Technical Approach

### Data Processing

I converted the raw JSONL data to Parquet format for better performance and used Polars for data manipulation. The preprocessing pipeline handles event categorization and creates the necessary data structures for each model.

### Top-Seller Model

This is straightforward - I count purchase events for each item and rank them by frequency. The top 100 products form our best-seller list. This model has the highest confidence since it's based on actual purchases.

### Frequently Bought Together Model

I analyze purchase sessions to find items that appear together frequently. The approach:
- Extract sessions containing purchase events
- Generate all item pairs from each session
- Count co-occurrence frequencies
- Filter to keep only meaningful associations

This gives us a co-occurrence matrix that powers the "frequently bought together" recommendations.

### Real-Time Collaborative Filtering

This uses item-item similarity based on user interaction patterns:
- Build a user-item interaction matrix from session data
- Weight different event types (purchases count more than clicks)
- Compute cosine similarity between items
- Use similarity scores to recommend related products

For the prototype, I worked with a sample of the data due to memory constraints. In production, this would scale to the full dataset using distributed computing.

## Model Evaluation

I focused on practical metrics rather than traditional ML metrics:

- Coverage: What percentage of the catalog can we recommend for?
- Diversity: Do we avoid showing the same items repeatedly?
- Data support: Are recommendations based on sufficient evidence?

The top-seller model covers 100 high-confidence products. The frequently bought together model has associations for over 85% of active items. The collaborative filtering model handles the top 5,000 most active items.

## Deployment Architecture

The system is designed to work in a cloud environment:

User requests go through an API gateway to a model service running FastAPI. The service loads pre-trained models from storage and returns recommendations. All endpoints are stateless, which means we can scale horizontally by adding more service instances behind a load balancer.

For production, I would add:
- Redis or similar for caching user sessions
- Proper logging and monitoring
- A/B testing framework
- Scheduled model retraining as new data arrives

## API Design

The notebook includes a unified API class that wraps all three models. Each model has a simple function signature:

- get_top_sellers(n) - returns the top N selling products
- get_frequently_bought_together(item_id, n) - returns N related items
- get_personalized_recommendations(user_history, n) - returns N personalized suggestions

This abstraction makes it easy to swap out model implementations or add new recommendation strategies without changing the API interface.

## Model Versioning

Each trained model saves metadata including version number, creation timestamp, and basic statistics. This supports:
- Tracking which model version is in production
- Rolling back to previous versions if needed
- Comparing performance across versions

## Getting Started

### Prerequisites

- Python 3.8 or higher
- 8GB RAM recommended
- Kaggle API credentials for dataset download

### Installation

Install the required packages:

```
pip install -r requirements.txt
```

Set up Kaggle API credentials by placing kaggle.json in your home directory under .kaggle/

### Running the Notebook

Open the Jupyter notebook and run all cells in order. The notebook will:
- Download the dataset if not already present
- Convert and preprocess the data
- Train all three models
- Generate evaluation metrics
- Display the deployment architecture

The first run takes longer due to data download and conversion. Subsequent runs are faster since processed data is cached.

## Project Structure

The notebook creates three directories:

- data/ - Contains raw and processed datasets
- models/ - Stores trained model artifacts and metadata
- output/ - Intermediate files used during training

All directories are created automatically when you run the notebook.

## Current Limitations

This is a prototype demonstrating feasibility. Some limitations:

- I sampled the data for the collaborative filtering model due to memory constraints
- No online learning or real-time model updates
- Cold-start for new items not fully addressed
- Evaluation is based on proxy metrics rather than live A/B tests

## Future Improvements

For production deployment, I would prioritize:

1. Scale to the full dataset using distributed computing
2. Implement proper A/B testing infrastructure
3. Add content-based filtering for cold-start items
4. Build real-time feature stores for user profiles
5. Add explainability features for recommendations
6. Implement automated model retraining pipelines

## Technical Decisions

Some key choices I made:

Why Polars instead of Pandas? Better performance with large datasets and native Parquet support.

Why item-item over user-user collaborative filtering? Items are more stable than user preferences in e-commerce.

Why three models? They complement each other and cover different use cases. The top-seller fallback ensures we always have something to recommend.

Why sample the data for CF? Memory constraints for the prototype. The architecture supports scaling to full data.

## Dataset Information

The Otto RecSys Challenge dataset includes:
- About 12 million user sessions
- About 1.8 million unique products
- Three event types: clicks, carts, orders
- Approximately 4 weeks of e-commerce activity

This represents realistic e-commerce behavior patterns including browsing, cart additions, and purchases.

## Contact

Ruhollah Jamali
Email: Jamali.Ruhollah@gmail.com


