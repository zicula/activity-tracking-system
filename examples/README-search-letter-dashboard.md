# Search Letter Activity Dashboard Example

## Overview
ตัวอย่าง Dashboard สำหรับแสดงกิจกรรมการค้นหาหนังสือในระบบ EDCS (Electronic Document Control System) ที่สร้างขึ้นจากข้อมูล log file จริง

## Files ในโฟลเดอร์นี้

### 1. `search-letter-example-dashboard.md`
- Dashboard แบบ Markdown ที่แสดงข้อมูลแบบ static
- มีการวิเคราะห์ log file อย่างละเอียด
- แสดงข้อมูล user session, search criteria, และ performance metrics

### 2. `search-letter-dashboard.html`
- Interactive HTML Dashboard
- Real-time updates ด้วย JavaScript
- Responsive design สำหรับ desktop และ mobile
- Modern UI/UX ด้วย CSS3 animations

### 3. `search-letter-activity-data.json`
- ข้อมูล JSON ที่ extract มาจาก log file
- โครงสร้างข้อมูลที่ครบถ้วนสำหรับการพัฒนา API
- รองรับการ integration กับระบบ monitoring

## การใช้งาน

### เปิดดู HTML Dashboard
```bash
# เปิดไฟล์ใน browser
open search-letter-dashboard.html

# หรือใช้ local server
python3 -m http.server 8000
# แล้วไปที่ http://localhost:8000/search-letter-dashboard.html
```

### ใช้ข้อมูล JSON
```javascript
// อ่านข้อมูลจาก JSON file
fetch('./search-letter-activity-data.json')
  .then(response => response.json())
  .then(data => {
    console.log('Search Activity:', data.searchLetterActivity);
    // ใช้ข้อมูลเพื่อสร้าง charts หรือ widgets
  });
```

## Features ของ Dashboard

### 📊 Real-time Metrics
- จำนวนผู้ใช้ที่ active
- การค้นหาในวันนี้
- เอกสารที่พบ
- Response time
- Memory usage
- Error rate

### 👤 User Information
- ข้อมูลผู้ใช้ปัจจุบัน
- Session details
- Access permissions
- Role-based information

### 🔍 Search Activity Details
- เกณฑ์การค้นหาที่ใช้
- ข้อมูล JSF form interaction
- AJAX request details
- Technical parameters

### 📅 Activity Timeline
- กิจกรรมล่าสุด
- User interactions
- System events
- Chronological order

### ⚡ Performance Analysis
- Response time breakdown
- Network latency
- Server processing time
- Database query estimation

### 🔐 Security Information
- Session security
- CSRF protection
- Access control
- Security recommendations

## ข้อมูลจาก Log File ตัวอย่าง

### Request Information
```
Method: POST
URL: /edcs/pages/edcs/app/process/search-letter/search-letter.xhtml
Content-Type: application/x-www-form-urlencoded
Content-Length: 1055 bytes
Remote Address: 0:0:0:0:0:0:0:1 (IPv6 Localhost)
```

### User Session
```
Session ID: E6A3F57C1E1D3DF53182003476A10999
User: sirat (นางสาวศรีรัตน์ โกมลสวัสดิ์)
Department: ฝ่ายสารบรรณ
Position: หัวหน้าฝ่ายสารบรรณ (อาวุโส)
```

### Search Criteria
```
ประเภทการลงทะเบียน: R (ลงทะเบียน)
หนังสือ Electronic: Y (ใช่)
ประเภทหนังสือ: 1
รหัสสมุด: 16375
วันที่: 22/07/2568 - 23/07/2568
เลขที่ลงทะเบียน: 233
เรื่อง: ทดสอบ
ผู้ส่ง: ทดสอบ
```

## การปรับแต่ง Dashboard

### เปลี่ยนสี Theme
```css
/* ใน search-letter-dashboard.html */
:root {
  --primary-color: #3498db;
  --secondary-color: #9b59b6;
  --success-color: #27ae60;
  --warning-color: #f39c12;
  --danger-color: #e74c3c;
}
```

### เพิ่ม Metrics ใหม่
```javascript
// เพิ่ม metric card ใหม่
const newMetric = {
  value: '95.5%',
  label: 'System Uptime',
  icon: '🟢',
  status: 'excellent'
};
```

### การ Integration กับ API
```javascript
// เชื่อมต่อกับ real-time API
const updateDashboard = async () => {
  try {
    const response = await fetch('/api/search-activity/current');
    const data = await response.json();
    updateMetrics(data);
  } catch (error) {
    console.error('Failed to update dashboard:', error);
  }
};

setInterval(updateDashboard, 5000); // Update ทุก 5 วินาที
```

## การพัฒนาต่อ

### 1. Real-time Data Connection
- เชื่อมต่อกับ WebSocket สำหรับ real-time updates
- Integration กับ log streaming systems
- Database connection สำหรับ historical data

### 2. Advanced Analytics
- Chart.js หรือ D3.js สำหรับ data visualization
- Time-series analysis
- Predictive analytics

### 3. Alert System
- Email notifications
- Slack integration
- SMS alerts สำหรับ critical events

### 4. Export Features
- PDF reports
- Excel export
- CSV data download

## Security Considerations

### Data Privacy
- Mask sensitive information (IP addresses, personal data)
- GDPR/PDPA compliance
- Role-based access control

### Authentication
- SSO integration
- Multi-factor authentication
- Session management

### Audit Trail
- User activity logging
- Change tracking
- Compliance reporting

## Browser Support
- Chrome 80+
- Firefox 75+
- Safari 13+
- Edge 80+

## Performance
- Optimized for 100+ concurrent users
- Lazy loading for large datasets
- Efficient DOM updates
- Minimal memory footprint

---

**Created:** July 23, 2025  
**Version:** 1.0  
**Author:** Activity Tracking System Team  
**License:** Internal Use Only
