# Activity-Log Project: Kiro-Style Development Example

## 🎯 ตัวอย่างการใช้ Kiro-Style Prompts สำหรับโปรเจ็ค Activity-Log

### Scenario: สร้างระบบ Activity Log สำหรับ Web Application

---

## Phase 1: Requirements Analysis

### Input Prompt:
```
[ใช้ Master Prompt + Requirements Analysis Prompt]

User Request: "ต้องการสร้างระบบ Activity Log สำหรับติดตามกิจกรรมของผู้ใช้งานใน web application system รวมถึงการ login, การเข้าถึงเอกสาร, การอนุมัติ และการใช้งานโมดูลต่างๆ"
```

### Expected Output:
```
## 📋 ANALYSIS
ระบบ Activity Log นี้ต้องการ:
- ติดตามกิจกรรมของพนักงาน 4 ประเภทหลัก (login, document access, approval, module usage)
- เก็บข้อมูลสำหรับ audit trail และ compliance
- ต้องมี security และ privacy protection
- รองรับการ query และ reporting

## 📝 PROPOSED APPROACH
1. สร้าง requirements.md เป็นอันดับแรก
2. ขอข้อมูลเพิ่มเติมเกี่ยวกับ:
   - จำนวนผู้ใช้งาน (scale)
   - ระยะเวลาเก็บข้อมูล (retention policy)
   - ระดับ detail ที่ต้องการติดตาม
   - Integration กับระบบ web application ปัจจุบัน

## 📄 DOCUMENTS TO CREATE
1. requirements.md (ต้องทำก่อน)
2. privacy-impact-assessment.md
3. design.md
4. security-requirements.md
5. tasks.md
6. implementation-plan.md

## ✅ VALIDATION CHECKPOINTS
- PDPA compliance check
- Security audit requirements
- Performance benchmarks
- Integration compatibility

## 🔄 NEXT STEPS
ขอให้ยืนยันข้อมูลเหล่านี้ก่อนสร้าง requirements document:
1. จำนวนผู้ใช้งานพร้อมกันที่คาดหวัง (Expected concurrent users)?
2. ระยะเวลาการเก็บข้อมูล (Data retention period)?
3. จุดเชื่อมต่อที่ต้องการกับระบบ web application ปัจจุบัน (Required integration points with existing web application)?
4. ข้อกำหนดด้านการปฏิบัติตามกฎระเบียบ (Compliance requirements: PDPA, ISO27001, etc.)?

### คำแนะนำสำหรับการตอบ:
**ใช้ภาษาไทยเป็นหลักในการตอบและอธิบาย** - แต่สามารถใช้คำศัพท์ทางเทคนิคภาษาอังกฤษได้ตามความเหมาะสม เพื่อความชัดเจนและเป็นมาตรฐานในวงการ IT
```

---

## Phase 2: Design Phase

### Input Prompt:
```
[ใช้ Design Documentation Prompt]

จาก requirements ที่ได้ สร้าง design.md สำหรับระบบ Activity Log:

Requirements Summary:
- รองรับ 1000 concurrent users
- เก็บข้อมูล 7 ปี
- Integration กับ Web Application API
- PDPA compliance required
- Real-time monitoring dashboard

### คำแนะนำ:
**ใช้ภาษาไทยในการอธิบายและเหตุผล** พร้อมรายละเอียดทางเทคนิคภาษาอังกฤษตามมาตรฐาน
```

### Expected Output:
```
# Activity Log System Design

## ภาพรวมของระบบ (System Overview)

### สถาปัตยกรรมระบบระดับสูง (High-Level Architecture)
```
[Frontend Dashboard] → [API Gateway] → [Activity Service] → [Database]
                                   ↓
                              [Message Queue] → [Analytics Service]
                                   ↓
                              [Notification Service]
