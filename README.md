# Production-Forecasting
Python-based production forecasting tool for pharmaceutical manufacturing. Calculates order completion dates, machine capacities, and on-time delivery status from CSV data.

Production Forecast:

import pandas as pd
from datetime import timedelta


# 1. Load CSV files

orders = pd.read_csv("Orders.csv")
production = pd.read_csv("Production.csv")
batches = pd.read_csv("Batch.csv")
inventory = pd.read_csv("Inventory.csv")
workforce = pd.read_csv("Workforce.csv")
downtime = pd.read_csv("Downtime.csv")


# 2. Calculate Effective Machine Capacity
production["Daily_Capacity"] = (production["Capacity_Per_Hour"] *
                               production["Shift_Hours"] *
                               production["No_of_Shifts"]) - production["Downtime_Hours"]

capacity_per_product = production.groupby("Product")["Daily_Capacity"].sum().reset_index()
capacity_per_product.rename(columns={"Daily_Capacity": "Total_Daily_Capacity"}, inplace=True)


# 3. Merge Orders with Capacity

orders_capacity = orders.merge(capacity_per_product, on="Product", how="left")


# 4. Calculate Forecasted Completion Time

orders_capacity["Days_Required"] = (orders_capacity["Quantity_Ordered"] /
                                    orders_capacity["Total_Daily_Capacity"]).round(1)

orders_capacity["Start_Date"] = pd.to_datetime("2025-09-08")  # Assume today
orders_capacity["Forecasted_Completion"] = orders_capacity["Start_Date"] + \
    orders_capacity["Days_Required"].apply(lambda x: timedelta(days=x))


# 5. Check if Orders Meet Deadline

orders_capacity["On_Time"] = orders_capacity["Forecasted_Completion"] <= orders_capacity["Delivery_Deadline"]


# 6. Display Final Forecast

print("\n📊 Production Forecast Summary:")
print(orders_capacity[["Order_ID", "Product", "Quantity_Ordered",
                       "Delivery_Deadline", "Total_Daily_Capacity",
                       "Days_Required", "Forecasted_Completion", "On_Time"]])


Result:

📊 Production Forecast Summary:
   Order_ID      Product  Quantity_Ordered Delivery_Deadline  Total_Daily_Capacity  Days_Required Forecasted_Completion  On_Time
0       101  Paracetamol            500000        2025-10-15               351994            1.4   2025-09-09 09:36:00     True
1       102  Amoxicillin            300000        2025-10-20                79999            3.8   2025-09-11 19:12:00     True
2       103  Cough_Syrup            200000        2025-11-01                31998            6.3   2025-09-14 07:12:00     True




# 📊 Production Forecasting Tool (Demo)

This is a Python-based demo tool for pharmaceutical manufacturing that forecasts order completion dates. 

It calculates:
- Effective machine capacities per product
- Estimated days required for each order
- Whether orders can be completed on time according to delivery deadlines

⚠️ Note: This repository uses **small demo data** for demonstration purposes. For real production use, replace the sample CSV files with actual production and order data.
