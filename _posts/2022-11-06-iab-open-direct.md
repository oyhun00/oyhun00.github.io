---
title: IAB OpenDirect
author: yong
date: 2022-11-06 11:58:00 +0800
categories: [AD Tech, IAB OpenDirect]
tags: [OpenDirect, ad, AD Tech, DSP, SSP, DMP]

---
*해당 포스트는 [IAB OpenDirect 공식 문서](https://github.com/InteractiveAdvertisingBureau/OpenDirect/blob/master/OpenDirect.v2.0.final.md)를 나름 번역하며 공부하고 정리한 글이며 빠진 내용 혹은 오역, 잘못된 내용이 있을 수 있음*

# 🖥️ Open Direct

Open Direct를 통해 퍼블리셔가 구매자에게 Open Direct의 표준에 맞춰 구축한 프로그래매틱한 인터페이스를 통해 광고 인벤토리를 제공할 수 있다. 

> **인벤토리란**? - 퍼블리셔가 판매할 수 있는 광고 지면, 매체를 뜻함
> 

각 기업 마다의 인터페이스를 통해 구매 및 판매를 전반적으로 걸쳐 재고를 관리한다. 각 구매자와 판매자의 시스템은 인터페이스가 서로 다르기 때문에, 시스템을 통합하기 위해선 많은 리소스를 소요하게 된다.

이를 위해 Open Direct를 통해 광고 인벤토리를 공급자와 수요자가 사용할 수 있게 끔 표준 인터페이스를 제공해준다.

판매자의 경우 프로그래매틱한 마켓 플레이스에서 구매자에게 다양한 *프리미엄 보장* 인벤토리를 제공할 수 있다. 또한 이 광고 인벤토리를 추적할 수 있어 가시적인 리포트를 제공하고, 노출 수와 송출 수를 계산할 수 있도록 기반을 제공해준다. 자동 보장 인벤토리를 제공하기 위한 인터페이스를 설계할 때 Open Direct API를 사용할 수 있다. 

공급자(판매자, *퍼블리셔) - 광고 지면을 제공하는 혹은 소유하고 있는 매체, 즉 광고 내용을 소비자에게 전달하는 매개체*

수요자(구매자) *- 광고주, 광고 지면을 필요로 하는 사람, 즉 광고주와 퍼블리셔는 광고 시장을 형성하는 핵심 주체, 광고를 제공하는 사람*

# 🔧 Open Direct의 주요 기능

- 광고 인벤토리 검색
- 가격 설정
- 타겟팅 및 빈도 수 제약 조건 적용
- 캠페인(Order) 혹은 광고 항목(Line) 추가
- 광고 소재(Creative) 업로드, 광고 항목(Line)에 광고 소재(Creative) 할당

# 🛣️ 작동 방식

| 수요자 | 공급자 |
| --- | --- |
| 수요자는 공급자로부터 OpenDirect에서 사용할 수 있는 ID를 얻는다. | 수요자가 광고 인벤토리를 사용하기 위해서 ID를 제공해준다. |
| 공급자로부터 얻은 계정을 통해 광고 인벤토리를 탐색한다. | 수요자가 Open Direct 기반의 시스템에서 액세스 할 수 있는 계정을 만든다. |
| 탐색한 광고 인벤토리에 캠페인(Order)에 광고 항목(Line)을 추가한다. | 수요자가 요청하는 API를 수신받고, 이에 응답한다. |
| 캠페인(Order) 설정이 완료되면, 공급자의 승인을 위해 광고 소재(Creative)를 제출하고, 승인이 되면 캠페인(Order)이 송출 될 수 있도록 예약할 수 있다. | 수요자가 제출한 광고 소재(Creative)를 승인 혹은 반려 처리한다. |

# 🗂️ Spec

![출처: IAB Open Direct Document](https://raw.githubusercontent.com/InteractiveAdvertisingBureau/OpenDirect/master/images/ODv2EntityRelationshipDiagram.png)
_출처: IAB Open Direct Document_

# 📋 Object

### Account

Account는 구매자(수요자) - 광고주 관계를 정의한다. 구매자는 **일반적으로 광고주를 대신하여 주문하는 대행사**다.

각 Account는 구매자를 광고주와 연결하고 판매자(공급자)의 캠페인(Order)을 관리하는 데 사용된다. 광고주는 여러 구매자들과 작업할 수 있으므로 작업하는 구매자에 대해 각각 별도의 계정을 갖게된다.

대행사(구매자)가 광고주를 대신하여 Account를 만들고 구매를 하기 위해선 광고주가 대행사(구매자)에게 권한을 부여해야 한다. 만약 권한을 부여받지 못한 대행사(구매자)가 Account 생성 시 실패해야한다. 권한을 부여하는 작업은 판매자(공급자)가 정의한다.

| 속성 | 내용 | 타입 |
| --- | --- | --- |
| id* | - 시스템에서 생성되는 식별 가능한 고유 opaque 타입 ID | string(36) |
| advertiserid* | - 광고주를 식별하는 ID <br> - 공급자가 수요자에게 생성하는 ID <br> - 자신을 나타내는 광고주는 AdvertiserId 혹은 BuyerId가 있어야함 | string(36) |
| buyerid* | - 구매자 역할을 하는 조직을 식별하는 ID <br> - 판매자(공급자)가 BuyerId를 생성함 <br> - 광고주가 직접 구매를 수행하는 경우, AdvertiserId와 BuyerId가 동일해야함 | string(36) |
| name* | - 계정의 이름 | string(255) |
| ext | - Optional vendor-specific extensions. | ext object |

### AdUnit

각 광고 단위에는 하나의 AdCOM 광고 사양이 포함됨

| 속성 | 내용 | 타입 |
| --- | --- | --- |
| id* | - 해당 광고 단위를 고유하게 식별하는 ID | string(36) |
| name | - Ad Unit의 고유한 이름 | string(255) |
| spec* | - The technical specifications of this Ad Unit | AdCOM Placement object |

### Address

주소 객체는 Organization에 대한 값을 제공하는데 사용

| 속성 | 내용 | 타입 |
| --- | --- | --- |
| city* | - organization의 주소 상에 입력된 도시 이름 | string(255) |
| country* | - organization의 지역 | string(255) |
| addressline1* | - organization의 기본 주소 | string(255) |
| addressline2* | - organization의 상세 주소 | string(255) |
| postalcode | - 주소의 우편번호 혹은 ZIP Code | string(15) |
| state | - organization의 행정 주소 | string(36) |

### Assignment

Assingment는 광고 소재(Creative)를 캠페인(Order)의 광고 항목(Line)과 연관시킨다. 광고 소재(Creative)는 하나 이상의 광고 항목(Line)에 할당될 수 있고, 광고 항목(Line)은 하나 이상의 광고 소재(Creative)에 할당될 수 있다.

| 속성 | 내용 | 타입 |
| --- | --- | --- |
| id | - 시스템에서 생성되는 식별 가능한 고유 opaque 타입 ID | string(36) |
| creativeid* | - 광고 항목에 표시되는 광고 소재의 ID | string(36) |
| placementid* | - 광고 소재를 표시할 placement의 ID | string(36) |
| status | - 광고 소재 표시 여부 결정 값 <br> - 비활성화 상태에서 활성화 상태로 못바꿀 수도 있음 | enum (Active, Inactive) |
| weight | - 동일한 광고 항목에 할당되어 있는 여러개의 광고 소재에서 해당 광고 소재 표시량의 가중치를 결정 <br> - 가중치를 지정하지 않으면 일정한 가중치 제공 <br> - 가중치가 하나라도 적용되어 있다면, 동일한 광고 항목에 속한 모든 Assignment는 가중치를 지정해야함 <br> - 모든 Assignment의 가중치 합이 최대 100이 되지 않으면 균등 순환이 적용됨 <br> - ex: 동일한 날짜의 광고 항목에 할당된 광고 소재 A의 가중치가 25, B의 가중치가 75라면 B는 A보다 3배 더 자주 표시된다. | integer (1-100) |
| ext | Optional vendor-specific extensions. | ext object |

### ChangeRequest

캠페인(Order)이 진행되고 있는 동안 변경이 필요할 때, ChangeRequest를 통해 변경을 요청하고 승인을 기다리는 동안 캠페인(Order)를 수정할 수 있다.

| 속성 | 내용 | 타입 |
| --- | --- | --- |
| id | - 시스템에서 생성되는 식별 가능한 고유 opaque 타입 ID | string(36) |
| accountid* | - ChangeRequest 권한을 소유한 광고주와 수요자를 식별하기 위한 Account ID <br> - 캠페인(Order)의 Account ID와 동일해야함 | string(36) |
| comments | - 변경이 필요한 설명 | string(1000) |
| contacts | - 연락처 목록 <br> - 구매자 및 광고주의 연락처 목록에 추가됨 <br> - 목록에는 고유한 연락처 Type이 포함되어야 함 | CONTACT array |
| orderid* | - 변경 요청된 캠페인(Order)의 ID | string(36) |
| requesterid* | - 대행사에서 변경 요청일 경우 AgencyID <br> - 판매자(공급자)가 변경 요청한 경우 PublisherID | string(36) |
| status | - 변경할 상태 지정 | enum (PENDING, APPROVED, REJECTED) |
| webhook | - 판매자(공급자)가 변경을 승인, 거부, 수정할 때 호출되는 URI <br> - URI는 변경 요청 ID가 포함된 PUT과 함께 호출 | string(36) |
| ext | Optional vendor-specific extensions. | ext object |

### Creative

광고 소재(Creative)는 캠페인(Order)의 광고 항목(Line)에 표시할 광고에 대한 정보를 제공한다. Assignment 리소스를 통해 캠페인(Order)의 광고 항목(Line) 리소스가 광고 소재(Creative)에 할당된다

| 속성 | 내용 | 타입 |
| --- | --- | --- |
| accountid* | - 광고 소재(Creative)를 소유한 Account ID | string(36) |
| name | - 광고 소재(Creative) 이름 | string(255) |
| ad | - 광고 소재(Creative)의 메타 데이터, 콘텐츠 | AdCOM Ad object |
| creativeapprovals | - 각 판매자(공급자)에 대한 승인 상태를 나타내는 항목 | Key/Value array |
| ext | Optional vendor-specific extensions. | ext object |

### Line

광고 항목(Line)은 캠페인(Order)에 포함되며, 제품의 상태, 시작 혹은 종료 날짜, 캠페인(Order) 항목에 대한 세부 정보를 제공한다. 

광고 소재(Creative)는 Assignment 리소스를 통해 캠페인의 광고 항목에 할당된다.

*광고 항목은 상태가 초안 상태일 때만 변경할 수 있다. 상태가 예약 혹은 거부 상태일 경우 광고 항목을 수정하기 위해 reset을 호출하여 광고 항목을 초안 상태로 다시 변경할 수 있다*


| 속성 | 내용 | 타입 |
| --- | --- | --- |
| id | - 시스템에서 생성되는 식별 가능한 고유 opaque 타입 ID | string(36) |
| name* | - 광고 항목(Line)의 고유한 이름 | string(200) |
| orderid* | - 광고 항목이 속한 캠페인(Order)의 ID | string(36) |
| productid* | - 광고 소재가 실행되는 제품의 ID | string(36) |
| bookingstatus* | - 광고 항목이 예약되었고, 광고를 전달할 수 있는지 결정하는 값 <br> - 광고 항목이 재설정되면 statechangereason 내용을 지워야한다. | enum (Draft, PendingReservation, Reserved, PendingBooking, Booked, InFlight, Finished, Stopped, Canceled, Pause, Expired, Declined, ChangePending) |
| statechangereason | - 상태를 변경한 이유 | string |
| startdate* | - 광고 항목이 시작되는 날짜 <br> - 시간은 오전 12시가 기본 값 <br> - 시작일은 현재보다 같거나 이후여야 하고, 캠페인(Campaign)의 시작일 보다 같거나 이후여야 한다. <br> - 캠페인의 시작일보다 이전일 경우, 날짜가 일치하도록 캠페인의 시작일을 변경해야함 | string (date-time) |
| enddate* | - 광고 항목이 종료될 날짜 <br> - 시간은 오후 11시 59분 59초가 기본 값 <br> - 종료일은 시작일 이후여야 하고, 캠페인(Campaign)의 종료일 보다 이전이거나 같아야한다. <br> - 캠페인의 종료일보다 이후일 경우, 날짜가 일치하도록 캠페인의 종료일을 변경해야함 | string (date-time) |
| ratetype* | - cost가 표현되는 측정 단위 정의 | enum (CPM, CPMV, CPC, CPD, FlatRate) |
| rate* | - 노출 단위 당 가격 1,000회 노출 마다 $10(CPM) <br> - 요금은 광고 항목이 추가, 수정, 예약될 때마다 결정 됨 | number |
| quantity* | - 지정된 기간에서 요청된 수량 <br> - 수량 값은 ratetype에 따라 달라짐 <br> - 광고 항목은 예약할 때 수량을 포함해야함 <br> - 요청된 수량을 사용할 수 없을 경우 bookingStatus는 Declined로 설정되어야함 | integer |
| cost* | The projected cost of the line is based on the specified quantity, rate and targeting. <br> The actual cost (the amount billed) is based on the actual number of mpressions. <br> The cost is specified in the currency for the order. <br> If the order uses a different currency than what the product uses, the cost for the line must be converted to the order’s currency. <br> The cost is determined at the time the line is saved with the following statuses: Drafted, Reserved, or Booked. | number |
| comment | - 참고하기 위한 항목 | string (255) |
| frequencycount | The maximum number of times that a unique user must see ads from this line during the specified interval (see FrequencyInterval). | integer |
| frequencyinterval | Defines the frequency cap intervals that the API supports. The frequency interval specifies the units in which the frequency count is expressed. <br> For example, if a line’s frequency count is 2 and interval is Day, display the ad to the same user a Max 2 times in the same calendar day. | enum (Day, Month, Week, Hour, LineDuration) |
| reservedexpirydate | The date and time that the reserved inventory will expire. If the line is reserved, the expiry date must be set. | string (date-time) |
| targeting | The creative assigned to the LINE resource is display when the line includes user segments and the delivery engine can determine whether the user matches the specified segments. | AdCOM Segment object array |
| pmp |  |  |
| ext | Optional vendor-specific extensions. | ext object |

### Message

### Order

### Organization

### Placement

### Product

### Stats

# 🔃 Workflow

### 대행사(구매자) 등록

대행사(구매자)는 퍼블리셔가 직접 등록한다. 해당 프로세스는 판매자(공급자)가 정의하며 판매자에 따라 다르다. 대행사에 조직이 추가되면 대행사에서 광고 고객을 위한 조직을 만들 수 있다. 조직에 존재하는 사용자들은 각자의 고유한 자격 증명을 가지고 있어야 한다.

### 광고주(Organization) 조직 등록

광고주는 퍼블리셔에게 직접 가입하거나, 대행사가 광고주를 대리할 수 있다. 광고주는 하나 이상의 조직을 만들 수 있다. 특정 조직을 만들면 해당 조직에 브랜드, 혹은 자회사에 대한 account를 만들 수 있다. 이러한 조직의 구성도는 광고주의 판단으로 자율적으로 구성할 수 있다. 그리고 각 조직의 사용자들은 고유한 자격 증명을 가지고 있어야 한다.

### OAuth 2.0 Access Token

공급자는 OAuth 2.0을 통해 사용자를 인증해야 한다. 모든 API 호출에서 헤더에 OAuth Access Token이 필요하다.

인증 서비스에서 Access Token과 Refresh Token 및 만료 시간을 반환한다. Refresh Token을 이용하여 Access Token을 받으면 된다.

### 계정 등록

광고주의 재량에 따라 조직을 구성하고 각 조직에 해당하는 account를 만들 수 있다 했다. 광고주를 대행하는 대행사는 광고주의 accept가 있어야 한다. 이러한 광고주의 account를 관리할 수 있는 권한을 대행사에게 부여하는 과정은 퍼블리셔가 직접 정의한다.

account는 캠페인(Order) 혹은 광고 소재(Creative)와의 관계가 포함된다.

### 인벤토리와 Availability, 가격 가져오기

The following provides several options for getting product inventory details. Typically, you’d use the first two options to present a product catalog and the last option to add and book a line.

사용자에게 보여줄 제품 카탈로그를 가져오려면 `/products` 에 `GET` 요청을 보낸다. 응답으로 `Product` 객체를 받고, 이 `Product` 객체에는 제품의 예상 일일 노출 수, base rate가 포함된다.

Providers should not use the avails search method (option 3) to determine estimated avails.

### 캠페인(Order) 생성

캠페인(Order)는 광고 항목(Line)의 상위 컨테이너이다. 캠페인(Order)에 광고 항목(Line)을 추가하려면 `POST` `/accounts/{id}/orders` 로 API Call을 보낸다.

### 캠페인(Order)에 광고 항목(Line) 추가

광고 항목(Line)은 예약할 광고 제품, 수량, 타겟팅 정보와 광고 항목이 실행되는 기간을 지정한다. 캠페인에 광고 항목을 추가하려면 `POST` `/accounts/{id}/orders/{id}/lines` 로 요청을 보낸다.

광고 항목의 상태는 초안 상태에서만 업데이트할 수 있다.

### 광고 소재(Creative) 업로드 및 Assignment(할당)

광고 소재를 업로드 한다. `POST` `/accounts/{id}/creatives` 요청을 보내면 된다.

대부분의 광고 소재는 광고 항목에 할당되기 전에 승인을 거쳐야 할당이 가능하다. 이 광고 소재의 상태를 확인하고 광고 항목에 할당한다. 광고 항목을 예약하기 전에 할당된 광고 소재가 있어야 한다. 이 광고 소재는 광고주가 실행하려는 실제 광고 소재이거나, 대체되는 광고 소재일 수 도 있다.

### 광고 항목 예약 및 취소

광고 항목을 예약 또는 취소하려면 각각 다음 URI에 PATCH 또는 PUT 요청을 보낸다.

`/accounts/{id}/orders/{id}/lines/{id}?reserve`

`/accounts/{id}/orders/{id}/lines/{id}?book`

`/accounts/{id}/orders/{id}/lines/{id}?cancel`

# 🗞️ 다이어그램

### 퍼블리셔 Workflow Diagram
<img src="/opendirect1.png" alt="IAB Open Direct" style="width: 450px;">
_출처: IAB Open Direct Document_

### 구매 대행사 및 광고주 Workflow Diagram
<img src="/opendirect2.png" alt="IAB Open Direct" style="width: 450px;">
_출처: IAB Open Direct Document_

작성중