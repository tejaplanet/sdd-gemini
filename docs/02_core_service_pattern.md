# 02. Core Service Pattern & Business Logic

เอกสารนี้กำหนดมาตรฐาน (Anatomy) ของการเขียน Business Logic สำหรับรับ Message จาก RabbitMQ มาประมวลผล 
**AI ต้องใช้โครงสร้างและปฏิบัติตามกฎ 8 ขั้นตอนในเอกสารนี้เสมอเมื่อมีการสร้างไฟล์ Service ใหม่**

## 1. The `SequenceService` Anatomy
ไฟล์ Use Case ทุกตัว (เช่น `CreateNewCustomerService.go`) ต้องเกาะอยู่กับ Struct `SequenceService` และมี Function Signature ที่ตายตัวดังนี้:

```go
func (s SequenceService) [SERVICE_NAME]Service(ctx rabbitmq.Context, info order.Info) (stepStatus string, results []orderflow.Result, payload string, err error) {
    // ... logic ...
}
```

## 2. ขั้นตอนการทำงานมาตรฐาน 8 Steps (Golden Pattern)
ในการเขียนเนื้อหาภายในฟังก์ชัน ให้ปฏิบัติตามลำดับ 8 ขั้นตอนนี้อย่างเคร่งครัด:

### Step 1: Init Variables
ประกาศตัวแปร `isWorked` และ `payloadStruct` ให้พร้อมใช้งาน
```go
	var (
		isWorked      = false
		payloadStruct payloadModel.Struct
	)
```

### Step 2: Init ZFlow Logger & Panic Handler (🚨 Mandatory)
ห้ามใช้ `log.Println` เด็ดขาด! ต้องประกาศ ZFlow Logger ทันทีและดัก Panic เสมอ:
```go
	logs := logger.InitZFlow(
		s,
		&info,
		logInfo, // ถ้ามี
		&payloadStruct,
		logger.BackendInfo{
			Endpoint:    AppConfig["[CONFIG_KEY]"], // ดึงจาก cons.go หรือ ETCD
			BackendName: "[SYSTEM_CODE].[CONFIG_KEY]",
		},
	)
	defer logs.PanicHandler()
```

### Step 3: Parse Payload
แกะข้อมูลจาก `info.Payload` มาใส่ใน `payloadStruct` โดยต้องจัดการ Error ผ่าน Logger:
```go
	err = json.Unmarshal([]byte(info.Payload), &payloadStruct)
	if err != nil {
		return logs.FailedRequest(errors.New(err.Error())).ResultSequence()
	}
```

### Step 4: Pre-Execution Check & Skip Logic
ต้องตรวจสอบเงื่อนไขจาก `StepData.PreExecCheckFM` เพื่อข้าม Operation ที่ไม่เข้าเงื่อนไข หรือเคยทำสำเร็จไปแล้ว:
```go
	var ok bool
	var preExecCheckResult []interface{}
	if info.StepData.PreExecCheckFM != "" {
		preExecCheckResult, ok = validator.ValidateStruct(payloadStruct, info.StepData.PreExecCheckFM).([]interface{})
		if !ok {
			preExecCheckResult = make([]interface{}, 0)
		}
	}

	targetExtIDs, err := preexeccheck.GetTargetExtID(preExecCheckResult)
	if err != nil {
		return logs.FailedRequest(errors.New(err.Error())).ResultSequence()
	}
	prevOperationIDs := preexeccheck.GetPrevOperationID(info.StepData.Result)
```

### Step 5: Business Logic Loop & API Call
วนลูปข้อมูลที่ต้องการประมวลผล, เช็ค Skip Logic, Map Request, และยิง API:
```go
	// สมมติว่าเป้าหมายคือ payloadStruct.Order.Customer
	if payloadStruct.Order != nil && payloadStruct.Order.Customer != nil {
		for i, item := range *payloadStruct.Order.Customer {
			logs.OperationID = pointy.StringValue(item.ExtID, "")
			
			// Skip Logic
			if info.StepData.PreExecCheckFM != "" && !preexeccheck.Contain(targetExtIDs, logs.OperationID) {
				continue
			}
			if preexeccheck.Contain(prevOperationIDs, logs.OperationID) {
				continue
			}

			logs.BackendExtID = uuid.New().String()
			
			// 1. Map Request (อ้างอิงจาก docs/04_payload_dictionary.md)
			reqBody, err := build[SERVICE_NAME]Request(item, payloadStruct.Order)
			if err != nil {
				return logs.FailedRequest(err).ResultSequence()
			}
			logs.InfoRequest(reqBody).ResultInfo()
			reqBytes, _ := json.Marshal(reqBody)

			// 2. HTTP Call (🚨 ห้ามใช้ net/http เด็ดขาด)
			headers := map[string]string{
				"Content-Type":  "application/json; charset=UTF-8",
				"Authorization": "Bearer " + AppConfig["authorization"],
			}
			resBytes, httpRes, err := request.CallWithETCDConfigTimeoutHttpStatus(ctx.KVS, info.QueueName, "POST", logs.Endpoint, reqBytes, headers)
			
			// 3. Handle API Error
			if err != nil {
				return logs.FailedResponse("400", errors.New(err.Error())).ResultSequence()
			}
			if httpRes.StatusCode != 200 {
				return logs.FailedResponse("400", errors.New(string(resBytes))).ResultSequence()
			}

			// ... (เข้าสู่ Step 6)
```

