# 04. Payload Data Dictionary (`payloadModel`)

เอกสารนี้คือ **Single Source of Truth (พจนานุกรมข้อมูล)** สำหรับโครงสร้าง Payload ขาเข้าและขาออกของระบบ

## สารบัญ (Table of Contents)

| Struct | คำอธิบาย | บรรทัด |
|--------|---------|--------|
| `Struct` | Root payload struct | Master |
| `Order` | ข้อมูล Order หลัก | Master |
| `Customer` | ข้อมูลลูกค้า | Customer |
| `CustomerContactList` | รายการติดต่อลูกค้า | Customer |
| `CustomerGeneralInfo` | ข้อมูลทั่วไปลูกค้า (บัตร, อาชีพ, วันเกิด) | Customer |
| `CustomerTypeInfo` | ประเภทลูกค้า | Customer |
| `Account` | ข้อมูลบัญชี | Account |
| `Subscriber` | ข้อมูลผู้ใช้บริการ (เบอร์, เลขหมาย, อุปกรณ์) | Subscriber |
| `Offers` | ข้อมูลโปรโมชั่น/แพ็กเกจ | Offer |
| `Address` | ที่อยู่ (ใช้ร่วมกันทั้ง Customer, Account, Subscriber) | Shared |
| `NameInfo` | ชื่อ-นามสกุล (ใช้ร่วมกันทั้ง Customer, Account, Subscriber) | Shared |
| `ActivityInfo` | เหตุผลการดำเนินการ | Shared |
| `ConvergenceInfo` | ข้อมูล Convergence/รวมบริการ | Convergence |
| `OU` | Organizational Unit (หน่วยงาน) | Hierarchy |
| `Agreement` | สัญญาบริการ | Agreement |

## อภิธานศัพท์ย่อ (Glossary)

| คำย่อ | คำเต็ม |
|------|--------|
| **IOM** | Integrated Order Management - ระบบจัดการคำสั่งซื้อหลัก |
| **FM** | Fulfillment Management - ระบบจัดการ Fulfillment |
| **BDH** | Backend Data Handler - ระบบจัดการข้อมูลฝั่ง Backend |
| **CVG** | Convergence - ระบบรวมบริการ (Mobile+Fixed+TV) |
| **Melon** | ระบบ CRM ภายใน - ใช้อ้างอิงใน field เช่น `IOMMelonCustomerID` |
| **BAG** | Billing Account Group - กลุ่มบัญชีเรียกเก็บเงิน, ใช้อ้างอิงใน field เช่น `IOMBAGID` |
| **QRUN** | Queue Runner - ระบบจัดคิวงาน, ใช้อ้างอิงใน field เช่น `IOMQRUNCanFlag` |
| **TVS** | TrueVisions - ระบบ TV |
| **LOV** | List of Values - ตารางค่าอ้างอิงสำหรับ Transform |
| **OU** | Organizational Unit - หน่วยจัดการบริการ |
| **MNP** | Mobile Number Portability - ย้ายค่ายเบอร์มือถือ |

--- 

**🚨 คำสั่งสำหรับ AI Agent (Mandatory Directives):**
1. ทุกครั้งที่มีการเขียนฟังก์ชัน `build...Request` หรือ `build...Response` **คุณต้องค้นหาชื่อ Field และ Data Type จากเอกสารนี้เท่านั้น**
2. **ห้าม (MUST NOT)** สร้างตัวแปร, สร้าง Struct, หรือเดาชื่อ Field ขึ้นมาเองเด็ดขาด 
3. หาก Field ที่ต้องการ Map ไม่มีในเอกสารนี้ ให้หยุดทำงานแล้วแจ้ง Developer ทันที
4. **Package Reference ในโค้ด:** `gitlab.com/ft25/iom/fm/model/payload`

---

## Master Data Structures