```

### องค์ประกอบของระบบ (System Components)
| องค์ประกอบ | หน้าที่รับผิดชอบ | เทคโนโลยี |
|-----------|---------------|------------|
| Activity Collector | รวบรวมกิจกรรมของผู้ใช้ | Node.js + Express |
| Data Pipeline | ประมวลผลและเก็บ logs | Apache Kafka + MongoDB |
| Dashboard API | ให้บริการข้อมูลแดชบอร์ด | Node.js + GraphQL |
| Analytics Engine | สร้างข้อมูลเชิงลึก | Python + Pandas |
| Notification Service | ส่งการแจ้งเตือน | Node.js + WebSocket |

## รายละเอียดทางเทคนิค (Technical Specifications)

### เทคโนโลยีที่เลือกใช้ (Technology Stack)
**เหตุผลในการเลือก:**
- **MongoDB**: รองรับการเขียนข้อมูลปริมาณมากสำหรับ activity logs
- **Kafka**: จัดการ event streaming ปริมาณสูง
- **Node.js**: รองรับ I/O เร็วสำหรับการบันทึกแบบ real-time
- **React**: สร้าง dashboard interface ที่ตอบสนองดี

### การออกแบบฐานข้อมูล (Database Design)
```mongodb
// Activities Collection
{
  _id: ObjectId,
  userId: String,          // รหัสผู้ใช้
  action: String,          // การกระทำที่ทำ
  resource: String,        // ทรัพยากรที่เข้าถึง
  timestamp: Date,         // เวลาที่ทำกิจกรรม
  metadata: Object,        // ข้อมูลเพิ่มเติม
  sessionId: String,       // รหัส session
  ipAddress: String (hashed),    // IP address ที่ปกปิด
  userAgent: String (sanitized)  // User agent ที่ทำความสะอาดแล้ว
}

// Users Collection
{
  _id: ObjectId,
  employeeId: String,      // รหัสพนักงาน
  department: String,      // แผนก
  role: String,           // บทบาท
  isActive: Boolean       // สถานะการใช้งาน
}
```

### การออกแบบ API (API Design)
```
POST /api/v1/activities      // บันทึกกิจกรรม
GET /api/v1/activities/search // ค้นหากิจกรรม
GET /api/v1/analytics/dashboard // ข้อมูลแดชบอร์ด
GET /api/v1/reports/export   // ส่งออกรายงาน
```

### สถาปัตยกรรมด้านความปลอดภัย (Security Architecture)
- เข้ารหัสข้อมูลส่วนบุคคลทั้งหมดแบบ at rest
- ทำให้ข้อมูลกิจกรรมเป็นนิรนามหลังจาก 90 วัน
- การยืนยันตัวตน API ผ่าน JWT
- จำกัดอัตราการเรียกใช้ต่อผู้ใช้
```

---

## Phase 3: Task Breakdown

### Input Prompt:
```
[ใช้ Task Breakdown Prompt]

จาก design document สร้าง tasks.md สำหรับ Activity Log system

### คำแนะนำ:
**ใช้ภาษาไทยในการอธิบาย task และ acceptance criteria** เพื่อให้ทีมไทยเข้าใจง่ายขึ้น
```

