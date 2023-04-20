<h1> Analyzing Drop in User engangement From Yammer Dataset</h1>

<h2>Problem Statement</h2>
<p>At the later half of August, Yammer detected a drop in overall user engagement through a graph that looks like this</p>
<h3>Query</h3>
<pre><code>
SELECT DATE_TRUNC('week', e.occurred_at) as weeks,
COUNT(DISTINCT e.user_id) AS weekly_active_users
FROM yammer_events e
WHERE e.event_type = 'engagement'
      AND e.event_name = 'login'
GROUP BY weeks
ORDER BY weeks
 </code></pre>
 

<img src="https://github.com/parthjain99/SQL-Portfolio/blob/438312486ab85fb71c049a859c4709674530ce54/Drop%20in%20user%20engagement%20/Graphs/DECRESE%20IN%20ACTIVE%20USERS.png"/>

<h3>Task</h3>
<p>My next step is to explore the reason behind the abrupt decrease in engagement and propose potential solutions to address it. To accomplish this, I will initially brainstorm and analyze the potential causes of the drop. Following that, I will examine the data and validate my assumptions using SQL. Once I have identified the most likely cause, I will provide recommendations for solutions and next steps. Finally, I will summarize by outlining improvements I could have made to this analysis for future reference.</p>

<h2>1. Understanding whats wrong with the graph</h2>
<p>When we sum information, we may inadvertently overlook details and lose granularity. The chart illustrates the overall count of weekly active users who accessed and utilized the app's features such as commenting, emailing, and searching, as listed in the event table. The data reveals that the number of weekly active users had been consistently rising in 2014 until the end of July, when it experienced a significant decline of 200 users, reverting back to the June level, and remained at this reduced engagement level for the duration of August.</p>
What we missed is the demographics of who the users really are. We missed on information on who actively uses Yammer :-
<ul>
<li>How old are the users who stopped interaction on the platform</li>
<li>Location</li>
<li>Age</li>
<li>Device they use </li>
<li>how they perceive weekly engagement emails</li>
<li>and any external factors</li>
 </ul>

I will use this starting intuition to analyze the data further.

<h2>2. Plan: Identifying potential causes</h2>
<ol>
<li>Bug: There can be some bug the UI of the or some feature might not be working and it has been unnoticed by the testing department. This is a little harder to pinpoint because different parts of the application would show differently in the metrics.</li>
<li> Holiday season: August, there was a decline in user engagement. Since we do not have data beyond August, it is possible that this trend is specific to that month only. There might be some festive/holiday season in some of the countries . To investigate this possibility, we could first take out the list of Top 5countries having maximum users and then filter user engagement by each country specifically for the month of August. </li>
<li>Regarding location, as Yammer is a global application, it's possible that local competitors could be affecting user engagement. However, it's not possible to verify whether Yammer has competitors in various countries with the current database. To investigate this possibility, we can test for growth in account registrations in August. If the numbers are steady, it suggests that competition is not the cause. It's worth noting that the chart above only defines user engagement by logins and in-app engagement, so this may not be the strongest hypothesis to rely on.</li>

<li>Device performance could also be a factor. To explore this possibility, we can filter user engagement by device.</li>

<li>Another aspect to consider is Yammer's outreach strategy. The platform uses email communication to keep in touch with its users, sending out weekly digests and reengagement emails. Yammer can track whether users open the emails and interact with the content by clicking on the embedded links. We can investigate if emails are causing the issue by examining how users interacted with their emails in August.</li>

<li>Finally, there is the possibility of a database problem. It's possible that there is an error in the database pipeline that is either not recording user activities or sending the information to the wrong place. To investigate this possibility, we can examine all the databases, links, and formulas.</li>
</ol>

## 3. Investigate the data
### 3.1 Holiday seaoson:-
<pre><code>
-- CTE query to fetch data From Top 5 Countries having Max_users and reporting the weekly active users 
with data as (SELECT DATE_TRUNC('week', a.occurred_at) AS weeks,a.location,
              COUNT(DISTINCT a.user_id) AS weekly_active_user
              FROM tutorial.yammer_events a 
              WHERE a.location IN 
                     (SELECT location
                     FROM tutorial.yammer_events
                     GROUP BY location
                     ORDER by COUNT(user_id) DESC
                     LIMIT 5) 
              AND a.event_type = 'engagement'
              AND a.event_name = 'login'
              GROUP BY weeks, location
              ORDER BY location, weeks DESC)

