---
title: mysql关联更新与删除
date: 2021-12-23 13:29:39
tags: mysql
categories: 数据库
---

#### 1.更新子查询语法：
```
UPDATE ycd_t_match_term t
INNER JOIN (
    SELECT r.`term_id`,
        DATE_ADD(
        MAX(m.`start_date`),
        INTERVAL 4 HOUR
        ) max_start_date
    FROM ycd_t_term_match_relation r, ycd_t_match m
    WHERE r.`match_id` = m.id
    GROUP BY r.`term_id`
) a ON a.term_id = t.`id`
SET t.`settlement_date` = a.max_start_date
WHERE t.`settlement_date` IS NULL;
```

#### 2.删除子查询语法：
```
DELETE d FROM ycd_t_user_activity d
WHERE d.id IN (
SELECT id
FROM
(
    SELECT a.id
    FROM ycd_t_user_activity a
    WHERE a.id != (
        SELECT MIN(e.id)
        FROM ycd_t_user_activity e
        WHERE e.`user_id` = a.user_id)
    ) AS X
);
```