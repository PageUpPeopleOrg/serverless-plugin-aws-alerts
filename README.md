# Serverless AWS Alerts Plugin
  [![NPM version][npm-image]][npm-url]
  [![Build Status][travis-image]][travis-url]
  [![Dependency Status][daviddm-image]][daviddm-url]
  [![Coverage percentage][coveralls-image]][coveralls-url]

A Serverless plugin to easily add CloudWatch alarms to functions

## Installation
`npm i serverless-plugin-aws-alerts`

## Usage

```yaml
service: your-service
provider:
  name: aws
  runtime: nodejs4.3

custom:

  notifications:
    - protocol: email
      endpoint: john@acloud.guru
  codeRedNotifications:
    - protocol: sms
      endpoint: +61407874400
    - protocol: email
      endpoint: xxxx@pageup.slack.com

  alerts:
    stages: # Optionally - select which stages to deploy alarms to
      - producton
      - staging
    topics:
      ok: ${self:service}-${opt:stage}-alerts-ok
      insufficientData: ${self:service}-${opt:stage}-alerts-insufficientData
      alarm:
        topic: ${self:service}-${opt:stage, self:provider.stage}-alerts-alarm
        notifications: ${self:custom.notifications}
      codeRedAlarm:
        topic: ${self:service}-${opt:stage, self:provider.stage}-alerts-alarm-code-red
        notifications: ${self:custom.codeRedNotifications}


    definitions:  # these defaults are merged with your definitions
      functionErrors:
        period: 300 # override period
      customAlarm:
        namespace: 'AWS/Lambda'
        metric: duration
        threshold: 200
        statistic: Average
        period: 300
        evaluationPeriods: 1
        comparisonOperator: GreaterThanThreshold

     functionCodeRed: #Something is _really_ going wrong and the team needs to know about it NOW!
        namespace: 'AWS/Lambda'
        metric: Errors
        threshold: 20
        statistic: Sum
        period: 300
        evaluationPeriods: 1
        comparisonOperator: GreaterThanOrEqualToThreshold
        topics:
          alarm: codeRedAlarm

    global:
      - functionCodeRed
      - functionThrottles
      - functionErrors
    function:
      - functionInvocations
      - functionDuration

plugins:
  - serverless-plugin-aws-alerts

functions:
  foo:
    handler: foo.handler
    alarms: # merged with function alarms
      - customAlarm
      - name: fooAlarm
        namespace: 'AWS/Lambda'
        metric: errors # define custom metrics here
        threshold: 1
        statistic: Minimum
        period: 60
        evaluationPeriods: 1
        comparisonOperator: GreaterThanThreshold
```

## SNS Topics

If topic name is specified, plugin assumes that topic does not exist and will create it. To use existing topics, specify ARNs instead.

## Metric Log Filters
You can monitor a log group for a function for a specific pattern. Do this by adding the pattern key.
You can learn about custom patterns at: http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/FilterAndPatternSyntax.html

The following would create a custom metric log filter based alarm named `barAlarm`. Any function that included this alarm would have its logs scanned for the pattern `exception Bar` and if found would trigger an alarm.

```yaml
custom:
  alerts:
    function:
      - name: barAlarm
        metric: barExceptions
        threshold: 0
        statistic: Minimum
        period: 60
        evaluationPeriods: 1
        comparisonOperator: GreaterThanThreshold
        pattern: 'exception Bar'
      - name: bunyanErrors
        metric: BunyanErrors
        threshold: 0
        statistic: Sum
        period: 60
        evaluationPeriods: 1
        comparisonOperator: GreaterThanThreshold
        pattern: '{$.level > 40}'
```

> Note: For custom log metrics, namespace property will automatically be set to stack name (e.g. `fooservice-dev`).

## Default Definitions
The plugin provides some default definitions that you can simply drop into your application. For example:

```yaml
alerts:
  global:
    - functionThrottles
    - functionErrors
  function:
    - functionInvocations
    - functionDuration
```

If these definitions do not quite suit i.e. the threshold is too high, you can override a setting without
creating a completely new definition.

```yaml
alerts:
  definitions:  # these defaults are merged with your definitions
    functionErrors:
      period: 300 # override period
```

The default definitions are below.

```yaml
definitions:
  functionInvocations:
    namespace: 'AWS/Lambda'
    metric: Invocations
    threshold: 100
    statistic: Sum
    period: 60
    evaluationPeriods: 1
    comparisonOperator: GreaterThanThreshold
  functionErrors:
    namespace: 'AWS/Lambda'
    metric: Errors
    threshold: 10
    statistic: Maximum
    period: 60
    evaluationPeriods: 1
    comparisonOperator: GreaterThanThreshold
  functionDuration:
    namespace: 'AWS/Lambda'
    metric: Duration
    threshold: 500
    statistic: Maximum
    period: 60
    evaluationPeriods: 1
    comparisonOperator: GreaterThanThreshold
  functionThrottles:
    namespace: 'AWS/Lambda'
    metric: Throttles
    threshold: 50
    statistic: Sum
    period: 60
    evaluationPeriods: 1
    comparisonOperator: GreaterThanThreshold
```

## License

MIT © [A Cloud Guru](https://acloud.guru/)


[npm-image]: https://badge.fury.io/js/serverless-plugin-aws-alerts.svg
[npm-url]: https://npmjs.org/package/serverless-plugin-aws-alerts
[travis-image]: https://travis-ci.org/ACloudGuru/serverless-plugin-aws-alerts.svg?branch=master
[travis-url]: https://travis-ci.org/ACloudGuru/serverless-plugin-aws-alerts
[daviddm-image]: https://david-dm.org/ACloudGuru/serverless-plugin-aws-alerts.svg?theme=shields.io
[daviddm-url]: https://david-dm.org/ACloudGuru/serverless-plugin-aws-alerts
[coveralls-image]: https://coveralls.io/repos/ACloudGuru/serverless-plugin-aws-alerts/badge.svg
[coveralls-url]: https://coveralls.io/r/ACloudGuru/serverless-plugin-aws-alerts