-- Pivot Table 
SELECT weeks,
SUM(CASE when location = 'France' then weekly_active_user Else 0 END) as france_weekly, 
SUM(CASE when location = 'United States' then weekly_active_user Else 0 END) as us_weekly, 
SUM(CASE when location = 'Japan' then weekly_active_user Else 0 END) as japan_weekly,
SUM(CASE when location = 'United Kingdom' then weekly_active_user Else 0 END) as uk_weekly,
SUM(CASE when location = 'Germany' then weekly_active_user Else 0 END) as germany_weekly
From data
GROUP by weeks;
</code></pre>

<img src="https://github.com/parthjain99/SQL-Portfolio/blob/9d1af3d7aa6aceccf94808e106213133c1c9756e/Drop%20in%20user%20engagement%20/Graphs/TOP%205%20COUNTRY%20WEEKLY%20ACTIVE%20USERS.png"/>

A drop in user engagement similar to the one in Chart 1 was observed towards the end of July and throughout August. The drop was particularly noticeable in the United States, which was one of Yammer's largest markets at the time. While user engagement in all Top 5 countries experienced a decline to levels seen in May, it is premature to assume that the drop in user engagement was solely due to August being a vacation month.


### 3.2 Location:-

Our hypothesis suggests that a decline in account registrations could indicate a potential competitor eroding Yammer's market share. To test this hypothesis, we need to compare the number of engagements with the number of sign-ups.

<pre><code>
with user_activity as(SELECT EXTRACT('month' FROM occurred_at) as month, 
                      event_type, 
                      COUNT(event_name) AS total_count
                      FROM tutorial.yammer_events events
                      GROUP BY month, event_type
                      ORDER BY event_type)

SELECT month,
SUM(CASE WHEN event_type = 'engagement' Then total_count else 0 END) as engagement_count,
SUM(CASE WHEN event_type = 'signup_flow' Then total_count else 0 END) as signup_flow_count
FROM user_activity
GROUP BY month
</code></pre>

<img src="https://github.com/parthjain99/SQL-Portfolio/blob/9d1af3d7aa6aceccf94808e106213133c1c9756e/Drop%20in%20user%20engagement%20/Graphs/MONTHLY%20SIGNUPS%20AND%20ENGAGEMENT%20COMPARISION.png"/>

As mentioned earlier, the decrease in usage is related to in-app activities, so it's improbable that potential customers would have any impact on existing customers. We can observe that the number of sign-ups actually went up in August, whereas engagement by existing users decreased.

### 3.3 GROWTH OF DAILY SIGNUPS:-
<pre><code>
SELECT DATE_TRUNC('day', created_at) AS days,
COUNT(CASE WHEN state='active' THEN 1 ElSE NULL END) AS active_users,
COUNT(*) AS all_users
From tutorial.yammer_users
WHERE created_at >= '2014-06-01'
      AND created_at < '2014-09-01'
GROUP By days
ORDER BY days
</code></pre>
<img src=https://github.com/parthjain99/SQL-Portfolio/blob/9d1af3d7aa6aceccf94808e106213133c1c9756e/Drop%20in%20user%20engagement%20/Graphs/GROWTH%20CHART%20SHOWING%20DAILY%20SIGNUPS%20OF%20ACTIVE%20USERS%20AND%20ALL%20USERS.png/>

One of the simplest metrics to investigate is growth because it's easy to quantify, and most companies, including Yammer, already track it closely. However, in this case, you have to create the measurement yourself. By analyzing the data, we can observe that there hasn't been a significant change in the growth rate. It remains high during the week and low on weekends.


