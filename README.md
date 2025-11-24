# The-Cloud-Resume-Challenge-Project
What is the Cloud Resume Challenge?
The Cloud Resume Challenge is a popular, hands-on project created by Forrest Brazeal, a cloud architect and advocate, to help aspiring cloud professionals gain real-world experience with cloud technologies. This challenge isn’t just another technical exercise — it’s designed to push you to combine your technical skills in a way that mirrors the actual responsibilities of a cloud engineer.

Here’s the typical AWS stack the challenge requires:
1. HTML/CSS Resume
You create a simple resume webpage.
Store it in Amazon S3 as a static website.
2. CloudFront
You serve your site globally using CloudFront CDN.
3. DNS (Route 53)
Connect a domain name (optional but recommended).
4. Visitor Counter
Your site has a real-time visitor counter:
Use DynamoDB to store the count
Use AWS Lambda to update count
Expose Lambda through API Gateway
Your JavaScript on the webpage calls the API and displays the count
5. Infrastructure as Code
Use Terraform, CloudFormation, or AWS SAM to automate everything.
6. Backend + Frontend Tests
Write small tests for Lambda + website.
7. CI/CD Pipelines
Use GitHub Actions, AWS CodeBuild, or similar to automatically deploy:
Frontend to S3/CloudFront
Backend to Lambda/APIGW/DynamoDB
🎯 What It's Meant to Teach You
Real-world cloud architecture
Serverless development
Databases (NoSQL)
CI/CD pipelines
Networking & DNS
Infrastructure as Code
Version control (Git/GitHub)

<img width="720" height="285" alt="image" src="https://github.com/user-attachments/assets/ff9f006f-c5b2-411d-b403-3bc4e7477ae0" />
Diagram for the completed project


# Part 1: Building the Frontend
Setting up the frontend for my resume website initially felt simple. I created a basic layout using HTML and CSS and hosted it in an S3 bucket. Later, I refined the design by customising a template from BootstrapMade for a more polished look.
When it came to serving the site globally, I planned to use CloudFront. Even though I had worked with CloudFront before, I decided to dig deeper into the documentation. That’s when I learned that using the S3 REST API endpoint as the origin—combined with an Origin Access Control (OAC)—is the more secure and AWS-recommended approach, rather than using the S3 website endpoint. CloudFront is a Content Delivery Network (CDN) that caches content on edge servers around the world. This ensures faster loading times for viewers regardless of location and adds a layer of security.

<img width="1390" height="378" alt="image" src="https://github.com/user-attachments/assets/984b6022-e4c9-47d9-95d3-ee11ad69a84e" />

With this configuration, I ensured that the site could only be accessed through CloudFront and not directly from the S3 bucket. 
The bucket policy statement was copied into S3 from CloudFront and it ensured me of independent access through CloudFront.

<img width="677" height="413" alt="image" src="https://github.com/user-attachments/assets/435e3c63-f8c0-4926-ac1e-6e8717ae11c1" />

Lastly, I purchased a domain from namecheap.com and configured the custom domain. I created an SSL/TLS certificate in ACM and attached it to the CloudFront distribution so the site would be served securely over HTTPS. Finally, I configured Amazon Route 53 to route my domain’s traffic to the CloudFront distribution. The alternate domain used was khalidresume.hudhayfah.cloud.

<img width="1844" height="574" alt="image" src="https://github.com/user-attachments/assets/2ad8fa56-356f-49ff-ac41-2f2a3113c418" />


# Part 2: Building the Backend
# Building the API

First, I created a DynamoDB table to store the view count. This table contains a single item with a unique primary key (id) and an attribute called views, which holds the current number of page visits.

<img width="759" height="107" alt="image" src="https://github.com/user-attachments/assets/1b715e9e-d6c4-406c-87ab-354c3f07e68c" />

I then developed an AWS Lambda function using Python and the Boto3 SDK. Every time the website loads, a request is sent to this Lambda function via Amazon API Gateway. The function reads the current view count from DynamoDB, increments it by one, and writes the updated value back to the table.
This process happens dynamically and in real-time, allowing the counter displayed on the website to update automatically with each visit. Because Lambda is serverless, it scales automatically and only runs when triggered, making the solution cost-effective and efficient. DynamoDB ensures fast, reliable data storage with low latency.
This architecture demonstrates the use of cloud-native services to create a scalable and automated visitor tracking system without managing any traditional servers.

