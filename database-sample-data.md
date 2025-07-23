# Database Sample Data Examples - ACTIVITY-TRACKING System

## ตัวอย่างข้อมูลที่เกิดขึ้นใน Database Tables

### 1. User Authentication Tracking (FR1)

#### Table: USER_AUTH_LOGS
```sql
-- ตัวอย่างข้อมูล Login/Logout Activities
INSERT INTO USER_AUTH_LOGS VALUES 
(1, 'USR001', 'LOGIN', '2025-07-23 08:30:15', '192.168.1.xxx', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/126.0.0.0', 'Windows 10', 'SESSION_12345', 'SUCCESS', NULL),
(2, 'USR001', 'LOGOUT', '2025-07-23 17:45:32', '192.168.1.xxx', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/126.0.0.0', 'Windows 10', 'SESSION_12345', 'SUCCESS', NULL),
(3, 'USR002', 'LOGIN', '2025-07-23 09:15:22', '192.168.2.xxx', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) Safari/605.1.15', 'macOS', 'SESSION_67890', 'SUCCESS', NULL),
(4, 'USR003', 'LOGIN_FAILED', '2025-07-23 14:22:10', '203.154.xxx.xxx', 'Mozilla/5.0 (iPhone; CPU iPhone OS 16_0) Mobile/15E148', 'iOS', NULL, 'FAILED', 'Invalid credentials'),
(5, 'USR001', 'CONCURRENT_SESSION', '2025-07-23 10:30:00', '192.168.3.xxx', 'Mozilla/5.0 (Android 12) Chrome/126.0.0.0', 'Android', 'SESSION_11111', 'WARNING', 'Multiple active sessions detected');
```

#### Table Structure:
- **LOG_ID**: Primary key
- **USER_ID**: รหัสผู้ใช้ (anonymized)
- **ACTION_TYPE**: LOGIN, LOGOUT, LOGIN_FAILED, CONCURRENT_SESSION
- **TIMESTAMP**: เวลาที่เกิดเหตุการณ์
- **IP_ADDRESS**: IP address (masked สำหรับ privacy)
- **USER_AGENT**: Browser และ device information
- **DEVICE_TYPE**: Windows, macOS, iOS, Android
- **SESSION_ID**: Session identifier
- **STATUS**: SUCCESS, FAILED, WARNING
- **NOTES**: ข้อมูลเพิ่มเติม

---

### 2. Document Access Tracking (FR2)

#### Table: DOCUMENT_ACCESS_LOGS
```sql
-- ตัวอย่างข้อมูล Document Activities
INSERT INTO DOCUMENT_ACCESS_LOGS VALUES 
(1, 'USR001', 'DOC_12345', 'VIEW', '2025-07-23 09:30:45', 'contract_template.pdf', 'PDF', 2048576, 45, 'HR_MODULE', NULL),
(2, 'USR002', 'DOC_67890', 'DOWNLOAD', '2025-07-23 10:15:30', 'employee_handbook.docx', 'DOCX', 15728640, NULL, 'HR_MODULE', 'Downloaded to local machine'),
(3, 'USR001', 'DOC_11111', 'UPLOAD', '2025-07-23 11:22:18', 'budget_proposal_2025.xlsx', 'XLSX', 8388608, NULL, 'FINANCE_MODULE', 'New document uploaded'),
(4, 'USR003', 'DOC_12345', 'SHARE', '2025-07-23 13:45:12', 'contract_template.pdf', 'PDF', 2048576, NULL, 'HR_MODULE', 'Shared with USR004, USR005'),
(5, 'USR002', 'DOC_22222', 'VIEW', '2025-07-23 14:30:22', 'sensitive_audit_report.pdf', 'PDF', 5242880, 120, 'AUDIT_MODULE', 'Sensitive document access logged');
```

