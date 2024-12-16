# Serverless-Email-Marketing-Application
This guide explains how to set up a process to send emails using AWS S3, SES, and Lambda with a scheduled trigger using EventBridge.

![Cloud Architecture](images/cloud-architecture.png)
# Sending Emails Using AWS S3, SES, and Lambda

This guide explains how to set up a process to send emails using AWS S3, SES, and Lambda with a scheduled trigger using EventBridge.

---

## Steps to Set Up

### Step 1: Create an S3 Bucket and Upload Files
1. Navigate to the **S3 Console**.
2. Create a bucket with a globally unique name (e.g., `t-email-marketing`).
   - Ensure **Block Public Access** is enabled (default and recommended for security).
![Step 1](images/1.png)
![Step 2](images/2.png)
3. Upload the following files into the bucket:
   - `email_template.html`
   - `contacts.csv` (list of contact details)
![Step 3](images/3.png)
![Step 4](images/4.png)

---

### Step 2: Configure Amazon SES
1. Open the **SES Console** and set up your environment:
   - Configure a sandbox if you are new.
   - Add and verify an email identity for sending emails.
![Step 5](images/5.png)
2. Go to **Identities** under **Configuration** and:
   - Add recipient email addresses (e.g., `example1@gmail.com` and `example2@gmail.com`).
   - Verify these emails by clicking on the link sent to them.
![Step 6](images/6.png)
![Step 9](images/9.png)
![Step 7](images/7.png)
![Step 11](images/11.png)

---

### Step 3: Create a Lambda Function

1. Navigate to the **Lambda Console** and create a new function:
   - **Name**: `SendSESEmailToContacts`
   - **Runtime**: Python 3.x

2. Paste the following code into the editor:
   ```python
   import boto3
   import csv

   # Initialize the boto3 client
   s3_client = boto3.client('s3')
   ses_client = boto3.client('ses')

   def lambda_handler(event, context):
       # Specify the S3 bucket name
       bucket_name = 't-email-marketing'  # Replace with your bucket name

       try:
           # Retrieve the CSV file from S3
           csv_file = s3_client.get_object(Bucket=bucket_name, Key='contacts.csv')
           lines = csv_file['Body'].read().decode('utf-8').splitlines()
           
           # Retrieve the HTML email template from S3
           email_template = s3_client.get_object(Bucket=bucket_name, Key='email_template.html')
           email_html = email_template['Body'].read().decode('utf-8')
           
           # Parse the CSV file
           contacts = csv.DictReader(lines)
           
           for contact in contacts:
               # Replace placeholders in the email template with contact information
               personalized_email = email_html.replace('{{FirstName}}', contact['FirstName'])
               
               # Send the email using SES
               response = ses_client.send_email(
                   Source='you@yourdomainname.com',  # Replace with your verified "From" address
                   Destination={'ToAddresses': [contact['Email']]},
                   Message={
                       'Subject': {'Data': 'Your Weekly Tiny Tales Mail!', 'Charset': 'UTF-8'},
                       'Body': {'Html': {'Data': personalized_email, 'Charset': 'UTF-8'}}
                   }
               )
               print(f"Email sent to {contact['Email']}: Response {response}")
       except Exception as e:
           print(f"An error occurred: {e}")

![Step 15](images/15.png)
![Step 17](images/17.png)
Replace your-bucket-name and your-verified-email@example.com with your actual bucket name and source SES verified email.

3. Deploy the function and create a test event (e.g., TestSendEmail).
![Step 18](images/18.png)
![Step 19](images/19.png)
4. Run the test. If an error occurs, it is likely due to insufficient permissions.
![Step 20](images/20.png)

### Step 4: Add IAM Permissions for Lambda

1. Under Execution Role, locate the role associated with your Lambda function (e.g., SendSESEmailToContacts-role-22j4ztrm) and click on it.
![Step 22](images/22.png)
2. This opens the IAM Console. You will see that only CloudWatch permissions are assigned by default.
![Step 24](images/24.png)
4. Create a new IAM policy:
Go to Policies in the IAM Console and click Create Policy.
Paste the following JSON into the policy editor:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": ["s3:GetObject"],
               "Resource": "arn:aws:s3:::your-bucket-name/*"
           },
           {
               "Effect": "Allow",
               "Action": ["ses:SendEmail", "ses:SendRawEmail"],
               "Resource": "*"
           }
       ]
   }

Replace your-bucket-name with your actual S3 bucket name.
![Step 25](images/25.png)
![Step 26](images/26.png)
![Step 27](images/27.png)
![Step 28](images/28.png)

4. Attach this new policy to your Lambda function's Execution Role.
![Step 30](images/30.png)
![Step 31](images/31.png)
![Step 32](images/32.png)
![Step 33](images/33.png)

5. Test the Lambda function to ensure it runs without errors.
![Step 38](images/38.png)

### Step 5: Schedule Lambda with EventBridge
![Step 39](images/39.png)
1. Go to the **EventBridge Console** and create a new schedule:
   - **Name**: `SendWeeklyEmail2`
   - **Schedule Pattern**: Choose either a **one-time** or **recurring** schedule.
 ![Step 40](images/40.png)
![Step 41](images/41.png)
2. Set the **Target** to invoke your Lambda function `SendSESEmailToContacts`.
![Step 42](images/42.png)
![Step 43](images/43.png)
![Step 44](images/44.png) 
4. Review and confirm the schedule.
![Step 45](images/45.png)
![Step 46](images/46.png)
---
5. You will receive the following mail at your scheduled time based on the given email template.
![Step 47](images/47.png)
## Monitoring and Debugging

- Use the **Monitor** tab in the **Lambda Console** to view execution logs.
- Check **CloudWatch Logs** for troubleshooting and insights.

![Step 49](images/49.png)
![Step 50](images/50.png)
![Step 52](images/52.png)
![Step 53](images/53.png)
---

## Conclusion

By following these steps, you can automate email delivery using **AWS S3**, **SES**, and **Lambda** with minimal setup.












