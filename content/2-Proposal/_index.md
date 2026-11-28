---
title: "Proposal"
date: 2026-11-28
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AI Exam Maker
## Cost-Optimized AWS Architecture for AI-Assisted Question Generation and Exam Approval

### 1. Executive Summary

AI Exam Maker is a web application that supports Vietnamese high-school teachers during the exam preparation process. A teacher selects a subject, grade, chapter, question type, difficulty, and an optional note. The application sends the request to an asynchronous AI pipeline, validates the generated result, and saves the questions as drafts. The teacher must review the content before it can enter the department approval workflow. Approved questions can then be reused to assemble versioned exams, publish an official version, and export a student paper and a separate answer document.

The project is designed as a focused AWS demonstration environment. It prioritizes a complete and accountable workflow over production-scale availability. The target is to keep the development and demonstration environment within a USD 200 credit while still demonstrating authentication, authorization, asynchronous processing, deployment automation, monitoring, rollback, and cost control.

My work covers the OPS-1 infrastructure scope. It includes the AWS decisions, the infrastructure runbook and IaC boundary, VPC and security groups, EC2 and EBS, Elastic IP, Session Manager, domain and HTTPS, the CloudFront design and its S3–SSM–EC2 fallback, GitHub OIDC, frontend CI/CD, and the end-to-end sign-in integration with the frontend team.

### 2. Project Goals

#### 2.1. General Objective

Build a working exam platform on AWS that allows a teacher to sign in, generate question drafts with AI, review and submit them, receive department-level approval, assemble an exam version, publish the approved version, and export the final documents.

#### 2.2. Functional Scope

- Sign in through Amazon Cognito using Authorization Code with PKCE.
- Support three roles: teacher, department head, and administrator.
- Generate question drafts by subject, grade, chapter, question type, and cognitive level.
- Process AI requests through Amazon SQS and a dedicated worker instead of blocking the browser.
- Validate the AI response, track job status, and require teacher review before approval.
- Maintain a reusable question bank with states such as Draft, Submitted, Approved, Rejected, and Archived.
- Assemble exams as immutable versions with snapshots and a content hash when submitted.
- Allow a department head to review content created by another member of the department.
- Publish at most one official version of an exam at a time.
- Export a student paper without answers and a separately protected answer document.
- Record state changes and provide operational logs and cost information.

#### 2.3. Infrastructure Objectives

- Keep the public entry point on HTTPS and avoid inbound SSH.
- Place EC2 in a public subnet and RDS in private subnets, without a NAT Gateway.
- Restrict database access to the EC2 security group and keep the security group rules reviewable.
- Use an EC2 instance profile and Systems Manager Session Manager for administration.
- Use GitHub OIDC and short-lived role credentials instead of long-lived AWS access keys.
- Build a repeatable frontend release path with checksums, health checks, and rollback.
- Test the Cognito callback, logout, protected API, and frontend behavior on the configured domain.

#### 2.4. Completion Criteria

| Area | Completion target |
|---|---|
| Network | Public EC2 subnet, private RDS subnet, controlled routes, and no unnecessary NAT cost. |
| Security | Port 22 remains closed; RDS accepts traffic only from EC2; IAM permissions are least privilege. |
| Compute | EC2, EBS, Elastic IP, Nginx, API, and worker are available and manageable through SSM. |
| Delivery | GitHub Actions builds the frontend, uploads a release, invokes SSM, checks health, and preserves a rollback target. |
| Identity | Cognito sign-in returns to the correct callback URL and the API accepts only valid access tokens. |
| Operations | Logs, health checks, alarms, budget notifications, and cleanup instructions are documented. |

### 3. Problem Statement

#### 3.1. Manual exam preparation is slow and difficult to reuse

Teachers often write questions repeatedly for different classes and exams. Separate documents also make it difficult to know which questions were reviewed, approved, or already used. AI Exam Maker provides structured generation and a searchable question bank, but the generated output remains a draft until a human reviews it.

#### 3.2. AI output cannot be trusted without controls

