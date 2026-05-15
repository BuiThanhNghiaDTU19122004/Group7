# W5 Evidence Pack — AWS Network Fortress

**XBrain AWS Accelerator · Tuần 5 · Evidence Submission**

---

## Cover

| Field | Value |
|-------|-------|
| **Group ID** | GROUP 07 |
| **Team Members** | 1. Bùi Thành Nghĩa<br>2. Hoàng Kim Hùng<br>3. Trần Minh Quang<br>4. Nguyễn Công Thịnh<br>5. Phạm Công Huy<br>6. Lê Thị Thuỳ Trang<br>7. Lê Nguyễn Nhật Thành<br>8. Đỗ Phúc<br>9. Nguyễn Tất Văn |
| **Repository Link** | Backend: https://github.com/lennhatthanh/Project_G7_BE <br> Frontend: https://github.com/lennhatthanh/Project_G7_FE|
| Prior evidence | W4 evidence pack link: `[paste commit/file link here]` |
| Live frontend | CloudFront frontend URL: `[paste here]` |
| Live backend | ALB or backend CloudFront health URL: `[paste here]` |

---

---

# MH1 — Multi-VPC Connectivity

## Lựa Chọn & Rationale

**Architecture:** VPC Peering (Path A)
SanGo uses two VPCs with non-overlapping CIDR blocks:

| VPC | CIDR | Purpose |
|---|---:|---|
| `sango-vpc-app` | `10.0.0.0/16` | Main 3-tier application stack: ALB, EC2 ASG, EFS, RDS, Redis, Lambda |
| `sango-vpc-mgmt` | `10.1.0.0/16` | Management plane: Bastion host for private access |

VPC Peering is the right choice because the project has exactly two VPCs, needs direct private connectivity from Bastion to the app tier, and does not need transitive routing or hub-and-spoke connectivity. Transit Gateway would add cost and complexity without a current routing need.

---

## Evidence

### Screenshot 1: VPC Peering Status

![MH1_Peering_Status](./img/MH1_peering_status.png)

---

### Screenshot 2: Route Table — AZ A

![MH1_RouteTable_VPC1](./img/MH1_route_table_az_a_vpc1.png)


---

### Screenshot 3: Route Table — AZ B

![MH1_RouteTable_VPC2](./img/MH1_route_table_az_b_vpc1.png)


---

### Screenshot 4: VPC Flow Logs — ACCEPT Entries

![MH1_FlowLogs_Accept](./screenshots/MH1_flowlogs_accept.png)

**Sample (from CloudWatch):**


**Ping test result:**

---

# MH2 — Network Firewall / Hardened SG+NACL

## Lựa Chọn & Rationale

**Architecture:** AWS Network Firewall + Stateful Domain-Based Rules

**Rationale:**
SanGo must use Network Firewall because private app EC2 instances have outbound internet access through NAT Gateway. The W5 guide says SG+NACL-only hardening is not valid when EC2 or Lambda reaches the internet through NAT.
- Defense-in-depth: Firewall → Private Subnet → Security Group
- Centralized logging (CloudWatch Logs)
---

## Evidence

### Screenshot 1: Firewall Configuration

![MH2_Firewall_Config](./img/MH2_firewall_config_overview.png)
![MH2_Firewall_Config](./img/MH2_firewall_config_AttachmentAttachment_Details.png)

---

### Screenshot 2: Rule Group — Domain Allowlist

![MH2_RuleGroup_Allowlist](./img/MH2_rulegroup_allowlist.png)

**Allowed Domains:**

---

### Screenshot 3: Route Table — Private Subnet

![MH2_Route_Private](./img/MH1_route_table_az_a_vpc1.png)
![MH2_Route_Private](./img/MH1_route_table_az_b_vpc1.png)

---

### Screenshot 4: Route Table — Firewall Subnet

![MH2_Route_Firewall](./img/MH2_route_firewall_az_a.png)
![MH2_Route_Firewall](./img/MH2_route_firewall_az_b.png)

---

### Screenshot 5: Test — Allowed Request (HTTP 200) ✅

![MH2_Test_Allowed_200](./img/MH2_test_allowed_200.png)


---

### Screenshot 6: Test — Blocked Request (Negative Test) ❌

![MH2_Test_Blocked_Timeout](./img/MH2_test_blocked_timeout.png)


---

### Screenshot 7: CloudWatch Alert Logs

![MH2_CloudWatch_Alerts](./img/MH2_cloudwatch_alerts.png)

**Alert entry (blocked domain):**

**Result:** 

**CloudWatch Evidence:** 

---

# MH3 — File Storage + Backup Plan

