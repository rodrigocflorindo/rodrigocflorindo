- 👋 Hi, I’m @rodrigocflorindo, loved Pets
- 👀 I’m interested in Code/Deploy/Pipeline/DevOps..
- 🌱 I’m currently learning CI/CD...
- 💞️ I’m looking to collaborate on people e tools ...
- 📫 How to reach me ...

<!---
rodrigocflorindo/rodrigocflorindo is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->


AWSTemplateFormatVersion: 2010-09-09
Parameters:
    QueueNameSQS:
        Type: String
    QueueNameSQSDLQ:
        Type: String
    FeatureName:
        Type: String
    MicroServiceName:
        Type: String

Resources:
  CloudWatchLogGroup:
    Type: AWS::Logs::LogGroup
    Properties: 
      LogGroupName: !Sub "${FeatureName}-${MicroServiceName}"
      RetentionInDays: 7

  404MetricFilter:
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName: !Sub "${FeatureName}-${MicroServiceName}"
      FilterPattern: "[ip, identity, user_id, timestamp, request, status_code = 404, size, ...]"
      MetricTransformations:
      - MetricValue: '1'
        MetricNamespace: !Sub "${FeatureName}-${MicroServiceName}/404s"
        MetricName: !Sub "${FeatureName}-${MicroServiceName}/404Count"
    DependsOn: CloudWatchLogGroup

  404Alarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Sub "Alarm-${FeatureName}-${MicroServiceName}-404s"
      AlarmDescription: "Alarm description"
      MetricName: !Sub "${FeatureName}-${MicroServiceName}/404Count"
      Namespace: !Sub "${FeatureName}-${MicroServiceName}/404s"
      Statistic: Sum
      Period: 60
      EvaluationPeriods: 1
      Threshold: 1
      AlarmActions: 
        - '{{resolve:ssm:/org/member/workload_local_sns_arn:1}}'
      OKActions:
        - '{{resolve:ssm:/org/member/workload_local_sns_arn:1}}'
      ComparisonOperator: GreaterThanThreshold
      TreatMissingData: notBreaching
    DependsOn: 404MetricFilter
    
  QueueSQS:
    Type: 'AWS::ServiceCatalog::CloudFormationProvisionedProduct'
    Properties:
      ProductName: 'SQS'
      ProvisionedProductName: 'SQS-ExampleHelloWorld-Queue'
      ProvisioningArtifactName: 'v1.2'
      ProvisioningParameters:
          - Key: QueueNameSQS
            Value: !Ref QueueNameSQS
          - Key: QueueNameSQSDLQ
            Value: !Ref QueueNameSQSDLQ 


{
  "Parameters": {
    "QueueNameSQS": "SQS-ExampleHelloWorld-Queue-DEV",
    "QueueNameSQSDLQ": "SQS-ExampleHelloWorld-DLQQueue-DEV",
    "FeatureName": "produto",
    "MicroServiceName": "aplicacao"
  }
}
