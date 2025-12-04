---
title: "AWS Static Website"
description: "Technical documentation of the AWS, Cloudflare, and Hugo (Book) architecture."
weight: 10
draft: false
---

# Cloud Infrastructure

**Project Status:** Live Production
**Stack:** AWS (S3, CloudFront), Cloudflare, Hugo (Book Theme)
**Architecture:** Serverless Static Website

## 1. Architecture Overview

This project utilizes a **Serverless Static Website** architecture designed for high availability, security, and zero-maintenance scaling. The cost is negligible (Free Tier eligible) while offering enterprise-grade performance.



* **DNS:** **Cloudflare** (Manages DNS records and SSL Strict Mode).
* **CDN:** **AWS CloudFront** (Global caching, SSL termination, and Edge Logic).
* **Storage:** **AWS S3** (Private bucket, accessible *only* via CloudFront Origin Access Control).
* **Generator:** **Hugo + Book Theme** (Lightweight, no NPM dependencies).

---

## 2. Local Development Environment

To manage this site locally, the following software stack is required.

| Software | Version | Purpose |
| :--- | :--- | :--- |
| **Hugo** | Standard or Extended | Generates the static HTML site. |
| **AWS CLI** | v2.x | Synchronizes local build files to the S3 bucket. |
| **Git** | Latest | Version control and Theme Submodule management. |

### Critical Environment Variables (Windows)
The Windows `PATH` environment variable was updated to include:
* `C:\Hugo\bin\` (Containing `hugo.exe`)
* `C:\Program Files\Amazon\AWSCLIV2\`

*(Note: Node.js and NPM are no longer required).*

---

## 3. AWS Infrastructure Setup

### A. S3 Bucket (Storage)
1.  Created a bucket named `[DOMAIN]`.
2.  **Region:** `us-east-1` (N. Virginia).
3.  **Public Access:** Blocked *all* public access (Private).
4.  **Encryption:** Enabled (SSE-S3).

### B. Certificate Manager (SSL)
1.  Requested a public certificate in **us-east-1** for `[DOMAIN]` and `*.[DOMAIN]`.
2.  Validated ownership via DNS records in Cloudflare.

### C. CloudFront (CDN)
1.  **Origin:** Pointed to S3 Bucket.
2.  **Origin Access:** set to **Origin Access Control (OAC)** to secure the bucket.
3.  **Viewer Protocol:** Redirect HTTP to HTTPS.
4.  **Alternate Domain Name:** `[DOMAIN]`.
5.  **Custom SSL:** Attached the ACM Certificate created above.
6.  **Default Root Object:** Set to `index.html`.

### D. Edge Logic (The Sub-Directory Fix)
To prevent S3 "Access Denied" XML errors when visiting sub-pages (e.g., `/docs/`), a CloudFront Function was implemented on the **Viewer Request** behavior.

**Function Name:** `append-index-html`
**Code:**
```javascript
function handler(event) {
    var request = event.request;
    var uri = request.uri;

    // If the URL ends with a slash '/', add 'index.html' to the end
    if (uri.endsWith('/')) {
        request.uri += 'index.html';
    }
    // If the URL doesn't have a file extension, add '/index.html'
    else if (!uri.includes('.')) {
        request.uri += '/index.html';
    }

    return request;
}
```
---