A language model may produce an invalid structure, a duplicate question, or an answer that does not match the explanation. The application therefore treats AI output as untrusted data. It validates the response, records the job result, applies duplicate checks, and uses a teacher and department-head review process before publication.

#### 3.3. Long-running generation should not block the user interface

AI calls can take several seconds and can fail temporarily. The API creates a job and places it on SQS. A worker claims the job, calls Bedrock, persists the result, and reports a terminal status. Retries, a dead-letter queue, leases, and recovery logic reduce the chance of lost or duplicated work.

#### 3.4. CloudFront was not available for the working deployment path

The original frontend design used S3 with CloudFront behaviors and a CloudFront Function. Because CloudFront could not be used in the current environment, the project needs a fallback that does not require a CDN to serve the static export. The fallback stores versioned artifacts in S3 and uses SSM to let EC2 download and activate them.

#### 3.5. Infrastructure access must be secure and repeatable

Opening SSH or putting permanent credentials in GitHub Actions would make the system harder to secure and review. The deployment design uses an EC2 instance profile, Session Manager, IAM least privilege, and GitHub OIDC. Environment-specific values remain outside the repository.

### 4. Solution Architecture

AI Exam Maker runs in the `ap-southeast-1` Singapore Region. The architecture intentionally uses one EC2 instance and a Single-AZ RDS database for the demonstration to reduce cost and operational complexity. This is an accepted trade-off and not a claim of production-level high availability.

![AI Exam Maker AWS architecture](/fcj-workshop-template/images/2-Proposal/project-architecture.jpg)

#### 4.1. Component Responsibilities

| Component | Responsibility |
|---|---|
| VPC, subnets, and Internet Gateway | Separate public compute from private database resources and provide the required network path. |
| Security groups | Allow HTTPS and certificate-validation traffic to EC2, and allow PostgreSQL only from EC2 to RDS. |
| EC2, EBS, and Elastic IP | Run Nginx, the static frontend, the .NET API, and the AI worker under systemd. |
| Nginx and domain | Terminate HTTPS, serve the active frontend release, and forward `/api/*` to the API. |
| Amazon RDS for PostgreSQL | Store users, departments, questions, jobs, exams, versions, and audit records in private subnets. |
| Amazon Cognito | Authenticate users and provide the teacher, head, and admin groups. |
| Amazon SQS and DLQ | Decouple generation requests from the worker and retain messages that repeatedly fail. |
| Amazon Bedrock | Generate draft questions and optionally verify answers according to the application workflow. |
| SSM Parameter Store and KMS | Provide configuration and encrypted values without committing them to Git. |
| Amazon S3 | Store frontend release packages and checksums for deployment and rollback. |
| GitHub Actions and OIDC | Build, test, package, and deploy without long-lived AWS credentials. |
| CloudWatch, SNS, and Budgets | Collect logs, notify on important alarms, and monitor spending. |

#### 4.2. Network and Security Design

The VPC has a public subnet for EC2 and private subnets for RDS. The design does not use a NAT Gateway because the demonstration does not need the additional fixed cost. EC2 receives only the traffic required for HTTPS and certificate validation. Port 22 is not opened. RDS accepts PostgreSQL connections only from the EC2 security group, rather than from a public CIDR range.

The EC2 instance receives permissions through an instance profile. Session Manager is used for administrative access and command execution. GitHub Actions assumes a deployment role through the GitHub OIDC provider. The role is restricted by repository, branch, and workflow conditions supported by IAM, and it receives only the S3 and SSM permissions required by the deployment.

#### 4.3. Frontend Delivery and CloudFront Fallback

The intended CDN path uses S3 as the frontend origin and CloudFront behaviors for static assets and API requests. If the CDN cannot be used, the working path is:

