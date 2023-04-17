<h1> Analyzing Drop in User engangement From Yammer Dataset</h1>
<h2>Context</h2>
<p>Yammer, a freemium enterprise social networking service, was established in 2008 with the aim of providing organizations with a private communication platform. It gained popularity quickly and in 2012, Microsoft acquired it for a staggering $1.2 billion, recognizing its potential in the enterprise collaboration space. Since the acquisition, Yammer has been integrated into Microsoft's Office 365 suite, making it widely accessible to businesses of all sizes.
This is a case study of Yammer’s drop in user engagement from May to August of 2014. Due to privacy reasons, the data used is not real and did not reflected real user engagement of the Yammer app at the time. 
</p>

<h2>Problem Statement</h2>
<p>At the later half of August, Yammer detected a drop in overall user engagement through a graph that looks like this</p>
<h4>Chart 1<h4>
<code>SELECT DATE_TRUNC('week', e.occurred_at) as weeks,
       COUNT(DISTINCT e.user_id) AS weekly_active_users
  FROM yammer_events e
 WHERE e.event_type = 'engagement'
   AND e.event_name = 'login'
 GROUP BY weeks
 ORDER BY weeks</code>
 

<img src="https://github.com/parthjain99/SQL-Portfolio/blob/c527f201291fcdc6c810aa814675ac02f2e2380e/Drop%20in%20user%20engagement%20/Graphs/Drop%20in%20user%20retention.png"/>

<h3>Below are the three data tables included in the dataset I utilized.<h4>
<h4> Table 1: User Table<h4>
<img src="https://github.com/parthjain99/SQL-Portfolio/blob/433ef1c6cdab4b3428deaf8ce73bfb41c318be3c/Drop%20in%20user%20engagement%20/Graphs/user_table.png"/>

<h4> Table 2: Event Table<h4>
<img src="https://github.com/parthjain99/SQL-Portfolio/blob/433ef1c6cdab4b3428deaf8ce73bfb41c318be3c/Drop%20in%20user%20engagement%20/Graphs/events_table.png"/>

<h4> Table 3: Email Event Table<h4>
<img src="https://github.com/parthjain99/SQL-Portfolio/blob/433ef1c6cdab4b3428deaf8ce73bfb41c318be3c/Drop%20in%20user%20engagement%20/Graphs/email_events.png"/>


<h3>Task</h3>
<p>My next step is to explore the reason behind the abrupt decrease in engagement and propose potential solutions to address it. To accomplish this, I will initially brainstorm and analyze the potential causes of the drop. Following that, I will examine the data and validate my assumptions using SQL. Once I have identified the most likely cause, I will provide recommendations for solutions and next steps. Finally, I will summarize by outlining improvements I could have made to this analysis for future reference.</p>

<h2>1. Understanding whats wrong with the graph</h2>
<p>When we sum information, we may inadvertently overlook details and lose granularity. The chart illustrates the overall count of weekly active users who accessed and utilized the app's features such as commenting, emailing, and searching, as listed in the event table. The data reveals that the number of weekly active users had been consistently rising in 2014 until the end of July, when it experienced a significant decline of 200 users, reverting back to the June level, and remained at this reduced engagement level for the duration of August.</p>
<p> What we missed is the demographics of who the users really are. We missed on information on who actively uses Yammer :-
<ul>
<li>How old are the users who stopped interaction on the platform</li>
<li>Location</li>
<li>Age</li>
<li>Device they use </li>
<li>how they perceive weekly engagement emails</li>
<li>and any external factors</li>


I will use this starting intuition to analyze the data further.</p>

<h2>2. Plan: Identify potential causes</h2>
<ol>
<li>Bug: There can be some bug the UI of the or some feature might not be working and it has been unnoticed by the testing department. This is a little harder to pinpoint because different parts of the application would show differently in the metrics.</li>
<li> Time </li>






