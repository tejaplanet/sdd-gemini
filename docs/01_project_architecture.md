# 01. Project Architecture & Global Rules

เอกสารนี้กำหนดภาพรวมโครงสร้างของโปรเจกต์ (FM Project Structure) ที่ใช้ในระบบ Order Management ของทีม iOM โดยอยู่ใน Module ชื่อ FM Service ที่ย่อมาจาก Fulfillment Management ซึ่งเป็นส่วนหนึ่งของระบบ Order Management ที่ทำหน้าที่ Fulfillment ข้อมูลของลูกค้า ในการทำ order

**AI Agent ต้องอ่านและทำความเข้าใจโครงสร้างนี้ก่อนเริ่มทำงานใดๆ**

## 1. Directory Structure (โครงสร้างโฟลเดอร์)
โปรเจกต์นี้ใช้โครงสร้างมาตรฐานสำหรับ Microservice โดยแบ่งแยกหน้าที่การทำงาน (Separation of Concerns) อย่างชัดเจน:

```text
.
├── go.mod
├── main.go
├── service/
│   ├── service.go
│   ├── [SERVICE_NAME]Service.go
│   └── cons.go
├── docs/
├── tests/
│   ├──testutils/
│	│	└── testutils.go
│   ├── root_test.go
│   └── [SERVICE_NAME]Service_test.go
└── version/
    └── version.go
```

- `go.mod` / `go.sum` :(Mandatory Dependency File) ไฟล์จัดการ Dependency ของโปรเจกต์ AI ต้องสร้างไฟล์นี้เป็นไฟล์แรกเมื่อเริ่มต้นโปรเจกต์ โดยให้เปลี่ยน `[MODULE_NAME]` เป็นชื่อโปรเจกต์ที่ถูกต้องเสมอ และ **ห้ามตัด block `replace` ด้านล่างสุดทิ้งเด็ดขาด**
	- Template `go.mod`
		```go
		module [MODULE_NAME] // ตัวอย่าง: sdd-bdh-customer-service-001 หรือตามที่ระบุใน Requirement
		go 1.22.2 // ต้องใช้ Go 1.22.2 ขึ้นไปเสมอ
		require (
			[github.com/go-errors/errors](https://github.com/go-errors/errors) v1.1.1
			[github.com/google/uuid](https://github.com/google/uuid) v1.4.0
			[github.com/openlyinc/pointy](https://github.com/openlyinc/pointy) v1.1.2
			[gitlab.com/ft25/iom/fm/model](https://gitlab.com/ft25/iom/fm/model) v0.0.9
			[gitlab.com/ft25/iom/fm/utils](https://gitlab.com/ft25/iom/fm/utils) v0.0.0-00010101000000-000000000000
			[gitlab.com/ft25/iom/framework](https://gitlab.com/ft25/iom/framework) v1.0.0
			[gitlab.com/ft25/iom/model](https://gitlab.com/ft25/iom/model) v1.0.0
		)
		// หมายเหตุสำหรับ AI: ให้รันคำสั่ง `go mod tidy` เพื่อจัดการ require indirect ที่เหลือโดยอัตโนมัติ
		replace (
			[gitlab.com/ft25/iom/fm/model](https://gitlab.com/ft25/iom/fm/model) => [gitlab.com/ft25/iom/prod/da/fm/model.git](https://gitlab.com/ft25/iom/prod/da/fm/model.git) latest
			[gitlab.com/ft25/iom/fm/utils](https://gitlab.com/ft25/iom/fm/utils) => [gitlab.com/ft25/iom/prod/da/fm/utils.git](https://gitlab.com/ft25/iom/prod/da/fm/utils.git) latest
			[gitlab.com/ft25/iom/framework](https://gitlab.com/ft25/iom/framework) => [gitlab.com/ft25/iom/prod/da/framework.git](https://gitlab.com/ft25/iom/prod/da/framework.git) latest
			[gitlab.com/ft25/iom/model](https://gitlab.com/ft25/iom/model) => [gitlab.com/ft25/iom/prod/da/model.git](https://gitlab.com/ft25/iom/prod/da/model.git) latest
		)
		```
