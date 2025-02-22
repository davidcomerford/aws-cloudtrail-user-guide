# Infrastructure security in AWS CloudTrail<a name="infrastructure-security"></a>

As a managed service, AWS CloudTrail is protected by the AWS global network security procedures that are described on the [Best Practices for Security, Identity, & Compliance](https://aws.amazon.com/architecture/security-identity-compliance/) page\.

You use AWS published API calls to access CloudTrail through the network\. Clients must support Transport Layer Security \(TLS\) 1\.0 or later\. We recommend TLS 1\.2 or later\. Clients must also support cipher suites with perfect forward secrecy \(PFS\) such as Ephemeral Diffie\-Hellman \(DHE\) or Elliptic Curve Ephemeral Diffie\-Hellman \(ECDHE\)\. Most modern systems such as Java 7 and later support these modes\.

Additionally, requests must be signed by using an access key ID and a secret access key that is associated with an IAM principal\. Or you can use the [AWS Security Token Service](https://docs.aws.amazon.com/STS/latest/APIReference/Welcome.html) \(AWS STS\) to generate temporary security credentials to sign requests\.

The following security best practices also address infrastructure security in CloudTrail:
+ [Consider Amazon VPC endpoints for trail access\.](cloudtrail-and-interface-VPC.md)
+ Consider Amazon VPC endpoints for Amazon S3 bucket access\. For more information, see [Example Bucket Policies for VPC Endpoints for Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies-vpc-endpoint.html)\. 
+ Identify and audit all Amazon S3 buckets that contain CloudTrail log files\. Consider using tags to help identify both your CloudTrail trails and the Amazon S3 buckets that contain CloudTrail log files\. You can then use resource groups for your CloudTrail resources\. For more information, see [AWS Resource Groups](https://docs.aws.amazon.com/ARG/latest/userguide/welcome.html)\.