```go
package payload

import (
	"[gitlab.com/ft25/iom/model/framework/timer](https://gitlab.com/ft25/iom/model/framework/timer)"
)

type Struct struct {
	ExtID *string `json:"extId,omitempty"`
	Order *Order  `json:"Order,omitempty"`
}

type Order struct {
	ExtID                       *string                        `json:"extId,omitempty"`
	CustOrderID                 *string                        `json:"custOrderId,omitempty"`
	Channel                     *string                        `json:"channel,omitempty"`
	RefTrackingID               *string                        `json:"refTrackingId,omitempty"`
	RefCustOrderID              *string                        `json:"refCustOrderId,omitempty"`
	RefFutureOrderID            *int                           `json:"refFutureOrderId,omitempty"`
	OrderType                   *int                           `json:"orderType,omitempty"`
	DealerCode                  *string                        `json:"dealerCode,omitempty"`
	DealerName                  *string                        `json:"dealerName,omitempty"`
	Media                       *string                        `json:"media,omitempty"`
	OldMedia                    *string                        `json:"oldMedia,omitempty"`
	CreateDate                  *timer.FormattedTime           `json:"createDate,omitempty"`
	OrderCaptureCode            *string                        `json:"orderCaptureCode,omitempty"`
	OrderCaptureName            *string                        `json:"orderCaptureName,omitempty"`
	OrderCaptureSaleGroup       *string                        `json:"orderCaptureSaleGroup,omitempty"`
	ChangeType                  *string                        `json:"changeType,omitempty"`
	SubType                     *string                        `json:"subType,omitempty"`
	SaleID                      *string                        `json:"saleId,omitempty"`
	SaleName                    *string                        `json:"saleName,omitempty"`
	SaleContact                 *string                        `json:"saleContact,omitempty"`
	SaleCode                    *string                        `json:"saleCode,omitempty"`
	EffectiveDate               *timer.FormattedTime           `json:"effectiveDate,omitempty"`
	CRMFlag                     *string                        `json:"crmFlag,omitempty"`
	SaleChannel                 *string                        `json:"saleChannel,omitempty"`
	MasterOrderID               *string                        `json:"masterOrderId,omitempty"`
	ConsolidateRequestType      *string                        `json:"consolidateRequestType,omitempty"`
	RequestName                 *string                        `json:"requestName,omitempty"`
	InstallationStatus          *string                        `json:"installationStatus,omitempty"`
	CurrentActivity             *string                        `json:"currentActivity,omitempty"`
	User                        *string                        `json:"user,omitempty"`
	Password                    *string                        `json:"password,omitempty"`
	SRNumber                    *string                        `json:"srNumber,omitempty"`
	IOMQrunCloseDate            *timer.FormattedTime           `json:"iomQrunCloseDate,omitempty"`
	TrackingID                  *string                        `json:"trackingId,omitempty"`
	IOMCustomerOrderID          *string                        `json:"iomCustomerOrderId,omitempty"`
	IOMActionName               *string                        `json:"iomActionName,omitempty"`
	IOMSourceType               *string                        `json:"iomSourceType,omitempty"`
	IOMQRUNCanFlag              *string                        `json:"iomQRUNCanFlag,omitempty"`
	IOMQRUNCanFlagFrom          *string                        `json:"iomQRUNCanFlagFrom,omitempty"`
	IOMQRUNCanFlagTo            *string                        `json:"iomQRUNCanFlagTo,omitempty"`
	IOMTVSCanFlag               *string                        `json:"iomTVSCanFlag,omitempty"`
	IOMImmediateFlag            *string                        `json:"iomImmediateFlag,omitempty"`
	IOMMFOrderType              *string                        `json:"iomMFOrderType,omitempty"`
	SaleEmail                   *string                        `json:"saleEmail,omitempty"`
	AsyncID                     *string                        `json:"asyncId,omitempty"`
	CrmGoldenId                 *string                        `json:"crmGoldenId,omitempty"`
	IOMCompanyCode              *string                        `json:"iomCompanyCode,omitempty"`
	OrderName                   *string                        `json:"orderName,omitempty"`
	ProjectName                 *string                        `json:"projectName,omitempty"`
	SendType                    *string                        `json:"sendType,omitempty"`
	SendFlag                    *string                        `json:"sendFlag,omitempty"`
	ConvergenceInfo             *ConvergenceInfo               `json:"ConvergenceInfo,omitempty"`
	ExtendedInfo                *map[string]interface{}        `json:"ExtendedInfo,omitempty"`
	Subcontractor               *Subcontractor                 `json:"Subcontractor,omitempty"`
	QrunNotification            *[]DeviceInfo                  `json:"QrunNotification,omitempty"`
	Customer                    *[]Customer                    `json:"Customer,omitempty"`
	ChangeInfoList              *[]ChangeInfoList              `json:"ChangeInfoList,omitempty"`
	MNPInfo                     *MNPInfo                       `json:"MNPInfo,omitempty"`
	SMSInfo                     *[]SMSInfo                     `json:"SMSInfo,omitempty"`
	EmailInfo                   *[]EmailInfo                   `json:"EmailInfo,omitempty"`
	OperatorID                  *string                        `json:"operatorId,omitempty"`
	EligibilityRuleCriteriaList *[]EligibilityRuleCriteriaList `json:"EligibilityRuleCriteriaList,omitempty"`
	EmployeeInfo                *EmployeeInfo                  `json:"EmployeeInfo,omitempty"`
	WebhookInfo                 *WebhookInfo                   `json:"WebhookInfo,omitempty"`
}

type ConvergenceInfo struct {
	ExtID                      *string                 `json:"extId,omitempty"`
	ConvergenceCode            *string                 `json:"convergenceCode,omitempty"`
	TrueLifeID                 *string                 `json:"trueLifeId,omitempty"`
	MobileFirstIndicator       *string                 `json:"mobileFirstIndicator,omitempty"`
	ConvergenceType            *string                 `json:"convergenceType,omitempty"`
	FamilyType                 *string                 `json:"familyType,omitempty"`
	ConvergenceGroupID         *string                 `json:"convergenceGroupId,omitempty"`
	IOMAllowCreate4PFamilyFlag *string                 `json:"iomAllowCreate4PFamilyFlag,omitempty"`
	IOM4PFamilyID              *string                 `json:"iom4PFamilyId,omitempty"`
	IOMOldConvergenceCode      *string                 `json:"iomOldConvergenceCode,omitempty"`
	CancelReasonCode           *string                 `json:"cancelReasonCode,omitempty"`
	ConsolidateRequestType     *string                 `json:"consolidateRequestType,omitempty"`
	SequenceNumber             *string                 `json:"sequenceNumber,omitempty"`
	ExtendedInfo               *map[string]interface{} `json:"ExtendedInfo,omitempty"`
	IomConvergenceCode         *string                 `json:"iomConvergenceCode,omitempty"`
	IomConvergenceType         *string                 `json:"iomConvergenceType,omitempty"`
	IomTrueLifeId              *string                 `json:"iomTrueLifeId,omitempty"`
	IomConvergenceGroupId      *string                 `json:"iomConvergenceGroupId,omitempty"`
}

type Customer struct {
	ExtID                *string                 `json:"extId,omitempty"`
	CustomerID           *int                    `json:"customerId,omitempty"`
	RefID                *string                 `json:"refId,omitempty"`
	BillCycleNo          *int                    `json:"billCycleNo,omitempty"`
	IOMCustomerAddressID *string                 `json:"iomCustomerAddressId,omitempty"`
	IOMCustomerID        *string                 `json:"iomCustomerId,omitempty"`
	IOMMelonCustomerID   *string                 `json:"iomMelonCustomerId,omitempty"`
	IOMBAGID             *string                 `json:"iomBAGId,omitempty"`
	CustomerAddress      *Address                `json:"CustomerAddress,omitempty"`
	CustomerName         *NameInfo               `json:"CustomerName,omitempty"`
	CustomerGeneralInfo  *CustomerGeneralInfo    `json:"CustomerGeneralInfo,omitempty"`
	CustomerTypeInfo     *CustomerTypeInfo       `json:"CustomerTypeInfo,omitempty"`
	CustomerContactList  *[]CustomerContactList  `json:"CustomerContactList,omitempty"`
	ExtendedInfo         *map[string]interface{} `json:"ExtendedInfo,omitempty"`
	ActivityInfo         *ActivityInfo           `json:"ActivityInfo,omitempty"`
	ProductList          *[]ProductList          `json:"ProductList,omitempty"`
}

type CustomerContactList struct {
	ExtID                *string                 `json:"extId,omitempty"`
	IOMCustomerContactID *string                 `json:"iomCustomerContactId,omitempty"`
	ContactType          *string                 `json:"contactType,omitempty"`
	Title                *string                 `json:"title,omitempty"`
	Name                 *string                 `json:"name,omitempty"`
	Lastname             *string                 `json:"lastname,omitempty"`
	MobilePhone          *string                 `json:"mobilePhone,omitempty"`
	WorkPhone            *string                 `json:"workPhone,omitempty"`
	FaxNumber            *string                 `json:"faxNumber,omitempty"`
	Email                *string                 `json:"email,omitempty"`
	HomePhone            *string                 `json:"homePhone,omitempty"`
	ExtendedInfo         *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type ActivityInfo struct {
	ExtID             *string                 `json:"extId,omitempty"`
	ActivityReason    *string                 `json:"activityReason,omitempty"`
	ActivitySubReason *string                 `json:"activitySubReason,omitempty"`
	UserText          *string                 `json:"userText,omitempty"`
	Remarks           *string                 `json:"remarks,omitempty"`
	ExtendedInfo      *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type CustomerGeneralInfo struct {
	ExtID                   *string                 `json:"extId,omitempty"`
	ContactLang             *string                 `json:"contactLang,omitempty"`
	Grading                 *string                 `json:"grading,omitempty"`
	Identification          *string                 `json:"identification,omitempty"`
	IdentificationEffDate   *timer.FormattedTime    `json:"identificationEffDate,omitempty"`
	IdentificationExpDate   *timer.FormattedTime    `json:"identificationExpDate,omitempty"`
	IdentificationType      *string                 `json:"identificationType,omitempty"`
	BirthDate               *timer.FormattedTime    `json:"birthDate,omitempty"`
	Occupation              *string                 `json:"occupation,omitempty"`
	TimeInBusiness          *string                 `json:"timeInBusiness,omitempty"`
	Salary                  *string                 `json:"salary,omitempty"`
	Nationality             *string                 `json:"nationality,omitempty"`
	InitTimeInAddress       *string                 `json:"initTimeInAddress,omitempty"`
	TimeInEmployee          *string                 `json:"timeInEmployee,omitempty"`
	TrueID                  *string                 `json:"trueId,omitempty"`
	TaxID                   *string                 `json:"taxId,omitempty"`
	BranchNumber            *string                 `json:"branchNumber,omitempty"`
	ContactChannel          *string                 `json:"contactChannel,omitempty"`
	OldIdentificationType   *string                 `json:"oldIdentificationType,omitempty"`
	OldIdentification       *string                 `json:"oldIdentification,omitempty"`
	OldCustName             *string                 `json:"oldCustName,omitempty"`
	VcareIdentificationType *string                 `json:"vcareIdentificationType,omitempty"`
	ExtendedInfo            *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type CustomerTypeInfo struct {
	ExtID           *string                 `json:"extId,omitempty"`
	CustomerSubType *string                 `json:"customerSubType,omitempty"`
	CustomerType    *string                 `json:"customerType,omitempty"`
	ExtendedInfo    *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type ProductList struct {
	ExtID        *string                 `json:"extId,omitempty"`
	ProductType  *string                 `json:"productType,omitempty"`
	ProductCode  *string                 `json:"productCode,omitempty"`
	IOMProdId    *int                    `json:"iomProdId,omitempty"`
	IOMProdCode  *string                 `json:"iomProdCode,omitempty"`
	OU           *[]OU                   `json:"OU,omitempty"`
	ExtendedInfo *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type EquipmentInfo struct {
	ExtID               *string                 `json:"extId,omitempty"`
	PointNo             *string                 `json:"pointNo,omitempty"`
	EquiptType          *string                 `json:"equiptType,omitempty"`
	SerialNo            *string                 `json:"serialNo,omitempty"`
	ReturnFlag          *string                 `json:"returnFlag,omitempty"`
	ModelType           *string                 `json:"modelType,omitempty"`
	ErrorCode           *string                 `json:"errorCode,omitempty"`
	ErrorDesc           *string                 `json:"errorDesc,omitempty"`
	AgreementDetailFrom *string                 `json:"agreementDetailFrom,omitempty"`
	AgreementDetailTo   *string                 `json:"agreementDetailTo,omitempty"`
	Material            *string                 `json:"material,omitempty"`
	OldPlant            *string                 `json:"oldPlant,omitempty"`
	OldLocation         *string                 `json:"oldLocation,omitempty"`
	ReturnMaterial      *string                 `json:"returnMaterial,omitempty"`
	NewPlant            *string                 `json:"newPlant,omitempty"`
	NewLocation         *string                 `json:"newLocation,omitempty"`
	SeqNo               *string                 `json:"seqNo,omitempty"`
	SrNo                *string                 `json:"srNo,omitempty"`
	ModelId             *string                 `json:"modelId,omitempty"`
	ModelName           *string                 `json:"modelName,omitempty"`
	SrStatus            *string                 `json:"srStatus,omitempty"`
	FromAgent           *string                 `json:"fromAgent,omitempty"`
	ToAgent             *string                 `json:"toAgent,omitempty"`
	HardwareGroup       *string                 `json:"hardwareGroup,omitempty"`
	AccountId           *string                 `json:"accountId,omitempty"`
	ChargeId            *string                 `json:"chargeId,omitempty"`
	ChargeAmount        *string                 `json:"chargeAmount,omitempty"`
	TaxInclude          *string                 `json:"taxInclude,omitempty"`
	ExtendedInfo        *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type TVSHardwareProduct struct {
	ExtID             *string                 `json:"extId,omitempty"`
	HardwareGroup     *string                 `json:"hardwareGroup,omitempty"`
	HardwareType      *string                 `json:"hardwareType,omitempty"`
	AgreementDetailID *string                 `json:"agreementDetailID,omitempty"`
	Point             *string                 `json:"point,omitempty"`
	ExtendedInfo      *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type AttributeList struct {
	ExtID        *string                 `json:"extId,omitempty"`
	Action       *string                 `json:"action,omitempty"`
	AttrType     *string                 `json:"attrType,omitempty"`
	AttrCode     *string                 `json:"attrCode,omitempty"`
	AttrName     *string                 `json:"attrName,omitempty"`
	AttrValue    *string                 `json:"attrValue,omitempty"`
	Description  *string                 `json:"description,omitempty"`
	ExtendedInfo *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type Account struct {
	ExtID                       *string                      `json:"extId,omitempty"`
	AccountID                   *int                         `json:"accountId,omitempty"`
	SequenceNumber              *int                         `json:"sequenceNumber,omitempty"`
	AgreementID                 *int                         `json:"agreementId,omitempty"`
	AccountRowID                *string                      `json:"accountRowId,omitempty"`
	IOMMelonAccountID           *string                      `json:"iomMelonAccountId,omitempty"`
	OURefID                     *string                      `json:"ouRefId,omitempty"`
	AgreementRefID              *string                      `json:"agreementRefId,omitempty"`
	BillingArrangementID        *string                      `json:"billingArrangementId,omitempty"`
	PayChannelCategory          *string                      `json:"payChannelCategory,omitempty"`
	PayChannelDescription       *string                      `json:"payChannelDescription,omitempty"`
	PayChannelID                *int                         `json:"payChannelId,omitempty"`
	RefID                       *string                      `json:"refId,omitempty"`
	BillCycleNo                 *string                      `json:"billCycleNo,omitempty"`
	AccountAddress              *Address                     `json:"AccountAddress,omitempty"`
	AccountName                 *NameInfo                    `json:"AccountName,omitempty"`
	AccountingManagementInfo    *AccountingManagementInfo    `json:"AccountingManagementInfo,omitempty"`
	BillingArrangementAddress   *Address                     `json:"BillingArrangementAddress,omitempty"`
	BillingArrangementName      *NameInfo                    `json:"BillingArrangementName,omitempty"`
	BillingArrangementBillInfo  *BillingArrangementBillInfo  `json:"BillingArrangementBillInfo,omitempty"`
	PayChannelPaymentMethodInfo *PayChannelPaymentMethodInfo `json:"PayChannelPaymentMethodInfo,omitempty"`
	ActivityInfo                *ActivityInfo                `json:"ActivityInfo,omitempty"`
	ExtendedInfo                *map[string]interface{}      `json:"ExtendedInfo,omitempty"`
	AccountCollectionFixInfo    *AccountCollectionFixInfo    `json:"AccountCollectionFixInfo,omitempty"`
	AlertToPayInfo              *AlertToPayInfo              `json:"AlertToPayInfo,omitempty"`
}

type AlertToPayInfo struct {
	ExtID           *string                 `json:"extId,omitempty"`
	TransactionType *string                 `json:"transactionType,omitempty"`
	AtpIDType       *string                 `json:"atpIdType,omitempty"`
	AtpIDValue      *string                 `json:"atpIdValue,omitempty"`
	BillerID        *string                 `json:"billerId,omitempty"`
	AtpBank         *string                 `json:"atpBank,omitempty"`
	AtpOwner        *string                 `json:"atpOwner,omitempty"`
	ExtendedInfo    *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type AccountCollectionFixInfo struct {
	ExtID                       *string                 `json:"extId,omitempty"`
	CollectionFixCsrCd          *string                 `json:"collectionFixCsrCd,omitempty"`
	CollectionPermanentWaiveInd *string                 `json:"collectionPermanentWaiveInd,omitempty"`
	CollectionFixPolicy         *string                 `json:"collectionFixPolicy,omitempty"`
	L9CollWaiverExpDate         *timer.FormattedTime    `json:"l9CollWaiverExpDate,omitempty"`
	ExtendedInfo                *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type AccountingManagementInfo struct {
	ExtID                    *string                 `json:"extId,omitempty"`
	CompanyCode              *string                 `json:"companyCode,omitempty"`
	CreditClass              *string                 `json:"creditClass,omitempty"`
	CreditLimitReasonCode    *string                 `json:"creditLimitReasonCode,omitempty"`
	CreditLimitWaiverExpDate *timer.FormattedTime    `json:"creditLimitWaiverExpDate,omitempty"`
	CreditLimitWaiverInd     *string                 `json:"creditLimitWaiverInd,omitempty"`
	PersonalCreditLimit      *float64                `json:"personalCreditLimit,omitempty"`
	WhtCertiNo               *string                 `json:"whtCertiNo,omitempty"`
	WhtInd                   *string                 `json:"whtInd,omitempty"`
	WHTEffectiveDate         *timer.FormattedTime    `json:"whtEffectiveDate,omitempty"`
	WHTExpireDate            *timer.FormattedTime    `json:"whtExpireDate,omitempty"`
	WhtTaxUpDate             *timer.FormattedTime    `json:"whtTaxUpDate,omitempty"`
	TaxID                    *string                 `json:"taxId,omitempty"`
	BranchNumber             *string                 `json:"branchNumber,omitempty"`
	AccountSubType           *string                 `json:"accountSubType,omitempty"`
	AccountCat               *string                 `json:"accountCat,omitempty"`
	AccountPriority          *string                 `json:"accountPriority,omitempty"`
	TempCreditLimit          *float64                `json:"tempCreditLimit,omitempty"`
	TempCreditLimitExpDate   *timer.FormattedTime    `json:"tempCreditLimitExpDate,omitempty"`
	PersonalClUpdDate        *timer.FormattedTime    `json:"personalClUpdDate,omitempty"`
	ConvergenceCode          *string                 `json:"convergenceCode,omitempty"`
	CharityCode              *string                 `json:"charityCode,omitempty"`
	IDDIndicator             *string                 `json:"iDDIndicator,omitempty"`
	IRIndicator              *string                 `json:"iRIndicator,omitempty"`
	InitiationReason         *string                 `json:"initiationReason,omitempty"`
	LegacyBan                *int64                  `json:"legacyBan,omitempty"`
	ManualBlacklistInd       *string                 `json:"manualBlacklistInd,omitempty"`
	ManualBlacklistRsnCd     *string                 `json:"manualBlacklistRsnCd,omitempty"`
	ManualBlacklistUpDate    *timer.FormattedTime    `json:"manualBlacklistUpDate,omitempty"`
	SpecialInstructions      *string                 `json:"specialInstructions,omitempty"`
	VcareAccountSubType      *string                 `json:"vcareAccountSubType,omitempty"`
	VcareAccountCat          *string                 `json:"vcareAccountCat,omitempty"`
	TaxCode                  *string                 `json:"taxCode,omitempty"`
	ColStatus                *string                 `json:"colStatus,omitempty"`
	WhtRate                  *int                    `json:"whtRate,omitempty"`
	ExtendedInfo             *map[string]interface{} `json:"ExtendedInfo,omitempty"`
	InterestWaiverInd        *string                 `json:"interestWaiverInd,omitempty"`
}

type BillingArrangementBillInfo struct {
	ExtID                 *string                 `json:"extId,omitempty"`
	BillFormat            *string                 `json:"billFormat,omitempty"`
	BillLanguage          *string                 `json:"billLanguage,omitempty"`
	PrintTemplateCategory *string                 `json:"printTemplateCategory,omitempty"`
	ConsolidateInd        *string                 `json:"consolidateInd,omitempty"`
	L9SplitParam          *string                 `json:"l9SplitParam,omitempty"`
	TaxSMS                *string                 `json:"taxSMS,omitempty"`
	TaxEmail              *string                 `json:"taxEmail,omitempty"`
	TaxContact            *string                 `json:"taxContact,omitempty"`
	TaxFormat             *string                 `json:"taxFormat,omitempty"`
	ShowUsageDetail       *string                 `json:"showUsageDetail,omitempty"`
	OptionImmediatly      *string                 `json:"optionImmediatly,omitempty"`
	ExtendedInfo          *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type PayChannelPaymentMethodInfo struct {
	ExtID                    *string                 `json:"extId,omitempty"`
	BankAccountNo            *string                 `json:"bankAccountNo,omitempty"`
	BankAccountType          *string                 `json:"bankAccountType,omitempty"`
	BankBranchNo             *string                 `json:"bankBranchNo,omitempty"`
	BankBranchName           *string                 `json:"bankBranchName,omitempty"`
	BankCode                 *string                 `json:"bankCode,omitempty"`
	BankName                 *string                 `json:"bankName,omitempty"`
	BranchCode               *string                 `json:"branchCode,omitempty"`
	CreditCardExpirationDate *timer.FormattedTime    `json:"creditCardExpirationDate,omitempty"`
	CreditCardNo             *string                 `json:"creditCardNo,omitempty"`
	CreditCardType           *string                 `json:"creditCardType,omitempty"`
	DdApprovalDate           *timer.FormattedTime    `json:"ddApprovalDate,omitempty"`
	IssueDate                *timer.FormattedTime    `json:"issueDate,omitempty"`
	ExpireDate               *timer.FormattedTime    `json:"expireDate,omitempty"`
	PaymentMethod            *string                 `json:"paymentMethod,omitempty"`
	PaymentName              *string                 `json:"paymentName,omitempty"`
	PaymentMeansOwnerDetails *string                 `json:"paymentMeansOwnerDetails,omitempty"`
	ExtendedInfo             *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type Address struct {
	ExtID              *string                 `json:"extId,omitempty"`
	Amphur             *string                 `json:"amphur,omitempty"`
	BuildingName       *string                 `json:"buildingName,omitempty"`
	City               *string                 `json:"city,omitempty"`
	Country            *string                 `json:"country,omitempty"`
	Floor              *string                 `json:"floor,omitempty"`
	HouseNo            *string                 `json:"houseNo,omitempty"`
	Moo                *string                 `json:"moo,omitempty"`
	RoomNo             *string                 `json:"roomNo,omitempty"`
	Soi                *string                 `json:"soi,omitempty"`
	SubSoi             *string                 `json:"subSoi,omitempty"`
	StreetName         *string                 `json:"streetName,omitempty"`
	TimeAtAddress      *string                 `json:"timeAtAddress,omitempty"`
	Tumbon             *string                 `json:"tumbon,omitempty"`
	TypeOfAccommodation *string                 `json:"typeOfAccommodation,omitempty"` // Note: JSON tag corrected from original 'typeOfAccomodation'
	Zip                *string                 `json:"zip,omitempty"`
	PoBox              *string                 `json:"poBox,omitempty"`
	MobilePhone        *string                 `json:"mobilePhone,omitempty"`
	WorkPhone          *string                 `json:"workPhone,omitempty"`
	Latitude           *string                 `json:"latitude,omitempty"`
	Longitude          *string                 `json:"longitude,omitempty"`
	LivingType         *string                 `json:"livingType,omitempty"`
	LivingStyle        *string                 `json:"livingStyle,omitempty"`
	ExtendedInfo       *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type OU struct {
	ExtID         *string                 `json:"extId,omitempty"`
	NumberOfIDD   *string                 `json:"numberOfIDD,omitempty"`
	NumberOfIR    *string                 `json:"numberOfIR,omitempty"`
	OuDescription *string                 `json:"ouDescription,omitempty"`
	OuID          *string                 `json:"ouId,omitempty"`
	OuName        *string                 `json:"ouName,omitempty"`
	RefID         *string                 `json:"refId,omitempty"`
	OU            *[]OU                   `json:"OU,omitempty"`
	ActivityInfo  *ActivityInfo           `json:"ActivityInfo,omitempty"`
	ExtendedInfo  *map[string]interface{} `json:"ExtendedInfo,omitempty"`
	Agreement     *Agreement              `json:"Agreement,omitempty"`
	Account       *Account                `json:"Account,omitempty"`
	Subscriber    *[]Subscriber           `json:"Subscriber,omitempty"`
}

type Agreement struct {
	ExtID                *string                 `json:"extId,omitempty"`
	RefID                *string                 `json:"refId,omitempty"`
	AgreementID          *int                    `json:"agreementId,omitempty"`
	AgreementType        *string                 `json:"agreementType,omitempty"`
	AgreementGeneralInfo *AgreementGeneralInfo   `json:"AgreementGeneralInfo,omitempty"`
	ContactInfo          *ContactInfo            `json:"ContactInfo,omitempty"`
	ActivityInfo         *ActivityInfo           `json:"ActivityInfo,omitempty"`
	ExtendedInfo         *map[string]interface{} `json:"ExtendedInfo,omitempty"`
	Offers               *[]Offers               `json:"Offers,omitempty"`
}

type AgreementGeneralInfo struct {
	ExtID                *string                 `json:"extId,omitempty"`
	AgreementDescription *string                 `json:"agreementDescription,omitempty"`
	ExtendedInfo         *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type ContactInfo struct {
	ExtID              *string                 `json:"extId,omitempty"`
	ContactCountryCode *string                 `json:"contactCountryCode,omitempty"`
	ContactExtensionNo *string                 `json:"contactExtensionNo,omitempty"`
	ContactFaxNo       *string                 `json:"contactFaxNo,omitempty"`
	ContactName        *string                 `json:"contactName,omitempty"`
	ContactRole        *string                 `json:"contactRole,omitempty"`
	ContactTelephoneNo *string                 `json:"contactTelephoneNo,omitempty"`
	Email              *string                 `json:"email,omitempty"`
	HomeTelephoneNo    *string                 `json:"homeTelephoneNo,omitempty"`
	MobileTelephoneNo1 *string                 `json:"mobileTelephoneNo1,omitempty"`
	MobileTelephoneNo2 *string                 `json:"mobileTelephoneNo2,omitempty"`
	WorkExtensionNo    *string                 `json:"workExtensionNo,omitempty"`
	WorkTelephoneNo    *string                 `json:"workTelephoneNo,omitempty"`
	ExtendedInfo       *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type Offers struct {
	ExtID                       *string                        `json:"extId,omitempty"`
	Action                      *string                        `json:"action,omitempty"`
	DeployMode                  *string                        `json:"deployMode,omitempty"`
	EffectiveDate               *timer.FormattedTime           `json:"effectiveDate,omitempty"`
	ExpirationDate              *timer.FormattedTime           `json:"expirationDate,omitempty"`
	OfferName                   *string                        `json:"offerName,omitempty"`
	OfferInstanceID             *int                           `json:"offerInstanceId,omitempty"`
	ParentOfferInstanceID       *int                           `json:"parentOfferInstanceID,omitempty"`
	EffectiveNextBillInd        *string                        `json:"effectiveNextBillInd,omitempty"`
	RequestProvisionInd         *string                        `json:"requestProvisionInd,omitempty"`
	ServiceType                 *string                        `json:"serviceType,omitempty"`
	Soc                         *string                        `json:"soc,omitempty"`
	SocProperties               *string                        `json:"socProperties,omitempty"`
	TargetPayChannelID          *int                           `json:"targetPayChannelId,omitempty"`
	Source                      *string                        `json:"source,omitempty"`
	OfferId                     *int                           `json:"offerId,omitempty"`
	OfferCode                   *string                        `json:"offerCode,omitempty"`
	OfferType                   *string                        `json:"offerType,omitempty"`
	Duration                    *int                           `json:"duration,omitempty"`
	DurationUOM                 *string                        `json:"durationUOM,omitempty"`
	IOMElementID                *string                        `json:"iomElementId,omitempty"`
	IOMOfferGroup               *string                        `json:"iomOfferGroup,omitempty"`
	OfferSubType                *string                        `json:"offerSubType,omitempty"`
	IOMRecurringChargeFee       *string                        `json:"iomRecurringChargeFee,omitempty"`
	IOMEffectiveType            *string                        `json:"iomEffectiveType,omitempty"`
	IOMBenefitType              *string                        `json:"iomBenefitType,omitempty"`
	IOMCircuitFlag              *string                        `json:"iomCircuitFlag,omitempty"`
	IOMProvisioningCode         *string                        `json:"iomProvisioningCode,omitempty"`
	IOMOfferDescription         *string                        `json:"iomOfferDescription,omitempty"`
	IOMOverrideOCAmount         *string                        `json:"iomOverrideOCAmount,omitempty"`
	IOMOverrideOCDesc           *string                        `json:"iomOverrideOCDesc,omitempty"`
	IOMOverrideRCAmount         *string                        `json:"iomOverrideRCAmount,omitempty"`
	IOMOverrideRCDesc           *string                        `json:"iomOverrideRCDesc,omitempty"`
	IOMCPCVersionID             *string                        `json:"iomCPCVersionId,omitempty"`
	IOMAssetCompRowID           *string                        `json:"iomAssetCompRowId,omitempty"`
	IOMParentAssetCompRowID     *string                        `json:"iomParentAssetCompRowId,omitempty"`
	IOMIntegrationID            *string                        `json:"iomIntegrationId,omitempty"`
	IOMParentIntegrationID      *string                        `json:"iomParentIntegrationId,omitempty"`
	CategoryCode                *string                        `json:"categoryCode,omitempty"`
	NewPeriodInd                *string                        `json:"newPeriodInd,omitempty"`
	Price                       *float64                       `json:"price,omitempty"`
	DiscountValue               *string                        `json:"discountValue,omitempty"`
	DiscountType                *string                        `json:"discountType,omitempty"`
	Quantity                    *int                           `json:"quantity,omitempty"`
	RateOverrideTP              *string                        `json:"rateOverrideTP,omitempty"`
	OverrideFixRate             *string                        `json:"overrideFixRate,omitempty"`
	ConvergenceInfo             *ConvergenceInfo               `json:"ConvergenceInfo,omitempty"`
	Offers                      *[]Offers                      `json:"Offers,omitempty"`
	ParameterInfo               *[]ParameterInfo               `json:"ParameterInfo,omitempty"`
	RelatedOffersArray          *[]RelatedOffersArray          `json:"RelatedOffersArray,omitempty"`
	ExtendedInfo                *map[string]interface{}        `json:"ExtendedInfo,omitempty"`
	ChargeTemplateList          *[]ChargeTemplateList          `json:"ChargeTemplateList,omitempty"`
	RelationList                *[]RelationList                `json:"RelationList,omitempty"`
	TariffPlanList              *[]TariffPlanList              `json:"TariffPlanList,omitempty"`
	SpeedInfo                   *SpeedInfo                     `json:"SpeedInfo,omitempty"`
	ServiceLevel                *string                        `json:"serviceLevel,omitempty"`
	RcIndicator                 *string                        `json:"rcIndicator,omitempty"`
	FutureType                  *string                        `json:"futureType,omitempty"`
	IomProvisioningFlag         *string                        `json:"iomProvisioningFlag,omitempty"`
	IomBillingFlag              *string                        `json:"iomBillingFlag,omitempty"`
	IomSpeed                    *string                        `json:"iomSpeed,omitempty"`
	IOMOfferGroupName           *string                        `json:"iomOfferGroupName,omitempty"`
	SwitchFeature               *[]SwitchFeature               `json:"SwitchFeature,omitempty"`
	RecurringFlag               *string                        `json:"recurringFlag,omitempty"`
	AttributeList               *[]AttributeList               `json:"AttributeList,omitempty"`
	EligibilityRuleCriteriaList *[]EligibilityRuleCriteriaList `json:"EligibilityRuleCriteriaList,omitempty"`
}

type RelatedOffersArray struct {
	ExtID           *string          `json:"extId,omitempty"`
	OfferName       *string          `json:"offerName,omitempty"`
	OfferInstanceID *int             `json:"offerInstanceId,omitempty"`
	ServiceType     *string          `json:"serviceType,omitempty"`
	Soc             *string          `json:"soc,omitempty"`
	ParameterInfo   *[]ParameterInfo `json:"ParameterInfo,omitempty"`
	SwitchFeature   *[]SwitchFeature `json:"SwitchFeature,omitempty"`
}

type ParameterInfo struct {
	ExtID                  *string                 `json:"extId,omitempty"`
	IOMAttributeRowID      *string                 `json:"iomAttributeRowId,omitempty"`
	IOMParentIntegrationID *string                 `json:"iomParentIntegrationId,omitempty"`
	IOMIntegrationID       *string                 `json:"iomIntegrationId,omitempty"`
	IOMType                *string                 `json:"iomType,omitempty"`
	ParamName              *string                 `json:"paramName,omitempty"`
	ValuesArray            *string                 `json:"valuesArray,omitempty"`
	OldValuesArray         *string                 `json:"oldValuesArray,omitempty"`
	Type                   *string                 `json:"type,omitempty"`
	EffectiveDate          *timer.FormattedTime    `json:"effectiveDate,omitempty"`
	ExpirationDate         *timer.FormattedTime    `json:"expirationDate,omitempty"`
	ExtendedInfo           *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type Subscriber struct {
	ExtID                       *string                        `json:"extId,omitempty"`
	Status                      *string                        `json:"status,omitempty"`
	AccountRefID                *string                        `json:"accountRefId,omitempty"`
	PayChannelIDPrimary         *int                           `json:"payChannelIdPrimary,omitempty"`
	PayChannelIDSecondary       *int                           `json:"payChannelIdSecondary,omitempty"`
	EventGroupItem              *string                        `json:"eventGroupItem,omitempty"`
	RefID                       *string                        `json:"refId,omitempty"`
	SubscriberID                *int                           `json:"subscriberId,omitempty"`
	SubscriberNumber            *string                        `json:"subscriberNumber,omitempty"`
	PrimaryResourceType         *string                        `json:"primaryResourceType,omitempty"`
	SubscriberType              *string                        `json:"subscriberType,omitempty"`
	OldBanDate                  *timer.FormattedTime           `json:"oldBanDate,omitempty"`
	OldBan                      *int                           `json:"oldBan,omitempty"`
	ProductOrderID              *string                        `json:"productOrderId,omitempty"`
	GroupProductID              *string                        `json:"groupProductId,omitempty"`
	AppointmentDate             *timer.FormattedTime           `json:"appointmentDate,omitempty"`
	AppointmentTimeSlot         *string                        `json:"appointmentTimeSlot,omitempty"`
	ServiceActivateDate         *timer.FormattedTime           `json:"serviceActivateDate,omitempty"`
	ResourceResvID              *string                        `json:"resourceResvId,omitempty"`
	WorkforceResvID             *string                        `json:"workforceResvId,omitempty"`
	RelatedProductID            *string                        `json:"relatedProductId,omitempty"`
	RelatedProductSpec          *string                        `json:"relatedProductSpec,omitempty"`
	RelatedSubscriberNumber     *string                        `json:"relatedSubscriberNumber,omitempty"`
	ProductID                   *string                        `json:"productId,omitempty"`
	Command                     *string                        `json:"command,omitempty"`
	ProductAddressID            *string                        `json:"productAddressId,omitempty"`
	SwitchOutletFlag            *string                        `json:"switchOutletFlag,omitempty"`
	EquipmentReturnFlag         *string                        `json:"equipmentReturnFlag,omitempty"`
	DisconnectDate              *timer.FormattedTime           `json:"disconnectDate,omitempty"`
	DisconnectType              *string                        `json:"disconnectType,omitempty"`
	SuspendNumberOfTimes        *int                           `json:"suspendNumberOfTimes,omitempty"`
	IOMAssetRowId               *string                        `json:"iomAssetRowId,omitempty"`
	IOMIntegrationID            *string                        `json:"iomIntegrationId,omitempty"`
	IOMProdSpecCode             *string                        `json:"iomProdSpecCode,omitempty"`
	IOMLastOrderID              *string                        `json:"iomLastOrderId,omitempty"`
	IOMOrderID                  *string                        `json:"iomOrderId,omitempty"`
	IOMOldSoftBundleCode        *string                        `json:"iomOldSoftBundleCode,omitempty"`
	IOMOldSoftBundleID          *string                        `json:"iomOldSoftBundleId,omitempty"`
	IOMDebundleSoftBundleFlag   *string                        `json:"iomDebundleSoftBundleFlag,omitempty"`
	EffectiveDate               *timer.FormattedTime           `json:"effectiveDate,omitempty"`
	IOMTapsTransactionID        *string                        `json:"iomTapsTransactionId,omitempty"`
	AssetRowID                  *string                        `json:"assetRowId,omitempty"`
	IOMOldAssetRowID            *string                        `json:"iomOldAssetRowId,omitempty"`
	IOMOldIntegrationID         *string                        `json:"iomOldIntegrationId,omitempty"`
	IOMServiceID                *string                        `json:"iomServiceId,omitempty"`
	IOMServiceAddressID         *string                        `json:"iomServiceAddressId,omitempty"`
	IOMOrderStatus              *string                        `json:"iomOrderStatus,omitempty"`
	SubscriberAddress           *Address                       `json:"SubscriberAddress,omitempty"` // here
	SubscriberGeneralInfo       *SubscriberGeneralInfo         `json:"SubscriberGeneralInfo,omitempty"`
	SubscriberName              *NameInfo                      `json:"SubscriberName,omitempty"`
	ResourceInfo                *[]ResourceInfo                `json:"ResourceInfo,omitempty"`
	ResourceRangeInfo           *[]ResourceRangeInfo           `json:"ResourceRangeInfo,omitempty"`
	ActivityInfo                *ActivityInfo                  `json:"ActivityInfo,omitempty"`
	ExtendedInfo                *map[string]interface{}        `json:"ExtendedInfo,omitempty"`
	Offers                      *[]Offers                      `json:"Offers,omitempty"`
	PropertyList                *map[string]interface{}        `json:"PropertyList,omitempty"`
	ReturnEquipt                *[]EquipmentInfo               `json:"ReturnEquipt,omitempty"`
	TVSReturnEquipt             *[]EquipmentInfo               `json:"TVSReturnEquipt,omitempty"`
	TVSSwitchDevice             *[]EquipmentInfo               `json:"TVSSwitchDevice,omitempty"`
	TVSHardwareProduct          *[]TVSHardwareProduct          `json:"TVSHardwareProduct,omitempty"`
	ServiceName                 *NameInfo                      `json:"ServiceName,omitempty"`
	ServiceAddress              *Address                       `json:"ServiceAddress,omitempty"`
	AttributeList               *[]AttributeList               `json:"AttributeList,omitempty"`
	NotifyInfo                  *[]NotifyInfo                  `json:"NotifyInfo,omitempty"`
	AdditionalDeviceList        *[]AdditionalDeviceList        `json:"AdditionalDeviceList,omitempty"`
	ConvergenceInfo             *ConvergenceInfo               `json:"ConvergenceInfo,omitempty"`
	ValueAddedList              *[]ValueAddedList              `json:"ValueAddedList,omitempty"`
	ReasonInfo                  *ActivityInfo                  `json:"ReasonInfo,omitempty"`
	SrvTrxNoInfo                *SrvTrxNoInfo                  `json:"SrvTrxNoInfo,omitempty"`
	SwitchFeatureInfo           *[]SwitchFeatureInfo           `json:"SwitchFeatureInfo,omitempty"`
	Source                      *string                        `json:"source,omitempty"`
	PromotionDescription        *string                        `json:"promotionDescription,omitempty"`
	SaleKey                     *string                        `json:"saleKey,omitempty"`
	SaleKeyEmpId                *string                        `json:"saleKeyEmpId,omitempty"`
	SofwareCPPrd                *string                        `json:"sofwareCPPrd,omitempty"`
	TVSPromotionId              *string                        `json:"tvsPromotionId,omitempty"`
	MultiSimInfo                *[]MultiSimInfo                `json:"MultiSimInfo,omitempty"`
	EligibilityRuleCriteriaList *[]EligibilityRuleCriteriaList `json:"EligibilityRuleCriteriaList,omitempty"`
	IOMOrderToStatus            *string                        `json:"iomOrderToStatus,omitempty"`
	IOMOrderFromStatus          *string                        `json:"iomOrderFromStatus,omitempty"`
}

type ValueAddedList struct {
	ExtId         *string                 `json:"extId,omitempty"`
	ProdId        *string                 `json:"prodId,omitempty"`
	ProdSpecCode  *string                 `json:"prodSpecCode,omitempty"`
	ProdSpecName  *string                 `json:"prodSpecName,omitempty"`
	Action        *string                 `json:"action,omitempty"`
	AttributeList *[]AttributeList        `json:"AttributeList,omitempty"`
	ExtendedInfo  *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type NotifyInfo struct {
	ExtID        *string                 `json:"extId,omitempty"`
	NotifySystem *string                 `json:"notifySystem,omitempty"`
	NotifyStatus *string                 `json:"notifyStatus,omitempty"`
	ExtendedInfo *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type AdditionalDeviceList struct {
	ExtID                       *string                        `json:"extId,omitempty"`
	DeviceType                  *string                        `json:"deviceType,omitempty"`
	Action                      *string                        `json:"action,omitempty"`
	DeviceName                  *string                        `json:"deviceName,omitempty"`
	Quantity                    *string                        `json:"quantity,omitempty"`
	ExtendedInfo                *map[string]interface{}        `json:"ExtendedInfo,omitempty"`
	EligibilityRuleCriteriaList *[]EligibilityRuleCriteriaList `json:"EligibilityRuleCriteriaList,omitempty"`
}

type ChargeTemplateList struct {
	ExtID                     *string                 `json:"extId,omitempty"`
	IOMChargeCode             *string                 `json:"iomChargeCode,omitempty"`
	IOMBillingChargeCode      *string                 `json:"iomBillingChargeCode,omitempty"`
	IOMPaymentDesignationCode *string                 `json:"iomPaymentDesignationCode,omitempty"`
	IOMRevenueCode            *string                 `json:"iomRevenueCode,omitempty"`
	IOMAmount                 *float64                `json:"iomAmount,omitempty"`
	IOMProvisioningFlag       *string                 `json:"iomProvisioningFlag,omitempty"`
	ExtendedInfo              *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type RelationList struct {
	ExtID               *string                 `json:"extId,omitempty"`
	IOMTagetOfferCode   *string                 `json:"iomTagetOfferCode,omitempty"`
	IOMRelationshipType *string                 `json:"iomRelationshipType,omitempty"`
	IOMRequireType      *string                 `json:"iomRequireType,omitempty"`
	ExtendedInfo        *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type TariffPlanList struct {
	ExtID                     *string                     `json:"extId,omitempty"`
	IOMTariffPlanSocCD        *string                     `json:"iomtariffPlanSocCD,omitempty"`
	IOMSocType                *string                     `json:"iomSocType,omitempty"`
	IOMSocLevel               *string                     `json:"iomSocLevel,omitempty"`
	IOMProvisioningFlag       *string                     `json:"iomProvisioningFlag,omitempty"`
	IOMPrimaryResource        *string                     `json:"iomPrimaryResource,omitempty"`
	IOMMaximumInstanceAllowed *string                     `json:"iomMaximumInstanceAllowed,omitempty"`
	IOMSpeed                  *string                     `json:"iomSpeed,omitempty"`
	TariffPlanPropertiesList  *[]TariffPlanPropertiesList `json:"TariffPlanPropertiesList,omitempty"`
	ExtendedInfo              *map[string]interface{}     `json:"ExtendedInfo,omitempty"`
}

type TariffPlanPropertiesList struct {
	ExtID                  *string                 `json:"extId,omitempty"`
	IOMParamType           *string                 `json:"iomParamType,omitempty"`
	IOMParamCategory       *string                 `json:"iomParamCategory,omitempty"`
	IOMRangeInd            *string                 `json:"iomRangeInd,omitempty"`
	IOMPrimaryResourceName *string                 `json:"iomPrimaryResourceName,omitempty"`
	ExtendedInfo           *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type SpeedInfo struct {
	ExtID        *string                 `json:"extId,omitempty"`
	DLSpeedRate  *string                 `json:"dlSpeedRate,omitempty"`
	DLSpeedUOM   *string                 `json:"dlSpeedUOM,omitempty"`
	ULSpeedRate  *string                 `json:"ulSpeedRate,omitempty"`
	ULSpeedUOM   *string                 `json:"ulSpeedUOM,omitempty"`
	ExtendedInfo *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type ResourceInfo struct {
	ExtID            *string                 `json:"extId,omitempty"`
	ResourceCategory *string                 `json:"resourceCategory,omitempty"`
	ResourceName     *string                 `json:"resourceName,omitempty"`
	ValuesArray      *string                 `json:"valuesArray,omitempty"`
	EffectiveDate    *timer.FormattedTime    `json:"effectiveDate,omitempty"`
	ExpirationDate   *timer.FormattedTime    `json:"expirationDate,omitempty"`
	ExtendedInfo     *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type ResourceRangeInfo struct {
	ExtID          *string                 `json:"extId,omitempty"`
	Action         *string                 `json:"action,omitempty"`
	ResourceName   *string                 `json:"resourceName,omitempty"`
	ValuesArray    *string                 `json:"valuesArray,omitempty"`
	EffectiveDate  *timer.FormattedTime    `json:"effectiveDate,omitempty"`
	ExpirationDate *timer.FormattedTime    `json:"expirationDate,omitempty"`
	ExtendedInfo   *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type SubscriberGeneralInfo struct {
	ExtID                    *string                 `json:"extId,omitempty"`
	EffectiveDate            *timer.FormattedTime    `json:"effectiveDate,omitempty"`
	Language                 *string                 `json:"language,omitempty"`
	ProofDate                *timer.FormattedTime    `json:"proofDate,omitempty"`
	ProofDoc                 *string                 `json:"proofDoc,omitempty"`
	SplitPeriod              *string                 `json:"splitPeriod,omitempty"`
	SmsInd                   *string                 `json:"smsInd,omitempty"`
	ProductSubtype           *string                 `json:"productSubtype,omitempty"`
	SaleID                   *string                 `json:"saleId,omitempty"`
	SmsLang                  *string                 `json:"smsLang,omitempty"`
	ImsiAlias                *string                 `json:"imsiAlias,omitempty"`
	DealerApp                *string                 `json:"dealerApp,omitempty"`
	ProjectCode              *string                 `json:"projectCode,omitempty"`
	InitActDate              *timer.FormattedTime    `json:"initActDate,omitempty"`
	ConvergenceCode          *string                 `json:"convergenceCode,omitempty"`
	RFBan                    *int                    `xml:"rfBan,omitempty" json:"rfBan,omitempty"`
	PrimResourceVal          *string                 `xml:"primResourceVal,omitempty" json:"primResourceVal,omitempty"`
	SmsLanguage              *string                 `xml:"smsLanguage,omitempty" json:"smsLanguage,omitempty"`
	RMVActDate               *timer.FormattedTime    `xml:"rmvActDate,omitempty" json:"rmvActDate,omitempty"`
	RecZone                  *string                 `xml:"recZone,omitempty" json:"recZone,omitempty"`
	BarringActDate           *timer.FormattedTime    `xml:"barringActDate,omitempty" json:"barringActDate,omitempty"`
	BarringActRsn            *string                 `xml:"barringActRsn,omitempty" json:"barringActRsn,omitempty"`
	BarByRater               *string                 `xml:"barByRater,omitempty" json:"barByRater,omitempty"`
	DonorZone                *string                 `xml:"donorZone,omitempty" json:"donorZone,omitempty"`
	RMVBan                   *int                    `xml:"rmvBan,omitempty" json:"rmvBan,omitempty"`
	InstallationType         *string                 `xml:"installationType,omitempty" json:"InstallationType,omitempty"`
	ActWaiveRsnCd            *string                 `xml:"actWaiveRsnCd,omitempty" json:"actWaiveRsnCd,omitempty"`
	PortInd                  *string                 `xml:"portInd,omitempty" json:"portInd,omitempty"`
	ColLastActDate           *timer.FormattedTime    `xml:"colLastActDate,omitempty" json:"colLastActDate,omitempty"`
	AddressFraudFlag         *string                 `xml:"addressFraudFlag,omitempty" json:"addressFraudFlag,omitempty"`
	BarringByReq             *string                 `xml:"barringByReq,omitempty" json:"barringByReq,omitempty"`
	DealerCode               *string                 `xml:"dealerCode,omitempty" json:"dealerCode,omitempty"`
	TMVBan                   *int                    `xml:"tmvBan,omitempty" json:"tmvBan,omitempty"`
	CreditStatus             *string                 `xml:"creditStatus,omitempty" json:"creditStatus,omitempty"`
	CreditLastActDate        *timer.FormattedTime    `xml:"creditLastActDate,omitempty" json:"creditLastActDate,omitempty"`
	LinkPrevSubNo            *int                    `xml:"linkPrevSubNo,omitempty" json:"linkPrevSubNo,omitempty"`
	Source                   *string                 `xml:"source,omitempty" json:"source,omitempty"`
	SubPassword              *string                 `xml:"subPassword,omitempty" json:"subPassword,omitempty"`
	CreditClass              *string                 `xml:"creditClass,omitempty" json:"creditClass,omitempty"`
	CalculatePaymentCategory *string                 `xml:"calculatePaymentCategory,omitempty" json:"calculatePaymentCategory,omitempty"`
	LastActivityID           *int                    `xml:"lastActivityId,omitempty" json:"lastActivityId,omitempty"`
	MarketingCode            *string                 `xml:"marketingCode,omitempty" json:"marketingCode,omitempty"`
	TMVSrvsLvl               *string                 `xml:"tmvSrvsLvl,omitempty" json:"tmvSrvsLvl,omitempty"`
	ColStatus                *string                 `xml:"colStatus,omitempty" json:"colStatus,omitempty"`
	PrimResourceTp           *string                 `xml:"primResourceTp,omitempty" json:"primResourceTp,omitempty"`
	CreditRsnCd              *string                 `xml:"creditRsnCd,omitempty" json:"creditRsnCd,omitempty"`
	TrueLifeID               *string                 `xml:"trueLifeId,omitempty" json:"trueLifeId,omitempty"`
	TMVActDate               *timer.FormattedTime    `xml:"tmvActDate,omitempty" json:"tmvActDate,omitempty"`
	RFServiceLevel           *string                 `xml:"rfServiceLevel,omitempty" json:"rfServiceLevel,omitempty"`
	L3FirstRechargeIndicator *string                 `xml:"l3FirstRechargeIndicator,omitempty" json:"l3FirstRechargeIndicator,omitempty"`
	PhaseCode                *string                 `xml:"phaseCode,omitempty" json:"phaseCode,omitempty"`
	RFActDate                *timer.FormattedTime    `xml:"rfActDate,omitempty" json:"rfActDate,omitempty"`
	RelatedSubscriber        *int                    `xml:"relatedSubscriber,omitempty" json:"relatedSubscriber,omitempty"`
	SaleChannel              *string                 `xml:"saleChannel,omitempty" json:"saleChannel,omitempty"`
	MultiSimInd              *string                 `xml:"multiSimInd,omitempty" json:"multiSimInd,omitempty"`
	TmvServiceLevel          *string                 `xml:"tmvServiceLevel,omitempty" json:"tmvServiceLevel,omitempty"`
	ExtendedInfo             *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type Subcontractor struct {
	ExtID             *string                 `json:"extId,omitempty"`
	SubcontractorCode *string                 `json:"subcontractorCode,omitempty"`
	SubcontractorName *string                 `json:"subcontractorName,omitempty"`
	OperStaff         *string                 `json:"operStaff,omitempty"`
	OperDate          *timer.FormattedTime    `json:"operDate,omitempty"`
	RuleType          *string                 `json:"ruleType,omitempty"`
	ChangeType        *string                 `json:"changeType,omitempty"`
	ReasonNo          *string                 `json:"reasonNo,omitempty"`
	ExtendedInfo      *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type DeviceInfo struct {
	ExtID        *string                 `json:"extId,omitempty"`
	Device       *string                 `json:"device,omitempty"`
	DeviceType   *string                 `json:"deviceType,omitempty"`
	DeviceCate   *string                 `json:"deviceCate,omitempty"`
	Brand        *string                 `json:"brand,omitempty"`
	Model        *string                 `json:"model,omitempty"`
	SerialNo     *string                 `json:"serialNo,omitempty"`
	MatCode      *string                 `json:"matCode,omitempty"`
	MatDesc      *string                 `json:"matDesc,omitempty"`
	Room         *string                 `json:"room,omitempty"`
	Floor        *string                 `json:"floor,omitempty"`
	ActivityDate *timer.FormattedTime    `json:"activityDate,omitempty"`
	DeviceStatus *string                 `json:"deviceStatus,omitempty"`
	CollectType  *string                 `json:"collectType,omitempty"`
	DeviceGroup  *string                 `json:"deviceGroup,omitempty"`
	ExtendedInfo *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type ChangeInfoList struct {
	ExtID        *string                 `json:"extId,omitempty"`
	Action       *string                 `json:"action,omitempty"`
	ExtendedInfo *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type NameInfo struct {
	ExtID              *string                 `json:"extId,omitempty"`
	NameType           *string                 `json:"nameType,omitempty"`
	Title              *string                 `json:"title,omitempty"`
	FirstName          *string                 `json:"firstName,omitempty"`
	MiddleName         *string                 `json:"middleName,omitempty"`
	LastName           *string                 `json:"lastName,omitempty"`
	NameSuffix         *string                 `json:"nameSuffix,omitempty"`
	MaritalStatus      *string                 `json:"maritalStatus,omitempty"`
	Gender             *string                 `json:"gender,omitempty"`
	OrgName            *string                 `json:"orgName,omitempty"`
	BranchCode         *string                 `json:"branchCode,omitempty"`
	BranchName         *string                 `json:"branchName,omitempty"`
	StoreID            *string                 `json:"storeId,omitempty"`
	FaxNumber          *string                 `json:"faxNumber,omitempty"`
	Email              *string                 `json:"email,omitempty"`
	Identification     *string                 `json:"identification,omitempty"`
	IdentificationType *string                 `json:"identificationType,omitempty"`
	Language           *string                 `json:"language,omitempty"`
	PrefContactNumber  *string                 `json:"prefContactNumber,omitempty"`
	HomePhone          *string                 `json:"homePhone,omitempty"`
	BizPhone           *string                 `json:"bizPhone,omitempty"`
	PrivatePhone       *string                 `json:"privatePhone,omitempty"`
	AuthFirstname      *string                 `json:"authFirstname,omitempty"`
	AuthLastname       *string                 `json:"authLastname,omitempty"`
	AuthPersonalID     *string                 `json:"authPersonalId,omitempty"`
	PoaName            *string                 `json:"poaName,omitempty"`
	PoaPersonalID      *string                 `json:"poaPersonalId,omitempty"`
	Education          *string                 `json:"education,omitempty"`
	JobTitle           *string                 `json:"jobTitle,omitempty"`
	AdditionalTitle    *string                 `json:"additionalTitle,omitempty"`
	ExtendedInfo       *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}
type MNPInfo struct {
	ExtID         *string                 `json:"extId,omitempty"`
	DonorOperator *string                 `json:"donorOperator,omitempty"`
	DonorZoneCode *string                 `json:"donorZoneCode,omitempty"`
	RcpOperator   *string                 `json:"rcpOperator,omitempty"`
	RcpZoneCode   *string                 `json:"rcpZoneCode,omitempty"`
	PinCode       *string                 `json:"pinCode,omitempty"`
	PinDate       *string                 `json:"pinDate,omitempty"`
	ChannelId     *string                 `json:"channelId,omitempty"`
	ExtendedInfo  *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type SMSInfo struct {
	ExtID          *string                 `json:"extId,omitempty"`
	SmsType        *string                 `json:"smsType,omitempty"`
	SmsMessage     *string                 `json:"smsMessage,omitempty"`
	SmsMobileNo    *string                 `json:"smsMobileNo,omitempty"`
	SmsLanguage    *string                 `json:"smsLanguage,omitempty"`
	SmsMessageType *string                 `json:"smsMessageType,omitempty"`
	ExtendedInfo   *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type EmailInfo struct {
	ExtID               *string                 `json:"extId,omitempty"`
	EmailAction         *string                 `json:"emailAction,omitempty"`
	EmailTemplateId     *string                 `json:"emailTemplateId,omitempty"`
	EmailVersion        *string                 `json:"emailVersion,omitempty"`
	EmailChannel        *string                 `json:"emailChannel,omitempty"`
	EmailLanguage       *string                 `json:"emailLanguage,omitempty"`
	EmailMessageType    *string                 `json:"emailMessageType,omitempty"`
	EmailToList         *[]string               `json:"emailToList,omitempty"`
	EmailCcList         *[]string               `json:"emailCcList,omitempty"`
	EmailBccList        *[]string               `json:"emailBccList,omitempty"`
	EmailParametersList *map[string]interface{} `json:"EmailParametersList,omitempty"`
	ExtendedInfo        *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type SwitchFeature struct {
	Action       *string                 `json:"action,omitempty"`
	Source       *string                 `json:"source,omitempty"`
	ItemId       *string                 `json:"itemId,omitempty"`
	SwParam      *string                 `json:"swParam,omitempty"`
	SwCode       *string                 `json:"swCode,omitempty"`
	ExtendedInfo *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type SrvTrxNoInfo struct {
	SrvTrxNo          *string                 `json:"srvTrxNo,omitempty"`
	SrvTrxTp          *string                 `json:"srvTrxTp,omitempty"`
	SrvTrxRouteStatus *string                 `json:"srvTrxRouteStatus,omitempty"`
	ExtendedInfo      *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type SwitchFeatureInfo struct {
	NewOrPrev *string `xml:"NEW_OR_PREV" json:"NEW_OR_PREV"`
	FtrGrp    *string `xml:"FTR_GRP" json:"FTR_GRP"`
	FtrParam  *string `xml:"FTR_PARAM" json:"FTR_PARAM"`
	FtrValue  *string `xml:"FTR_VALUE" json:"FTR_VALUE"`
}

type TVSListMappingCharge struct {
	CPID       *string `json:"cpId,omitempty"`
	RelateType *string `json:"relateType,omitempty"`
	SocName    *string `json:"socName,omitempty"`
}
type MultiSimInfo struct {
	ExtID             *string                 `json:"extId,omitempty"`
	SimType           *string                 `json:"simType,omitempty"`
	Sim               *string                 `json:"sim,omitempty"`
	Imsi              *string                 `json:"imsi,omitempty"`
	Alias             *string                 `json:"alias,omitempty"`
	RcOfferIn         *string                 `json:"rcOfferIn,omitempty"`
	RcOfferOut        *string                 `json:"rcOfferOut,omitempty"`
	SimStatus         *string                 `json:"simStatus,omitempty"`
	Source            *string                 `json:"source,omitempty"`
	SrvTrxNoInfo      *SrvTrxNoInfo           `json:"SrvTrxNoInfo,omitempty"`
	SwitchFeature     *[]SwitchFeature        `json:"SwitchFeature,omitempty"`
	ExtendedInfo      *map[string]interface{} `json:"ExtendedInfo,omitempty"`
	SwitchFeatureInfo *[]SwitchFeatureInfo    `json:"SwitchFeatureInfo,omitempty"`
}

type EligibilityRuleCriteriaList struct {
	ExtID                        *string                 `json:"extId,omitempty"`
	EligibilityRuleCriteria      *string                 `json:"eligibilityRuleCriteria,omitempty"`
	EligibilityRuleCriteriaValue *string                 `json:"eligibilityRuleCriteriaValue,omitempty"`
	ExtendedInfo                 *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}
type EmployeeInfo struct {
	ExtID        *string                 `json:"extId,omitempty"`
	EmployeeType *string                 `json:"employeeType,omitempty"`
	EmployeeCode *string                 `json:"employeeCode,omitempty"`
	ExtendedInfo *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}
type WebhookInfo struct {
	ExtID        *string                 `json:"extId,omitempty"`
	EndPoint     *string                 `json:"endPoint,omitempty"`
	ExtendedInfo *map[string]interface{} `json:"ExtendedInfo,omitempty"`
}

type SimInfo struct {
	ExtID             *string       `json:"extId,omitempty"`
	SubcontractorID   *string       `json:"subcontractorId,omitempty"`
	SubcontractorName *string       `json:"subcontractorName,omitempty"`
	SimSerialNo       *string       `json:"simSerialNo,omitempty"`
	ProductCode       *string       `json:"productCode,omitempty"`
	StockCode         *string       `json:"stockCode,omitempty"`
	ShopType          *string       `json:"shopType,omitempty"`
	DetailsList       []DetailsList `json:"DetailsList,omitempty"`
}

type DetailsList struct {
	ExtID            *string            `json:"extId,omitempty"`
	PropositionCode  *string            `json:"propositionCode,omitempty"`
	PromotionSetCode *string            `json:"promotionSetCode,omitempty"`
	ProductCode      *string            `json:"productCode,omitempty"`
	ProductType      *string            `json:"productType,omitempty"`
	Price            *string            `json:"price,omitempty"`
	DiscountBaht     *string            `json:"discountBaht,omitempty"`
	OtherPaymentList []OtherPaymentList `json:"OtherPaymentList,omitempty"`
}

type OtherPaymentList struct {
	ExtID  *string `json:"extId,omitempty"`
	Code   *string `json:"code,omitempty"`
	Amount *string `json:"amount,omitempty"`
}
```
