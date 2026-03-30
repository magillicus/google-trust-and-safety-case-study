-- Q4, 2025 Abuse Rate 
SELECT 
    DATE(p.prompt_timestamp) AS date,
    COUNT(*) AS total_prompts,
    COUNT(CASE WHEN m.violation_type IS NOT NULL THEN 1 END) AS flagged_prompts,
    ROUND(
        COUNT(CASE WHEN m.violation_type IS NOT NULL THEN 1 END) * 1.0 / COUNT(*),
        4
    ) AS abuse_rate
FROM prompts p
JOIN model_outputs m 
    ON p.prompt_id = m.prompt_id
GROUP BY DATE(p.prompt_timestamp)
ORDER BY date;

-- Violation type breakdown
SELECT 
    violation_type,
    COUNT(*) AS number_of_cases
FROM model_outputs
WHERE violation_type IS NOT NULL
GROUP BY violation_type
ORDER BY number_of_cases DESC;

-- Single vs Repeat Offenders
WITH user_violations AS (
    SELECT 
        p.user_id,
        COUNT(*) AS violations
    FROM prompts p
    JOIN model_outputs m 
        ON p.prompt_id = m.prompt_id
    WHERE m.violation_type IS NOT NULL
    GROUP BY p.user_id
)
SELECT 
    CASE 
        WHEN violations = 1 THEN 'Single Offender'
        ELSE 'Repeat Offender'
    END AS offender_type,
    COUNT(*) AS user_count
FROM user_violations
GROUP BY offender_type;

-- Detection time frames/bins
SELECT 
    CASE 
        WHEN hours_to_detection < 1 THEN '<1 hour'
        WHEN hours_to_detection < 3 THEN '1-3 hours'
        WHEN hours_to_detection < 6 THEN '3-6 hours'
        WHEN hours_to_detection < 12 THEN '6-12 hours'
        ELSE '12+ hours'
    END AS detection_bin,
    COUNT(*) AS case_count
FROM (
    SELECT 
        (JULIANDAY(m.detected_at) - JULIANDAY(p.prompt_timestamp)) * 24 AS hours_to_detection
    FROM prompts p
    JOIN model_outputs m 
        ON p.prompt_id = m.prompt_id
    WHERE m.detected_at IS NOT NULL
)
GROUP BY detection_bin
ORDER BY 
    CASE detection_bin
        WHEN '<1 hour' THEN 1
        WHEN '1-3 hours' THEN 2
        WHEN '3-6 hours' THEN 3
        WHEN '6-12 hours' THEN 4
        ELSE 5
    END;