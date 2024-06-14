# Real-Time Data Pipeline with AWS and Snowflake

![Kinesis_Apigateway_S3_Snowflake_Lambda](https://github.com/Moiz081/Real-Time-Data-Pipeline-with-AWS-and-Snowflake/blob/main/Kinesis_Apigateway_S3_Snowflake_Lambda.png?raw=true)


Use Postman to make an API call, the Amazon API Gateway will trigger a Lambda function, the function will write into the s3 bucket, then Snowpipe will start to write the data into a Snowflake. To not activate Snowpipe for every API call and optimize the process, we will use Amazon Kinesis data Stream to collect the data first (dump the data based on the buffer size or the time or if there are few messages), then use Kinesis Firehouse to leave the data into s3. 

 

Step 1: Create the Lambda Role 
- 


<img width="957" alt="Screen Shot 2024-01-02 at 12 55 54 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/3594ab96-fceb-4bce-90af-e6567cf32f83">





Step 2: Create the lambda function to read the data from API Gateway and put it in Kinesis Data Stream
- 

<img width="1263" alt="Screen Shot 2024-01-02 at 12 57 50 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/8f5184fa-e91b-4922-b215-15cef8978c14">


<img width="1254" alt="Screen Shot 2024-01-02 at 12 57 59 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/0d5a7ea4-7224-4a1f-89e3-b7553e2eaf0d">


			import json
			import datetime
			import random
			import boto3
			
			client = boto3.client('kinesis')
			
			def lambda_handler(event, context):
			    TODO implement
			    data = json.dumps(event['body'])  
			    client.put_record(StreamName="project3_kinesis_apigateway", Data=data, PartitionKey="1")
			    print("Data Inserted")
 

Step 3: Create API Gateway and make the integration with AWS Lambda created in Step 2 

- Create a simple HTTP API and integrate it with the lambda function.

<img width="1264" alt="Screen Shot 2024-01-02 at 1 03 12 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/7463dcef-ea73-4f20-a476-c7cb484d2310">




- Configure the route: POST method.

 <img width="1267" alt="Screen Shot 2024-01-02 at 1 07 18 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/505f5ede-d66a-43ed-9dff-0a282159ba55">


 <img width="1239" alt="Screen Shot 2024-01-02 at 1 07 37 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/d77a0bb0-68be-4104-8147-56efb3fc24da">


 <img width="1274" alt="Screen Shot 2024-01-02 at 1 08 08 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/d2f81373-df97-4cc3-8b8a-65f48ee9d8cf">
	

 <img width="1263" alt="Screen Shot 2024-01-02 at 1 10 11 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/2bababbe-58cf-4821-b673-841575dc3b1a">
	


Get the API endpoint " https://fwpwl5qova.execute-api.us-east-1.amazonaws.com/dev/project3_lambda1_kinesis_apigateway" 

 

Step 4: Create a Kinesis Data Stream to consume data from AWS Lambda created in Step 2 
 - 

<img width="1268" alt="Screen Shot 2024-01-02 at 1 23 24 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/cad76676-cc3d-44d0-acdf-7afa49c27464">


Step 5: Create a second Lambda function for processing the data before s3 dump
- 

<img width="1257" alt="Screen Shot 2024-01-02 at 1 36 27 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/06f00a74-b04e-47be-b75b-1f19db6889e2">



This second lambda function will transform the data( decode the data to put a delimiter in between each record) to make it easy to put on Snowflake. We will convert the binary data into string data. Then, add a \n to make a new line. 

			import json
			import boto3
			import base64
			output = []
			
			def lambda_handler(event, context):
			    print(event)
			    for record in event['records']:
			        payload = base64.b64decode(record['data']).decode('utf-8')
			        print('payload:', payload)
			        
			        row_w_newline = payload + "\n"
			        print('row_w_newline type:', type(row_w_newline))
			        row_w_newline = base64.b64encode(row_w_newline.encode('utf-8'))
			        
			        output_record = {
			            'recordId': record['recordId'],
			            'result': 'Ok',
			            'data': row_w_newline
			        }
			        output.append(output_record)
			
			    print('Processed {} records.'.format(len(event['records'])))
			    
			    return {'records': output}
-





Step 6: Create a S3 bucket that will be the kinesis Firehose destination
- 

S3:
<img width="1263" alt="Screen Shot 2024-01-02 at 1 40 26 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/3cdf54ee-2d52-4c43-96de-d89e82407690">


Step 7: Create Kinesis Firehose
-	

- Create kinesis firehouse 

<img width="1242" alt="Screen Shot 2024-01-02 at 1 40 45 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/66974308-33e6-489d-b09d-61db456a8173">


<img width="1267" alt="Screen Shot 2024-01-02 at 1 41 48 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/09c44659-85b0-44f2-9f3a-a6fe35cb232c">


- Activate Transform source records with AWS Lambda. 
 
<img width="1261" alt="Screen Shot 2024-01-02 at 1 43 21 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/7a9ef4c0-4557-4fcd-bbff-f55e2776ec82">

 

Step 8: Create a Snowflake role. 
- 

<img width="1249" alt="Screen Shot 2024-01-02 at 2 02 22 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/8ffa3bb8-b66e-4e64-b34a-d9a66f501de5">



Storage Integration Creation: Copy the snowflake role arm into the snowflake console and copy the s3 bucket arm role. 

			create warehouse s3_to_snowflake_wh;
			use s3_to_snowflake_wh;
			--Specify the role
			use role ACCOUNTADMIN;
			
			drop database if exists s3_to_snowflake;
			
			--Database Creation 
			create database if not exists s3_to_snowflake;
			
			--Specify the active/current database for the session.
			use s3_to_snowflake;
			
			
			--Storage Integration Creation
			create or replace storage integration s3_int
			TYPE = EXTERNAL_STAGE
			STORAGE_PROVIDER = S3
			ENABLED = TRUE 
			STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::200105849428:role/project3_snwoflake_role'
			STORAGE_ALLOWED_LOCATIONS = ('s3://gakas-project3-kinesis-apigateway')
			COMMENT = 'Testing Snowflake getting refresh or not';
			
			--Describe the Integration Object
			DESC INTEGRATION  s3_int;
			
			
			--External Stage Creation
			
			create stage mystage
			  url = 's3://gakas-project3-kinesis-apigateway'
			  storage_integration = s3_int;
			
			list @mystage;
			
			--File Format Creation
			create or replace file format my_json_format
			type = json;
			
			 
			--Table Creation
			create or replace external table s3_to_snowflake.PUBLIC.Person with location = @mystage file_format ='my_json_format';
			show external tables;
			
			select * from s3_to_snowflake.public.person;
			 
			 
			--Query the table
			select parse_json(VALUE):Age as Age  , trim(parse_json(VALUE):Name,'"') as Name from  s3_to_snowflake.PUBLIC.Person;

- 

Copy the 'STORAGE_AWS_IAM_USER_ARN' ARN and 'STORAGE_AWS_EXTERNAL_ID' from Snowflake and update the Trust Policy in the Snowflake role in IAM. 

 <img width="1271" alt="Screen Shot 2024-01-02 at 2 19 25 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/52f82acd-e723-4280-918a-963af3ffb56f">
 
- 

Create an event notification for the s3 bucket. 

<img width="1263" alt="Screen Shot 2024-01-02 at 2 26 11 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/4d487de5-ef90-477c-ad28-64ed8f62b31e">
 
-	

<img width="1269" alt="Screen Shot 2024-01-02 at 2 26 17 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/aa4106d0-68b9-4cea-a7b9-2dfa9b8e432b">

-	

<img width="1228" alt="Screen Shot 2024-01-02 at 2 26 26 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/4b502a0f-8d27-4596-b991-84a4f6cdcc07">


Step 9: Test the pipeline 
- 

- Open Postman and pass a series of records 
		{"Name": "wang", "Age":4}
		{"Name": "ali", "Age":32}
		{"Name": "li", "Age":54}
		{"Name": "Moctar", "Age":44}
		{"Name": "she", "Age":86}
		{"Name": "Abdoul", "Age":22}
		{"Name": "lie", "Age":34}
		{"Name": "Cheng", "Age":55}
		{"Name": "Karim", "Age":23}
		{"Name": "ram", "Age":34}
		{"Name": "li", "Age":23}
		{"Name": "she", "Age":36}
 
- Check Kinesis Data Stream and Kinesis Firehose metrics:
  <img width="1156" alt="Screen Shot 2024-01-02 at 2 32 54 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/282d172d-0c09-4d69-8a21-f4620d1bfe39">


<img width="1073" alt="Screen Shot 2024-01-02 at 3 27 12 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/88b99ad9-d590-42b9-9954-aa146f68300c">

- Check S3
<img width="1115" alt="Screen Shot 2024-01-02 at 3 27 30 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/f7e6ac3e-8dcc-4884-b504-6b6e08b680d6">

- Check the data into snowflake

<img width="1273" alt="Screen Shot 2024-01-02 at 3 07 30 PM" src="https://github.com/gakas14/Serverless-data-stream-with-kinesis-data-stream-kinesis-firehouse-S3-and-snowflake/assets/74584964/52ab3533-281c-4dda-a626-b86333d488b6">
 