#### Table Structure:
- **LOG_ID**: Primary key
- **USER_ID**: รหัสผู้ใช้
- **DOCUMENT_ID**: รหัสเอกสาร
- **ACTION_TYPE**: VIEW, DOWNLOAD, UPLOAD, SHARE, DELETE
- **TIMESTAMP**: เวลาที่เกิดเหตุการณ์
- **DOCUMENT_NAME**: ชื่อไฟล์ (อาจ hash สำหรับ sensitive files)
- **FILE_TYPE**: PDF, DOCX, XLSX, etc.
- **FILE_SIZE**: ขนาดไฟล์ (bytes)
- **ACCESS_DURATION**: ระยะเวลาการดู (วินาที)
- **MODULE_NAME**: Module ที่เข้าถึงเอกสาร
- **NOTES**: รายละเอียดเพิ่มเติม

---

### 3. Approval Process Tracking (FR3)

#### Table: APPROVAL_PROCESS_LOGS
```sql
-- ตัวอย่างข้อมูล Approval Activities
INSERT INTO APPROVAL_PROCESS_LOGS VALUES 
(1, 'WORKFLOW_001', 'USR001', 'INITIATE', '2025-07-23 08:45:30', 'LEAVE_REQUEST', 'Requesting 3 days annual leave', 1, 'PENDING', NULL, NULL),
(2, 'WORKFLOW_001', 'USR010', 'APPROVE', '2025-07-23 09:30:15', 'LEAVE_REQUEST', 'Approved by direct supervisor', 2, 'APPROVED', 'USR001', 'Approved - sufficient leave balance'),
(3, 'WORKFLOW_002', 'USR003', 'INITIATE', '2025-07-23 10:15:45', 'PURCHASE_ORDER', 'Office equipment procurement - 150,000 THB', 1, 'PENDING', NULL, NULL),
(4, 'WORKFLOW_002', 'USR011', 'REVIEW', '2025-07-23 11:22:30', 'PURCHASE_ORDER', 'Budget verification completed', 2, 'UNDER_REVIEW', 'USR003', 'Budget approved, forwarding to finance'),
(5, 'WORKFLOW_003', 'USR005', 'REJECT', '2025-07-23 13:45:18', 'EXPENSE_CLAIM', 'Missing required receipts', 2, 'REJECTED', 'USR004', 'Please resubmit with complete documentation'),
(6, 'WORKFLOW_002', 'USR012', 'DELEGATE', '2025-07-23 14:30:45', 'PURCHASE_ORDER', 'Delegated approval authority', 3, 'DELEGATED', 'USR011', 'Delegated to finance director due to amount');
```

#### Table Structure:
- **LOG_ID**: Primary key
- **WORKFLOW_ID**: รหัส workflow instance
- **USER_ID**: ผู้ที่ทำ action
- **ACTION_TYPE**: INITIATE, APPROVE, REJECT, REVIEW, DELEGATE, REVISE
- **TIMESTAMP**: เวลาที่เกิดเหตุการณ์
- **WORKFLOW_TYPE**: LEAVE_REQUEST, PURCHASE_ORDER, EXPENSE_CLAIM, etc.
- **DESCRIPTION**: รายละเอียดของ workflow
- **STEP_NUMBER**: ขั้นตอนใน workflow
- **STATUS**: PENDING, APPROVED, REJECTED, UNDER_REVIEW, DELEGATED
- **ORIGINAL_SUBMITTER**: ผู้ส่งคำขอเดิม
- **COMMENTS**: ความคิดเห็นของผู้อนุมัติ

---

### 4. Module Usage Tracking (FR4)