### Expected Output:
```
# Activity Log System - แผนการแตกงาน (Task Breakdown)

## Epic 1: การตั้งค่าโครงสร้างพื้นฐาน (Core Infrastructure Setup)
**ระยะเวลาประมาณการ:** 2 สัปดาห์

### Task 1.1: การตั้งค่าฐานข้อมูล (Database Setup)
**คำอธิบาย:** ตั้งค่า MongoDB cluster พร้อม indexing ที่เหมาะสม
**เกณฑ์การยอมรับ (Acceptance Criteria):**
- [ ] ติดตั้งและปรับแต่ง MongoDB cluster เสร็จสิ้น
- [ ] สร้าง indexes เพื่อเพิ่มประสิทธิภาพการ query
- [ ] วางแผนกลยุทธ์การสำรองข้อมูล
**ระยะเวลาประมาณการ:** 3 วัน
**ลำดับความสำคัญ:** สูง
**กฎการตรวจสอบ (Validation Rules):**
- เมื่อเสร็จสิ้น ให้ตรวจสอบ connection strings ใน config
- อัปเดตเอกสารการ deployment

### Task 1.2: การตั้งค่า Message Queue
**คำอธิบาย:** ปรับแต่ง Kafka สำหรับ event streaming
**เกณฑ์การยอมรับ:**
- [ ] ตั้งค่า Kafka cluster เสร็จสิ้น
- [ ] ใช้งานกลยุทธ์การแบ่ง topic partitioning
- [ ] ปรับแต่ง dead letter queue
**ระยะเวลาประมาณการ:** 2 วัน
**ลำดับความสำคัญ:** สูง

## Epic 2: บริการรวบรวมกิจกรรม (Activity Collection Service)
**ระยะเวลาประมาณการ:** 1.5 สัปดาห์

### Task 2.1: API รวบรวมกิจกรรม (Activity Collector API)
**คำอธิบาย:** สร้าง REST API สำหรับรับข้อมูล activity events
**เกณฑ์การยอมรับ:**
- [ ] ทำ POST /activities endpoint เสร็จสิ้น
- [ ] ตรวจสอบและทำความสะอาด input data
- [ ] ปรับแต่งการจำกัดอัตราการเรียกใช้ (rate limiting)
**ระยะเวลาประมาณการ:** 4 วัน
**ลำดับความสำคัญ:** สูง
**กฎการตรวจสอบ:**
- เมื่อ API เปลี่ยนแปลง ให้อัปเดตเอกสาร API
- เมื่อเพิ่ม endpoint ใหม่ ให้ทำ security review

### Task 2.2: การเชื่อมต่อกับ Web Application (Web Application Integration)
**คำอธิบาย:** เชื่อมต่อกับระบบ web application ที่มีอยู่
**เกณฑ์การยอมรับ:**
- [ ] ทำ authentication middleware เสร็จสิ้น
- [ ] ดึงข้อมูล user context ได้ถูกต้อง
- [ ] รักษาความเข้ากันได้แบบย้อนหลัง (backward compatibility)
**ระยะเวลาประมาณการ:** 3 วัน
**ลำดับความสำคัญ:** สูง

## Epic 3: ระบบประมวลผลข้อมูล (Data Processing Pipeline)
**ระยะเวลาประมาณการ:** 1 สัปดาห์

### Task 3.1: การประมวลผล Event (Event Processing)
**คำอธิบาย:** ประมวลผล incoming events และเก็บในฐานข้อมูล
**กฎการตรวจสอบ:**
- เมื่อแก้ไข data schema ให้อัปเดต migration scripts
- เมื่อเปลี่ยน processing logic ให้อัปเดต test cases

## ลำดับการทำงาน (Implementation Order)
1. การตั้งค่าฐานข้อมูล (รากฐาน)
2. Activity Collector API (ฟังก์ชันหลัก)
3. การเชื่อมต่อ Web Application (การเชื่อมต่อที่สำคัญ)
4. ระบบประมวลผลข้อมูล (ประสิทธิภาพ)
5. Dashboard API (ส่วนติดต่อผู้ใช้)
6. Analytics Engine (ฟีเจอร์ขั้นสูง)

## เส้นทางวิกฤต (Critical Path)
การตั้งค่าฐานข้อมูล → Activity Collector → การเชื่อมต่อ Web Application → Dashboard
```

---

## Phase 4: Implementation Example

### Input Prompt:
```
[ใช้ Implementation Prompt]

ทำ Task 2.1: Activity Collector API

รายละเอียด Task:
- สร้าง REST API สำหรับรับข้อมูล activity events
- ใช้งานการตรวจสอบและทำความสะอาด input data
- ปรับแต่งการจำกัดอัตราการเรียกใช้

### คำแนะนำ:
**ใช้ภาษาไทยในการอธิบายขั้นตอนและการตรวจสอบ** แต่ใช้ภาษาอังกฤษในโค้ดและ technical terms ตามมาตรฐาน
```

### Expected Output:
```
## การตรวจสอบก่อนเริ่มงาน (Pre-Implementation Check)
- [x] ทบทวน requirements สำหรับการรวบรวม activity แล้ว
- [x] ตรวจสอบ design specifications สำหรับ API endpoints แล้ว
- [x] ยืนยันการออกแบบ database schema แล้ว
- [x] ยืนยันข้อกำหนดด้านความปลอดภัยแล้ว

## ขั้นตอนการทำงาน (Implementation Steps)

### 1. สร้าง Test Cases ก่อน (TDD Approach)
```javascript
// tests/activity-collector.test.js
describe('Activity Collector API', () => {
  test('ควรยอมรับ activity event ที่ถูกต้อง', async () => {
    const validEvent = {
      userId: 'EMP001',
      action: 'document_view',
      resource: 'DOC123',
      timestamp: new Date()
    };
    
    const response = await request(app)
      .post('/api/v1/activities')
      .send(validEvent)
      .expect(201);
      
    expect(response.body.status).toBe('success');
  });
  
  test('ควรปฏิเสธ activity event ที่ไม่ถูกต้อง', async () => {
    const invalidEvent = {
      action: 'document_view'
      // ข้อมูลที่จำเป็นหายไป
    };
    
    await request(app)
      .post('/api/v1/activities')
      .send(invalidEvent)
      .expect(400);
  });
});
```

### 2. การพัฒนา API (API Implementation)
```javascript
// routes/activities.js
const express = require('express');
const rateLimit = require('express-rate-limit');
const { body, validationResult } = require('express-validator');
const ActivityService = require('../services/ActivityService');