### 3.4 Old user investigation:-
Since the growth of new users is steady, it is plausible that the decline in engagement is attributed to existing users rather than new ones. One of the most efficient methods to investigate this is by cohort analysis, which groups users according to the time they signed up for the product. This chart demonstrates a drop in engagement for users who signed up more than 10 weeks ago:
<pre><code>
SELECT DATE_TRUNC('week',z.occurred_at) AS "week",
AVG(z.age_at_event) AS "Average age during week",
COUNT(DISTINCT CASE WHEN z.user_age > 70 THEN z.user_id ELSE NULL END) AS "10+ weeks",
COUNT(DISTINCT CASE WHEN z.user_age < 70 AND z.user_age >= 63 THEN z.user_id ELSE NULL END) AS "9 weeks",
COUNT(DISTINCT CASE WHEN z.user_age < 63 AND z.user_age >= 56 THEN z.user_id ELSE NULL END) AS "8 weeks",
COUNT(DISTINCT CASE WHEN z.user_age < 56 AND z.user_age >= 49 THEN z.user_id ELSE NULL END) AS "7 weeks",
COUNT(DISTINCT CASE WHEN z.user_age < 49 AND z.user_age >= 42 THEN z.user_id ELSE NULL END) AS "6 weeks",
COUNT(DISTINCT CASE WHEN z.user_age < 42 AND z.user_age >= 35 THEN z.user_id ELSE NULL END) AS "5 weeks",
COUNT(DISTINCT CASE WHEN z.user_age < 35 AND z.user_age >= 28 THEN z.user_id ELSE NULL END) AS "4 weeks",
COUNT(DISTINCT CASE WHEN z.user_age < 28 AND z.user_age >= 21 THEN z.user_id ELSE NULL END) AS "3 weeks",
COUNT(DISTINCT CASE WHEN z.user_age < 21 AND z.user_age >= 14 THEN z.user_id ELSE NULL END) AS "2 weeks",
COUNT(DISTINCT CASE WHEN z.user_age < 14 AND z.user_age >= 7 THEN z.user_id ELSE NULL END) AS "1 week",
COUNT(DISTINCT CASE WHEN z.user_age < 7 THEN z.user_id ELSE NULL END) AS "Less than a week"
FROM (
        SELECT e.occurred_at,
               u.user_id,
               DATE_TRUNC('week',u.activated_at) AS activation_week,
               EXTRACT('day' FROM e.occurred_at - u.activated_at) AS age_at_event,
               EXTRACT('day' FROM '2014-09-01'::TIMESTAMP - u.activated_at) AS user_age
          FROM tutorial.yammer_users u
          JOIN tutorial.yammer_events e
            ON e.user_id = u.user_id
           AND e.event_type = 'engagement'
           AND e.event_name = 'login'
           AND e.occurred_at >= '2014-05-01'
           AND e.occurred_at < '2014-09-01'
         WHERE u.activated_at IS NOT NULL
       ) z

GROUP BY 1
ORDER BY 1
LIMIT 100
</code></pre>
<img src="https://github.com/parthjain99/SQL-Portfolio/blob/9d1af3d7aa6aceccf94808e106213133c1c9756e/Drop%20in%20user%20engagement%20/Graphs/USER%20RETENTION%20GRAPH.png/">


### 3.5 Devices:-
Based on our understanding that the issue is concentrated among older users, it seems unlikely that it's caused by a one-time surge in marketing traffic or anything that's impacting new traffic, such as being blocked or experiencing a change in search engine rankings. We should now investigate different device types to determine whether the problem is limited to a specific product.
<pre><code>
SELECT DATE_TRUNC('week', e.occurred_at) as weeks,
      COUNT(DISTINCT e.user_id) AS weekly_active_users,
      COUNT(DISTINCT CASE when device IN 
      ('iphone 5','samsung galaxy s4','nexus 5','iphone 5s','iphone 4s','nokia lumia 635',
       'htc one','samsung galaxy note','amazon fire phone') 
      Then  e.user_id Else NULL END) as mobile_user,
      COUNT(DISTINCT CASE when device IN 
      ('macbook pro','lenovo thinkpad','macbook air','dell inspiron notebook',
          'asus chromebook','dell inspiron desktop','acer aspire notebook','hp pavilion desktop','acer aspire desktop','mac mini') 
      Then e.user_id Else NULL END) as desktop_user,
      COUNT(DISTINCT CASE when device IN 
      ('ipad air','nexus 7','ipad mini','nexus 10','kindle fire','windows surface','samsumg galaxy tablet') 
      Then e.user_id Else NULL END) as tablet_user
FROM tutorial.yammer_events e
WHERE e.event_type = 'engagement'
AND e.event_name = 'login'
GROUP BY weeks
ORDER BY weeks
</code></pre>
<img src="https://github.com/parthjain99/SQL-Portfolio/blob/9d1af3d7aa6aceccf94808e106213133c1c9756e/Drop%20in%20user%20engagement%20/Graphs/USER%20RETENTION%20BASED%20ON%20DEVICE%20USED.png"/>


### 3.6 Weekly Digest:
Upon examining the data, a significant decline in phone engagement rates can be observed, indicating that the issue is likely connected to the long-term user retention through the mobile app. At this stage, it would be beneficial to investigate whether any recent changes have been made to the mobile app and to inquire about the factors that motivate users to engage with the product. The weekly digest email mentioned earlier is intended to encourage users to return to the product. As the problem appears to be related to the retention of long-term users, it would be worthwhile to explore whether the email is a contributing factor.


