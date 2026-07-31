# Margin-Watch-E-Commerce

# margin-watch-ecommerce-kpi-dashboard/

```
margin-watch-ecommerce-kpi-dashboard/
├── data/
│   ├── raw/
│   │   └── ecommerce_portfolio_raw_data.xlsx   # Raw data, untouched
│   ├── cleaned/
│   │   └── ecommerce_cleaned.xlsx              # Cleaned data
│   └── data_dictionary.md                      # Description of all columns in the data
│
├── notebooks_or_workbook/
│   └── analysis_workbook.xlsx                  # Working file: pivot tables, calculated
│                                                # columns, exploration charts (built from
│                                                # the cleaned data)
│
├── outputs/
│   ├── charts/
│   │   └── *.png                               # Exported PNGs of the best charts
│   └── margin_watch_dashboard.xlsx              # Final polished dashboard — KPIs,
│                                                 # slicers, no scratch work
│
├── report/
│   └── findings_summary.md                     # Write-up of key findings
│
├── PROJECT_BRIEF.md                             # Project scope, goals, questions
└── README.md                                    # Overview & how to navigate the repo
```