1. GitHub Actions installs dependencies, builds the static Next.js export, and runs CI checks.
2. The workflow packages the output as a versioned release and creates a SHA-256 checksum.
3. The artifact and checksum are uploaded under an S3 release prefix.
4. The workflow calls SSM Run Command on the target EC2 instance.
5. EC2 downloads the artifact, verifies the checksum, and extracts it into a new release directory.
6. The script updates the `current` symlink only after the new release is ready.
7. Nginx is reloaded and the public health or asset check is performed.
8. The previous release remains available so a failed activation can be rolled back.

This approach keeps the deployment auditable: the GitHub run, S3 artifact, SSM invocation, EC2 release directory, and active symlink can be inspected separately.

#### 4.4. Authentication Flow

The user opens the application over HTTPS and selects sign-in. Cognito authenticates the user and returns an authorization code to the configured callback URL. The frontend exchanges the code using PKCE, stores the session according to the application design, and calls the API with an access token. The backend validates issuer, audience, expiry, and the internal user record before returning protected data. Logout clears the local session and returns the user to the configured sign-out path.

#### 4.5. AI Job Flow

The API validates the request and checks the user’s role, department, quota, and selected catalog values. It creates a job and sends a message to SQS. The worker claims the job, checks the token budget before each model call, invokes Bedrock, validates and sanitizes the response, removes duplicates when required, and saves the draft question result. The frontend polls the job status and displays a recoverable error if the worker has not completed. Repeated messages must not create duplicate questions.

### 5. Technical Implementation Plan

#### Phase 0 — AWS decisions and infrastructure boundary

- Confirm the region, domain approach, EC2 family, model and quota assumptions, IaC/runbook boundary, and fallback strategy.
- Place infrastructure examples and scripts under `/infra` without real IDs, ARNs, secrets, or account-specific values.
- Define the review points for VPC rules, IAM permissions, deployment artifacts, and cleanup.

#### Phase 1 — Network, compute, and HTTPS

- Create the VPC, public and private subnets, route tables, Internet Gateway, and security groups.
- Prepare EC2, EBS, Elastic IP, instance profile, SSM Agent access, and systemd service boundaries.
- Configure the application domain, Nginx reverse proxy, HTTPS certificate, renewal timer, and origin-header checks where applicable.
- Confirm that the backend can reach RDS privately and that unwanted inbound paths remain closed.

#### Phase 2 — Identity and deployment automation

- Configure the GitHub OIDC provider and deployment IAM role with the smallest practical S3 and SSM permissions.
- Build the frontend workflow and package each release with a checksum.
- Upload artifacts to S3 and trigger the EC2 deployment through SSM instead of SSH.
- Extract to a new release, update `current`, reload Nginx, check the result, and retain a previous release for rollback.

#### Phase 3 — Integration and evidence

- Test Cognito sign-in, callback, logout, protected API access, and the frontend behavior with the real domain.
- Test successful and unsuccessful deployment paths, including checksum failure and rollback.
- Capture non-sensitive evidence from VPC, EC2, SSM, IAM, S3, GitHub Actions, and the application health check.
- Review cost controls, logs, alarms, and cleanup steps before the team demo.

### 6. Project Timeline and Milestones

The team roadmap runs from 17 November to 1 December 2026. Work is parallelized so the application team can develop while infrastructure is prepared.

| Period | Focus | Infrastructure deliverables |
|---|---|---|
| 17–19 Nov | Foundation | AWS decisions, infrastructure runbook, repository rules, and initial CI boundary. |
| 20–24 Nov | Platform | VPC, security groups, EC2, EBS, Elastic IP, Session Manager, domain, Nginx, HTTPS. |
| 25–29 Nov | Features and delivery | OIDC role, S3 release path, SSM deployment, frontend rollback, and sign-in integration. |
| 30 Nov–1 Dec | Finalization | End-to-end checks, recovery evidence, budget review, cleanup, and demo preparation. |

The main demo flow is protected from optional scope. If time is short, additional availability improvements and nonessential automation are deferred rather than weakening the network, authentication, or rollback controls.

### 7. Budget Estimation and Cost Control

The estimate covers the development and demonstration period in `ap-southeast-1`. Actual charges depend on runtime hours, storage, traffic, AI tokens, and the current AWS price list, so the numbers are planning values rather than a bill guarantee.

