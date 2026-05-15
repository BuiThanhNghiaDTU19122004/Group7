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

## Architecture Diagram

![00_architecture_w5_final](./img/w5_architecture_overview.png)
![00_architecture_w5_final](./img/w5_architecture_peering_connection.png)
---

# MH1 — Multi-VPC Connectivity

## Chosen Path And Rationale

**Architecture:** VPC Peering (Path A)
SanGo uses two VPCs with non-overlapping CIDR blocks:

| VPC | CIDR | Purpose |
|---|---:|---|
| `hung-service-app-vpc` | `10.0.0.0/16` | Main 3-tier application stack: ALB, EC2 ASG, EFS, RDS, Redis, Lambda |
| `hung-service-mgmt-vpc` | `10.1.0.0/16` | Management plane: Bastion host for private access |

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

![MH1_FlowLogs_Accept](./img/MH1_flowlogs_accept.png)


**Ping test result:**
![MH1_Ping_Result](./img/MH1_ping_result.png)

---

# MH2 — Network Firewall / Hardened SG+NACL

## Chosen Path & Alert entry (blocked domain):

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

![MH2_CloudWatch_Alerts](./img/MH2_cloudwatch_alerts1.png)
![MH2_CloudWatch_Alerts](./img/MH2_cloudwatch_alerts2.png)

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

---

### Screenshot 4: Backup Job — COMPLETED

![MH3_BackupJob_Completed](./img/MH3_backupjob_completed.png)


---

### Screenshot 5: Restore Job — COMPLETED

![MH3_RestoreJob_Completed](./img/MH3_restorejob_completed.png)


---

# MH4 — API Gateway

## Lựa Chọn & Rationale

**Architecture:** REST API + Lambda Authorizer (or API Key)

This is the easiest meaningful endpoint because it maps directly to the existing serverless layer in Terraform:

- API Gateway HTTP API: `chatbox-usage-plan`
- Route: `POST /chatbox`
- Integration: Lambda proxy -> `project-g7-get-san-choi`
- Authentication: Lambda authorizer -> `chatbox-api-keyr`
- Header checked: `x-api-key`
- Throttling: HTTP API stage default route settings, rate `100`, burst `200`

---

## Evidence

### Screenshot 1: API Gateway Route

![MH4_01_api_route_post_ask](./img/MH4_01_apigw_route.png)

---

### Screenshot 2: API KEY

![MH4_Auth_Config](./img/MH4_auth_config.png)

---
### Screenshot 3: Stage Throttling
![MH4_Stage_Throttling](./img/MH4_stage_throttling.png)


---
### Screenshot 3: Test 200 — Authorized Request ✅

![MH4_Test_200_Authorized](./img/MH4_test_200_authorized.png)


---

### Screenshot 4: Test 403 — Unauthorized Request ❌

![MH4_Test_403_Unauthorized](./img/MH4_test_403_unauthorized.png)

---

**Completion Status:** ✅ **Ready for Presentation**

**Group:** [GROUP 07]  
**Date Completed:** [2026-05-14]  
<!-- **Trainer Sign-off:** [Trainer name]  
**Notes:** W5 thêm hardening layers, đủ 3 business domains (connectivity, storage, scaling) -->
