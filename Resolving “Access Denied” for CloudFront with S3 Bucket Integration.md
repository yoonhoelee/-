## Introduction

While configuring Amazon CloudFront to serve content from an S3 bucket, I encountered an issue where accessing specific files resulted in an “Access Denied” error. This issue was particularly tricky because the S3 bucket and CloudFront distribution were correctly configured at first glance.

This post documents the problem, analysis, and solution to help others who might face a similar issue.

## Problem

The goal was to serve files stored in an S3 bucket through a CloudFront distribution. The structure of the bucket was as follows:
```
/
├── 2025/
│   └── 01/
│       └── 09/
│           ├── file1.mp4
│           ├── file2.mp4
│           └── ...
```

However, the CloudFront behavior was configured with the path pattern /live-stream/*. The requirement was to make files in the S3 bucket (e.g., 2025/01/09/file.mp4) accessible via CloudFront URLs like:
```
https://<cloudfront-domain>/live-stream/2025/01/09/file.mp4
```
Despite correct IAM and S3 bucket policy configurations, attempting to access files through CloudFront resulted in a 403 Access Denied error.

## Root Cause

The issue was traced to how CloudFront forwards requests to the S3 origin. CloudFront directly appends the path specified in the URL to the S3 origin. For example:
	1.	Request: https://<cloudfront-domain>/live-stream/2025/01/09/file.mp4
	2.	CloudFront forwards: /live-stream/2025/01/09/file.mp4 to S3.

Since the files in the S3 bucket were located under 2025/01/09/ without the live-stream prefix, the forwarded path did not match any files in the bucket, leading to the “Access Denied” error.

## Solution

If modifying the bucket structure is feasible,the solution is to include the live-stream prefix in the S3 folder structure. For example:
```
/
├── live-stream/
│   └── 2025/
│       └── 01/
│           └── 09/
│               ├── file1.mp4
│               ├── file2.mp4
│               └── ...
```
This ensures that CloudFront forwards the path /live-stream/2025/01/09/file.mp4 directly to the S3 bucket.
## Conclusion

Configuring CloudFront to work seamlessly with S3 requires careful attention to how request paths are mapped. In this case, adjusting the Origin Path in the CloudFront behavior was a simple yet effective solution to the problem. Hopefully, this post helps others avoid the same pitfall!
