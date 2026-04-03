# 06. Backend Model Dictionary (BDH / interfaces)

เอกสารนี้เก็บโครงสร้างข้อมูล (Struct) ของระบบ Backend BDH เพื่อใช้เป็นปลายทาง (Destination) ในการทำ Data Mapping

**🚨 คำสั่งสำหรับ AI Agent (Mandatory Directives):**
1. ทุกครั้งที่มีการสร้างฟังก์ชัน `build...Request` หรือ `build...Response` **คุณต้องอ้างอิงชื่อฟิลด์และ Data Type จากเอกสารนี้เท่านั้น**
2. **ห้าม (MUST NOT)** เดาชื่อฟิลด์เอง หากไม่มีในนี้ให้แจ้ง Developer ทันที
3. **Package Reference:** อ้างอิงตาม `gitlab.com/ft25/iom/fm/model/bdh/customer_service`

---

## 1. Request Models 📩

### L9CreateNewCustomerRequest
```go
type L9CreateNewCustomerRequest struct {
    CustomerList *[]CustomerList `json:"customerList,omitempty"`
}

type CustomerList struct {
    ActivityInfo           *ActivityInfo      `json:"activityInfo,omitempty"`
    AddressInfoList        []*AddressInfoList `json:"addressInfoList,omitempty"`
    CustomerNo             *int               `json:"customerNo,omitempty"`
    BillCycleNo            *int               `json:"billCycleNo,omitempty"`
    BirthDate              *string            `json:"birthDate,omitempty"`
    CustomerType           *string            `json:"customerType,omitempty"`
    Identification         *string            `json:"identification,omitempty"`
    IdentificationType     *string            `json:"identificationType,omitempty"`
    InitTimeInAdd          *string            `json:"initTimeInAdd,omitempty"`
    NameInfoList           []*NameInfoList    `json:"nameInfoList,omitempty"`
    Occupation             *string            `json:"occupation,omitempty"`
    OpenDate               *string            `json:"openDate,omitempty"`
}
```

### Request Components
```go
type AddressInfoList struct {
    AddressInfo *AddressInfo `json:"addressInfo,omitempty"`
}

type AddressInfo struct {
    HouseNo            *string `json:"houseNo,omitempty"`
    Soi                *string `json:"soi,omitempty"`
    StreetName         *string `json:"streetName,omitempty"`
    Tumbon             *string `json:"tumbon,omitempty"`
    Amphur             *string `json:"amphur,omitempty"`
    City               *string `json:"city,omitempty"`
    Zip                *string `json:"zip,omitempty"`
    Country            *string `json:"country,omitempty"`
    TypeOfAccommodation *string `json:"typeOfAccommodation,omitempty"` // Note: corrected from original 'typeOfAccomodation'
}

type NameInfoList struct {
    NameInfo *NameInfo `json:"nameInfo,omitempty"`
}

type NameInfo struct {
    NameType      *string `json:"nameType,omitempty"`
    Title         *string `json:"title,omitempty"`
    FirstName     *string `json:"firstName,omitempty"`
    LastName      *string `json:"lastName,omitempty"`
    MaritalStatus *string `json:"maritalStatus,omitempty"`
}

type ActivityInfo struct {
    ActivityDate   *string `json:"activityDate,omitempty"`
    ActivityReason *string `json:"activityReason,omitempty"`
}
```

---

## 2. Response Models 📤

### L9CreateNewCustomerResponse
```go
type L9CreateNewCustomerResponse struct {
    Results *[]Result `json:"results,omitempty"`
}

type Result struct {
    Data       *Data   `json:"data,omitempty"`
    Code       *int    `json:"code,omitempty"`
    CustomerNo *int    `json:"customerNo,omitempty"`
    Message    *string `json:"message,omitempty"`
    Status     *string `json:"status,omitempty"`
}

type Data struct {
    CustomerNo               *int                      `json:"customerNo,omitempty"`
    CustomerBillingCycleInfo *CustomerBillingCycleInfo `json:"customerBillingCycleInfo,omitempty"`
}

type CustomerBillingCycleInfo struct {
    BillCycleNo *int    `json:"billCycleNo,omitempty"`
    BillCycleDay *int   `json:"billCycleDay,omitempty"`
    Status       *string `json:"status,omitempty"`
}
```

> **หมายเหตุ:** เอกสารนี้ยังครอบคลุมเฉพาะ API `L9CreateNewCustomer` เท่านั้น หากมี API เพิ่มเติม (เช่น UpdateCustomer, DeleteCustomer) ให้เพิ่ม Request/Response Model ในรูปแบบเดียวกัน
