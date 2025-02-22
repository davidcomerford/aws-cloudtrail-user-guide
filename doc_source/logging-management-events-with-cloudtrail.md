# Logging management events for trails<a name="logging-management-events-with-cloudtrail"></a>

By default, trails log management events and don't include data or Insights events\. Additional charges apply for data or Insights events\. For more information, see [AWS CloudTrail Pricing](https://aws.amazon.com/cloudtrail/pricing/)\.

**Contents**
+ [Management events](#logging-management-events)
  + [Logging management events with the AWS Management Console](#logging-management-events-with-the-cloudtrail-console)
+ [Read and write events](#read-write-events-mgmt)
+ [Logging events with the AWS Command Line Interface](#creating-mgmt-event-selectors-with-the-AWS-CLI)
+ [Logging events with the AWS SDKs](#logging-management-events-with-the-AWS-SDKs)
+ [Sending events to Amazon CloudWatch Logs](#sending-management-events-to-cloudwatch-logs)

## Management events<a name="logging-management-events"></a>

Management events provide visibility into management operations that are performed on resources in your AWS account\. These are also known as control plane operations\. Example management events include:
+ Configuring security \(for example, IAM `AttachRolePolicy` API operations\)
+ Registering devices \(for example, Amazon EC2 `CreateDefaultVpc` API operations\)
+ Configuring rules for routing data \(for example, Amazon EC2 `CreateSubnet` API operations\)
+ Setting up logging \(for example, AWS CloudTrail `CreateTrail` API operations\)

Management events can also include non\-API events that occur in your account\. For example, when a user logs in to your account, CloudTrail logs the `ConsoleLogin` event\. For more information, see [Non\-API events captured by CloudTrail](cloudtrail-non-api-events.md)\. For a list of supported management events that CloudTrail logs for AWS services, see [CloudTrail supported services and integrations](cloudtrail-aws-service-specific-topics.md)\.

By default, trails are configured to log management events\. For a list of supported management events that CloudTrail logs for AWS services, see [CloudTrail supported services and integrations](cloudtrail-aws-service-specific-topics.md)\.

**Note**  
The CloudTrail **Event history **feature supports only management events\. Not all management events are supported in event history\. You cannot exclude AWS KMS or Amazon RDS Data API events from **Event history **; settings that you apply to a trail do not apply to **Event history **\. For more information, see [Viewing events with CloudTrail Event history](view-cloudtrail-events.md)\. 

### Logging management events with the AWS Management Console<a name="logging-management-events-with-the-cloudtrail-console"></a>

1. Open the **Trails** page of the CloudTrail console and choose the trail name\.

1. For **Management events**, choose **Edit**\.
   + Choose if you want your trail to log **Read** events, **Write** events, or both\.
   + Choose **Exclude AWS KMS events** to filter AWS Key Management Service \(AWS KMS\) events out of your trail\. The default setting is to include all AWS KMS events\.

     The option to log or exclude AWS KMS events is available only if you log management events on your trail\. If you choose not to log management events, AWS KMS events are not logged, and you cannot change AWS KMS event logging settings\.

     AWS KMS actions such as `Encrypt`, `Decrypt`, and `GenerateDataKey` typically generate a large volume \(more than 99%\) of events\. These actions are now logged as **Read** events\. Low\-volume, relevant AWS KMS actions such as `Disable`, `Delete`, and `ScheduleKey` \(which typically account for less than 0\.5% of AWS KMS event volume\) are logged as **Write** events\.

     To exclude high\-volume events like `Encrypt`, `Decrypt`, and `GenerateDataKey`, but still log relevant events such as `Disable`, `Delete` and `ScheduleKey`, choose to log **Write** management events, and clear the check box for **Exclude AWS KMS events**\.
   + Choose **Exclude Amazon RDS Data API events** to filter Amazon Relational Database Service Data API events out of your trail\. The default setting is to include all Amazon RDS Data API events\. For more information about Amazon RDS Data API events, see [Logging Data API calls with AWS CloudTrail](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/logging-using-cloudtrail-data-api.html) in the *Amazon RDS User Guide for Aurora*\.

   Choose **Update trail** when you are finished\.

## Read and write events<a name="read-write-events-mgmt"></a>

When you configure your trail to log management events, you can specify whether you want read\-only events, write\-only events, or both\.
+ **Read**

  Read\-only events include API operations that read your resources, but don't make changes\. For example, read\-only events include the Amazon EC2 `DescribeSecurityGroups` and `DescribeSubnets` API operations\. These operations return only information about your Amazon EC2 resources and don't change your configurations\.
+ **Write**

  Write\-only events include API operations that modify \(or might modify\) your resources\. For example, the Amazon EC2 `RunInstances` and `TerminateInstances` API operations modify your instances\.

**Example: Logging read and write events for separate trails**

The following example shows how you can configure trails to split log activity for an account into separate S3 buckets: one bucket receives read\-only events and a second bucket receives write\-only events\.

1. You create a trail and choose an S3 bucket named `read-only-bucket` to receive log files\. You then update the trail to specify that you want **Read** management events\.

1. You create a second trail and choose an S3 bucket named `write-only-bucket` to receive log files\. You then update the trail to specify that you want **Write** management events\.

1. The Amazon EC2 `DescribeInstances` and `TerminateInstances` API operations occur in your account\.

1. The `DescribeInstances` API operation is a read\-only event and it matches the settings for the first trail\. The trail logs and delivers the event to the `read-only-bucket`\.

1. The `TerminateInstances` API operation is a write\-only event and it matches the settings for the second trail\. The trail logs and delivers the event to the `write-only-bucket`\.

## Logging events with the AWS Command Line Interface<a name="creating-mgmt-event-selectors-with-the-AWS-CLI"></a>

You can configure your trails to log management events using the AWS CLI\.

To view whether your trail is logging management events, run the `get-event-selectors` command\.

```
aws cloudtrail get-event-selectors --trail-name TrailName
```

The following example returns the default settings for a trail\. By default, trails log all management events, log events from all event sources, and don't log data events\.

```
{
    "TrailARN": "arn:aws:cloudtrail:us-east-1:111122223333:trail/TrailName",
    "AdvancedEventSelectors": [
        {
            "Name": "Management events selector",
            "FieldSelectors": [
                {
                    "Field": "eventCategory",
                    "Equals": [
                        "Management"
                    ]
                }
            ]
        }
    ]
}
```

To configure your trail to log management events, run the `put-event-selectors` command\. The following example shows how to configure your trail to include all management events for two S3 objects\. You can specify from 1 to 5 event selectors for a trail\. You can specify from 1 to 250 data resources for a trail\.

**Note**  
The maximum number of S3 data resources is 250, regardless of the number of event selectors\.

```
aws cloudtrail put-event-selectors --trail-name TrailName --event-selectors '[{ "ReadWriteType": "All", "IncludeManagementEvents":true, "DataResources": [{ "Type": "AWS::S3::Object", "Values": ["arn:aws:s3:::mybucket/prefix", "arn:aws:s3:::mybucket2/prefix2"] }] }]'
```

The following example returns the event selector configured for the trail\.

```
{
    "TrailARN": "arn:aws:cloudtrail:us-east-1:111122223333:trail/TrailName",
    "EventSelectors": [
        {
            "ReadWriteType": "All",
            "IncludeManagementEvents": true,
            "DataResources": [
                {
                    "Type": "AWS::S3::Object",
                    "Values": [
                        "arn:aws:s3:::mybucket/prefix",
                        "arn:aws:s3:::mybucket2/prefix2",
                    ]  
                }
            ],
            "ExcludeManagementEventSources": []
        }
    ]
}
```

To exclude AWS Key Management Service \(AWS KMS\) events from a trail's logs, run the `put-event-selectors` command and add the attribute `ExcludeManagementEventSources` with a value of `kms.amazonaws.com`\. The following example creates an event selector for a trail named *TrailName* to include read\-only and write\-only management events, but exclude AWS KMS events\. Because AWS KMS can generate a high volume of events, the user in this example might want to limit events to manage the cost of a trail\.

```
aws cloudtrail put-event-selectors --trail-name TrailName --event-selectors '[{"ReadWriteType": "All","ExcludeManagementEventSources": ["kms.amazonaws.com"],"IncludeManagementEvents": true}]'
```

The example returns the event selector configured for the trail\.

```
{
    "TrailARN": "arn:aws:cloudtrail:us-east-1:111122223333:trail/TrailName",
    "EventSelectors": [
        {
            "ReadWriteType": "All",
            "IncludeManagementEvents": true,
            "DataResources": [],
            "ExcludeManagementEventSources": [
                "kms.amazonaws.com"
            ]
        }
    ]
}
```

To exclude Amazon RDS Data API events from a trail's logs, run the `put-event-selectors` command and add the attribute `ExcludeManagementEventSources` with a value of `rdsdata.amazonaws.com`\. The following example creates an event selector for a trail named *TrailName* to include read\-only and write\-only management events, but exclude Amazon RDS Data API events\. Because Amazon RDS Data API can generate a high volume of events, the user in this example might want to limit events to manage the cost of a trail\.

```
{
    "TrailARN": "arn:aws:cloudtrail:us-east-1:111122223333:trail/TrailName",
    "EventSelectors": [
        {
            "ReadWriteType": "All",
            "IncludeManagementEvents": true,
            "DataResources": [],
            "ExcludeManagementEventSources": [
                "rdsdata.amazonaws.com"
            ]
        }
    ]
}
```

To start logging AWS KMS or Amazon RDS Data API events to a trail again, pass an empty string as the value of `ExcludeManagementEventSources`, as shown in the following command\.

```
aws cloudtrail put-event-selectors --trail-name TrailName --event-selectors '[{"ReadWriteType": "All","ExcludeManagementEventSources": [],"IncludeManagementEvents": true}]'
```

To log relevant AWS KMS events to a trail like `Disable`, `Delete` and `ScheduleKey`, but exclude high\-volume AWS KMS events like `Encrypt`, `Decrypt`, and `GenerateDataKey`, log write\-only management events, and keep the default setting to log AWS KMS events, as shown in the following example\.

```
aws cloudtrail put-event-selectors --trail-name TrailName --event-selectors '[{"ReadWriteType": "WriteOnly","ExcludeManagementEventSources": [],"IncludeManagementEvents": true}]'
```

## Logging events with the AWS SDKs<a name="logging-management-events-with-the-AWS-SDKs"></a>

Use the [GetEventSelectors](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_GetEventSelectors.html) operation to see whether your trail is logging management events for a trail\. You can configure your trails to log management events with the [PutEventSelectors](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_PutEventSelectors.html) operation\. For more information, see the [AWS CloudTrail API Reference](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/)\.

## Sending events to Amazon CloudWatch Logs<a name="sending-management-events-to-cloudwatch-logs"></a>

CloudTrail supports sending data and management events to CloudWatch Logs\. When you configure your trail to send events to your CloudWatch Logs log group, CloudTrail sends only the events that you specify in your trail\. For example, if you configure your trail to log management events only, your trail delivers management events only to your CloudWatch Logs log group\. For more information, see [Monitoring CloudTrail Log Files with Amazon CloudWatch Logs](monitor-cloudtrail-log-files-with-cloudwatch-logs.md)\. 