## Lựa Chọn & Rationale

**Architecture:** Amazon EFS + AWS Backup

**Rationale:**
SanGo app tier runs on Amazon Linux EC2 instances in private app subnets, so EFS is appropriate for shared POSIX/NFS storage. It supports multi-AZ mount targets and lets app instances share uploaded venue files or fallback shared content under `/mnt/efs/uploads`.

---

## Evidence

### Screenshot 1: EFS Mount — Private Subnet
![MH3_EFS_Mount_Private](./img/MH3_efs_mount_private.png)

![MH3_EFS_Mount](./img/MH3_efs_mount_console.png)

**EFS File System And Mount Targets**

![MH3_EFS_Mount_Target](./img/MH3_efs_mount_target.png)

---

### Screenshot 2: EFS Read/Write Test

![MH3_EFS_Read_Write](./img/MH3_efs_read_write.png)

---

### Screenshot 3: Backup Plan Configuration

![MH3_BackupPlan_Config](./img/MH3_backupplan_config.png)

**Info (1-2 lines):**

---

### Screenshot 4: Backup Job — COMPLETED

![MH3_BackupJob_Completed](./img/MH3_backupjob_completed.png)

**Info:**

---

### Screenshot 5: Restore Job — COMPLETED

![MH3_RestoreJob_Completed](./img/MH3_restorejob_completed.png)

**Info:**

---

### Screenshot 6:   ✅

![MH3_Restored_Data_Verified](./screenshots/MH3_restored_data_verified.png)

**Mounted restored EFS at /mnt/restored-efs:**

---

# MH4 — API Gateway

## Lựa Chọn & Rationale

**Architecture:** REST API + Lambda Authorizer (or API Key)

**Rationale:**
- Expose backend functions securely via HTTPS
- Authorization enforced (Lambda Authorizer or API Key)
- 200: authorized requests allowed
- 403: unauthorized requests rejected

---

## Evidence

### Screenshot 1: API Resource Tree

![MH4_API_Resources](./screenshots/MH4_api_resources.png)

**Structure:**

---

### Screenshot 2: Authorization Configuration

![MH4_Auth_Config](./screenshots/MH4_auth_config.png)

**Method Details (1 line):**

---

### Screenshot 3: Test 200 — Authorized Request ✅

![MH4_Test_200_Authorized](./screenshots/MH4_test_200_authorized.png)


---

### Screenshot 4: Test 403 — Unauthorized Request ❌

![MH4_Test_403_Unauthorized](./screenshots/MH4_test_403_unauthorized.png)


---

### Checklist MH4

- [ ] API created with resources (/users, /products, etc.)
- [ ] Authorization method configured (Lambda / API Key)
- [ ] curl 200: valid token → success
- [ ] curl 403: invalid token → unauthorized

---

---

# MH5 — Serverless Scaling Pattern

## Lựa Chọn & Rationale

**Pattern:** [Reserved Concurrency / Async + DLQ] ← **chọn 1**

### Option A: Reserved Concurrency

**Rationale:**
- Guarantee max N concurrent executions
- Prevent runaway costs + throttling
- Good for batch processing (S3, SQS triggers)

---

### Option B: Async + DLQ

**Rationale:**
- Handle failures gracefully (retry + DLQ)
- Decoupling invocation from execution
- Good for long-running tasks (Bedrock, DB queries)

---

## Evidence (Choose ONE)

---

### Option A — Reserved Concurrency Evidence

#### Screenshot 1: Lambda Configuration

![MH5_Lambda_Config_Reserved](./screenshots/MH5_lambda_config_reserved.png)

**Info:**

---

#### Screenshot 2: CloudWatch Throttles Metric

![MH5_CloudWatch_Throttles](./screenshots/MH5_cloudwatch_throttles.png)

**During load test (20 concurrent invokes, limit=5):**

---

#### Screenshot 3: Load Test Output

![MH5_LoadTest_Output](./screenshots/MH5_loadtest_output.png)


---

### Option B — Async + DLQ Evidence

#### Screenshot 1: Lambda DLQ Configuration

![MH5_Lambda_Config_DLQ](./screenshots/MH5_lambda_config_dlq.png)

**Info:**

---

#### Screenshot 2: Async Invocation Test

![MH5_Async_Invocation](./screenshots/MH5_async_invocation.png)


---

#### Screenshot 3: DLQ Message

![MH5_DLQ_Message](./screenshots/MH5_dlq_message.png)

**After 2 retries + ~5 min:**

---

<!-- ### Checklist MH5

- [ ] **Option A (Reserved Concurrency):**
  - Reserved concurrency set on production function
  - Load test shows throttling (HTTP 429)
  - CloudWatch Throttles metric > 0 ✅

