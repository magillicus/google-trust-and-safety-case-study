-- Core KPIs
SELECT 
    COUNT(*) AS total_prompts,
    COUNT(CASE WHEN m.violation_type IS NOT NULL THEN 1 END) AS flagged_prompts,
    ROUND(COUNT(CASE WHEN m.violation_type IS NOT NULL THEN 1 END) * 1.0 / COUNT(*), 4) AS abuse_rate,
    ROUND(COUNT(CASE WHEN r.composite_risk_score >= 0.7 THEN 1 END) * 1.0 / COUNT(*), 4) AS high_risk_prompt_rate,
    ROUND(COUNT(CASE WHEN m.escalated_at IS NOT NULL THEN 1 END) * 1.0 / COUNT(*), 4) AS escalation_rate,
    ROUND(AVG(
        CASE 
            WHEN m.detected_at IS NOT NULL 
            THEN (JULIANDAY(m.detected_at) - JULIANDAY(p.prompt_timestamp)) * 24
        END
    ), 2) AS avg_hours_to_detection
FROM prompts p
JOIN model_outputs m ON p.prompt_id = m.prompt_id
JOIN risk_signals r ON p.prompt_id = r.prompt_id;

-- Repeat Offender Rate
WITH flagged_users AS (
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
    ROUND(
        COUNT(CASE WHEN violations > 1 THEN 1 END) * 1.0 / COUNT(*),
        4
    ) AS repeat_offender_rate
FROM flagged_users;