
```python

# TODO: import the required dependencies
import pandas as pd

# Write code here
sql_query = """
    SELECT country, revenue 
    FROM table_3196ce98cf3ce3223735e1d61022e9d2 
    ORDER BY revenue DESC 
    LIMIT 5;
"""
top_countries_df = execute_sql_query(sql_query)

# Declare result var
result = {
    "type": "dataframe",
    "value": top_countries_df
}
```