const router = express.Router();

// Rate limiting: 100 requests per minute per user
const activityRateLimit = rateLimit({
  windowMs: 60 * 1000,
  max: 100,
  keyGenerator: (req) => req.user?.id || req.ip
});

// Validation middleware
const validateActivity = [
  body('userId').isString().notEmpty(),
  body('action').isIn(['login', 'logout', 'document_view', 'document_edit', 'approval']),
  body('resource').optional().isString(),
  body('timestamp').isISO8601()
];

router.post('/activities', 
  activityRateLimit,
  validateActivity,
  async (req, res) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return res.status(400).json({
          status: 'error',
          errors: errors.array()
        });
      }

      const activity = await ActivityService.create({
        ...req.body,
        sessionId: req.sessionID,
        ipAddress: req.ip,
        userAgent: req.get('User-Agent')
      });

      res.status(201).json({
        status: 'success',
        data: { id: activity._id }
      });
    } catch (error) {
      res.status(500).json({
        status: 'error',
        message: 'Internal server error'
      });
    }
  }
);

module.exports = router;
```

### 3. Service Layer
```javascript
// services/ActivityService.js
const Activity = require('../models/Activity');
const crypto = require('crypto');

class ActivityService {
  static async create(activityData) {
    // Sanitize PII data
    const sanitizedData = {
      ...activityData,
      ipAddress: this.hashIP(activityData.ipAddress),
      userAgent: this.sanitizeUserAgent(activityData.userAgent)
    };

    return await Activity.create(sanitizedData);
  }

  static hashIP(ip) {
    return crypto.createHash('sha256').update(ip).digest('hex');
  }

  static sanitizeUserAgent(userAgent) {
    // Remove potentially sensitive information
    return userAgent.replace(/\(.*?\)/g, '(sanitized)');
  }
}

module.exports = ActivityService;
```

## Post-Implementation Validation

### Validation Checks Completed:
- [x] API endpoints documented in OpenAPI spec
- [x] Rate limiting tested with load testing
- [x] Input validation covers all edge cases
- [x] Error handling follows project standards
- [x] Security review completed
- [x] Performance benchmarks met

### Documentation Updates:
- [x] Updated API documentation
- [x] Added deployment notes
- [x] Updated security guidelines

### Validation Hooks Triggered:
- API documentation auto-updated ✅
- Security scan passed ✅
- Integration tests updated ✅

## Next Task Ready: Task 2.2 - Web Application Integration
Dependencies satisfied, ready to proceed.
```

---

## Phase 5: Quality Assurance Example

### Input Prompt:
```
[ใช้ Validation & QA Prompt]

ตรวจสอบความสอดคล้องของ Activity Collector API ที่เพิ่งทำเสร็จ
```

### Expected Output:
```
# QA Report: Activity Collector API

## Document Consistency Check

### ✅ Requirements vs Implementation
- [x] All required endpoints implemented
- [x] Rate limiting matches requirements (100 req/min)
- [x] Input validation covers all specified fields
- [x] Error handling meets requirements

### ✅ Design vs Code
- [x] API follows RESTful design principles
- [x] Database interactions match schema design
- [x] Security measures implemented as designed

### ⚠️ Issues Found:
1. **API Documentation**: Missing response examples for error cases
   - **Fix**: Add error response examples to OpenAPI spec
   - **Priority**: Medium

2. **Test Coverage**: Integration tests missing for rate limiting
   - **Fix**: Add rate limiting tests
   - **Priority**: High

## Code Quality Check

### ✅ Passed:
- Code follows project standards
- Error handling comprehensive
- Security best practices applied
- Performance optimization in place

### ⚠️ Recommendations:
1. Add API response caching for frequently accessed data
2. Implement circuit breaker pattern for external dependencies
3. Add more detailed logging for debugging

## Testing Validation

### Coverage Report:
- Unit Tests: 95% ✅
- Integration Tests: 85% ⚠️ (Target: 90%)
- End-to-End Tests: 80% ⚠️ (Target: 85%)

### Missing Test Cases:
- [ ] Rate limiting edge cases
- [ ] Concurrent request handling
- [ ] Database connection failure scenarios

## Documentation Quality

### ✅ Complete and Accurate:
- API endpoints documented
- Security requirements documented
- Deployment instructions updated

### 🔧 Needs Update:
- Error code reference
- Troubleshooting guide
- Performance tuning guide

## Updated Validation Hooks

Added new validation rules:
- When modifying rate limiting → Update load testing scenarios
- When changing error responses → Update client integration guides
- When adding new validation rules → Update API documentation

## Recommendations for Next Phase:
1. Fix integration test coverage before proceeding
2. Complete API documentation
3. Prepare monitoring dashboards for production deployment
```

