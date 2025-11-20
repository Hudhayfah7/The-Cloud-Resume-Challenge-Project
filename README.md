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

Lastly, I created an SSL/TLS certificate in ACM and attached it to the CloudFront distribution so the site would be served securely over HTTPS. Finally, I configured Amazon Route 53 to route my domain’s traffic to the CloudFront distribution. 

# Part 2: Building the Backend