#### Table: MODULE_USAGE_LOGS
```sql
-- ตัวอย่างข้อมูล Module Usage
INSERT INTO MODULE_USAGE_LOGS VALUES 
(1, 'USR001', 'HR_MODULE', 'ENTER', '2025-07-23 08:30:45', 'SESSION_12345', NULL, NULL, 'Direct navigation'),
(2, 'USR001', 'HR_MODULE', 'EXIT', '2025-07-23 08:45:30', 'SESSION_12345', 900, 'DOCUMENT_MODULE', 'Switched to document management'),
(3, 'USR002', 'FINANCE_MODULE', 'ENTER', '2025-07-23 09:15:22', 'SESSION_67890', NULL, 'HR_MODULE', 'Navigation from HR module'),
(4, 'USR001', 'DOCUMENT_MODULE', 'ENTER', '2025-07-23 08:45:30', 'SESSION_12345', NULL, 'HR_MODULE', 'Switched from HR module'),
(5, 'USR003', 'APPROVAL_MODULE', 'ENTER', '2025-07-23 10:30:15', 'SESSION_99999', NULL, NULL, 'Direct access from dashboard'),
(6, 'USR002', 'REPORT_CENTER', 'ENTER', '2025-07-23 11:45:18', 'SESSION_67890', NULL, 'FINANCE_MODULE', 'Generating financial reports'),
(7, 'USR001', 'DOCUMENT_MODULE', 'EXIT', '2025-07-23 09:30:12', 'SESSION_12345', 2682, 'DASHBOARD', 'Returned to main dashboard');
```

#### Table Structure:
- **LOG_ID**: Primary key
- **USER_ID**: รหัสผู้ใช้
- **MODULE_NAME**: ชื่อ module
- **ACTION_TYPE**: ENTER, EXIT
- **TIMESTAMP**: เวลาที่เกิดเหตุการณ์
- **SESSION_ID**: Session identifier
- **DURATION_SECONDS**: เวลาที่ใช้ใน module (วินาที)
- **PREVIOUS_MODULE**: Module ที่มาก่อนหน้า
- **NOTES**: รายละเอียดเพิ่มเติม

---

### 5. Real-time Dashboard Data (FR5)

#### Table: REAL_TIME_METRICS
```sql
-- ตัวอย่างข้อมูล Real-time Metrics
INSERT INTO REAL_TIME_METRICS VALUES 
('2025-07-23 15:30:00', 1247, 15, 3, 2, 0, 8, 45.67, 1.23, 99.95),
('2025-07-23 15:31:00', 1251, 12, 5, 1, 1, 6, 46.12, 1.18, 99.95),
('2025-07-23 15:32:00', 1245, 18, 2, 3, 0, 12, 44.89, 1.31, 99.94),
('2025-07-23 15:33:00', 1256, 20, 4, 2, 0, 15, 47.23, 1.15, 99.96),
('2025-07-23 15:34:00', 1263, 14, 6, 1, 2, 9, 48.45, 1.27, 99.93);
```

#### Table Structure:
- **TIMESTAMP**: เวลาการวัด
- **ACTIVE_USERS**: จำนวนผู้ใช้ที่ active
- **NEW_LOGINS**: Login ใหม่ในช่วง 1 นาที
- **DOCUMENT_ACCESSES**: การเข้าถึงเอกสารในช่วง 1 นาที
- **APPROVAL_ACTIONS**: การอนุมัติในช่วง 1 นาที
- **SECURITY_ALERTS**: แจ้งเตือนความปลอดภัย
- **MODULE_SWITCHES**: การสลับ module ในช่วง 1 นาที
- **CPU_USAGE_PERCENT**: การใช้ CPU
- **RESPONSE_TIME_AVG**: Response time เฉลี่ย (วินาที)
- **UPTIME_PERCENT**: System uptime

---

### 6. Security and Audit Events

#### Table: SECURITY_AUDIT_LOGS
```sql
-- ตัวอย่างข้อมูล Security Events
INSERT INTO SECURITY_AUDIT_LOGS VALUES 
(1, 'USR001', 'SUSPICIOUS_LOGIN', '2025-07-23 02:30:15', 'HIGH', '203.154.xxx.xxx', 'Login attempt from unusual location and time', 'ALERT_SENT', 'Security team notified'),
(2, 'USR999', 'BRUTE_FORCE_ATTEMPT', '2025-07-23 03:45:22', 'CRITICAL', '185.220.xxx.xxx', 'Multiple failed login attempts', 'IP_BLOCKED', 'IP address blocked for 24 hours'),
(3, 'USR003', 'SENSITIVE_DOC_ACCESS', '2025-07-23 14:30:22', 'MEDIUM', '192.168.2.xxx', 'Access to confidential salary data', 'LOGGED', 'Normal business access, logged for audit'),
(4, 'SYS_ADMIN', 'PRIVILEGE_ESCALATION', '2025-07-23 16:15:45', 'HIGH', '192.168.1.xxx', 'Admin privileges granted to USR007', 'APPROVED', 'Approved by IT director'),
(5, 'USR005', 'DATA_EXPORT_LARGE', '2025-07-23 13:22:18', 'MEDIUM', '192.168.1.xxx', 'Large data export: 50,000 records', 'LOGGED', 'Monthly report generation');
```