---

## 📈 ประโยชน์ของการใช้ Kiro-Style Approach

### 1. **ความเป็นระเบียบ (Organization)**
- ทุกขั้นตอนมีเอกสารรองรับเป็นภาษาไทยที่เข้าใจง่าย
- ไม่มีการเขียนโค้ดโดยไม่มีแผนหรือเหตุผล
- ตรวจสอบความสอดคล้องได้ตลอดเวลาด้วยภาษาที่เข้าใจ

### 2. **ลดความเสี่ยง (Risk Reduction)**
- Validation hooks ป้องกันการลืมอัปเดตเอกสารภาษาไทย
- การทำงานเป็นขั้นตอนลดโอกาสผิดพลาดจากการสื่อสารไม่เข้าใจ
- มีการ review และ validation ในทุกขั้นตอนเป็นภาษาไทย

### 3. **ง่ายต่อการบำรุงรักษา (Maintainability)**
- เอกสารและโค้ดสอดคล้องกันเสมอ พร้อมคำอธิบายภาษาไทย
- มี test cases ครอบคลุมพร้อมคำอธิบายที่เข้าใจง่าย
- การเปลี่ยนแปลงมีการควบคุมและอธิบายเป็นภาษาไทย

### 4. **เหมาะสำหรับทีมงานไทย (Thai Team Optimization)**
- ทุกคนเข้าใจขั้นตอนการทำงานเป็นภาษาไทย
- มีมาตรฐานเดียวกันในการใช้ภาษาไทยร่วมกับเทคนิค
- Knowledge transfer ทำได้ง่ายด้วยเอกสารภาษาไทย
- ลดความเข้าใจผิดจากการแปลหรือใช้ภาษาอังกฤษเท่านั้น

---

## 🎯 แนวทางการใช้ภาษาในโปรเจ็ค

### ภาษาไทย ใช้สำหรับ:
- **การอธิบายและเหตุผล** ในทุกเอกสาร
- **คำอธิบาย tasks และ acceptance criteria** 
- **การสื่อสารความต้องการและข้อกำหนด**
- **การอธิบายขั้นตอนและกระบวนการ**
- **Error messages และ user-facing content**
- **Comment ในโค้ดที่อธิบายตรรกะทางธุรกิจ**

### ภาษาอังกฤษ ใช้สำหรับ:
- **โค้ดและตัวแปร** (variable names, function names)
- **ชื่อเทคโนโลยีและเครื่องมือ** (MongoDB, React, API endpoints)
- **คำศัพท์เทคนิค** ที่เป็นมาตรฐานในวงการ IT
- **Configuration files และ technical specifications**

### ตัวอย่างการผสมผสาน:
```javascript
// ฟังก์ชันสำหรับการเข้ารหัสข้อมูลผู้ใช้
function encryptUserData(userData) {
    // ใช้ AES-256 เพื่อเข้ารหัสข้อมูลส่วนตัว
    return crypto.encrypt(userData, ENCRYPTION_KEY);
}

// Model สำหรับเก็บข้อมูลกิจกรรมของผู้ใช้
const UserActivity = {
    userId: String,          // รหัสผู้ใช้
    actionType: String,      // ประเภทการกระทำ
    timestamp: Date,         // เวลาที่ทำกิจกรรม
    description: String      // คำอธิบายกิจกรรม
};
```

---

*ใช้แนวทางนี้เป็นมาตรฐานในการพัฒนาโปรเจ็ค Activity-Log แบบ Kiro IDE โดยเน้นการใช้ภาษาไทยเป็นหลักในการสื่อสารและอธิบาย เพื่อให้ทีมงานไทยเข้าใจและทำงานได้อย่างมีประสิทธิภาพสูงสุด*