- `main.go` : จุดเริ่มต้นของโปรเจกต์ (Entrypoint)
	- Template `main.go`
		```go
		package main

		import (
			"[MODULE_NAME]/version"    // TODO: เปลี่ยน [MODULE_NAME] ตามไฟล์ go.mod
			"[gitlab.com/ft25/iom/framework/service](https://gitlab.com/ft25/iom/framework/service)"

			fmService "[MODULE_NAME]/service" // TODO: เปลี่ยน [MODULE_NAME] ตามไฟล์ go.mod
		)

		func main() {
			forever := make(chan bool)

			ctx, closeConnection, err := service.InitConnection()
			if err != nil {
				panic(err)
			}
			defer closeConnection()

			err = fmService.InitAppConf(ctx)
			if err != nil {
				panic(err)
			}

			ctx.Logger.SetVersion(version.Number)

			// TODO: ตัวแปร BackendService อาจเปลี่ยนชื่อตามระบบเป้าหมาย (เช่น BdhBackendService, CvgBackendService) 
			// ให้อ้างอิงชื่อตัวแปรตามที่ประกาศไว้ใน service/cons.go ของโปรเจกต์นั้นๆ
			err = fmService.BdhBackendService.NewService(ctx.KVS)
			if err != nil {
				panic(err)
			}
			defer fmService.BdhBackendService.Destroy()

			// TODO: พารามิเตอร์ที่ 2 ใน service.Load คือ [SYSTEM_CODE] ให้อ้างอิงจากที่ตกลงกันไว้ (เช่น "bdh", "cvg")
			err = service.Load(ctx, "[SYSTEM_CODE]", []interface{}{fmService.SequenceService{}})
			if err != nil {
				panic(err)
			}

			service.Healthcheck()
			<-forever
		}
		```


- `service/` : **(Core Business Logic)** เก็บไฟล์ Use Case / Service ทั้งหมดที่รับ Request จาก RabbitMQ (เช่น `[SERVICE_NAME]Service.go`) และไฟล์ `cons.go` สำหรับตั้งค่าตัวแปรระดับระบบ
- `service/service.go`
	- Template `service.go`
		```go
		package service

		import (
			"[gitlab.com/ft25/iom/framework/service](https://gitlab.com/ft25/iom/framework/service)"
		)

		// SequenceService ใช้เป็น Receiver สำหรับผูกฟังก์ชัน Use Case ทั้งหมดของ Microservice นี้
		type SequenceService struct {
			service.SequenceService
		}
		```