- [ ] **Option B (Async + DLQ):**
  - DLQ configured on Lambda function
  - Async invocation (type=Event) returns 202
  - Failed message appears in DLQ with error details ✅ -->

---

---

# Application Carry-Forward Verification

---

## Screenshot 1: CI/CD Pipeline Execution

![CarryForward_Pipeline](./screenshots/CarryForward_pipeline.png)

**Info (1-2 lines):**

---

## Screenshot 2: Bedrock Retrieval Test

![CarryForward_Bedrock](./screenshots/CarryForward_bedrock.png)

**Prompt + Response (1-2 lines):**

---

## Screenshot 3: Database Query Execution

![CarryForward_Database](./screenshots/CarryForward_database.png)

**Query + Result (1-2 lines):**

---

<!-- ### Checklist Carry-Forward

- [ ] CI/CD pipeline executed successfully (all stages PASSED)
- [ ] Bedrock query returned response (working)
- [ ] Database query returned results (working) -->

---

---

# Negative Security Tests

---

## Test 1 — SQL Injection Prevention

**Attack Vector:** Input: `{"username":"admin\" OR \"1\"=\"1\""}`

**Mitigation:** 

**Screenshot:**

![SecurityTest_SQLi](./screenshots/SecurityTest_1_sqli.png)

**Result:** 

---

## Test 2 — XSS Prevention

**Attack Vector:** Input: `<script>alert('XSS')</script>`

**Mitigation:** HTML escaping + Content Security Policy headers

**Screenshot:**

![SecurityTest_XSS](./screenshots/SecurityTest_2_xss.png)

**Result:** HTTP 400 - Unsafe characters detected ✅

---

## Test 3 — Firewall Domain Block (MH2)

**Attack Vector:** `curl https://google.com` from private EC2

**Mitigation:** 

**Screenshot:**

![SecurityTest_FirewallBlock](./screenshots/SecurityTest_3_firewall_block.png)

**Result:** 

---

## Test 4 — API Authentication Rejection (MH4)

**Attack Vector:** `curl ... -H "Authorization: Bearer invalid"`

**Mitigation:** 

**Screenshot:**

![SecurityTest_AuthReject](./screenshots/SecurityTest_4_auth_reject.png)

**Result:** 

---

## Test 5 — CORS Policy Blocking

**Attack Vector:** 

**Mitigation:** 

**Screenshot:**

![SecurityTest_CORS](./screenshots/SecurityTest_5_cors.png)

**Result:** 

---

## Test 6 — Rate Limiting / Lambda Throttling (MH5)

**Attack Vector:** 

**Mitigation:** 

**Screenshot:**

![SecurityTest_RateLimit](./screenshots/SecurityTest_6_rate_limit.png)

**Result:** 

---

<!-- ### Checklist Security Tests

- [ ] SQLi test: HTTP 400 (blocked) ✅
- [ ] XSS test: HTTP 400 (blocked) ✅
- [ ] Firewall domain block: timeout (blocked) ✅
- [ ] API auth rejection: HTTP 403 (blocked) ✅
- [ ] CORS blocking: policy rejected ✅
- [ ] Rate limiting: HTTP 429 (throttled) ✅ -->

---

---

# Bonus Section (Optional)

---

## NACL Configuration (if applicable)

**Inbound Rules:**


**Outbound Rules:** 

---

## Security Group Rules (for reference)

**App Tier SG (Inbound):**

**EFS Mount SG (Inbound):**
---

## Additional Hardening Steps

- [ ] Enable VPC Flow Logs
- [ ] Enable CloudTrail logging
- [ ] Set up CloudWatch alarms for failed authentications
- [ ] Enable WAF on API Gateway (optional)
- [ ] Encrypt RDS at rest + in transit

---

---

# Final Summary

| Component | Status | Screenshot |
|-----------|--------|-----------|
| **MH1 — VPC Peering** | ______ | ______ |
| **MH2 — Network Firewall** | ______ | ______ |
| **MH3 — EFS + Backup** | ______ | ______ |
| **MH4 — API Gateway** | ______ | ______ |
| **MH5 — Scaling Pattern** | ______ | ______ |
| **Carry-Forward** | ______ | ______ |
| **Security Tests** | ______ | ______ |

---

**Completion Status:** ✅ **Ready for Presentation**

**Group:** [GROUP 07]  
**Date Completed:** [2026-05-14]  
<!-- **Trainer Sign-off:** [Trainer name]  
**Notes:** W5 thêm hardening layers, đủ 3 business domains (connectivity, storage, scaling) -->
