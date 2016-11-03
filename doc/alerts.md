# Alerts and notification specification language

The alerts and notification specification language defines alerts based on temporal conditions. They are defined in the `modules/alerts/check.xml` file that must be deployed inside the application configuration collection.

The alerts are computed and created on a regular basis by the scheduler of the database by invoking the `modules/alerts/job.xql` script (see eXist-DB documentation), and they can result in different side effects :

* add an entry to the todo list of the application; this is used to display personalized todo lists to users;
* send an e-mail reminder messages;
* automatically change workflow status.

The temporal conditions are defined by the time ellapsed between a start and end event. The event may be define by the entry or the exit in a particular workflow status, or by a date stored in a case or activity document.

The alerts module is implemented in the `modules/alerts` folder. The todo list is stored in a `check` collection inside the application data collection. The *check* collection must be created by the deployment script. There is one resource named after the alert number (e.g. *1.xml*) for each type of alert.

## Examples

### An alert independent of the current workflow status

This example alert is created when the case resource does not contain a grant signature date. It is added to the todo list of the coaching assistants.

```xml
<Check No="2">
  <When Case="*"/>
  <Title>Grant signature date missing</Title>
  <Eval>fn:collection('/db/sites/cctracker/cases')//Case[Information/Contract[not(Date)]]</Eval>
  <Responsible>
    <Meet>g:coaching-assistant</Meet>
  </Responsible>
</Check>
```

### An alert dependent of the current workflow status and bounded by two dates

This example alert is created when the needs analysis document of the case resource shows that more than 10 days have ellapsed since the date of the first contact with the SME. It is added to the todo list of the manager for the case (KAM) and to the todo list of the region manager coordinator (KAMCO).

```xml
<Check No="5" Threshold="10">
  <Title>Needs Analysis not finished</Title>
  <When Case="Needs Analaysis">3</When>
  <Start>$base/NeedsAnalysis/Contact/Date</Start>
  <Stop>$base/NeedsAnalysis/Analysis/Date</Stop>
  <Responsible>
    <Meet>r:kam</Meet>
  </Responsible>
  <Supervisor>
    <Meet>r:region-manager</Meet>
  </Supervisor>
</Check>
```

### An alert dependent of the current workflow status and bounded by a status time span with an automatic e-mail notification

This example alert is created when the activity has entered the coaching plan status for more than 10 days and is still in the coaching plan status. It is added to the todo list of the coach of the activity. In addition an e-mail reminder is sent to the coach exactly 8 days after the coaching plan status initiation if still in the coaching plan status. This reminder is only sent once, meaning it is not sent again in case the activity workflow is returned back to the coaching plan status afterwards.

```xml
<Check No="8" Threshold="10">
  <Title>Coaching plan not finished</Title>
  <When Activity="Coaching plan">2</When>
  <Start>enter</Start>
  <Stop>leave</Stop>
  <Responsible>
    <Meet>r:coach</Meet>
  </Responsible>
  <Email Elapsed="8">
    <Description>reminds coach to write coaching plan</Description>
    <Template>coach-assignment-reminder</Template>
    <Recipients Max="1" Key="coach-1 reminder">r:coach</Recipients>
  </Email>
</Check>
```

### An automatic e-mail notification and an automatic status change

This example alert is created when the activity is in the feedback status. It is not added to the todo list but it triggers an e-mail reminder exactly 7 days after the feedback status initiation. The recipient of the e-mail is not specified in the alert (it must be specified directly in the e-mail template *sme-feedback-reminder*). The reminder is only sent once, meaning it is not sent again in case the activity workflow is returned back to the feedback status afterwards. In addition the alert also moves forward the workflow status to the closed status if the activity is still in the feedback status 21 days after it has been initiated.

```xml
<Check>
  <Title>Remind evaluation feedback from SME</Title>
  <When Activity="Evaluation">8</When>
  <Start>enter</Start>
  <Stop>leave</Stop>
  <Email Elapsed="7">
    <Description>reminds SME to complete feedback formular</Description>
    <Template>sme-feedback-reminder</Template>
    <Recipients Max="1" Key="sme-2 reminder"/>
  </Email>
  <Status Elapsed="21" To="10">
    <Id>no-feedback</Id>
    <Description>automatic closing of activity with incomplete feedbacks after 21 days</Description>
  </Status>
</Check>
```

## The `Check` element

The `Check` element defines an alert. 

The optional `@No` attribute configure the alert to be stored in the todo list. If absent the alert is not stored, but it can still be used to send an e-mail reminder or to automatically change the workflow status.

## The `Responsible` element

The `Responsible` element defines the category of users concerned by an alert. It is used only for numbered alerts which are stored in the todo list.

## The `Email` element

The `Email` element send an e-mail reminder in reaction to an alert.

## The `Status` element

The `Status` element changes a workflow status in reaction to an alert.

