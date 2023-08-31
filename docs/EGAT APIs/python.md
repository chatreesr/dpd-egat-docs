# Example - Python

ตัวอย่างการดึงข้อมูลผ่านภาษา Python

```python
import requests
import pandas as pd
import urllib
      
response = requests.get(
    url='http://172.17.113.142:8080/billing',
    headers={
        'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoibWVhX2FwcCJ9.YQeO5hci1qAlSCKQXgGje7WBMj1CXBJtlXO_EEeTdtY'
    },
    params={
        'fisc_period': 'eq.2023-07', # Use the same syntax as PostgREST
        'line_feeder_name': 'eq.BN/69 MEA#2 M' # requests package will handle URL encoding for you
    }
)

# Print status code
print(response.status_code)

# Import data as a Pandas DataFrame
billing = pd.DataFrame.from_records(response.json())

# Print the DataFrame information
print(billing.info())

# Print some data
print(billing.head())
```