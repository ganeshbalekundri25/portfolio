# Portfolio CI/CD: GitHub Actions to AWS S3

A personal portfolio website that deploys itself. Every push to the `main` branch triggers a GitHub Actions pipeline that syncs the site files to an Amazon S3 bucket configured for static website hosting, so changes go live within seconds with no manual upload.

**Live site:** http://ganesh-portfolio-bckt.s3-website-ap-southeast-2.amazonaws.com

## How it works

```
 Developer                GitHub                 GitHub Actions              AWS
 ---------   git push   ----------   triggers   ----------------   sync    ----------------
 edit code  ---------->  main branch ---------->  Portfolio         -----> S3 bucket
                                                  Deployment               (static website)
                                                  (ubuntu runner)               |
                                                                                v
                                                                         Visitors' browsers
```

1. Code is pushed to the `main` branch.
2. The `Portfolio Deployment` workflow starts on a GitHub-hosted Ubuntu runner.
3. The runner checks out the repo and authenticates to AWS using GitHub Secrets.
4. `aws s3 sync` uploads changed files to the bucket and removes deleted ones.
5. The updated site is served from the S3 website endpoint.

## Tech stack

- **Frontend:** HTML, CSS, JavaScript
- **Hosting:** Amazon S3 (static website hosting)
- **CI/CD:** GitHub Actions
- **Auth:** IAM user with access keys stored in GitHub Secrets
- **Region:** `ap-southeast-2` (Sydney)

## Pipeline

Workflow file: [`.github/workflows/main.yaml`](.github/workflows/main.yaml)

| Step | What it does |
|------|--------------|
| Checkout | Pulls the repository code onto the runner |
| Configure AWS Credentials | Authenticates to AWS using repository secrets |
| Deploy static site to S3 bucket | Syncs site files to the bucket, skipping repo metadata |

## Setup guide

### 1. Create the S3 bucket
- Create a bucket (for example `ganesh-portfolio-bckt`) in `ap-southeast-2`.
- Under **Properties**, enable **Static website hosting** and set `index.html` as the index document.
- Under **Permissions**, turn off "Block all public access" for this bucket.

### 2. Add a bucket policy for public read
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::ganesh-portfolio-bckt/*"
    }
  ]
}
```

### 3. Create an IAM user for the pipeline
Create a dedicated IAM user with programmatic access and a policy limited to this bucket:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::ganesh-portfolio-bckt"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:DeleteObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::ganesh-portfolio-bckt/*"
    }
  ]
}
```

### 4. Add GitHub Secrets
In the repo go to **Settings, Secrets and variables, Actions** and add:

| Secret | Value |
|--------|-------|
| `AWS_ACCESS_KEY_ID` | Access key ID of the IAM user |
| `AWS_SECRET_ACCESS_KEY` | Secret access key of the IAM user |

Never commit these values to the repository.

### 5. Push the workflow
Place the workflow at `.github/workflows/main.yaml`, commit, and push to `main`. The first run deploys the site.

## Screenshots

### Live portfolio
![Live portfolio](images/Screenshot%202026-11-09%20143627.png)

### Successful GitHub Actions run
![GitHub Actions run](screenshots/github-actions-run.png)

### S3 bucket contents
![S3 bucket objects](screenshots/s3-bucket-objects.png)

### S3 bucket policy
![S3 bucket policy](screenshots/s3-bucket-policy.png)


## Author

**Ganesh Balekundri** - (AWS, DevOps)
GitHub: [@ganeshbalekundri25](https://github.com/ganeshbalekundri25)