### Step 6: Map Response & Update State
เมื่อยิง API สำเร็จ ให้เซ็ตผลลัพธ์ลง Payload ขากลับ และเก็บ Log Success
```go
			// 4. Map Response
			var response interfaces.[SERVICE_NAME]Response
			if err = logs.SetResponseResult(resBytes, &response); err != nil {
				return logs.FailedResponse("400", errors.New(err.Error())).ResultSequence()
			}
			
			build[SERVICE_NAME]Response(&item, response) // อัปเดตค่ากลับเข้า Payload

			// 5. Success Logging
			logs.SetNewPayload(payloadStruct)
			logs.InfoResponse(response).ResultInfo()
			logs.AddResultComplete()

			isWorked = true
		}
	}
```

### Step 7: Return Sequence
จบการทำงานของฟังก์ชันด้วยการ Return ผ่าน Logger เสมอ ห้ามใช้ `return stepStatus, results, payload, err` ตรงๆ:
```go
	return logs.ReturnSequence(isWorked)
}
```
### Step 8: Backend Data Mapping Standards (Request & Response) 🔄

**🚨 สำคัญ: แหล่งอ้างอิง Model (Model Source of Truth)**
โครงสร้างข้อมูล (Struct) ทั้งหมดที่ใช้ในส่วนนี้ **ต้องยึดตามรายละเอียดใน `docs/06_backend_model_dictionary.md` เป็นหลัก** โค้ดที่แสดงในหัวข้อนี้คือ **ตัวอย่าง (Example Only)** เพื่อแสดงโครงสร้างการเขียน (Pattern) เท่านั้น AI ต้องอ่าน Model จากไฟล์ 06 เพื่อนำมาปรับใช้กับฟิลด์จริงของแต่ละ Service และ field mapping ให้ดูจาก `spec.md`ในส่วนของ Request Mapping และ Response Mapping


#### 8.1 การทำ Map Request (`build...Request`)
ฟังก์ชันนี้มีหน้าที่แปลงข้อมูลจาก **Internal Payload** ไปเป็น **Backend Struct** ของระบบปลายทาง:
- **Model Reference:** อ้างอิงโครงสร้างจาก `interfaces.[BACKEND_REQ_STRUCT]` ในไฟล์ 06
- **Implementation Rule:** AI ต้องเลือก Map ฟิลด์ตามชื่อฟิลด์ที่ปรากฏจริงใน Model ของ Backend และใช้กฎการแปลงข้อมูล (Transform) ตามที่ระบุใน `spec.md`

**[ตัวอย่างแนวทางการเขียน - Template Example]**
```go
func build[SERVICE_NAME]Request(cust payloadModel.Customer, ord *payloadModel.Order) (*interfaces.[BACKEND_REQ_STRUCT], error) {
    // 💡 AI NOTE: ให้ปรับเปลี่ยนฟิลด์ด้านล่างนี้ตาม interfaces.[BACKEND_REQ_STRUCT] ใน 06_backend_model_dictionary.md
    
    var activityDate string
    // ... (Logic การเตรียมข้อมูล เช่น Time Parsing)

    // Mapping ข้อมูลเข้าสู่ Struct จริงจาก 06
    customerList := interfaces.CustomerList{
        // รายละเอียดฟิลด์ต้องตรงตาม Model ในไฟล์ 06
        BillCycleNo: cust.BillCycleNo,
        ActivityInfo: &interfaces.ActivityInfo{
            ActivityDate:   pointy.String(activityDate),
            ActivityReason: pointy.String("CREQ"),
        },
    }

    // Mapping ส่วนอื่นๆ (Name, Address) ตามจริง
    // ...
    
    return &interfaces.[BACKEND_REQ_STRUCT]{
        CustomerList: &[]interfaces.CustomerList{customerList},
    }, nil
}
```

#### 8.2 การทำ Map Response (`build...Response`)
ฟังก์ชันนี้มีหน้าที่รับผลลัพธ์จาก Backend และอัปเดตกลับเข้าสู่ Payload ของระบบเรา:
- **Model Reference:** อ้างอิงโครงสร้างจาก `interfaces.[BACKEND_RES_STRUCT]` ในไฟล์ 06
- **Implementation Rule:** AI ต้องตรวจสอบความมีอยู่ของข้อมูล (Nil Check) และอัปเดตฟิลด์ให้ถูกต้องตามสัญญา (Contract) ที่ระบุไว้ใน `spec.md`

**[ตัวอย่างแนวทางการเขียน - Template Example]**
```go
func build[SERVICE_NAME]Response(cust *payloadModel.Customer, res interfaces.[BACKEND_RES_STRUCT]) {
    // 💡 AI NOTE: ให้ปรับเปลี่ยน Logic การดึงข้อมูลตามโครงสร้างใน 06_backend_model_dictionary.md
    
    if res.Results != nil && len(*res.Results) > 0 {
        result := (*res.Results)[0]
        if result.Data != nil {
            // อัปเดตค่ากลับเข้าสู่ Payload ตามกฎใน spec.md
            cust.CustomerID = result.Data.CustomerNo
            // ...
        }
    }
}
```

## 3. กฎเหล็กของการทำ Error Handling (Strict Rule)
- **ห้าม** Return Standard Error (`return "", nil, "", err`) ให้หลุดออกไปจากฟังก์ชัน
- หากพังก่อนยิง API (เช่น แกะ JSON ผิด, เตรียม Request ผิด) ให้ใช้:
  👉 `return logs.FailedRequest(err).ResultSequence()`
- หากพังตอนยิง API หรือหลังยิง API (เช่น HTTP Code ไม่ใช่ 2XX, Backend พ่น Error, แกะ Response ผิด) ให้ใช้:
  👉 `return logs.FailedResponse("400", err).ResultSequence()`
