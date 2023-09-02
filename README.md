# react-ui
## 1. Run locally
- Update `package.json` as needed
- Install dependencies : `npm install`
- Run : `npm run dev`

## 2. Deploy on AWS with CI/CD
### Objective
- Push on GitHub triggers the CodePipeline
- CodePipeline builds and deploys the static website
- Static website is deployed on S3 with CloudFront
### Process
#### 1. Create an S3 bucket
- In properties, click on Static website hosting & choose use this bucket to host a website, indicate the index file (generally index.html)
- In authorisations, deactivate the field "Block all public access" 
- In authorisations, under compartiment strategies, add the following code (change the bucket name)
```
{
"Version": "2012-10-17",
"Statement": [
    {
        "Sid": "AddPerm",
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::react-ui/*"
    }
]
}
```
#### 2. Create a CloudFront distribution
- Choose the domain name origin of your S3 bucket
- Give it a name
- Activate the WAF protection if needed

#### 3. Create a CodePipeline
- Give it a name and create a service role
##### 1. Source stage
- Choose GitHub 2.0 and follow the steps to connect the repository
##### 2. Build stage
- Choose AWS CodeBuild and create a new CodeBuild project
- The buildspec.yml file is already in the repository
- Set this configuration for the execution environment : `aws/codebuild/amazonlinux2-x86_64-standard:5.0`
- Modify the buildspec.yml file, change the S3 bucket name and the distribution ID:
    - `aws s3 sync dist s3://immo-ui --delete`
    - `aws cloudfront create-invalidation --distribution-id E2WPOPMTKTJRTN --paths '/*'`

#### 4. Push your code on GitHub
To test if everything worked, just push your code on GitHub and go to the distribution URL, example : 
https://d18hwm4oh7cgmg.cloudfront.net
