# 05. Testing Standards & Quality Assurance

เอกสารนี้กำหนดมาตรฐานการเขียน Unit Test สำหรับโปรเจกต์นี้ 
**AI ต้องปฏิบัติตามโครงสร้างและกฎเกณฑ์ในเอกสารนี้เสมอเมื่อได้รับคำสั่งให้เขียนหรือแก้ไขไฟล์ Test**

## 1. 🚨 กฎเหล็กสำหรับการเขียน Test (Mandatory Rules)
- **Framework:** ห้ามใช้ Library อื่นในการทำ Assertion นอกเหนือจาก `github.com/stretchr/testify/assert` และ `github.com/stretchr/testify/mock`
- **Location & Naming:** - ไฟล์ Test ต้องลงท้ายด้วย `_test.go` เสมอ (เช่น `CreateNewCustomerService_test.go`)
  - โค้ด Test ของ Business Logic ให้อยู่ใน `package service` (หรือ `package service_test`)
- **Pattern:** ต้องเขียน Test ในรูปแบบ **Table-Driven Tests** เสมอ เพื่อให้ง่ายต่อการเพิ่ม Test Case ใหม่ในอนาคต
- **No Real Connections:** ห้ามให้โค้ด Test มีการต่อ RabbitMQ, Redis, หรือ ETCD ของจริงเด็ดขาด ต้องทำการ Mock ตัวแปร `rabbitmq.Context`, `AppConfig` และฟังก์ชัน HTTP Call เสมอ

## 2. โครงสร้างมาตรฐาน (Table-Driven Test Anatomy)

AI ต้องใช้โครงสร้างต่อไปนี้เป็น Template เริ่มต้นในการเขียน Unit Test สำหรับฟังก์ชัน `SequenceService`:

```go
package service

import (
	"encoding/json"
	"errors"
	"testing"

	"[github.com/stretchr/testify/assert](https://github.com/stretchr/testify/assert)"
	"[gitlab.com/ft25/iom/model/framework/order](https://gitlab.com/ft25/iom/model/framework/order)"
	"[gitlab.com/ft25/iom/fm/model/payload](https://gitlab.com/ft25/iom/fm/model/payload)"
	"[gitlab.com/ft25/iom/framework/rabbitmq](https://gitlab.com/ft25/iom/framework/rabbitmq)"
)

func Test[SERVICE_NAME]Service(t *testing.T) {
	// 1. Setup Global Mocks (เช่น AppConfig)
	AppConfig = map[string]string{
		"[ConfigKey]":   "[http://mock-endpoint.com/api](http://mock-endpoint.com/api)",
		"authorization": "mock-token",
	}

	// 2. Define Test Cases (Table-Driven)
	tests := []struct {
		name           string
		mockPayload    string
		mockStepData   order.StepData
		setupMock      func() // สำหรับ Setup Mock HTTP หรือฟังก์ชันที่เกี่ยวข้อง
		expectedStatus string
		expectedError  bool
	}{
		{
			name: "Success: Should process payload and return completed",
			mockPayload: `{"Order": {"Customer": [{"extId": "123"}]}}`,
			mockStepData: order.StepData{PreExecCheckFM: ""},
			setupMock: func() {
				// TODO: Mock HTTP Request ให้ Return HTTP 200
			},
			expectedStatus: "completed", // หรือค่าที่ระบบคาดหวัง
			expectedError:  false,
		},
		{
			name: "Skip: Should skip process if ExtID is not in PreExecCheckFM target",
			mockPayload: `{"Order": {"Customer": [{"extId": "999"}]}}`,
			mockStepData: order.StepData{
                PreExecCheckFM: "...", 
                Result: "...", // ใส่ Mock Result เพื่อทดสอบ GetPrevOperationID
            },
			setupMock: func() {
				// ไม่ต้องยิง API
			},
			expectedStatus: "completed",
			expectedError:  false,
		},
		{
			name: "Error: Should fail request when payload is invalid JSON",
			mockPayload: `{"Order": Invalid JSON}`,
			mockStepData: order.StepData{},
			setupMock: func() {},
			expectedStatus: "failed",
			expectedError:  true,
		},
		{
			name: "Error: Should fail response when API returns HTTP 400",
			mockPayload: `{"Order": {"Customer": [{"extId": "123"}]}}`,
			mockStepData: order.StepData{},
			setupMock: func() {
				// TODO: Mock HTTP Request ให้ Return HTTP 400
			},
			expectedStatus: "failed",
			expectedError:  true,
		},
	}

	// 3. Execute Tests
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			// Arrange
			tt.setupMock()
			svc := SequenceService{}
			ctx := rabbitmq.Context{} // Mock Context
			info := order.Info{
				Payload:  tt.mockPayload,
				StepData: tt.mockStepData,
			}

			// Act
			status, _, _, err := svc.[SERVICE_NAME]Service(ctx, info)

			// Assert
			if tt.expectedError {
				assert.Error(t, err)
			} else {
				assert.NoError(t, err)
			}
			assert.Equal(t, tt.expectedStatus, status)
		})
	}
}
```

## 3. Test Cases Requirement (สิ่งที่ต้อง Test เสมอ)
ทุกครั้งที่ AI สร้าง Unit Test สำหรับ Service ใหม่ **จะต้องมี Test Cases อย่างน้อย 4 สถานการณ์นี้เสมอ**:

1. **Happy Path (Success Case):** ตรวจสอบว่าฟังก์ชันทำงานจนจบ Map ข้อมูลถูกต้อง ยิง API ผ่าน และคืนค่า Sequence กลับมาสมบูรณ์
2. **Skip Logic (Pre-Execution Validation):** ตรวจสอบว่าหากส่ง `extId` ที่ไม่ผ่านเงื่อนไข `PreExecCheckFM` หรือ `extId` ที่เคยทำสำเร็จไปแล้ว (`prevOperationIDs`) ระบบจะต้อง ข้าม (Continue) ลูปนั้นโดยไม่ยิง API และคืนค่าไม่เกิด Error
3. **Payload Parsing Error:** ตรวจสอบว่าระบบสามารถจัดการ Error และคืนค่า `logs.FailedRequest()` ได้ถูกต้องหาก Payload ขาเข้าเป็น JSON ที่พังหรือไม่ตรงตาม Struct
4. **API Failure (Error Handling):** ตรวจสอบว่าหาก API ปลายทางล่ม, Timeout, หรือตอบกลับเป็น HTTP Code ที่ไม่ใช่ 200 ระบบจะต้องคืนค่า `logs.FailedResponse()` อย่างถูกต้อง