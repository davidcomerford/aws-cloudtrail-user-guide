# Example sharedEventID<a name="shared-event-ID"></a>

The following is an example that describes how CloudTrail delivers two events for the same action:

1. Alice has AWS account \(111111111111\) and creates an AWS KMS key\. She is the owner of this KMS key\. 

1. Bob has AWS account \(222222222222\)\. Alice gives Bob permission to use the KMS key\. 

1. Each account has a trail and a separate bucket\.

1. Bob uses the KMS key to call the `Encrypt` API\. 

1. CloudTrail sends two separate events\. 
   + One event is sent to Bob\. The event shows that he used the KMS key\.
   + One event is sent to Alice\. The event shows that Bob used the KMS key\.
   + The events have the same `sharedEventID`, but the `eventID` and `recipientAccountID` are unique\.

![\[How the sharedEventID field appears in logs\]](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/images/event-reference-sharedEventId.png)

## Shared event IDs in CloudTrail Insights<a name="shared-event-ID-insights"></a>

A `sharedEventID` for CloudTrail Insights events differs from the `sharedEventID` for the management and data types of CloudTrail events\. In Insights events, a `sharedEventID` is a GUID that is generated by CloudTrail Insights to uniquely identify a start and end pair of Insights events\. `sharedEventID` is common between the start and the end Insights event, and helps to create a correlation between both events to uniquely identify unusual activity\.

You can think of the `sharedEventID` as the overall Insights event ID\.