# Case tracker data model

## Data mapping

The main entities are a Person, an Enterprise, a Case, an Activity.

| Entity                 | Collection                    | Resource               |
|:-----------------------|:------------------------------|:-----------------------|
| Person                 | /db/sites/{app}/persons       | persons.xml            |
| Enterprise             | /db/sites/{app}/enterprises   | enterprises.xml        |
| Case                   | /db/sites/{app}/cases/YYYY/MM | case.xml               |
| Activity               | inside Case                   | inside Case            |

## Person entity

Here is a sample Person record. Empty tags are shown to reveal the full data model but the convention is to not store them in database.

```xml
<Person>
  <Id>3</Id>
  <Sex>M</Sex>
  <Civility>Dr</Civility>
  <Name>
    <LastName>Ducharme</LastName>
    <FirstName>Bob</FirstName>
    <SortString/>
  </Name>
  <Country>FR</Country>
  <EnterpriseRef>1</EnterpriseRef>
  <Function>Manager</Function>
  <Contacts>
    <Phone>+33 (0) 2 29 97 41 65</Phone>
    <Mobile>+33 (0) 6 62 94 21 50</Mobile>
    <Email>bob@ducharme.com</Email>
  </Contacts>
  <Photo/>
  <UserProfile/>
</Person>
```

The `Photo` is a path to an image uploaded into the `/db/sites/{app}/persons/images` collection.

The `UserProfile` contains information about the user if the Person entity represents a user with access to the case tracker.

## Enterprise entity

Here is a sample Enterprise record. Empty tags are shown to reveal the full data model but the convention is to not store them in database.

```xml
<?xml version="1.0"?>
<Enterprise>
  <Id>1</Id>
  <Name>Acme Software Ltd</Name>
  <ShortName>Acme</ShortName>
  <CreationYear>2014</CreationYear>
  <SizeRef>2</SizeRef>
  <DomainActivityRef>J62</DomainActivityRef>
  <MainActivities>web application development</MainActivities>
  <TargetedMarkets>
    <TargetedMarketRef>572010</TargetedMarketRef>
    <TargetedMarketRef>581010</TargetedMarketRef>
  </TargetedMarkets>
  <Address>
    <StreetNameAndNo>1 impasse des Papillons</StreetNameAndNo>
    <PO-Box/>
    <Co/>
    <PostalCode>85000</PostalCode>
    <Town>La Roche Sur Yon</Town>
    <Country>FR</Country>
  </Address>
</Enterprise>
```

The `DomainActivityRef` belongs to the NACE code classification.

The `TargetedMarketRef` belongs to the Thomson Reuters classification.

## Case skeleton

    Case
      No
      CreationDate
      StatusHistory<+>
      Information
        Title
        ClientEnterprise<+>
          [PILOTE] EnterpriseRef (live copy from enterprises.xml)
        [NOT USED] ContactPerson<+>
        [NOT USED] ManagingEntity
          RegionalEntityRef
          AssignedByRef
          Date
      Alerts<+>
      Management
        AccountManagerRef
        [NOT USED] AssignedByRef
        [NOT USED] Date
        Conformity<+>
      NeedsAnalysis
        [NOT USED]Contact
          Date
          Agent
        ContactPerson<+>
        Analysis
          Date
          [PILOTE] Agent
          [PILOTE] FundingSource
        Tools
        Stats
        Context<+>
        Impact<+>
        Comments
      Activities
        Acitvity<+>
      [PILOTE ???] Evaluation
        KAMReport
          Recognition
          Tools
      [NOT USED] Proxies
        KAMReportNAProxy
          Recognition
          Tools

## Activity skeleton

    Activity
      No
      CreationDate
      StatusHistory<+>
      [DEAD COPY] NeedsAnalysis
      Assignment
        Weights
        Description
        ServiceRef
        ResponsibleCoachRef
        AssignedByRef
        Date
      Alerts<+>
      FundingRequest<+>
      Opinions
        KAM-Opinion<+>
        ServiceHeadOpinion<+>
      FundingDecision<+>
      FinalReport (coach feedback)
      FinalReportApproval ([a] from KAMReportNAProxy and [b] form poll module)
        [a] Recognition
        [a] Tools
        [b] Profiles
        [b] Dialogue
        [b] PastRegionalInvolvement
        [b] RegionalInvolvement
        [b] FutureRegionalInvolvement
        [b] FutureSupport
        CoachingManagerVisa
      Evaluation (connection with poll module)
       Order (for SME)
       Order (for KAM, transcoded into FinalReportApproval [b])

## The `StatusHistory` data type

The `StatusHistory` encodes the current workflow status and a trace of the last date in each status already reached by the workflow (shallow history). The encoding uses status selectors values declared in global information.

```xml
<StatusHistory>
  <CurrentStatusRef>3</CurrentStatusRef>
  <PreviousStatusRef>2</PreviousStatusRef>
  <Status>
    <Date>2015-02-13</Date>
    <ValueRef>1</ValueRef>
  </Status>
  <Status>
    <Date>2015-03-18T14:22:07.713+01:00</Date>
    <ValueRef>2</ValueRef>
  </Status>
  <Status>
    <Date>2015-04-23T11:28:51.186+02:00</Date>
    <ValueRef>3</ValueRef>
  </Status>
</StatusHistory>
```

## The `UserProfile` data type

It contains information about the user if the Person entity represents a user with access to the case tracker.

  