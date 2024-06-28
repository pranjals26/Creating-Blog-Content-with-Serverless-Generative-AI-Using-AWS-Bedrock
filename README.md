![alt text](https://github.com/pranjals26/Creating-Blog-Content-with-Serverless-Generative-AI-Using-AWS-Bedrock/blob/main/llama%20AWS.jpg)

# Creating-Blog-Content-with-Serverless-Generative-AI-Using-AWS-Bedrock

Generative AI has revolutionized various industries, offering innovative solutions that were previously unimaginable. Recently, I undertook an exciting project to create a serverless Generative AI product using AWS. This project leverages AWS Bedrock, API Gateway, S3, and Lambda to generate blog content from user queries.

![alt text](https://github.com/pranjals26/Creating-Blog-Content-with-Serverless-Generative-AI-Using-AWS-Bedrock/blob/main/flow%20diagram.png)


The project aims to develop a serverless Generative AI that generates blog content based on user queries. The architecture consists of:
- API Gateway: Manages API requests.
- Lambda: Handles backend logic.
- AWS Bedrock: Provides foundational models for content generation.
- S3: Stores the generated content.

This is a step by step guide on how you can use AWS Bedrock to create AI applications on cloud.
## Setup Instructions

### AWS Bedrock Configuration

AWS Bedrock provides access to foundational AI models

1. **Configure Bedrock**:
   - Explore available models in AWS Bedrock.
   - Ensure correct permissions and model access for the Lambda function.
   - In the project, I have used LLaMa 2 Chat 13B

<img width="468" alt="image" src="https://github.com/pranjals26/Creating-Blog-Content-with-Serverless-Generative-AI-Using-AWS-Bedrock/assets/41803622/c074fac0-840c-4705-8459-20710912f0eb">


### API Creation

### Lambda Function

1. **Create Lambda Function**:
   - Navigate to the [AWS Lambda console](https://console.aws.amazon.com/lambda).
   - Create a new Lambda function.
   - Set up environment variables and permissions for accessing Bedrock and S3.

2. **Lambda Function Code**:
   - Create a virtual environment and install necessary libraries:
     ```bash
     python3 -m venv venv
     source venv/bin/activate
     pip install boto3
     ```
   
   - Compress and save the boto3 file in a folder, then compress it. Create a custom layer in the Lambda function and add the zip file to it.
   - Write Python code to handle the API request, generate content using Bedrock, and save to S3.Refer to app.py file in the repoistory. 

     ```python
      import boto3
      import botocore.config
      import json
      
      from datetime import datetime
      
      def blog_generate_using_bedrock(blogtopic:str)-> str:
          prompt=f"""<s>[INST]Human: Write a 200 words blog on the topic {blogtopic}
          Assistant:[/INST]
          """
      
          body={
              "prompt":prompt,
              "max_gen_len":512,
              "temperature":0.5,
              "top_p":0.9
          }
      
          try:
              bedrock=boto3.client("bedrock-runtime",region_name="us-west-1",
                                   config=botocore.config.Config(read_timeout=300,retries={'max_attempts':3}))
              response=bedrock.invoke_model(body=json.dumps(body),modelId="meta.llama2-13b-chat-v1")
      
              response_content=response.get('body').read()
              response_data=json.loads(response_content)
              print(response_data)
              blog_details=response_data['generation']
              return blog_details
          except Exception as e:
              print(f"Error generating the blog:{e}")
              return ""
      
      def save_blog_details_s3(s3_key,s3_bucket,generate_blog):
          s3=boto3.client('s3')
      
          try:
              s3.put_object(Bucket = s3_bucket, Key = s3_key, Body =generate_blog )
              print("Code saved to s3")
      
          except Exception as e:
              print("Error when saving the code to s3")
      
      
      
      def lambda_handler(event, context):
          # TODO implement
          event=json.loads(event['body'])
          blogtopic=event['blog_topic']
      
          generate_blog=blog_generate_using_bedrock(blogtopic=blogtopic)
      
          if generate_blog:
              current_time=datetime.now().strftime('%H%M%S')
              s3_key=f"blog-output/{current_time}.txt"
              s3_bucket='aws-bedrock-bloggen-pranjal'
              save_blog_details_s3(s3_key,s3_bucket,generate_blog)
      
      
          else:
              print("No blog was generated")
      
          return{
              'statusCode':200,
              'body':json.dumps('Blog Generation is completed')
          }
     ```
     - Change the S3 bucket name, region and foundational model using for application


1. **Create API Gateway**:
   - Go to the [Amazon API Gateway console](https://console.aws.amazon.com/apigateway).
   - Create a new API.
   - Define endpoints and methods (e.g., POST for content generation).
   - Create a dev stage

<img width="468" alt="image" src="https://github.com/pranjals26/Creating-Blog-Content-with-Serverless-Generative-AI-Using-AWS-Bedrock/assets/41803622/c093e414-8785-49d5-abcf-37355c2a8bfd">


<img width="468" alt="image" src="https://github.com/pranjals26/Creating-Blog-Content-with-Serverless-Generative-AI-Using-AWS-Bedrock/assets/41803622/d7bebc34-01a6-453d-8663-369f8642800e">


2. **Postman Setup**:
   - Use Postman to send HTTP requests to the API and test its functionality.

### S3 for Storage

Create an S3 bucket to store the generated blog content:

1. **Create S3 Bucket**:
   - Go to the [S3 console](https://console.aws.amazon.com/s3).
   - Create a new bucket with appropriate permissions.
   - Configure the Lambda function to save the generated content in this bucket.
   - In the lambda code, change the bucket name with what you have used

## Testing and Debugging

1. **API Testing**:
   - Use Postman to send requests
   - Ensure there are no extra space in the API and the API triggers the Lambda function correctly 
  
     <img width="468" alt="image" src="https://github.com/pranjals26/Creating-Blog-Content-with-Serverless-Generative-AI-Using-AWS-Bedrock/assets/41803622/9f7c3485-6ad0-4a51-a782-245356ff0517">

   
2. **Monitoring**:
   - Use AWS CloudWatch to monitor logs and debug any issues with the Lambda function.
   - In the Cloudwatch you can see you the steps being follwed from API to output text file being saved in the S3 bucket.

     <img width="468" alt="image" src="https://github.com/pranjals26/Creating-Blog-Content-with-Serverless-Generative-AI-Using-AWS-Bedrock/assets/41803622/2fa39d12-e520-4426-8698-b72d51c93578">


## Key Features

- **Serverless Architecture**: Utilizes AWS serverless services for scalability and cost efficiency.
- **Generative AI**: Employs advanced AI models in AWS Bedrock for high-quality content generation.
- **Seamless Integration**: Demonstrates the integration of multiple AWS services for automated content generation.
- **Extensibility**: The solution can be extended to various applications like document generation, Q&A systems, etc.

## Conclusion

This project showcases a comprehensive guide to building a serverless Generative AI solution on AWS. By following the step-by-step instructions, you can replicate this process for similar use cases, leveraging AWS’s robust infrastructure to develop scalable and efficient AI-driven applications.

## Contributing

Contributions are welcome! Please fork this repository and submit a pull request for any enhancements or bug fixes.

## Video Result
https://github.com/pranjals26/Creating-Blog-Content-with-Serverless-Generative-AI-Using-AWS-Bedrock/assets/41803622/116d7a93-4abd-4145-9b31-524385833d1c