#### Table Structure:
- **LOG_ID**: Primary key
- **USER_ID**: รหัสผู้ใช้หรือระบบ
- **EVENT_TYPE**: ประเภทเหตุการณ์ความปลอดภัย
- **TIMESTAMP**: เวลาที่เกิดเหตุการณ์
- **SEVERITY**: LOW, MEDIUM, HIGH, CRITICAL
- **IP_ADDRESS**: IP address ที่เกี่ยวข้อง
- **DESCRIPTION**: รายละเอียดเหตุการณ์
- **ACTION_TAKEN**: การดำเนินการที่ทำ
- **NOTES**: หมายเหตุเพิ่มเติม

---

## Data Volume Estimates

### Daily Data Generation (Based on 2,000-8,000 users)

| Table | Records/Day | Size/Day (Approx) |
|-------|-------------|-------------------|
| USER_AUTH_LOGS | 16,000-64,000 | 2-8 MB |
| DOCUMENT_ACCESS_LOGS | 50,000-200,000 | 8-32 MB |
| APPROVAL_PROCESS_LOGS | 5,000-20,000 | 1-4 MB |
| MODULE_USAGE_LOGS | 100,000-400,000 | 15-60 MB |
| REAL_TIME_METRICS | 1,440 | 0.1 MB |
| SECURITY_AUDIT_LOGS | 100-500 | 0.1-0.5 MB |

**Total Daily Size: 26-104 MB**  
**Annual Size: 9.5-38 GB**

---

## Data Archiving Strategy

### Archive After 12 Months
```sql
-- ตัวอย่าง Archived Data Table
CREATE TABLE USER_AUTH_LOGS_ARCHIVE_2024 AS 
SELECT * FROM USER_AUTH_LOGS 
WHERE TIMESTAMP < DATE '2025-01-01';
```

### Data Retention Compliance
- **Active Period**: 12 months in main tables
- **Archive Period**: 24 months in archive tables
- **Total Retention**: 36 months (3 years)
- **Auto-deletion**: After 36 months for PDPA compliance

---

## Performance Considerations

### Indexing Strategy
```sql
-- Primary Performance Indexes
CREATE INDEX idx_user_auth_timestamp ON USER_AUTH_LOGS(TIMESTAMP);
CREATE INDEX idx_user_auth_user_id ON USER_AUTH_LOGS(USER_ID);
CREATE INDEX idx_doc_access_timestamp ON DOCUMENT_ACCESS_LOGS(TIMESTAMP);
CREATE INDEX idx_doc_access_user_doc ON DOCUMENT_ACCESS_LOGS(USER_ID, DOCUMENT_ID);
CREATE INDEX idx_approval_workflow ON APPROVAL_PROCESS_LOGS(WORKFLOW_ID);
CREATE INDEX idx_module_usage_session ON MODULE_USAGE_LOGS(SESSION_ID);
```

### Partitioning Strategy
```sql
-- Time-based Partitioning for Large Tables
ALTER TABLE USER_AUTH_LOGS 
PARTITION BY RANGE (TIMESTAMP) 
INTERVAL (NUMTOYMINTERVAL(1, 'MONTH'));

ALTER TABLE DOCUMENT_ACCESS_LOGS 
PARTITION BY RANGE (TIMESTAMP) 
INTERVAL (NUMTOYMINTERVAL(1, 'MONTH'));
```

---

**Note**: ข้อมูลทั้งหมดจะถูก anonymized และ masked ตาม PDPA requirements โดยไม่เก็บข้อมูลส่วนบุคคลที่สามารถระบุตัวตนได้
