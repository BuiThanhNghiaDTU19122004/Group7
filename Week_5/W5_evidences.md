# W5 Evidence Pack — AWS Network Fortress

**XBrain AWS Accelerator · Tuần 5 · Evidence Submission**

---

## Cover

| Field | Value |
|-------|-------|
| **Group ID** | GROUP 07 |
| **Team Members** | 1. Bùi Thành Nghĩa<br>2. Hoàng Kim Hùng<br>3. Trần Minh Quang<br>4. Nguyễn Công Thịnh<br>5. Phạm Công Huy<br>6. Lê Thị Thuỳ Trang<br>7. Lê Nguyễn Nhật Thành<br>8. Đỗ Phúc<br>9. Nguyễn Tất Văn |
| **Repository Link** | https://github.com/BuiThanhNghiaDTU19122004/Group7.git |
| **W5 Implementation Notes** | W5 thêm lớp hardening vào ứng dụng W4 — cùng business domain nhưng tăng security, redundancy, và scaling. |

---

---

# MH1 — Multi-VPC Connectivity

## Lựa Chọn & Rationale

**Architecture:** VPC Peering (Path A)

**Rationale:**
- Direct connectivity giữa 2 VPC (10.0.0.0/16 ↔ 10.1.0.0/16)
- Chi phí thấp, setup nhanh (2 route + 1 peering connection)
- Phù hợp với topology 2 VPC: App VPC + Bastion Host VPC
- Không cần Transit Gateway (đơn giản hóa)

---

## Evidence

### Screenshot 1: VPC Peering Status

![MH1_Peering_Status](./screenshots/MH1_peering_status.png)

**Info to capture (1-2 lines):**


---

### Screenshot 2: Route Table — VPC 1

![MH1_RouteTable_VPC1](./screenshots/MH1_route_table_vpc1.png)


---

### Screenshot 3: Route Table — VPC 2

![MH1_RouteTable_VPC2](./screenshots/MH1_route_table_vpc2.png)


---

### Screenshot 4: VPC Flow Logs — ACCEPT Entries

![MH1_FlowLogs_Accept](./screenshots/MH1_flowlogs_accept.png)

**Sample (from CloudWatch):**


**Ping test result:**

---

<!-- ### Checklist MH1

- [ ] Peering Connection status = **Active**
- [ ] Route Table 2 chiều (RT1 + RT2) cấu hình đúng
- [ ] VPC Flow Logs group: `/aws/vpc/w5-flowlogs`
- [ ] Flow Logs có ACCEPT entries cho cross-VPC traffic
- [ ] Ping test: 0% packet loss -->

---

---

# MH2 — Network Firewall / Hardened SG+NACL

## Lựa Chọn & Rationale

**Architecture:** AWS Network Firewall + Stateful Domain-Based Rules

**Rationale:**
- Layer 7 domain filtering (allowlist: .amazonaws.com, github.com, pypi.org)
- Defense-in-depth: Firewall → Private Subnet → Security Group
- Egress traffic control (outbound whitelist)
- Centralized logging (CloudWatch Logs)

---

## Evidence

### Screenshot 1: Firewall Configuration

![MH2_Firewall_Config](./screenshots/MH2_firewall_config.png)

**Info:**

---

### Screenshot 2: Rule Group — Domain Allowlist

![MH2_RuleGroup_Allowlist](./screenshots/MH2_rulegroup_allowlist.png)

**Allowed Domains:**

---

### Screenshot 3: Route Table — Private Subnet

![MH2_Route_Private](./screenshots/MH2_route_private.png)


---

### Screenshot 4: Route Table — Firewall Subnet

![MH2_Route_Firewall](./screenshots/MH2_route_firewall.png)


---

### Screenshot 5: Test — Allowed Request (HTTP 200) ✅

![MH2_Test_Allowed_200](./screenshots/MH2_test_allowed_200.png)


---

### Screenshot 6: Test — Blocked Request (Negative Test) ❌

![MH2_Test_Blocked_Timeout](./screenshots/MH2_test_blocked_timeout.png)


---

### Screenshot 7: CloudWatch Alert Logs

![MH2_CloudWatch_Alerts](./screenshots/MH2_cloudwatch_alerts.png)

**Alert entry (blocked domain):**

**Result:** 

**CloudWatch Evidence:** 

---

<!-- ### Checklist MH2

- [ ] Firewall subnet created (10.0.3.0/28 separate)
- [ ] Rule Group with 5 domains (ALLOWLIST)
- [ ] Private route: 0.0.0.0/0 → Firewall Endpoint
- [ ] Firewall subnet route: 0.0.0.0/0 → NAT Gateway
- [ ] Logging: `/aws/network-firewall/w5-alerts` with ALERT entries
- [ ] curl allowed domains: HTTP 200 ✅
- [ ] curl blocked domain: Connection timeout ✅ -->

---

---

# MH3 — File Storage + Backup Plan

## Lựa Chọn & Rationale

**Architecture:** Amazon EFS + AWS Backup (3 resource types)

**Rationale:**
- Shared NFS storage, mountable from private subnet
- AWS Backup covers EFS + RDS + EBS (redundancy across W2, W3, W5)
- Restore test mandatory (backup without restore = no recovery)
- Encryption by default + NFS over TLS

---

## Evidence

### Screenshot 1: EFS Mount — Private Subnet

![MH3_EFS_Mount_Private](./screenshots/MH3_efs_mount_private.png)

**Info (1 line):**

---

### Screenshot 2: EFS Read/Write Test

![MH3_EFS_Read_Write](./screenshots/MH3_efs_read_write.png)

**app-config.txt:**

**session-store.json:**

---

### Screenshot 3: Backup Plan Configuration

![MH3_BackupPlan_Config](./screenshots/MH3_backupplan_config.png)

**Info (1-2 lines):**

---

### Screenshot 4: Backup Job — COMPLETED

![MH3_BackupJob_Completed](./screenshots/MH3_backupjob_completed.png)

**Info:**

---

### Screenshot 5: Restore Job — COMPLETED

![MH3_RestoreJob_Completed](./screenshots/MH3_restorejob_completed.png)

**Info:**

---

### Screenshot 6: Restored Data — Verified ✅

![MH3_Restored_Data_Verified](./screenshots/MH3_restored_data_verified.png)

**Mounted restored EFS at /mnt/restored-efs:**

---

<!-- ### Checklist MH3

- [ ] EFS created, mount target in private subnet
- [ ] EFS SG: only from app-tier SG (no 0.0.0.0/0)
- [ ] Files written + read successfully
- [ ] Backup Plan covers 3 resources (EFS + RDS + EBS)
- [ ] Schedule: daily at 2AM UTC, retention: 7 days
- [ ] Restore job: COMPLETED
- [ ] Restored data verified — exact match ✅ -->

---

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
