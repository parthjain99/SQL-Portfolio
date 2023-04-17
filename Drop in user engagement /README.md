<h1> Analyzing Drop in User engangement From Yammer Dataset</h1>
<h2>Problem Statement</h2>

<code>SELECT DATE_TRUNC('week', e.occurred_at),
       COUNT(DISTINCT e.user_id) AS weekly_active_users
  FROM tutorial.yammer_events e
 WHERE e.event_type = 'engagement'
   AND e.event_name = 'login'
 GROUP BY 1
 ORDER BY 1</code>
 
<!--  <iframe src="https://drive.google.com/file/d/1jj3Z03FQnXoLX_O8zZ8nog2sre4Walse/preview" width="640" height="480" allow="autoplay"></iframe>
 -->
<img src="https://drive.google.com/file/d/1jj3Z03FQnXoLX_O8zZ8nog2sre4Walse/view"/>
