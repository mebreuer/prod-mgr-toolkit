# Preset Query Patterns

Common SQL patterns for enhancing PRDs with data from Preset. Use these as starting points — adapt table/column names to match your actual data warehouse schema.

## Tips for Preset MCP Usage

1. **Start with dashboards:** Use `preset_get_dashboards` to find existing dashboards relevant to the feature area. Often the data you need already exists.
2. **Check datasets:** Use `preset_get_datasets` to understand available tables and columns before writing custom queries.
3. **Use `preset_query`** for custom SQL when existing charts don't cover your needs.
4. **Limit results:** Always include `LIMIT` clauses — Preset will timeout on unbounded queries.
5. **Date ranges:** Default to last 90 days unless the PRD requires a different window.

## Feature Usage Patterns

### Daily/Weekly/Monthly Active Users for a Feature
```sql
-- Adapt event_name and properties to your tracking schema
SELECT
  DATE_TRUNC('week', event_timestamp) AS week,
  COUNT(DISTINCT user_id) AS unique_users,
  COUNT(*) AS total_events
FROM events
WHERE event_name = '{feature_event}'
  AND event_timestamp >= CURRENT_DATE - INTERVAL '90 days'
GROUP BY 1
ORDER BY 1;
```

### Feature Adoption Over Time (Cumulative)
```sql
SELECT
  first_use_date,
  COUNT(*) AS new_adopters,
  SUM(COUNT(*)) OVER (ORDER BY first_use_date) AS cumulative_adopters
FROM (
  SELECT
    user_id,
    MIN(DATE(event_timestamp)) AS first_use_date
  FROM events
  WHERE event_name = '{feature_event}'
  GROUP BY 1
) first_uses
GROUP BY 1
ORDER BY 1;
```

### Feature Usage Frequency Distribution
```sql
SELECT
  usage_bucket,
  COUNT(*) AS user_count
FROM (
  SELECT
    user_id,
    CASE
      WHEN cnt = 1 THEN '1 time'
      WHEN cnt BETWEEN 2 AND 5 THEN '2-5 times'
      WHEN cnt BETWEEN 6 AND 20 THEN '6-20 times'
      ELSE '20+ times'
    END AS usage_bucket
  FROM (
    SELECT user_id, COUNT(*) AS cnt
    FROM events
    WHERE event_name = '{feature_event}'
      AND event_timestamp >= CURRENT_DATE - INTERVAL '90 days'
    GROUP BY 1
  ) usage
) bucketed
GROUP BY 1
ORDER BY MIN(CASE usage_bucket
  WHEN '1 time' THEN 1
  WHEN '2-5 times' THEN 2
  WHEN '6-20 times' THEN 3
  ELSE 4
END);
```

## Funnel & Conversion Patterns

### Step-by-Step Funnel
```sql
WITH funnel AS (
  SELECT
    COUNT(DISTINCT CASE WHEN event_name = '{step_1_event}' THEN user_id END) AS step_1,
    COUNT(DISTINCT CASE WHEN event_name = '{step_2_event}' THEN user_id END) AS step_2,
    COUNT(DISTINCT CASE WHEN event_name = '{step_3_event}' THEN user_id END) AS step_3
  FROM events
  WHERE event_timestamp >= CURRENT_DATE - INTERVAL '90 days'
)
SELECT
  step_1,
  step_2,
  step_3,
  ROUND(100.0 * step_2 / NULLIF(step_1, 0), 1) AS step_1_to_2_pct,
  ROUND(100.0 * step_3 / NULLIF(step_2, 0), 1) AS step_2_to_3_pct,
  ROUND(100.0 * step_3 / NULLIF(step_1, 0), 1) AS overall_conversion_pct
FROM funnel;
```

