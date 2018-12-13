AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Resources:
  DatalogFunction:
    Type: 'AWS::Serverless::Function'
    Properties:
      Handler: datalog.lambda_handler
      Runtime: python3.6
      CodeUri: 'lambda/datalog.py'
      MemorySize: 256
      Timeout: 15
      Policies:
       - AWSLambdaKinesisExecutionRole
      Events:
        StreamData:
          Type: Kinesis
          Properties:
            BatchSize: 50
            StartingPosition: "TRIM_HORIZON"
            Stream: !GetAtt DataStream.Arn


  DatastoreFunction:
    Type: 'AWS::Serverless::Function'
    Properties:
      Handler: datastore.lambda_handler
      Runtime: python3.6
      CodeUri: 'lambda/datastore.py'
      MemorySize: 256
      Timeout: 15
      Policies:
       - AWSLambdaFullAccess
      Environment:
        Variables:
          TABLE_NAME: !Ref DatapipeTable
      Events:
        StreamData:
          Type: Kinesis
          Properties:
            Stream: !GetAtt DataStream.Arn
            StartingPosition: "TRIM_HORIZON"
            BatchSize: 50

  DataStream:
    Type: AWS::Kinesis::Stream
    Properties:
      Name: datastream
      ShardCount: 1
      Tags:
        -
          Key: 'Project'
          Value: 'SEIS665'

  DynamoDBTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: DatapipeTable
      AttributeDefinitions:
        -
          AttributeName: 'id'
          AttributeType: 'S'
      KeySchema:
        -
          AttributeName: 'id'
          KeyType: 'HASH'
      ProvisionedThroughput:
        ReadCapacityUnits: 5
        WriteCapacityUnits: 5
      SSESpecification:
        SSEEnabled: true