<pre><code>
SELECT DATE_TRUNC('week', occurred_at) AS week, 
COUNT(CASE when action  = 'sent_weekly_digest' Then 1 Else Null END) as weekly_emails,
COUNT(CASE when action  = 'email_open' Then 1 Else Null END) as emails_open,
COUNT(CASE when action  = 'email_clickthrough' Then 1 Else Null END) as email_clickthrough,
COUNT(CASE when action  = 'sent_reengagement_email' Then 1 Else Null END) as sent_reengagement_email
FROM tutorial.yammer_emails
GROUP BY week
ORDER BY week
</code></pre>
<img src="https://github.com/parthjain99/SQL-Portfolio/blob/9d1af3d7aa6aceccf94808e106213133c1c9756e/Drop%20in%20user%20engagement%20/Graphs/EMAIL%20ACTIONS.png"/>



### 3.7 Dveling deeper into weekly digest:-

If you apply a filter to clickthroughs by clicking the dot in the legend, you will notice a significant drop in clickthrough rates. To delve deeper into this issue, the following chart displays clickthrough and open rates of emails, which strongly suggest that the problem is related to digest emails as well as the mobile app.



<pre><code>
SELECT week,
       weekly_opens/CASE WHEN weekly_emails = 0 THEN 1 ELSE weekly_emails END::FLOAT AS weekly_open_rate,
       weekly_ctr/CASE WHEN weekly_opens = 0 THEN 1 ELSE weekly_opens END::FLOAT AS weekly_ctr,
       retain_opens/CASE WHEN retain_emails = 0 THEN 1 ELSE retain_emails END::FLOAT AS retain_open_rate,
       retain_ctr/CASE WHEN retain_opens = 0 THEN 1 ELSE retain_opens END::FLOAT AS retain_ctr
FROM (
       SELECT DATE_TRUNC('week',e1.occurred_at) AS week,
              COUNT(CASE WHEN e1.action = 'sent_weekly_digest' THEN e1.user_id ELSE NULL END) AS weekly_emails,
              COUNT(CASE WHEN e1.action = 'sent_weekly_digest' THEN e2.user_id ELSE NULL END) AS weekly_opens,
              COUNT(CASE WHEN e1.action = 'sent_weekly_digest' THEN e3.user_id ELSE NULL END) AS weekly_ctr,
              COUNT(CASE WHEN e1.action = 'sent_reengagement_email' THEN e1.user_id ELSE NULL END) AS retain_emails,
              COUNT(CASE WHEN e1.action = 'sent_reengagement_email' THEN e2.user_id ELSE NULL END) AS retain_opens,
              COUNT(CASE WHEN e1.action = 'sent_reengagement_email' THEN e3.user_id ELSE NULL END) AS retain_ctr
         FROM tutorial.yammer_emails e1
         LEFT JOIN tutorial.yammer_emails e2
                 ON e2.occurred_at >= e1.occurred_at
                 AND e2.occurred_at < e1.occurred_at + INTERVAL '5 MINUTE'
                 AND e2.user_id = e1.user_id
                 AND e2.action = 'email_open'
         LEFT JOIN tutorial.yammer_emails e3
                 ON e3.occurred_at >= e2.occurred_at
                 AND e3.occurred_at < e2.occurred_at + INTERVAL '5 MINUTE'
                 AND e3.user_id = e2.user_id
                 AND e3.action = 'email_clickthrough'
        WHERE e1.occurred_at >= '2014-06-01'
                 AND e1.occurred_at < '2014-09-01'
                 AND e1.action IN ('sent_weekly_digest','sent_reengagement_email')
        GROUP BY 1
       ) a
 ORDER BY 1
</code></pre>
<img src="https://github.com/parthjain99/SQL-Portfolio/blob/9d1af3d7aa6aceccf94808e106213133c1c9756e/Drop%20in%20user%20engagement%20/Graphs/OPEN%20AND%20CLICK%20THROUGH%20RATES.png"/>



## 4. Solutions

Ultimately, we identified that the decrease in user engagement was due to a decline in clickthrough rates for the weekly digest emails, particularly on smartphone devices. As a result, it is necessary to investigate whether the email formatting for smartphones is appropriate and user-friendly.
Here are some potential areas for further exploration:
<ul>
<li>Conducting a bug check for email formatting on mobile devices</li>
<li>Implementing A/B testing for the weekly digest email content</li>
<li>Administering a survey to collect user feedback on the weekly digest emails.</li>
</ul>
