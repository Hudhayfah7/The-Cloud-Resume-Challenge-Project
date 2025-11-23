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

<img width="716" height="428" alt="image" src="https://github.com/user-attachments/assets/26d5d9c4-c86d-465c-8943-18dd72e46240" />

Next, I created a JavaScript code in order to create a visitor counter that displays how many people have accessed the site. 
The JavaScript code is not talking directly to the DynamoDB.
Instead, Amazon API Gateway is set with one POST route, proxying request to a Lambda function responsible for updating a visitor counter.



