# Overview

โครงสร้างพื้นฐานการใช้งาน EGAT APIs มี 2 ข้อได้แก่

- สนับสนุนเฉพาะ HTTP __GET__  เท่านั้น
- กรองข้อมูลผ่าน Query Parameters โดยใช้รูปแบบเดียวกับ [PostgREST](https://postgrest.org/en/stable/references/api/tables_views.html)

## API Endpoints

__Base URL__: http://172.17.113.142:8080

| Dataset           | Endpoint     |
|-------------------|--------------|
| __Substations__   | /sub_station |
| __Line Feeders__  | /line_feeder |
| __Billing__       | /billing     |
| __Load Profiles__ | /loadprofile |

ในส่วนของข้อมูล Load Profile แนะนำให้ Filter เป็นรายเดือน หรือ ตามความต้องการใช้งาน เนื่องจากข้อมูลมีปริมาณมาก (~ 1GB ณ วันที่ 29 สิงหาคม 2566)

## API Authentication

เพิ่ม Request Header ด้วย Key-Value ดังนี้

| Key           | Value                                                                                                                    |
|---------------|--------------------------------------------------------------------------------------------------------------------------|
| Authorization | Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9<br>.eyJyb2xlIjoibWVhX2FwcCJ9<br>.YQeO5hci1qAlSCKQXgGje7WBMj1CXBJtlXO_EEeTdtY |

## Supported Operations

### Read

อ่านข้อมูลทั้งหมดที่มีในระบบ __ไม่แนะนำสำหรับ Load Profile Endpoint__ เนื่องจากข้อมูลมีปริมาณมาก

#### Substations

```bash
GET /sub_station
```

Response:
```javascript
{
  "code": "STB",
  "name": "สถานีไฟฟ้าแรงสูง ธนบุรีใต้",
  "type_code": "MEA"
}
```

#### Billing

```bash
GET /billing
```

Response:
```javascript
{
  "substation_code": "SSUT-F2",
  "line_feeder_name": "SSUT-F2/115 MEA#1 M",
  "fisc_period": "2021-02",
  "record_type": "C1",
  "a_plus_t1": 0,
  "a_plus_t1_status": "0",
  "a_plus_t2": 0,
  "a_plus_t2_status": "0",
  "a_minus_t1": 19883170,
  "a_minus_t1_status": "0",
  "a_minus_t2": 26966200,
  "a_minus_t2_status": "0",
  "id": 1
}
```

#### cURL Example

```
curl -X GET -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoibWVhX2FwcCJ9.YQeO5hci1qAlSCKQXgGje7WBMj1CXBJtlXO_EEeTdtY" http://172.17.113.142:8080/sub_station
```

### Filtering

กรองข้อมูลเท่าที่จำเป็นใช้งานโดยมีโครงสร้างหลักดังนี้

```
http://172.17.113.142:8080/dataset?field_name=operator.value
```

สามารถตรวจสอบ Field Name ได้ที่หน้า Metadata ของแต่ละชุดข้อมูล และ รายการ Operator ที่สนับสนุนตามตารางด้านล่าง

#### Supported Operators

สำหรับ Advanced Operators สามารถดูได้ที่ [PostgREST](https://postgrest.org/en/stable/references/api/tables_views.html)

| Operator | Meaning               |
|----------|-----------------------|
| eq       | equals                |
| gt       | greater than          |
| gte      | greater than or equal |
| lt       | less than             |
| lte      | less than or equal    |
| neq      | not equal             |
| fts      | full text search      |
| and      | logical AND           |
| or       | logical OR            |

#### Bills by Feeder Name
ข้อมูลบิลเฉพาะ `line_feeder_name=BN/69 MEA#2 M` โดยชื่อของสายป้อนจะต้องอยู่ในรูปแบบของ URL Encoded String

```
GET /billing?line_feeder_name=eq.BN%2F69%20MEA%232%20M
```

#### Bills by Fiscal Period
ข้อมูลบิลเดือนกรกฎาคมปี 2023

```
GET /billing?fisc_period=eq.2023-07
```

#### Bills by Substation
ข้อมูลบิลแยกตามสถานีย่อย

```
/billing?substation_code=eq.SSUT-F2
```

#### Bills by Substation and Fiscal Period
ข้อมูลบิลของสถานีย่อย `SSUT-F2` เดือนกรกฎาคมปี 2023

```
/billing?substation_code=eq.SSUT-F2&fisc_period=eq.2023-07
```

#### Load Profile by Substation
ข้อมูล Load Profile ของทุกสายป้อนภายในสถานีย่อย (ข้อมูลโดยเฉลี่ยประมาณ 140MB ใช้เวลาดึงข้อมูลประมาณ 20 วินาที)

```
/loadprofile?substation_code=eq.SSUT-F2
```

#### Load Profile by Time Range
ข้อมูล Load Profile ของช่วงวันที่ต้องการ (ตั้งแต่วันที่ 2023-07-15 เที่ยงคืน จนถึงวันที่ 2023-07-20 23:30 น.)

```
/loadprofile?and=(time_local.gte.2023-07-15, time_local.lt.2023-07-21)
```

#### Load Profile by Line Feeder Name
ข้อมูล Load Profile ของสายป้อนที่ต้องการ (ถ้าไม่กำหนดวัน ระบบจะตอบกลับข้อมูลที่มีทั้งหมดในฐานข้อมูลของสายป้อนที่กำหนด) โดยชื่อของสายป้อนต้องอยู่ในรูปแบบ URL Encoded String

```
/loadprofile?line_feeder_name=eq.BK%2F230%20MEA%232%20M
```

#### Load Profile by Feeder and Time Range
ข้อมูล Load Profile ของสายป้อนที่ต้องการ ภายในระยะวันที่กำหนด

```
/loadprofile?and=(time_local.gte.2023-07-15, time_local.lt.2023-07-16)&line_feeder_name=eq.BK%2F230%20MEA%232%20M
```

### Full Text Search
ค้นหาข้อมูลที่มีคำที่ต้องการ ตัวอย่างนี้ค้นหาข้อมูล Load Profile ทั้งหมดที่ชื่อสายป้อนมีตัวหนังสือ `BK/230` อยู่ภายในชื่อ __โดยปกติการใช้งาน Full Text Search จะใช้เวลานานกว่าวิธีดึงข้อมูลอื่นๆ__

```
/loadprofile?line_feeder_name=fts.BK%2F230
```

### Projection
เลือก Field ใน Response ตามต้องการ ตัวอย่างนี้เลือก 2 คอลัมน์จาก 3 คอลัมน์ในชุดข้อมูล `line_feeder`

```
/line_feeder?select=line_feeder_name,meter_type
```

### Renaming Columns
เปลี่ยนชื่อคอลัมน์ตามต้องการ ตัวอย่างนี้เปลี่ยนชื่อจาก `line_feeder_name` เป็น `Feeder`, `meter_type` เป็น `Type` และ `type_code` เป็น `Code`

```
/line_feeder?select=Feeder:line_feeder_name,Type:meter_type,Code:type_code
```

### Limit and Pagination
จำกัดปริมาณข้อมูลที่เรียก และ การแบ่งหน้า โดยสามารถตรวจสอบจำนวน Records ได้ด้วยการเพิ่ม Request Header ด้วย Key-Value ตามตารางด้านล่าง

| Key      | Value         |
|----------|---------------|
| `Prefer` | `count=exact` |


```
/loadprofile?limit=10000&offset=0
```

### CSV Export
Export ข้อมูลออกในรูปแบบ CSV ทำได้โดยกำหนด Request Header ด้วย Key-Value ตามตารางด้านล่าง

| Key      | Value      |
|----------|------------|
| `Accept` | `text/csv` |


```
/billing
```