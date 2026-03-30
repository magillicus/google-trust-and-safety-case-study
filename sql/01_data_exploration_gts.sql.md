-- Row counts
SELECT 'users' AS table_name, COUNT(*) AS row_count FROM users
UNION ALL
SELECT 'prompts', COUNT(*) FROM prompts
UNION ALL
SELECT 'risk_signals', COUNT(*) FROM risk_signals
UNION ALL
SELECT 'model_outputs', COUNT(*) FROM model_outputs;

--Nulls (model_outputs)
SELECT
    SUM(CASE WHEN violation_type IS NULL THEN 1 ELSE 0 END) AS violation_type_nulls,
    SUM(CASE WHEN detected_at IS NULL THEN 1 ELSE 0 END) AS detected_at_nulls,
    SUM(CASE WHEN escalated_at IS NULL THEN 1 ELSE 0 END) AS escalated_at_nulls,
    SUM(CASE WHEN review_outcome IS NULL THEN 1 ELSE 0 END) AS review_outcome_nulls
FROM model_outputs;

-- Distinct violation categories
SELECT DISTINCT violation_type
FROM model_outputs
WHERE violation_type IS NOT NULL
ORDER BY violation_type;