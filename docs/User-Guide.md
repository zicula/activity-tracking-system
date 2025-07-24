# User Guide - ACTIVITY-TRACKING System

## Document Information
**Project:** ACTIVITY-TRACKING System  
**Version:** 1.0  
**Date:** December 2024  
**Target Audience:** End Users, System Administrators, Dashboard Users  
**Last Updated:** December 2024

---

## 📋 Table of Contents

1. [Getting Started](#-getting-started)
2. [Dashboard Overview](#-dashboard-overview)
3. [User Authentication](#-user-authentication)
4. [Log Search and Analysis](#-log-search-and-analysis)
5. [Reports and Analytics](#-reports-and-analytics)
6. [System Administration](#-system-administration)
7. [Troubleshooting](#-troubleshooting)
8. [FAQ](#-frequently-asked-questions)

---

## 🚀 Getting Started

### System Overview
The Activity Tracking System provides real-time monitoring and analysis of user activities across E-Office applications including:
- **EDCS** (Electronic Document Circulation System)
- **LEAV** (Leave Management System)  
- **OFC-APP** (Office Application Portal)

### Accessing the System
1. **URL:** https://activity-tracking.eoffice.local
2. **Supported Browsers:** Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
3. **Minimum Screen Resolution:** 1280x720

### User Roles and Permissions
| Role | Permissions | Description |
|------|-------------|-------------|
| **Dashboard User** | View dashboards, basic analytics | Standard users who monitor system activity |
| **Log Viewer** | Search and view logs, export data | Users who need detailed log analysis |
| **Report Generator** | Create and schedule reports | Users who generate analytical reports |
| **System Admin** | Full system access, configuration | Administrators with complete system control |

---

## 🏠 Dashboard Overview

### Main Dashboard Layout
```
┌─────────────────────────────────────────────────────┐
│ [Logo] Activity Tracking System    [User Menu] ☰   │
├─────────────────────────────────────────────────────┤
│ Navigation: Dashboard | Search | Reports | Admin    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  📊 System Overview Cards                          │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │ Total   │ │ Active  │ │ Success │ │ Error   │   │
│  │ Users   │ │Sessions │ │ Rate    │ │ Rate    │   │
│  │ 1,250   │ │   48    │ │ 99.2%   │ │ 0.8%    │   │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘   │
│                                                     │
│  📈 Charts and Analytics                           │
│  ┌─────────────────────┐ ┌─────────────────────┐   │
│  │ Authentication      │ │ Document Access     │   │
│  │ Trends             │ │ Patterns           │   │
│  │                     │ │                     │   │
│  │  [Line Chart]       │ │  [Bar Chart]        │   │
│  └─────────────────────┘ └─────────────────────┘   │
│                                                     │
│  📋 Recent Activity Table                          │
│  Time     │ User    │ System │ Action    │ Status   │
│  14:30:15 │ sirat   │ EDCS   │ LOGIN     │ SUCCESS  │
│  14:31:20 │ somchai │ LEAV   │ DOWNLOAD  │ SUCCESS  │
│  └─────────────────────────────────────────────────┘
```

### Dashboard Features

#### 1. Time Range Selection
```html
<!-- Time Range Selector -->
<select id="time-range-selector">
  <option value="1h">Last 1 Hour</option>
  <option value="6h">Last 6 Hours</option>
  <option value="24h" selected>Last 24 Hours</option>
  <option value="7d">Last 7 Days</option>
  <option value="30d">Last 30 Days</option>
</select>
```

**How to use:**
1. Click on the time range dropdown in the top-right corner
2. Select your desired time period
3. Dashboard automatically refreshes with new data
4. Charts and statistics update to reflect the selected timeframe

#### 2. System Overview Cards
Each card displays real-time metrics:

**Total Users Card**
- Shows unique users active in the selected time period
- Updates every 30 seconds
- Click to drill down to user list

**Active Sessions Card**  
- Displays currently active user sessions
- Shows sessions across all systems (EDCS, LEAV, OFC-APP)
- Green indicator for healthy session count

**Success Rate Card**
- Percentage of successful operations
- Calculated from login attempts, document access, etc.
- Red warning if below 95%

**Error Rate Card**
- Percentage of failed operations
- Includes authentication failures, system errors
- Yellow warning if above 1%, red if above 5%

#### 3. Interactive Charts

**Authentication Trends Chart**
- Line chart showing login/logout patterns
- Hover over points to see exact values
- Legend shows different systems
- Click legend items to show/hide system data

**Document Access Patterns**
- Bar chart showing document operations
- Categories: View, Download, Upload, Modify
- Grouped by system (EDCS, LEAV, OFC-APP)
- Click bars to filter recent activity table

#### 4. Recent Activity Table
- Real-time log entries from all systems
- Auto-refreshes every 15 seconds
- Sortable columns (click column headers)
- Color-coded status indicators:
  - 🟢 Green: Successful operations
  - 🟡 Yellow: Warnings or retries
  - 🔴 Red: Errors or failures

**Table Actions:**
- Click row to view detailed log entry
- Use pagination controls for older entries
- Export visible data to CSV/Excel

---

## 🔐 User Authentication

### Login Process
1. Navigate to https://activity-tracking.eoffice.local
2. Enter your E-Office credentials
3. Complete two-factor authentication if enabled
4. System redirects to appropriate dashboard based on role

### Single Sign-On (SSO)
The system integrates with E-Office SSO:
- Automatic login if already authenticated in other E-Office systems
- Session sharing across EDCS, LEAV, and OFC-APP
- Automatic logout when E-Office session expires

### Password Requirements
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter  
- At least one number
- At least one special character (@, #, $, %, etc.)

### Session Management
- **Session Duration:** 8 hours of inactivity
- **Concurrent Sessions:** Maximum 3 per user
- **Session Extension:** Automatic with activity
- **Forced Logout:** Admin can terminate sessions

---

## 🔍 Log Search and Analysis

### Accessing Log Search
1. Click "Search" in the main navigation
2. Or click "View Details" from any dashboard chart
3. Advanced search form loads with filters

### Search Interface Layout
```
┌─────────────────────────────────────────────────────┐
│ 🔍 Log Search and Analysis                         │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Search Filters                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │ Username: [sirat        ] 🔍                │   │
│  │ System:   [EDCS    ▼] Action: [LOGIN  ▼]   │   │
│  │ From:     [2024-12-15 09:00] To: [Now     ] │   │
│  │ IP:       [192.168.1.   ] Status: [All ▼] │   │
│  │                                             │   │
│  │ [Clear] [Search] [Save Search] [Export]    │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Search Results (1,234 entries found)              │
│  ┌─────────────────────────────────────────────┐   │
│  │ Timestamp         │User    │System│Action  │   │
│  │ 2024-12-15 14:30  │sirat   │EDCS  │LOGIN   │   │
│  │ 2024-12-15 14:31  │sirat   │EDCS  │DOWNLOAD│   │
│  │ 2024-12-15 14:32  │somchai │LEAV  │VIEW    │   │
│  │ ...               │        │      │        │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  [◀ Previous] Page 1 of 62 [Next ▶]               │
└─────────────────────────────────────────────────────┘
```

### Search Filters

#### Basic Filters
**Username Filter**
- Enter exact username or partial match
- Supports wildcards: `sir*` finds "sirat", "siriwat"
- Case-insensitive search
- Leave blank to search all users

**System Filter**
- Dropdown with options: All, EDCS, LEAV, OFC-APP
- Filter logs from specific systems
- Multi-select available (hold Ctrl/Cmd)

**Action Filter**
- Common actions: LOGIN, LOGOUT, DOWNLOAD, UPLOAD, VIEW, MODIFY
- Dropdown with most frequent actions
- Custom action entry available

**Date/Time Range**
- From/To date picker with time selection
- Quick ranges: Today, Yesterday, Last Week, Last Month
- Maximum range: 6 months for performance

**IP Address Filter**
- Enter full or partial IP address
- Supports CIDR notation: `192.168.1.0/24`
- Useful for tracking location-based access

**Status Filter**
- SUCCESS: Completed operations
- FAILED: Failed operations
- WARNING: Operations with issues
- All: No status filtering

#### Advanced Filters
**Free Text Search**
- Search across all log fields
- Supports Boolean operators: AND, OR, NOT
- Example: `document AND (download OR upload)`
- Phrase search with quotes: `"login failed"`

**Response Time Filter**
- Filter by operation duration
- Options: < 1s, 1-5s, 5-10s, > 10s
- Useful for performance analysis

**Session ID Filter**
- Track specific user sessions
- Enter full or partial session ID
- Helps in troubleshooting user issues

### Search Results

#### Result Columns
| Column | Description | Sortable |
|--------|-------------|----------|
| Timestamp | When the event occurred | ✓ |
| Username | User who performed action | ✓ |
| System | Source system (EDCS/LEAV/OFC-APP) | ✓ |
| Action | Type of operation | ✓ |
| Target | Document/resource accessed | ✓ |
| IP Address | User's IP address | ✓ |
| Status | Operation result | ✓ |
| Duration | Response time | ✓ |

#### Result Actions
**View Details**
- Click any row to see full log entry
- Shows complete event information
- Includes metadata and context

**Export Results**
- Export current page or all results
- Formats: CSV, Excel, PDF
- Includes applied filters in export metadata

**Save Search**
- Save frequently used filter combinations
- Give custom names to saved searches
- Share saved searches with team members

### Quick Search Examples

#### Common Search Scenarios

**1. Find All Failed Logins**
```
Filters:
- Action: LOGIN
- Status: FAILED
- Time Range: Last 24 hours
```

**2. Track Specific User Activity**
```
Filters:
- Username: sirat
- Time Range: Today
- All systems and actions
```

**3. Document Download Analysis**
```
Filters:
- Action: DOWNLOAD
- System: EDCS
- Time Range: Last week
```

**4. Security Investigation**
```
Filters:
- IP Address: 192.168.1.100
- Status: FAILED
- Time Range: Last 30 days
```

**5. Performance Issues**
```
Advanced Filters:
- Response Time: > 10s
- Time Range: Last 6 hours
- All systems
```

---

## 📊 Reports and Analytics

### Report Types

#### 1. User Activity Report
**Purpose:** Analyze individual user behavior patterns
**Contents:**
- Login frequency and timing
- System usage breakdown
- Document access patterns
- Error rates and common issues

**How to Generate:**
1. Navigate to Reports → User Activity
2. Select target user from dropdown
3. Choose date range (up to 3 months)
4. Select report format (PDF, Excel, CSV)
5. Click "Generate Report"

**Sample Report Structure:**
```
User Activity Report - sirat
Report Period: 2024-12-01 to 2024-12-15
Generated: 2024-12-15 15:30:45

SUMMARY
- Total Sessions: 28
- Total Actions: 342
- Average Session Duration: 2h 15m
- Success Rate: 99.1%

SYSTEM USAGE
┌─────────┬─────────┬─────────┬─────────┐
│ System  │ Logins  │ Actions │ % Time  │
├─────────┼─────────┼─────────┼─────────┤
│ EDCS    │   15    │   198   │  58%    │
│ LEAV    │    8    │    89   │  26%    │
│ OFC-APP │    5    │    55   │  16%    │
└─────────┴─────────┴─────────┴─────────┘

ACTIVITY TIMELINE
[Daily activity chart]

DOCUMENT ACCESS
Top 10 most accessed documents...
```

#### 2. System Performance Report
**Purpose:** Monitor system health and performance metrics
**Contents:**
- Response time statistics
- Error rate analysis
- Peak usage periods
- Resource utilization trends

**Automated Scheduling:**
- Daily performance summaries
- Weekly detailed reports
- Monthly trend analysis
- Critical alert reports (immediate)

#### 3. Security Audit Report
**Purpose:** Compliance and security monitoring
**Contents:**
- Failed authentication attempts
- Unusual access patterns
- IP address analysis
- After-hours activity
- Privilege escalation events

**Compliance Features:**
- PDPA compliance reporting
- Audit trail completeness
- Data retention verification
- Access control effectiveness

#### 4. Document Usage Analytics
**Purpose:** Understand document workflow and usage
**Contents:**
- Most accessed documents
- Download/upload patterns
- Document lifecycle analysis
- Storage usage trends

### Creating Custom Reports

#### Report Builder Interface
```
┌─────────────────────────────────────────────────────┐
│ 📋 Custom Report Builder                           │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Report Configuration                               │
│  ┌─────────────────────────────────────────────┐   │
│  │ Report Name: [Monthly Security Audit     ] │   │
│  │ Description: [Security events analysis   ] │   │
│  │                                             │   │
│  │ Data Sources:                               │   │
│  │ ☑ Authentication Events                    │   │
│  │ ☐ Document Access                          │   │
│  │ ☑ System Errors                            │   │
│  │ ☐ Performance Metrics                      │   │
│  │                                             │   │
│  │ Filters:                                    │   │
│  │ Date Range: [Last 30 days             ▼]  │   │
│  │ Systems:    [All systems              ▼]  │   │
│  │ Users:      [All users                ▼]  │   │
│  │                                             │   │
│  │ Output Format:                              │   │
│  │ ○ PDF  ○ Excel  ○ CSV  ● Dashboard View   │   │
│  │                                             │   │
│  │ Schedule:                                   │   │
│  │ ○ One-time  ● Monthly  ○ Weekly  ○ Daily  │   │
│  │                                             │   │
│  │ [Preview] [Save] [Generate] [Cancel]       │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

#### Step-by-Step Custom Report Creation

**Step 1: Define Report Scope**
1. Choose report name and description
2. Select relevant data sources
3. Define target audience and purpose

**Step 2: Configure Filters**
1. Set date range (consider performance impact)
2. Select specific systems if needed
3. Filter by user groups or departments
4. Add advanced criteria if required

**Step 3: Choose Output Format**
- **PDF:** Professional reports for presentations
- **Excel:** Data analysis and manipulation
- **CSV:** Raw data for external tools
- **Dashboard View:** Interactive online report

**Step 4: Set Scheduling (Optional)**
1. One-time: Generate immediately
2. Daily: Early morning delivery
3. Weekly: Monday morning summaries
4. Monthly: First business day reports

**Step 5: Preview and Validate**
1. Click "Preview" to see sample output
2. Verify data accuracy and formatting
3. Adjust filters if needed
4. Save template for future use

### Report Delivery Options

#### Email Delivery
- Automatic email to specified recipients
- PDF attachments for formal reports
- Excel files for analytical reports
- Email summary with key metrics

#### Dashboard Integration
- Real-time report widgets
- Embedded charts and graphs
- Drill-down capabilities
- Export options from dashboard

#### API Access
- RESTful API for custom integrations
- JSON format for web applications
- Scheduled data feeds
- Real-time query capabilities

---

## ⚙️ System Administration

### Admin Dashboard Access
**Requirements:**
- System Admin role
- Valid admin credentials
- VPN connection (for remote access)

**Navigation:**
1. Login with admin account
2. Click "Admin" in main navigation
3. Admin dashboard loads with system controls

### Admin Dashboard Layout
```
┌─────────────────────────────────────────────────────┐
│ ⚙️  System Administration                          │
├─────────────────────────────────────────────────────┤
│                                                     │
│  System Health                                      │
│  ┌─────────────────────────────────────────────┐   │
│  │ 🟢 Overall Status: Healthy                  │   │
│  │ 🟢 Application: Running (3/3 instances)     │   │
│  │ 🟢 Elasticsearch: Healthy (3/3 nodes)      │   │
│  │ 🟡 Redis: Warning (2/3 nodes)               │   │
│  │ 🟢 Database: Connected                       │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Quick Actions                                      │
│  ┌─────────────────────────────────────────────┐   │
│  │ [Restart Services] [Clear Cache] [Reindex]  │   │
│  │ [View Logs] [Backup Data] [Update Config]   │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Configuration Management                           │
│  User Management | Log Processing | Security |     │
│  Monitoring | Backup/Restore | System Updates      │
└─────────────────────────────────────────────────────┘
```

### User Management

#### User Account Operations
**Add New User**
1. Click "User Management" tab
2. Click "Add User" button
3. Fill user information form:
   ```
   Username: [required, 3-50 characters]
   Email: [required, valid email format]
   Full Name: [required]
   Department: [dropdown selection]
   Role: [Dashboard User/Log Viewer/Report Generator/System Admin]
   Status: [Active/Inactive]
   ```
4. Set initial password or enable auto-generation
5. Configure role-specific permissions
6. Click "Create User"

**Modify User Account**
1. Search for user in user list
2. Click "Edit" action for target user
3. Modify fields as needed
4. Update role and permissions
5. Save changes

**Disable/Enable Users**
1. Select user(s) from list
2. Use bulk actions or individual toggles
3. Disabled users cannot login
4. Sessions are terminated immediately

**Password Reset**
1. Select user requiring password reset
2. Click "Reset Password"
3. Choose: Generate new password or Send reset email
4. User receives secure reset link
5. Force password change on next login

#### Role Management
**Available Roles:**
```
System Admin
├── Full system access
├── User management
├── Configuration changes
├── System maintenance
└── All other role permissions

Report Generator
├── Create custom reports
├── Schedule reports
├── Export all data formats
├── Access analytical tools
└── Log Viewer permissions

Log Viewer
├── Search and view logs
├── Export search results
├── Access basic reports
├── View dashboards
└── Dashboard User permissions

Dashboard User
├── View main dashboard
├── Basic filtering
├── Export dashboard data
└── View assigned reports
```

**Permission Matrix:**
| Feature | Dashboard User | Log Viewer | Report Generator | System Admin |
|---------|---------------|------------|------------------|--------------|
| View Dashboard | ✓ | ✓ | ✓ | ✓ |
| Search Logs | ✗ | ✓ | ✓ | ✓ |
| Export Data | ✗ | ✓ | ✓ | ✓ |
| Create Reports | ✗ | ✗ | ✓ | ✓ |
| User Management | ✗ | ✗ | ✗ | ✓ |
| System Config | ✗ | ✗ | ✗ | ✓ |

### Log Processing Configuration

#### Data Source Management
**Production Log Sources:**
1. **EDCS Logs**
   - Path: `/var/log/edcs/application.log`
   - Format: Standard Java logging
   - Processing: Real-time via Logstash
   - Retention: 90 days

2. **LEAV Logs**
   - Path: `/var/log/leav/app.log`
   - Format: Custom application format
   - Processing: Batch every 5 minutes
   - Retention: 90 days

3. **OFC-APP Logs**
   - Path: `/var/log/ofc-app/access.log`
   - Format: Apache access log format
   - Processing: Real-time via Logstash
   - Retention: 90 days

**Configuration Interface:**
```
┌─────────────────────────────────────────────────────┐
│ 📂 Log Source Configuration                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│  EDCS Log Processing                                │
│  ┌─────────────────────────────────────────────┐   │
│  │ Status: 🟢 Active                           │   │
│  │ Source: /var/log/edcs/application.log       │   │
│  │ Parser: Java Application Log Parser          │   │
│  │ Processing: Real-time                        │   │
│  │ Last Processed: 2024-12-15 15:29:45         │   │
│  │ Records/Hour: 1,234                          │   │
│  │                                             │   │
│  │ [Configure] [Test] [Pause] [View Sample]   │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  [Add New Source] [Bulk Configuration]             │
└─────────────────────────────────────────────────────┘
```

#### Parser Configuration
**Log Pattern Configuration:**
```javascript
// EDCS Login Pattern
{
  "pattern": "%{TIMESTAMP_ISO8601:timestamp} \\[%{DATA:thread}\\] %{LOGLEVEL:level}\\s+%{DATA:class} - User \\[%{USERNAME:username}\\] logged in successfully from IP \\[%{IP:client_ip}\\] with session \\[%{DATA:session_id}\\]",
  "fields": {
    "system": "EDCS",
    "action": "LOGIN",
    "status": "SUCCESS"
  }
}

// LEAV Document Access Pattern
{
  "pattern": "%{TIMESTAMP_ISO8601:timestamp} \\[%{DATA:thread}\\] %{LOGLEVEL:level}\\s+%{DATA:class} - User \\[%{USERNAME:username}\\] %{WORD:action} document \\[%{DATA:document_id}\\]",
  "fields": {
    "system": "LEAV",
    "target_type": "document"
  }
}
```

#### Processing Rules
**Data Transformation Rules:**
1. **Timestamp Standardization**
   - Convert all timestamps to ISO 8601 format
   - Handle timezone differences
   - Fill missing timestamps

2. **Username Normalization**
   - Convert to lowercase
   - Remove domain suffixes
   - Handle service accounts

3. **IP Address Processing**
   - Extract client IP from proxy headers
   - Geolocation enrichment
   - Private IP range detection

4. **Session Correlation**
   - Link related events by session ID
   - Track session duration
   - Identify session anomalies

### Security Configuration

#### Authentication Settings
**Password Policy Configuration:**
```
┌─────────────────────────────────────────────────────┐
│ 🔐 Password Policy Settings                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Password Requirements                              │
│  ┌─────────────────────────────────────────────┐   │
│  │ Minimum Length: [8    ] characters          │   │
│  │ ☑ Require uppercase letters                 │   │
│  │ ☑ Require lowercase letters                 │   │
│  │ ☑ Require numbers                           │   │
│  │ ☑ Require special characters                │   │
│  │ ☐ Prohibit common passwords                 │   │
│  │ ☑ Prohibit username in password             │   │
│  │                                             │   │
│  │ Password History: [5    ] previous passwords │   │
│  │ Password Expiry: [90   ] days               │   │
│  │ Account Lockout: [5    ] failed attempts   │   │
│  │ Lockout Duration: [30  ] minutes            │   │
│  │                                             │   │
│  │ [Save Changes] [Test Policy] [Reset]        │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

#### Session Management
**Session Configuration:**
- **Session Timeout:** 8 hours default, configurable 1-24 hours
- **Max Concurrent Sessions:** 3 per user, configurable 1-10
- **Session Sharing:** Enabled across E-Office systems
- **Remember Me:** Disabled for security (configurable)

#### Access Control
**IP Restriction Configuration:**
```
Allowed IP Ranges:
- 192.168.1.0/24    (Office Network)
- 10.0.0.0/16       (VPN Network)
- 172.16.0.0/12     (DMZ Network)

Blocked IP Addresses:
- Individual IPs that have shown suspicious activity
- Geographic restrictions (if enabled)
- Known malicious IP ranges
```

### System Monitoring

#### Performance Metrics
**Key Performance Indicators:**
```
┌─────────────────────────────────────────────────────┐
│ 📊 System Performance Metrics                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Response Time (Last 24h)                          │
│  ┌─────────────────────────────────────────────┐   │
│  │ Average: 245ms                               │   │
│  │ 95th Percentile: 1.2s                       │   │
│  │ Max: 8.5s                                    │   │
│  │ [📈 View Trend Chart]                       │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Throughput                                         │
│  ┌─────────────────────────────────────────────┐   │
│  │ Requests/Hour: 12,456                       │   │
│  │ Peak Hour: 14:00-15:00 (1,890 req/hr)      │   │
│  │ Log Processing: 99.2% real-time             │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Error Rates                                        │
│  ┌─────────────────────────────────────────────┐   │
│  │ Application Errors: 0.3%                    │   │
│  │ Authentication Failures: 1.2%               │   │
│  │ System Errors: 0.1%                         │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

#### Alert Configuration
**Alert Types and Thresholds:**
```
Critical Alerts (Immediate Notification):
- System down or unresponsive
- Elasticsearch cluster failure
- Authentication system failure
- Data processing stopped

Warning Alerts (15-minute delay):
- Response time > 5 seconds
- Error rate > 2%
- Disk space < 10%
- Memory usage > 90%

Info Alerts (Hourly summary):
- High activity periods
- New user registrations
- Configuration changes
- Scheduled maintenance
```

**Notification Channels:**
- **Email:** Primary contact and distribution lists
- **Slack:** Real-time team notifications
- **SMS:** Critical alerts only
- **Dashboard:** Visual indicators and status

### Backup and Restore

#### Backup Configuration
**Automated Backup Schedule:**
```
Daily Backups (Incremental):
- Time: 02:00 AM local time
- Retention: 30 days
- Includes: Configuration, user data, recent logs

Weekly Backups (Full):
- Time: Sunday 01:00 AM
- Retention: 12 weeks
- Includes: Complete system state, all logs

Monthly Backups (Archive):
- Time: First Sunday of month, 00:00 AM
- Retention: 12 months
- Includes: Full system backup, compressed
```

**Manual Backup Process:**
1. Navigate to Admin → Backup/Restore
2. Click "Create Manual Backup"
3. Select backup scope:
   - Configuration only
   - Configuration + Recent data (7 days)
   - Configuration + All data
4. Enter backup description
5. Click "Start Backup"
6. Monitor progress in real-time
7. Download backup file when complete

#### Restore Procedures
**System Restore Process:**
1. **Preparation:**
   - Schedule maintenance window
   - Notify users of downtime
   - Verify backup integrity

2. **Restore Steps:**
   - Stop application services
   - Clear existing data (if full restore)
   - Import backup data
   - Verify data integrity
   - Restart services
   - Validate system functionality

3. **Post-Restore Validation:**
   - Test user authentication
   - Verify dashboard data
   - Check log processing
   - Confirm all integrations

**Disaster Recovery:**
- **RTO (Recovery Time Objective):** 4 hours
- **RPO (Recovery Point Objective):** 1 hour
- **Backup Storage:** Multiple locations including off-site
- **Testing:** Quarterly disaster recovery drills

---

## 🔧 Troubleshooting

### Common Issues and Solutions

#### 1. Dashboard Not Loading
**Symptoms:**
- Blank dashboard page
- Loading spinner never stops
- Error messages in browser console

**Possible Causes and Solutions:**

**A. Browser Compatibility Issues**
```
Solution:
1. Check browser version (Chrome 90+, Firefox 88+, Safari 14+)
2. Clear browser cache and cookies
3. Disable browser extensions temporarily
4. Try incognito/private browsing mode
```

**B. Network Connectivity Problems**
```
Solution:
1. Check internet connection
2. Verify VPN connection (if required)
3. Test connection to: ping activity-tracking.eoffice.local
4. Check firewall settings
```

**C. Authentication Issues**
```
Solution:
1. Verify login credentials
2. Check if account is active (contact admin)
3. Clear saved passwords and re-login
4. Try logging into other E-Office systems first
```

**D. Server-Side Issues**
```
Solution:
1. Check system status page
2. Contact system administrator
3. Wait for automatic recovery (5-10 minutes)
4. Report issue with browser console errors
```

#### 2. Search Results Not Appearing
**Symptoms:**
- Search returns 0 results unexpectedly
- Search takes very long time
- Error message during search

**Troubleshooting Steps:**

**Step 1: Verify Search Criteria**
```
Check:
- Date range is not too narrow
- Username spelling is correct
- System selection includes relevant systems
- No conflicting filters applied
```

**Step 2: Test Basic Search**
```
Try:
1. Clear all filters
2. Set date range to "Last 24 hours"
3. Leave all fields empty
4. Click Search
```

**Step 3: Check Data Availability**
```
Verify:
- Log processing is active (Admin dashboard)
- Selected time period has data
- Your role has permission to view logs
- System is not in maintenance mode
```

**Step 4: Browser Issues**
```
Try:
1. Refresh the page (F5)
2. Clear browser cache
3. Use different browser
4. Check browser console for errors
```

#### 3. Slow Performance
**Symptoms:**
- Dashboard takes long time to load
- Search results appear slowly
- Charts don't render properly

**Performance Optimization:**

**Client-Side Optimization**
```
Actions:
1. Close unnecessary browser tabs
2. Clear browser cache regularly
3. Disable unused browser extensions
4. Use latest browser version
5. Check available RAM (4GB+ recommended)
```

**Search Optimization**
```
Best Practices:
1. Use specific date ranges (avoid "All time")
2. Add username filter to narrow results
3. Use system filter when possible
4. Limit result size (use pagination)
5. Avoid very broad text searches
```

**Server-Side Issues**
```
If problems persist:
1. Check system status dashboard
2. Contact administrator
3. Report performance metrics
4. Schedule during off-peak hours
```

#### 4. Export/Report Issues
**Symptoms:**
- Export fails with error
- Generated reports are empty
- Email delivery not working

**Export Troubleshooting:**

**File Format Issues**
```
Solutions:
1. Try different export format (CSV instead of Excel)
2. Reduce data size (smaller date range)
3. Check browser download settings
4. Ensure sufficient disk space
```

**Permission Issues**
```
Check:
1. Your role allows export functionality
2. You have access to selected data
3. Export quota not exceeded
4. Contact admin for permission verification
```

**Email Delivery Problems**
```
Verify:
1. Email address is correct
2. Check spam/junk folder
3. Email server is working
4. File size is under limit (10MB)
```

### Error Messages Reference

#### Authentication Errors
```
Error: "Invalid username or password"
Solution: 
- Verify credentials are correct
- Check if account is locked
- Reset password if necessary
- Contact admin if problem persists

Error: "Account has been disabled"
Solution:
- Contact system administrator
- Account may need reactivation
- Check if temporary suspension

Error: "Session expired"
Solution:
- Re-login to the system
- Check session timeout settings
- Contact admin if frequent occurrences
```

#### Data Access Errors
```
Error: "Insufficient permissions"
Solution:
- Verify your role allows this action
- Contact admin for role update
- Check if specific permission needed

Error: "Data not available"
Solution:
- Check if logs are being processed
- Verify date range has data
- Contact admin if persistent

Error: "Search timeout"
Solution:
- Narrow search criteria
- Use smaller date ranges
- Try during off-peak hours
```

#### System Errors
```
Error: "Service temporarily unavailable"
Solution:
- Wait 5-10 minutes and retry
- Check system status page
- Contact admin if extended downtime

Error: "Database connection error"
Solution:
- Report to administrator immediately
- System maintenance may be in progress
- Check status updates

Error: "Processing queue full"
Solution:
- Try again in a few minutes
- Reduce request size
- Contact admin if frequent
```

### Contacting Support

#### When to Contact Support
- **Immediate (Critical):**
  - Cannot access system at all
  - Data security concerns
  - System showing incorrect data
  - Suspected security breach

- **Normal Business Hours:**
  - Performance issues
  - Feature questions
  - Report problems
  - Account modifications

- **Self-Service First:**
  - Password resets
  - Basic troubleshooting
  - User guide questions
  - Training requests

#### Support Contact Information
```
IT Support Desk:
- Email: it-support@eoffice.local
- Phone: +66-2-XXX-XXXX (ext. 1234)
- Hours: Monday-Friday, 8:00-17:00

Emergency Contact:
- Phone: +66-8X-XXX-XXXX
- Available: 24/7 for critical issues

System Administrator:
- Email: admin@eoffice.local
- Response: Within 4 business hours
```

#### Information to Include in Support Requests
```
Required Information:
1. Your username
2. Date and time of issue
3. Browser and version
4. Steps to reproduce the problem
5. Error messages (exact text)
6. Screenshots if applicable

Optional but Helpful:
- Browser console errors
- Network configuration details
- Other affected users
- Business impact description
```

---

## ❓ Frequently Asked Questions

### General Usage

**Q: How often is the dashboard data updated?**
A: Dashboard data updates automatically every 30 seconds. The "Last Updated" timestamp at the bottom of the dashboard shows when data was last refreshed. You can also manually refresh by clicking the refresh icon.

**Q: Can I access the system from mobile devices?**
A: Yes, the system is responsive and works on tablets and smartphones. However, for best experience, we recommend using a desktop or laptop computer with a screen resolution of at least 1280x720.

**Q: How far back can I search logs?**
A: Log data is retained for 90 days by default. You can search any data within this period. For historical analysis beyond 90 days, contact your administrator about archived data availability.

**Q: Can I save my search filters for reuse?**
A: Yes, click "Save Search" after configuring your filters. Saved searches appear in the "Quick Searches" dropdown for easy access. You can save up to 10 different search configurations.

### User Accounts and Access

**Q: I forgot my password. How can I reset it?**
A: Click "Forgot Password" on the login page and enter your username. You'll receive a secure reset link via email. The link expires in 24 hours for security.

**Q: Why can't I see certain menu options?**
A: Menu visibility is based on your user role. Contact your administrator if you need access to additional features. Your current role is displayed in the top-right corner of the interface.

**Q: Can I have multiple concurrent sessions?**
A: Yes, you can have up to 3 concurrent sessions by default. If you exceed this limit, the oldest session will be automatically terminated. This setting can be adjusted by administrators.

**Q: How long do sessions last?**
A: Sessions remain active for 8 hours of inactivity by default. Your session will extend automatically when you interact with the system. A warning appears 15 minutes before session expiry.

### Data and Reports

**Q: Why do I see different numbers in different reports?**
A: Numbers may differ due to:
- Different time ranges or filters applied
- Real-time data updates between report generations
- Different calculation methods (unique users vs. total actions)
- Data processing delays (usually under 5 minutes)

**Q: Can I export data to Excel for further analysis?**
A: Yes, most data can be exported in Excel, CSV, or PDF formats. Look for the "Export" button in search results, reports, or dashboard widgets. Large exports may take several minutes to prepare.

**Q: How accurate is the real-time data?**
A: Real-time data has a processing delay of 30 seconds to 2 minutes depending on the data source. Critical security events are processed faster (under 30 seconds) while routine activity may have slightly longer delays.

**Q: Can I schedule reports to be sent automatically?**
A: Yes, if you have Report Generator role or higher. Use the Report Builder to create custom reports and set delivery schedules (daily, weekly, monthly). Reports are delivered via email as PDF attachments.

### Technical Issues

**Q: The system seems slow. What can I do?**
A: Try these steps:
1. Close unnecessary browser tabs
2. Clear browser cache
3. Use more specific search filters
4. Try during off-peak hours (before 9 AM or after 5 PM)
5. Contact support if problems persist

**Q: I'm getting timeout errors when searching.**
A: Reduce your search scope:
- Use shorter date ranges (1 week instead of 1 month)
- Add username or system filters
- Avoid very broad text searches
- Use pagination instead of requesting all results

**Q: Can I use the system with Internet Explorer?**
A: Internet Explorer is not supported due to security and compatibility issues. Please use Chrome 90+, Firefox 88+, Safari 14+, or Edge 90+ for the best experience.

**Q: What should I do if I see suspicious activity in the logs?**
A: Immediately:
1. Take a screenshot of the suspicious activity
2. Note the exact time and details
3. Report to IT Security team
4. Do not attempt to investigate further yourself
5. Document any related observations

### Security and Compliance

**Q: Is my search activity logged?**
A: Yes, all user activities within the system are logged for security and audit purposes. This includes logins, searches, exports, and report generation. Only authorized administrators can access these audit logs.

**Q: How is sensitive data protected?**
A: The system implements multiple security measures:
- All data transmission is encrypted (HTTPS/TLS)
- User passwords are hashed and salted
- Role-based access controls limit data visibility
- Regular security audits and updates
- Compliance with PDPA regulations

**Q: Can I share my login credentials with colleagues?**
A: No, sharing login credentials is strictly prohibited for security and audit reasons. Each user must have their own account. Contact your administrator to create accounts for additional users.

**Q: What happens to data when an employee leaves?**
A: When an employee leaves:
1. Their account is immediately disabled
2. Active sessions are terminated
3. Saved searches and reports are archived
4. Account data is retained per company policy
5. Access logs are preserved for audit purposes

### System Maintenance

**Q: When does system maintenance occur?**
A: Regular maintenance is scheduled for:
- Weekly: Sunday 2:00-4:00 AM (minor updates)
- Monthly: First Sunday 1:00-5:00 AM (major updates)
- Emergency: As needed with advance notice when possible

**Q: How will I know about planned downtime?**
A: Maintenance notifications are sent via:
- Email to all users (48 hours advance notice)
- System banner messages (24 hours advance)
- Dashboard announcements
- Company intranet announcements

**Q: What should I do during maintenance windows?**
A: During maintenance:
- Save your work and log out before scheduled start time
- Plan alternative workflows if needed
- Check email for updates if maintenance extends beyond schedule
- Do not attempt to log in during maintenance

**Q: How can I stay informed about system updates?**
A: Stay informed through:
- Release notes published after updates
- Email notifications for major changes
- User guide updates
- Training sessions for new features
- Monthly system newsletter (if available)

---

## 📞 Support and Contact Information

### Primary Support Channels

#### IT Help Desk
- **Email:** it-support@eoffice.local
- **Phone:** +66-2-XXX-XXXX (ext. 1234)
- **Hours:** Monday-Friday, 8:00 AM - 5:00 PM
- **Response Time:** Within 4 business hours
- **Languages:** Thai, English

#### Emergency Support
- **Phone:** +66-8X-XXX-XXXX
- **Available:** 24/7 for critical system issues
- **Use For:** System outages, security incidents, data corruption

#### System Administrator
- **Email:** admin@activity-tracking.eoffice.local
- **Response Time:** Within 2 business hours
- **Use For:** Account issues, configuration requests, technical questions

### Self-Service Resources

#### Online Documentation
- **User Guide:** https://docs.eoffice.local/activity-tracking/user-guide
- **FAQ:** https://docs.eoffice.local/activity-tracking/faq
- **Video Tutorials:** https://training.eoffice.local/activity-tracking
- **Release Notes:** https://docs.eoffice.local/activity-tracking/releases

#### Training and Learning
- **Monthly Training Sessions:** First Wednesday of each month, 2:00-3:00 PM
- **On-Demand Training:** Available 24/7 via learning portal
- **User Certification:** Optional certification program available
- **Best Practices Guide:** Available in documentation portal

### Feedback and Suggestions

#### Feature Requests
- **Email:** feature-requests@eoffice.local
- **Process:** Submit detailed description and business justification
- **Review:** Quarterly review by development team
- **Updates:** Notification of status changes

#### Bug Reports
- **Email:** bug-reports@eoffice.local
- **Include:** Steps to reproduce, screenshots, browser information
- **Priority:** Critical bugs addressed within 24 hours
- **Updates:** Status tracking via ticket system

---

**Document Version:** 1.0  
**Last Updated:** December 15, 2024  
**Next Review Date:** March 15, 2025  
**Document Owner:** E-Office Development Team

---

**Related Documents:**
- [System Requirements](./system-requirements.md)
- [Administrator Guide](./administrator-guide.md)
- [API Documentation](./api-documentation.md)
- [Security Guide](./security-guide.md)
