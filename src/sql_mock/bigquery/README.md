# Use SQL Mock with Google BigQuery

## Settings:

Ensure you have the `GOOGLE_APPLICATION_CREDENTIALS` environment variable correctly set to point to a service account file.

## Example: Testing Subscription Counts in BigQuery

```python
import datetime
from sql_mock.bigquery import column_mocks as col
from sql_mock.bigquery.table_mocks import BigQueryMockTable

# Define mock tables for your data model that inherit from BigQueryMockTable
class UserTable(BigQueryMockTable):
    user_id = col.Int(default=1)
    user_name = col.String(default='Mr. T')

class SubscriptionTable(BigQueryMockTable):
    subscription_id = col.Int(default=1)
    period_start_date = col.Date(default=datetime.date(2023, 9, 5))
    period_end_date = col.Date(default=datetime.date(2023, 9, 5))
    user_id = col.Int(default=1)

# Define a mock table for your expected results
class SubscriptionCountTable(BigQueryMockTable):
    subscription_count = col.Int(default=1)
    user_id = col.Int(default=1)

# Your original SQL query
query = """
SELECT
    count(*) AS subscription_count,
    user_id
FROM data.users
LEFT JOIN data.subscriptions USING(user_id)
GROUP BY user_id
"""

# Create mock data for the 'data.users' and 'data.subscriptions' tables
users = UserTable(data=[{'user_id': 1}, {'user_id': 2}])
subscriptions = SubscriptionTable(
    data=[
        {'subscription_id': 1, 'user_id': 1},
        {'subscription_id': 2, 'user_id': 1},
        {'subscription_id': 2, 'user_id': 2},
    ]
)

# Define your expected results
expected = [
    {'user_id': 1, 'subscription_count': 2}, 
    {'user_id': 2, 'subscription_count': 1}
]

# Simulate the SQL query using SQL Mock
res = SubscriptionCountTable.from_inputs(
    query=query, 
    input_data={
        'data.users': users, 
        'data.subscriptions': subscriptions
    }
)

# Assert the results
res.assert_equal(expected)
```