| Cost area | Planning assumption | Control measure |
|---|---|---|
| EC2 and public IPv4 | Small ARM instance and Elastic IP used only during working hours. | Stop the instance outside the schedule and release unused IPs after the demo. |
| EBS | One small general-purpose volume for application releases and logs. | Remove unattached volumes and keep only required release history. |
| RDS | Small Single-AZ PostgreSQL instance with encrypted storage and short backups. | Stop outside working hours, monitor storage, and remove temporary snapshots after review. |
| S3 | A small number of frontend release packages and checksums. | Use a release prefix and lifecycle cleanup for old artifacts. |
| AI services | Development uses a lower-cost model and the demo uses limited requests. | Enforce per-user/system quotas and token ceilings before model calls. |
| Logs and alarms | Seven-day log retention and a small set of high-value alarms. | Avoid verbose payload logging and keep the alarm count controlled. |

AWS Budgets alerts are configured at 50%, 80%, and 100% of the USD 200 credit. The application also limits questions per request and daily generation requests. Together, infrastructure schedules and application-level quotas reduce the risk that a test or repeated job consumes the entire budget.

### 8. Risk Assessment and Mitigation

| Risk | Impact | Probability | Mitigation and contingency |
|---|---|---|---|
| AI generates an incorrect answer | High | Medium | Keep output as draft, validate structure, allow independent verification, and require human approval. |
| Duplicate SQS delivery or worker crash | Medium | Medium | Use idempotent job claims, unique constraints, leases, heartbeat, retry, and a DLQ. |
| Unauthorized access to drafts or answers | Critical | Low | Enforce backend authorization using role, department, ownership, and content state; test the permission matrix. |
| CloudFront cannot be used | Medium | High | Use the S3–SSM–EC2 fallback and serve the static frontend from Nginx over HTTPS. |
| Release is corrupted or deployment fails | High | Medium | Verify SHA-256, run health checks, activate through `current`, and keep the previous release for rollback. |
| EC2 or Single-AZ RDS fails | High | Low | Accept the demo trade-off, use systemd restart and RDS backups, and test a restore procedure. |
| EC2 is exposed to scanning | Medium | Medium | Allow only 443 and necessary certificate-validation traffic; keep SSH closed and use Session Manager. |
| GitHub credential is leaked | Critical | Low | Use OIDC short-lived credentials, repository/branch restrictions, least privilege, and secret scanning. |
| Cognito callback is incorrect | Medium | Medium | Test login, callback, logout, and protected API access from the deployed frontend. |
| Two-week schedule slips | High | Medium | Work in parallel, prioritize the main demo, and defer optional production-scale features. |

### 9. Expected Deliverables

- A documented AWS architecture and infrastructure decision record.
- VPC, subnet, route, security-group, EC2, EBS, Elastic IP, Nginx, HTTPS, and Session Manager configuration.
- GitHub OIDC provider and least-privilege deployment role.
- Frontend CI/CD workflow, S3 release artifacts, SSM deployment script, health check, and rollback path.
- End-to-end Cognito sign-in integration with the frontend team.
- Non-sensitive screenshots and runbook evidence for the worklog and final demonstration.
- Cleanup instructions covering snapshots, EBS volumes, Elastic IP, S3 artifacts, and stopped resources.

### 10. Conclusion and Future Development

AI Exam Maker combines AI-assisted question drafting with a reviewable and auditable exam workflow. The infrastructure design balances security, cost, and delivery speed: the database is private, administration uses Session Manager, GitHub uses OIDC, and frontend releases can be verified and rolled back. The CloudFront fallback keeps the project demonstrable even when the original CDN path is unavailable.

The next evolution could add a load balancer, Multi-AZ database, CDN and WAF, stronger infrastructure-as-code coverage, document-based question generation, exam blueprints, multiple exam variants, and DOCX export. Those improvements can be introduced after the core demonstration is stable and the budget assumptions are rechecked.