### Drop-off Analysis
```sql
SELECT
  last_completed_step,
  COUNT(*) AS users_dropped,
  ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (), 1) AS pct_of_dropoffs
FROM (
  SELECT
    user_id,
    CASE
      WHEN completed_step_3 THEN 'completed'
      WHEN completed_step_2 THEN 'step_2'
      WHEN completed_step_1 THEN 'step_1'
      ELSE 'never_started'
    END AS last_completed_step
  FROM (
    SELECT
      user_id,
      BOOL_OR(event_name = '{step_1_event}') AS completed_step_1,
      BOOL_OR(event_name = '{step_2_event}') AS completed_step_2,
      BOOL_OR(event_name = '{step_3_event}') AS completed_step_3
    FROM events
    WHERE event_timestamp >= CURRENT_DATE - INTERVAL '90 days'
    GROUP BY 1
  ) steps
  WHERE completed_step_1  -- only users who entered the funnel
) dropoffs
WHERE last_completed_step != 'completed'
GROUP BY 1
ORDER BY users_dropped DESC;
```

## Error & Quality Patterns

### Error Rate by Feature
```sql
SELECT
  DATE_TRUNC('day', event_timestamp) AS day,
  COUNT(*) FILTER (WHERE event_name = '{error_event}') AS errors,
  COUNT(*) FILTER (WHERE event_name = '{attempt_event}') AS attempts,
  ROUND(100.0 *
    COUNT(*) FILTER (WHERE event_name = '{error_event}') /
    NULLIF(COUNT(*) FILTER (WHERE event_name = '{attempt_event}'), 0)
  , 2) AS error_rate_pct
FROM events
WHERE event_name IN ('{error_event}', '{attempt_event}')
  AND event_timestamp >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY 1
ORDER BY 1;
```

### Top Error Types
```sql
SELECT
  error_type,
  COUNT(*) AS occurrences,
  COUNT(DISTINCT user_id) AS affected_users,
  ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (), 1) AS pct_of_errors
FROM events
WHERE event_name = '{error_event}'
  AND event_timestamp >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY 1
ORDER BY occurrences DESC
LIMIT 10;
```

## Opportunity Sizing Patterns

### Total Addressable Users
```sql
SELECT
  COUNT(DISTINCT user_id) AS total_active_users,
  COUNT(DISTINCT CASE WHEN {eligibility_condition} THEN user_id END) AS eligible_users,
  ROUND(100.0 *
    COUNT(DISTINCT CASE WHEN {eligibility_condition} THEN user_id END) /
    NULLIF(COUNT(DISTINCT user_id), 0)
  , 1) AS eligible_pct
FROM users
WHERE last_active_date >= CURRENT_DATE - INTERVAL '30 days';
```

### Revenue Impact Estimation
```sql
SELECT
  COUNT(DISTINCT user_id) AS affected_users,
  SUM(revenue) AS current_revenue,
  AVG(revenue) AS avg_revenue_per_user,
  -- Estimate impact with assumed lift
  SUM(revenue) * 0.10 AS estimated_10pct_lift,
  SUM(revenue) * 0.25 AS estimated_25pct_lift
FROM user_revenue
WHERE {segment_condition}
  AND period >= CURRENT_DATE - INTERVAL '90 days';
```

## Segment Analysis Patterns

### Feature Usage by User Segment
```sql
SELECT
  u.segment,
  COUNT(DISTINCT e.user_id) AS users_with_feature_use,
  COUNT(DISTINCT u.user_id) AS total_segment_users,
  ROUND(100.0 *
    COUNT(DISTINCT e.user_id) /
    NULLIF(COUNT(DISTINCT u.user_id), 0)
  , 1) AS adoption_pct
FROM users u
LEFT JOIN events e
  ON u.user_id = e.user_id
  AND e.event_name = '{feature_event}'
  AND e.event_timestamp >= CURRENT_DATE - INTERVAL '90 days'
WHERE u.last_active_date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY 1
ORDER BY adoption_pct DESC;
```

## Workflow

1. **Identify relevant dashboards** — `preset_get_dashboards` → scan for feature area
2. **Pull existing chart data** — `preset_get_chart_data` for relevant charts
3. **Check available datasets** — `preset_get_datasets` → understand schema
4. **Run custom queries** — `preset_query` with adapted patterns above
5. **Summarize findings** — extract key numbers for the PRD evidence table
