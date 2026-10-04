# Amazon S3-Based Organization Document Storage System

## Project Overview

This project demonstrates how to build a secure, versioned document
storage system for an organization using **Amazon S3**.

The storage system organizes documents by department, keeps the bucket
private, enables S3 Versioning, and demonstrates how an earlier version
of a document can be recovered.

## Architecture

``` text
Amazon S3 Bucket
│
├── HR/
│   └── employee-policy.txt
│
├── Finance/
│   └── budget.txt
│
└── IT/
    └── server-documentation.txt
```

## AWS Services Used

-   Amazon S3
-   AWS IAM / S3 bucket permissions
-   S3 Bucket Versioning

## Project Requirements

-   Create an S3 bucket for organizational documents
-   Organize files into department folders
-   Keep public access blocked
-   Configure appropriate access permissions
-   Enable S3 Versioning
-   Upload an initial document
-   Modify and upload the same document again
-   Display multiple object versions
-   Recover an earlier version

## Implementation Steps

### 1. Create the S3 Bucket

Created the bucket:

`pranav-org-doc-storage-2026`

Region used:

`US East (N. Virginia) - us-east-1`

Bucket settings:

-   Object Ownership: ACLs disabled
-   Block Public Access: Enabled
-   Bucket Versioning: Enabled

![S3 bucket created](screenshots/step-1.png)

### 2. Enable S3 Versioning

Open:

**S3 → Bucket → Properties → Bucket Versioning → Edit**

Select:

**Enable**

Versioning allows S3 to retain previous versions of an object instead of
permanently replacing the old data.

![Versioning enabled](screenshots/step-2.png)

### 3. Create Department Folders

The bucket was organized into:

``` text
HR/
Finance/
IT/
```

These prefixes make the storage easier to manage by department.

### 4. Upload HR Document

The file `employee-policy.txt` was uploaded to:

``` text
s3://pranav-org-doc-storage-2026/HR/
```

![HR document upload](screenshots/step-3.png)

### 5. Upload IT Document

The file `server-documentation.txt` was uploaded to:

``` text
s3://pranav-org-doc-storage-2026/IT/
```

![IT document upload](screenshots/step-4.png)

### 6. Create a New Version

The original `employee-policy.txt` contained:

``` text
Employee Leave Policy

Employees are allowed 20 paid leaves per year.

Document Version: 1
```

The file was then changed to:

``` text
Employee Leave Policy

Employees are allowed 25 paid leaves per year.

Document Version: 2
```

The same filename was uploaded again.

Because Versioning was enabled, S3 retained both versions.

![Multiple object versions](screenshots/step-5.png)

## Version Recovery

When **Show versions** is enabled in the S3 console, the object displays
multiple Version IDs.

Example:

``` text
employee-policy.txt
├── Version 2 - Current
└── Version 1 - Previous
```

To recover an earlier version:

1.  Open the S3 bucket.
2.  Open the `HR/` folder.
3.  Turn on **Show versions**.
4.  Locate `employee-policy.txt`.
5.  Identify the older Version ID.
6.  Select the older version.
7.  Use **Download** to retrieve the earlier file.
8.  If the old content must become the current version, upload the
    recovered file again using the same object key.

This demonstrates recovery from an accidental modification.

## Security Configuration

The bucket was kept private by leaving:

**Block all public access = ON**

This prevents anonymous public access to organizational documents.

For a production environment, access should be granted through IAM
users, groups, or roles using least-privilege policies.
Department-specific permissions can also be implemented using S3 object
prefixes such as:

``` text
HR/*
Finance/*
IT/*
```

## Final Project Structure

``` text
aws-s3-document-storage-project/
│
├── README.md
├── AWS_S3_Document_Storage_Project.pdf
└── screenshots/
    ├── step-1.png
    ├── step-2.png
    ├── step-3.png
    ├── step-4.png
    └── step-5.png
```

## Result

The project successfully demonstrates:

-   S3 bucket creation
-   Department-wise document organization
-   Secure private bucket configuration
-   Document upload
-   S3 Versioning
-   Multiple versions of the same object
-   Recovery of an earlier document version

## Interview Explanation

> I created an Amazon S3-based document storage system for an
> organization. I organized documents using department-wise prefixes
> such as HR, Finance, and IT. I kept Block Public Access enabled to
> prevent public access. I enabled S3 Versioning so that previous
> versions of documents were retained. I then modified and re-uploaded
> an employee policy document and demonstrated how the previous version
> could be identified and recovered.

## Conclusion

Amazon S3 provides durable object storage with access control and
version management. S3 Versioning is especially useful for protecting
important organizational documents from accidental overwrites or
deletions.
