<h1> Analyzing Drop in User engangement From Yammer Dataset</h1>
<h2>Problem Statement</h2>
<code>
SELECT DATE_TRUNC('week', e.occurred_at),
       COUNT(DISTINCT e.user_id) AS weekly_active_users
  FROM tutorial.yammer_events e
 WHERE e.event_type = 'engagement'
   AND e.event_name = 'login'
 GROUP BY 1
 ORDER BY 1
</code>

<img align="right" src="https://app.mode.com/modeanalytics/reports/cbb8c291ee96/runs/7925c979521e/viz1/cfcdb6b78885" width="500"/>
<h3> 👨🏻‍💻 About Me </h3>