- `service/cons.go` (Configuration & Constants) ไฟล์ศูนย์กลางสำหรับการตั้งค่าระบบ, โหลด ETCD Config, และฟังก์ชัน Transform (LOV) AI ต้องสร้างไฟล์นี้ในโฟลเดอร์ `service/` และต้องเปลี่ยน `[MODULE_NAME]`, `[APP_NAME]`, `[OPERATION_SYSTEM]`, และ `[SYSTEM_CODE]` ให้ตรงกับ Requirement ของระบบเป้าหมายเสมอ
	- Template `cons.go`
		```go
		package service

		import (
			"os"
			// [OPTIONAL_REDIS_IMPORT_START : Include only if Redis/Backend service is required]
			"[MODULE_NAME]/pkg/redis"
			// [OPTIONAL_REDIS_IMPORT_END]
			"[MODULE_NAME]/version"

			"[gitlab.com/ft25/iom/fm/utils/logger](https://gitlab.com/ft25/iom/fm/utils/logger)"
			"[gitlab.com/ft25/iom/fm/utils/lov](https://gitlab.com/ft25/iom/fm/utils/lov)"
			"[gitlab.com/ft25/iom/framework/etcd](https://gitlab.com/ft25/iom/framework/etcd)"
			"[gitlab.com/ft25/iom/framework/rabbitmq](https://gitlab.com/ft25/iom/framework/rabbitmq)"
		)

		var (
			// [OPTIONAL_BACKEND_SERVICE_START : Use this if require redis in this project]
			// TODO: ตัวแปร BackendService อาจเปลี่ยนชื่อตามระบบเป้าหมาย (เช่น CvgBackendService)
			BdhBackendService redis.Service = &redis.BdhService{}
			// [OPTIONAL_BACKEND_SERVICE_END]

			logInfo                         = &logger.TransactionInfo{
				Version:         version.Number,
				AppName:         "[APP_NAME]",         // ตัวอย่าง: "fm-bdh"
				OperationSystem: "[OPERATION_SYSTEM]", // ตัวอย่าง: "BDH"
				EnvName:         os.Getenv("ENV"),
			}

			AppConfig   = map[string]string{}
			InitAppConf = func(ctx rabbitmq.Context) (err error) {
				// TODO: เพิ่มหรือลด Path ETCD ตาม Requirement ของฟีเจอร์ที่ต้องทำ
				path := []etcd.ETCDPath{
					{
						Path: "fm/[SYSTEM_CODE]/customerService/authorization",
						Key:  "authorization",
					},
					{
						Path: "fm/[SYSTEM_CODE]/customerService/createCustomer",
						Key:  "createCustomer",
					},
				}
				if AppConfig, err = etcd.GetETCD(ctx.KVS, path); err != nil {
					return err
				}
				return nil
			}

			//BKService service
			//BKService backend.Service = &backend.CCBSService{}
		)

		// [OPTIONAL_LOV_START : Use this if require redis in this project]
		// TODO: เพิ่ม Transform Functions (LOV) ตามความจำเป็นของ Business Logic
		var TransformTitle = func(title string) string {
			return BdhBackendService.ServiceInfo().LovData.Title(title)[lov.CCBS.Sys()]
		}

		var TransformOccupation = func(occcupation string) string {
			return BdhBackendService.ServiceInfo().LovData.Occupation(occcupation)[lov.CCBS.Sys()]
		}

		var TransformMarry = func(marry string) string {
			return BdhBackendService.ServiceInfo().LovData.Marry(marry)[lov.CCBS.Sys()]
		}

		var TransformAccommodation = func(accommodation string) string {
			return BdhBackendService.ServiceInfo().LovData.Accommodation(accommodation)[lov.CCBS.Sys()]
		}
		// [OPTIONAL_LOV_END]
		```

- `docs/` : **(AI Context & Specifications)** เก็บเอกสารคู่มือ, Data Dictionary (`payload_model.md`), และกฎเกณฑ์ต่างๆ เพื่อใช้เป็น Context สำหรับ AI Vibe Coding
- `tests/` : **(Quality Assurance)** เก็บไฟล์ Unit Test และ Mock data ทั้งหมด
- `tests/root_test.go` : Test utils, Test configuration
- `tests/testutils/testutils.go` : Test utils, Test configuration
- `version/version.go` : **(Versioning)** เก็บไฟล์ที่ระบุ Semantic Version ของโปรเจกต์ (เช่น `version.go`) โดยใช้ตัวแปร GitLab CI (`{{CI_COMMIT_REF_SLUG}}` = ชื่อ branch, `{{CI_COMMIT_SHA}}` = commit hash) เพื่อ inject version ตอน build
	- Template `version.go`
		```go
		package version

		const Number = "{{CI_COMMIT_REF_SLUG}}-{{CI_COMMIT_SHA}}"
		```



## 2. Dependency & Framework Rules (กฎการจัดการ `go.mod`)
โปรเจกต์นี้มีการใช้งาน Internal/Private Library ของทีมเป็นหลัก **AI ต้องปฏิบัติตามกฎเหล่านี้อย่างเคร่งครัดเมื่อจัดการ Dependency:**

- **Go Version:** โปรเจกต์นี้ใช้ Go version **1.22.2** หรือใหม่กว่าเสมอ
- **❌ ห้ามใช้ External Libraries พร่ำเพรื่อ:** ไม่อนุญาตให้ใช้ Library ภายนอกในการจัดการ HTTP Request, Message Queue (RabbitMQ), หรือ Logging ให้ใช้ Framework ของทีมแทน
- **🚨 กฎการใช้ Private Dependency (`replace` directive):**
  เนื่องจาก Framework และ Model ของทีมถูกเก็บอยู่ใน Private Repository หาก AI ต้องแก้ไขหรือสร้างไฟล์ `go.mod` ใหม่ **ห้ามลืมใส่ Block `replace` ด้านล่างนี้เด็ดขาด** มิฉะนั้นโปรเจกต์จะ Build ไม่ผ่าน:
  ```go
  replace (
      [gitlab.com/ft25/iom/fm/model](https://gitlab.com/ft25/iom/fm/model) => [gitlab.com/ft25/iom/prod/da/fm/model.git](https://gitlab.com/ft25/iom/prod/da/fm/model.git) latest
      [gitlab.com/ft25/iom/fm/utils](https://gitlab.com/ft25/iom/fm/utils) => [gitlab.com/ft25/iom/prod/da/fm/utils.git](https://gitlab.com/ft25/iom/prod/da/fm/utils.git) latest
      [gitlab.com/ft25/iom/framework](https://gitlab.com/ft25/iom/framework) => [gitlab.com/ft25/iom/prod/da/framework.git](https://gitlab.com/ft25/iom/prod/da/framework.git) latest
      [gitlab.com/ft25/iom/model](https://gitlab.com/ft25/iom/model) => [gitlab.com/ft25/iom/prod/da/model.git](https://gitlab.com/ft25/iom/prod/da/model.git) latest
  )
  ```
