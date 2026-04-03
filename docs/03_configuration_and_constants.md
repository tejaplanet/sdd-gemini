# 03. Configuration & Constants Management

เอกสารนี้กำหนดมาตรฐานการจัดการค่า Configuration, ค่าคงที่ (Constants), และการดึงข้อมูลจาก ETCD ภายในโปรเจกต์ 
**AI ต้องปฏิบัติตามกฎในเอกสารนี้ทุกครั้งที่มีการอ้างอิงถึง Endpoint, Token หรือตั้งค่าตัวแปรระดับระบบ**

## 1. ศูนย์กลางการตั้งค่า (`service/cons.go`)
โปรเจกต์นี้จะรวบรวมค่าคงที่และตัวแปร Global ที่ต้องใช้ข้ามไฟล์ไว้ที่ `service/cons.go` เสมอ โดยมีองค์ประกอบหลักดังนี้:
- `AppConfig`: ตัวแปรประเภท `map[string]string` ที่ทำหน้าที่เก็บค่า Configuration ทั้งหมดที่โหลดมาจาก ETCD
- `AppName` / `OperationSystem`: ชื่อระบบที่ใช้ในการแนบไปกับระบบ Logging (ZFlow)
- `[System]BackendService`: ตัวแปรโครงสร้าง `service.BackendService` สำหรับผูกกับ `main.go`

### ตัวอย่างโครงสร้างตัวแปรใน `cons.go`
```go
package service

import "[gitlab.com/ft25/iom/framework/service](https://gitlab.com/ft25/iom/framework/service)"

var (
	AppName         = "fm-bdh-customer-service" // ชื่อ Microservice
	OperationSystem = "BDH"                     // ระบบใหญ่ที่แอปนี้สังกัดอยู่
	AppConfig       = make(map[string]string)   // ที่เก็บ Config จาก ETCD
	
	// ชื่อตัวแปร BackendService อาจเปลี่ยนไปตามเป้าหมายของระบบ (เช่น CvgBackendService)
	BdhBackendService = &service.BackendService{
		AppName: AppName,
	}
)
```

## 2. กฎการจัดการ Config และ ETCD (🚨 Mandatory)
ห้าม Hardcode URL, Endpoint API, Timeout, หรือ Authentication Token ภายในโค้ด Business Logic เด็ดขาด! ทุกค่าต้องดึงผ่านตัวแปร `AppConfig` เท่านั้น

### 2.1 การเรียกใช้งานใน Use Case (`SequenceService`)
เมื่อต้องการระบุ URL ปลายทาง หรือ Token ให้เรียกผ่าน Key ที่กำหนดไว้ เช่น:
```go
// 1. การดึง Endpoint สำหรับ Logger BackendInfo
logger.BackendInfo{
    Endpoint:    AppConfig["createCustomer"],
    BackendName: "CVG",
}

// 2. การดึง Token สำหรับใส่ใน HTTP Header
headers := map[string]string{
    "Content-Type":  "application/json; charset=UTF-8",
    "Authorization": "Bearer " + AppConfig["authorization"],
}
```

### 2.2 การเพิ่ม Config ใหม่ (`InitAppConf`)
หาก Requirement มีการระบุให้เชื่อมต่อกับ API ใหม่ (เช่น ให้โหลด Config key: `updateProfile`) **AI จะต้องเข้าไปอัปเดตฟังก์ชัน `InitAppConf` ในไฟล์ `service/cons.go`** เพื่อสั่งให้ระบบไปดึงค่าจาก ETCD มาเก็บไว้ใน `AppConfig` เสมอ:

```go
func InitAppConf(ctx rabbitmq.Context) error {
	// ... (โค้ดดึง config เดิมที่มีอยู่แล้ว) ...
	
	// เพิ่มการดึงค่า Config ใหม่
	AppConfig["updateProfile"], _ = etcd.GetETCD(ctx.KVS, "[SYSTEM_CODE]/updateProfile")
	
	// ดึงค่าอื่นๆ ที่จำเป็น (เช่น authorization token ถ้ายังไม่มี)
	AppConfig["authorization"], _ = etcd.GetETCD(ctx.KVS, "[SYSTEM_CODE]/authorization")
	
	return nil
}
```
*(หมายเหตุ: `[SYSTEM_CODE]` จะเป็น Prefix ของระบบ เช่น `bdh` หรือ `cvg` ตามที่ระบุในโปรเจกต์)*

## 3. กฎเหล็กสำหรับ AI Vibe Coding
1. **Zero Hardcoding:** ห้ามเขียน String ที่เป็น URL ตรงๆ ลงในโค้ด (เช่น `"http://api.domain.com/v1/..."`) 
2. **Key Context:** เมื่อ AI อ่าน `spec.md` แล้วเจอประโยคว่า *"ใช้ Endpoint จาก ETCD ด้วย Key X"* AI ต้องแปลงผลว่านั่นคือการเรียกใช้โค้ด `AppConfig["X"]` โดยอัตโนมัติ
3. **Safe Initialization:** หากต้องเพิ่มตัวแปร Config ใหม่ อย่าลืมเช็คไฟล์ `cons.go` ว่ามีฟังก์ชัน `InitAppConf` โหลดค่านั้นขึ้นมาหรือยัง ถ้ายังให้แก้ไขเพิ่มเข้าไปด้วย