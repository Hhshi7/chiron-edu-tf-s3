# Static Website on AWS — Terraform Learning Project

Personal learning project to understand Terraform basics by deploying a static website to AWS.

> [!NOTE]
> This is a learning/portfolio project, not a production setup. The focus is on understanding Terraform concepts and AWS services.

---

## Goals

- Learn Terraform syntax and workflow (`init`, `plan`, `apply`)
- Understand how Terraform manages infrastructure state
- Practice Infrastructure as Code on AWS
- Deploy something real and functional

---

## What it does

Deploys a static website using:
- **AWS S3** for file storage
- **Cloudfront** fpr HTTPS and CDN

>[!NOTE] All infrastructure is defined in Terraform files. Running `terraform apply` creates everything from scratch.

---
## Prerequisites

If you want to use this setup you need:

1. **AWS Account** 
2. **AWS CLI** configured with credentials (`aws configure`)
3. **Terraform** installed

---

## How to Use

### Initial Setup

```bash
# Clone the repo
git clone https://github.com/Hhshi7/chiron-edu-tf-s3.git
cd chiron-edu-tf-s3/

# Copy example config
cp terraform.tfvars.example terraform.tfvars

# Default region in variables.tf was set to "eu-central-1"
# Choose suitable region by changing terraform.tfvars

# Edit terraform.tfvars and set a unique bucket name
# bucket_name must be globally unique across all AWS accounts

```
### Deploy Infrastructure

```bash
# Download AWS provider
terraform init

# See what will be created
terraform plan

# Create the infrastructure
terraform apply
```

Terraform will create:
- S3 bucket with website hosting enabled
- Bucket policy for public access
- CloudFront distribution

Takes about 5-10 minutes (CloudFront is slow to deploy ~ 4 minutes in my case).

### Upload Website Files

```bash
# S3 bucket was created but it is empty
# We are uploading the html files by syncing the ./site/ directory with s3 bucket 
aws s3 sync ./site/ s3://bucket-name/
```

[!NOTE] Replace `bucket-name` with whatever you set in `terraform.tfvars`.

### Get Website URL

After deployment terraform will provide all the set outputs. If console was cleared just use the following command to get cloudfront URL:

```bash
terraform output cloudfront_url
```

If everything went smoothly visitng the URL should display our deployed index.html. (Keep in mind that CloudFront may take up to 20 minutes to fully deploy)

### Clean Up

>[!NOTE] Terraform by default needs to empty the S3 bucket before deleting it. In this setup force_destroy was set to true, so terraform can destroy the bucket even though it contains our .html files.

```bash
# Delete everything
terraform destroy
```

>[!NOTE] Be carefull when using force_destroy. If you want to empty the bucket by hand use CLI:

```bash
aws s3 rm s3://bucket-name/ --recursive
```

In case of this project we sync the .html files from our local repository so there is no need to worry about force destroy as we still have original files. Just keep that in mind.

## Project Structure

```
.
├── main.tf                  # Main infrastructure config
├── variables.tf             # Input variables
├── outputs.tf               # Values to display after deployment
├── terraform.tfvars.example # Template for your config
├── .gitignore              # Don't commit state files!!
└── site/                   # Website files to upload
    └── index.html
```

## What I Learned
### Terraform Basics
- **Resources** - Things Terraform creates (S3 bucket, CloudFront distribution)
- **Variables** - Input values I can change without editing the code
- **Outputs** - Display important info after deployment (like the website URL)
- **State** - Terraform tracks what it created in `.tfstate` file

### S3 for Static Sites
- S3 can host websites directly
- Need to enable "static website hosting" on the bucket
- Must make bucket publicly readable (bucket policy)
- Blocking public access needs to be disabled first (this caught me initially)

### CloudFront
- CloudFront = AWS's CDN service
- Provides HTTPS for free (using CloudFront's certificate)
- Takes forever to deploy (10-20 minutes)
- Using S3 website endpoint as origin (not the bucket directly) lets S3 handle index.html routing

---

## How It Works

### Architecture

[WIP]

### Terraform Workflow

```
terraform init   → Downloads AWS provider plugin
terraform plan   → Shows what will be created/changed
terraform apply  → Actually creates the infrastructure
terraform output → Shows the website URL
terraform destroy → Deletes everything
```


## Design Decisions

| Decision | Reasoning |
|----------|-----------|
| **CloudFront instead of just S3** | S3 website URLs are HTTP only, CloudFront gives HTTPS for free |
| **Public bucket instead of OAI** | Simpler for learning, OAI can be added later |
| **Local state instead of remote** | Just me working on this, remote state (S3 backend) is for teams |
| **No custom domain** | Costs $12/year for domain, CloudFront URL is fine for learning |
| **Separate website files** | Terraform manages infrastructure, not content |


---
## Known Limitations

This is a learning project, so I kept it simple:
- No custom domain (using CloudFront URL)
- No CI/CD pipeline (manual upload via AWS CLI)
- Local state only (would use S3 backend for team projects)
- No monitoring/alerting
- No multi-environment setup (dev/prod)

These would be good additions for a real production setup or future projects.


## Technologies

- **Terraform** - Infrastructure as Code
- **AWS S3** - Object storage / static hosting
- **AWS CloudFront** - CDN for global distribution and HTTPS
- **AWS CLI** - Uploading files to S3



## Resources

- [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- HashiCorp Learn tutorials

---

## Next Steps

Things I might add:
- [ ] GitHub Actions workflow to upload files automatically
- [ ] S3 backend for Terraform state

For now, this achieves the goal: learn Terraform basics and deploy real infrastructure to AWS.