<img width="671" height="601" alt="image" src="https://github.com/user-attachments/assets/e3e6948d-f2bc-403f-979e-0ddee05907ed" />


Next, I created a JavaScript code in order to create a visitor counter that displays how many people have accessed the site. 
The JavaScript code is not talking directly to the DynamoDB.
Instead, Amazon API Gateway is set with one POST route, proxying request to a Lambda function responsible for updating a visitor counter.

<img width="784" height="663" alt="image" src="https://github.com/user-attachments/assets/416b774e-5bd5-4dc9-90c4-521fc9522c57" />

<img width="299" height="259" alt="image" src="https://github.com/user-attachments/assets/d3fd8c76-0fe2-449d-8b83-b3fbecaa5717" />
The value coming from DynamoDB through AWS Lambda into the JavaScript code, allows the page to dynamically count and display the visitors number. Upon completion of this step my my frontend and backend have fully been integrated.















# Problem that occured during the Project
Cloud Resume View Counter — Issues Encountered & How They Were Fixed
Throughout implementing the serverless view counter for my Cloud Resume Challenge, I encountered several issues across Lambda, IAM, DynamoDB, CORS, CloudFront caching, and frontend JavaScript. Below is a detailed breakdown of each problem and how it was resolved.

1. DynamoDB UpdateItem Failed (AccessDeniedException)
Problem
My Lambda function failed with:
AccessDeniedException: User is not authorized to perform dynamodb:UpdateItem
Cause
The Lambda execution role did not have permissions to access my DynamoDB table (cloudresume-test).
Fix
In IAM → Roles → MyResumeCounterFunction-role, I attached a policy allowing:
dynamodb:GetItem
dynamodb:UpdateItem
On the specific table ARN.
This allowed Lambda to update the view count successfully.

2. Reserved Keyword Error for “views”
Problem
Lambda threw:
Invalid UpdateExpression: Attribute name is a reserved keyword
Cause
DynamoDB treats views as a reserved keyword, so it cannot be used directly inside an UpdateExpression.
Fix
Used ExpressionAttributeNames to alias it:
UpdateExpression="SET #v = if_not_exists(#v, :start) + :inc",
ExpressionAttributeNames={'#v': 'views'}

3. Writing to Wrong Item ID
Problem
My earlier Lambda code read from:
Key={'id': '0'}
…but wrote back to:
'id': '1'
Cause
This created a mismatch: item “0” never updated, causing crashes later when it was missing attributes.
Fix
Wrote back to the same key:
'id': '0'

4. Missing DynamoDB Item
Problem
get_item() failed because id = 0 didn’t exist yet.
Fix
Manually created the item in DynamoDB:
{
  "id": "0",
  "views": 0
}

5. CORS Error: “Access-Control-Allow-Origin cannot contain more than one origin”
Problem
Browser console showed:
Access-Control-Allow-Origin cannot contain more than one origin.
Cause
Lambda (or API Gateway) was returning multiple CORS headers or an incorrectly formatted header.
Fix
Ensured Lambda returned ONLY this header:
"Access-Control-Allow-Origin": "*"
No duplicates, no extra spaces, no arrays.

6. CloudFront Cache Not Updating
Problem
HTML changes did not appear (counter stuck showing “Error”).
Cause
CloudFront was serving a cached version of the old index.html.
Fix
Created an invalidation:
/*
CloudFront then fetched the new HTML + JS and the counter updated correctly.

7. Frontend Could Not Hit Lambda URL (Failed Fetch)
Problem
Browser console showed:
Failed to load resource due to access control checks.
TypeError: Load failed
Cause
This was caused by:
CORS failing (fixed earlier), or
Cached HTML serving old JS, or
Frontend not being refreshed through CloudFront.
Fix
Once CORS and CloudFront invalidation were resolved, the fetch request worked correctly.




