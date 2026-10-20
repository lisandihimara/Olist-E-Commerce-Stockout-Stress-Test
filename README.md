# Dataset Instructions

## Source

Olist Brazilian E-Commerce Public Dataset, available on Kaggle.

Dataset page:
https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

## Required Files

Download the dataset and place these files inside `data/raw/`:

- `olist_orders_dataset.csv`
- `olist_order_items_dataset.csv`

## Data Handling

The raw CSV files are excluded from Git tracking because datasets
can be large and should be obtained from the original source.

This project analyses delivered-order item records and does not
directly observe inventory levels, actual stockouts, or lost demand.
