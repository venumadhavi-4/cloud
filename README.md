**Cloud-Based CSV Processing System**  

**Overview**  
This project demonstrates the development of a local cloud-based system that processes CSV files using **Localstack**, an open-source AWS cloud emulator. The system handles file uploads to an S3 bucket, extracts metadata, and stores the metadata in a database.  



**Project Features**  
1. **CSV File Processing**:  
   - Upload CSV files (up to 10MB) to an S3 bucket.  
   - Automatically trigger a Lambda function to process the file.  
   - Extract metadata (row count, column count, column names, etc.).  

2. **Metadata Storage**:  
   - Store extracted metadata in a database (AWS RDS or DynamoDB).  

3. **Notifications** (Optional):  
   - Send notifications after processing is completed.  

4. **Error Handling**:  
   - Handle invalid CSV formats or upload failures gracefully.  

---

 **Project Structure**  
```
cloud-engineer-assignment/
├── app/
│   ├── lambda_function.py         # Code for AWS Lambda
│   ├── utils.py                   # Helper functions
│   ├── requirements.txt           # Dependencies for Lambda
├── database/
│   ├── setup_dynamodb.py          # Script to configure DynamoDB
│   ├── setup_rds.py               # Script to configure RDS
├── scripts/
│   ├── s3_upload.py               # Script to upload CSV files to S3
├── README.md                      # Documentation
```

---
**System Workflow**  
1. **CSV File Upload**:  
   Users upload a CSV file to an S3 bucket.  
2. **Lambda Trigger**:  
   The upload triggers an AWS Lambda function via an S3 event.  
3. **File Processing**:  
   The Lambda function:  
   - Downloads the file from S3.  
   - Extracts metadata (e.g., file size, row/column count, column names).  
   - Stores metadata in the database (DynamoDB or RDS).  
4. **Notifications** (Optional):  
   - Notify users about the successful file processing.  

---

**Technologies Used**  
1. **Localstack**: For emulating AWS services (S3, Lambda, DynamoDB).  
2. **Python**: For implementing Lambda functions and scripts.  
3. **Boto3**: AWS SDK for Python to interact with AWS services.  
4. **Pandas**: For processing CSV files.  
5. **DynamoDB or RDS**: To store extracted metadata.  

---

**Prerequisites**  
1. Install Python and Pip.  
2. Install Docker to run Localstack.  
3. Install required Python packages:
   ```bash
   pip install boto3 pandas
   ```
4. Set up Localstack:
   ```bash
   docker run -d -p 4566:4566 -p 4571:4571 localstack/localstack
   ```

---

## **Detailed Instructions**  

### **Step 1: Setting Up S3 Bucket**  
1. Create an S3 bucket in Localstack:
   ```bash
   aws --endpoint-url=http://localhost:4566 s3 mb s3://my-csv-bucket
   ```

2. Verify the bucket:
   ```bash
   aws --endpoint-url=http://localhost:4566 s3 ls
   ```

---

### **Step 2: Lambda Function Deployment**  
1. Package the Lambda function:
   ```bash
   cd app/
   zip function.zip lambda_function.py utils.py
   ```

2. Deploy the Lambda function:
   ```bash
   aws --endpoint-url=http://localhost:4566 lambda create-function \
       --function-name process_csv \
       --runtime python3.9 \
       --role arn:aws:iam::000000000000:role/lambda-role \
       --handler lambda_function.lambda_handler \
       --code S3Bucket="__local__",S3Key="function.zip"
   ```

3. Add an S3 event trigger for the Lambda function:
   ```bash
   aws --endpoint-url=http://localhost:4566 s3api put-bucket-notification-configuration \
       --bucket my-csv-bucket \
       --notification-configuration file://s3-notification.json
   ```

---

### **Step 3: Database Setup**  
1. **For DynamoDB**:
   - Create a table:
     ```bash
     aws --endpoint-url=http://localhost:4566 dynamodb create-table \
         --table-name MetadataTable \
         --attribute-definitions AttributeName=FileName,AttributeType=S \
         --key-schema AttributeName=FileName,KeyType=HASH \
         --billing-mode PAY_PER_REQUEST
     ```

2. **For RDS**:
   - Use the `setup_rds.py` script to configure an RDS database.  

---

### **Step 4: Upload CSV File**  
Use the `s3_upload.py` script to upload a CSV file to the bucket:
```bash
python scripts/s3_upload.py --file sample.csv
```

---

### **Step 5: Testing**  
1. Verify metadata is stored in DynamoDB or RDS.  
2. Check the Lambda function logs for any errors:
   ```bash
   aws --endpoint-url=http://localhost:4566 logs describe-log-streams --log-group-name /aws/lambda/process_csv
   ```

---

## **Sample Metadata**  
Here’s an example of extracted metadata:
```json
{
  "filename": "example.csv",
  "upload_timestamp": "2024-12-18 10:00:00",
  "file_size_bytes": 1048576,
  "row_count": 1000,
  "column_count": 5,
  "column_names": ["id", "name", "age", "city", "date"]
}
```

---

## **Future Improvements**  
1. Add email notifications using Amazon SNS.  
2. Implement advanced error handling for file processing.  
3. Support larger files by using multipart uploads.

---

