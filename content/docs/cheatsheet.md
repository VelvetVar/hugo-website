---
title: "Commands: Cheatsheet"
description: "Technical documentation of the AWS, Cloudflare, and Hugo (Book) architecture."
weight: 90
draft: false
menu:
  docs:
    parent: "aws-folder"
---

## Hugo & AWS Cloud Portfolio Cheatsheet

This reference guide contains the essential commands for managing some of the projects.

Note: I used placeholders to remove sensitive information from commands or file names.

# 1. Setup & Installation

Install Book Theme (Git Submodule)
Use this to install the theme without adding node_modules dependencies.
```powershell
git submodule add [https://github.com/alex-shpak/hugo-book](https://github.com/alex-shpak/hugo-book) themes/book
```

# 2. Content Management

Create New Documentation Page
Generates a new markdown file with the default front matter in the specified directory.
```powershell
hugo new content/docs/projects/[filename].md
```

# 3. Local Development

```powershell
Start Local Server
Builds the site in memory and hosts it locally for preview.

hugo server


Access: http://localhost:1313
```

Stop: Press Ctrl + C



# 4. Deployment Workflow

Step 1: Clean Previous Build (PowerShell)
Removes the public folder to ensure no stale files remain.
```powershell
Remove-Item -Path "public" -Recurse -Force -ErrorAction SilentlyContinue
```

Step 2: Build Production Site
Generates the static HTML files into the public directory, minifying assets for performance.
```powershell
hugo --minify
```

Step 3: Sync to AWS S3
Uploads the new build to your S3 bucket. The --delete flag ensures files removed locally are also removed from S3.
```powershell
aws s3 sync ./public s3://[YOUR_DOMAIN] --delete
```

Step 4: Invalidate CloudFront Cache
Forces CloudFront to fetch the latest version of the site from S3 immediately.
```powershell
aws cloudfront create-invalidation --distribution-id [DISTRIBUTION_ID] --paths "/*"
```