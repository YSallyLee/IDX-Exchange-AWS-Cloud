# IDX Exchange AWS Cloud Engineering Internship

This repository documents my learning and hands-on labs during the
IDX Exchange AWS Cloud Engineering Internship.

---

## Week 01 — Cloud Fundamentals & Account Setup

### For week 1, we focus on foundational AWS account security and the critical distinction between the root user and IAM users. The root user is the parent account created when the account was created, and has unrestricted power over the entire AWS account, including closing the account, deleting all data, and modifying credit card payment information. Therefore, we need to lock the MFA and put it in a safe, never use it for routine operations. The routine task must be operated by an IAM user, and needs to follow the principle of least privilege: grant only the necessary permissions, and the access key can be rotated or revoked at any time to reduce the disaster in the event of a leak.
### Under the Shared Responsibility Model, Amazon famously states that "AWS secures OF the cloud, you secure IN the cloud." Their responsibility covers physical and underlying defenses, physical data centers, hardware facilities, and virtualization layers. And for the customer side, including  operating system updates, security groups, identity and access control (IAM), and data encryption. This is achieved through the zero-spend budget alert set this week. All of these reinforces the fundamental requirement for cloud architects to strike a balance between system security and cost control.


## Week 02 — IAM & Security Foundations


In the week2 json file, the custom IAM policy conforming to the Least Privilege principle was established and validated. This policy strictly restricts the test user `s3-test-user` to only perform object-level read and upload operations `s3:GetObject and s3:PutObject` under the specified path my-training-bucket-sally/*, while deliberately excluding global storage enumeration permissions `s3:ListAllMyBuckets` and storage-level list permissions `s3:ListBucket`. 

When executing `aws s3 ls --profile s3 test` on the terminal triggers an AccessDenied error, it fully verifies the default denial logic in the AWS policy evaluation mechanism, proving that the user is precisely restricted to the minimum authorized scope and cannot arbitrarily snoop on or laterally access other resources in the account. 

Furthermore, during the troubleshooting process in Debug Lab 2.3, I also gained a deeper understanding of the key differences between object-level ARNs (ending with /*) and storage-level ARNs (without /*). If a storage-level path is assigned to an object operation (such as s3:PutObject), it will be blocked due to resource mismatch. This demonstrates the importance of accurately defining cloud permission boundaries for system security and problem diagnosis.