- **การ Import ในโค้ด:** แม้ใน `go.mod` จะใช้ `replace` ไปยัง `.git latest` แต่เวลาใช้งานในโค้ด Go (เช่น ในไฟล์ `.go`) **ให้ Import ด้วยชื่อโมดูลดั้งเดิมเสมอ** เช่น:
  - `import "gitlab.com/ft25/iom/framework/service"`
  - `import "gitlab.com/ft25/iom/fm/model/payload"`

## 3. Application Entrypoint (`main.go`)
เมื่อมีการสร้างหรือแก้ไขไฟล์ `main.go` **AI ต้องปฏิบัติตาม Pattern มาตรฐานด้านล่างนี้ 100%** **🚨 กฎการ Import ใน `main.go`:**
ห้าม Hardcode Path ของ Package ภายในโปรเจกต์เด็ดขาด! ให้ AI ไปอ่านชื่อ Module จากบรรทัดแรกของไฟล์ `go.mod` ก่อนเสมอ (สมมติให้ชื่อว่า `[MODULE_NAME]`) และนำมาจัดรูปแบบ Import ดังนี้:
- `"[MODULE_NAME]/version"`
- `fmService "[MODULE_NAME]/service"`

### Template `main.go` มาตรฐาน
```go
package main

import (
	"[MODULE_NAME]/version"    // เปลี่ยน [MODULE_NAME] ตามไฟล์ go.mod
	"[gitlab.com/ft25/iom/framework/service](https://gitlab.com/ft25/iom/framework/service)"

	fmService "[MODULE_NAME]/service" // เปลี่ยน [MODULE_NAME] ตามไฟล์ go.mod
)

func main() {
	forever := make(chan bool)

	ctx, closeConnection, err := service.InitConnection()
	if err != nil {
		panic(err)
	}
	defer closeConnection()

	err = fmService.InitAppConf(ctx)
	if err != nil {
		panic(err)
	}

	ctx.Logger.SetVersion(version.Number)

	// ตัวแปร BdhBackendService อาจเปลี่ยนชื่อตามระบบเป้าหมาย (เช่น CvgBackendService) 
	// ให้อ้างอิงตามที่ประกาศไว้ใน service/cons.go
	err = fmService.BdhBackendService.NewService(ctx.KVS)
	if err != nil {
		panic(err)
	}
	defer fmService.BdhBackendService.Destroy()

	// พารามิเตอร์ "bdh" ใน service.Load คือ SystemCode ให้อ้างอิงจากที่ตกลงกันไว้
	err = service.Load(ctx, "bdh", []interface{}{fmService.SequenceService{}})
	if err != nil {
		panic(err)
	}

	service.Healthcheck()
	<-forever
}
```

## 4. System Identity (ป้ายชื่อระบบ)
ในการระบุตัวตนของ Microservice สำหรับระบบ Tracing และ Logging ให้ยึดค่าต่อไปนี้ (มักจะตั้งค่าใน `service/cons.go`):
- **SystemCode:** (เช่น `"bdh"`, `"cvg"`) ใช้ตอน Register service ใน `main.go`
- **AppName / SystemName:** ชื่อเต็มของโปรเจกต์ (เช่น `"fm-bdh-customer-service"`)
- **OperationSystem:** ชื่อระบบใหญ่ (เช่น `"BDH"`)
