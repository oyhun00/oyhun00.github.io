---
title: IAB OpenDirect
author: yong
date: 2022-11-06 11:58:00 +0800
categories: [AD Tech, IAB OpenDirect]
tags: [OpenDirect, ad, AD Tech, DSP, SSP, DMP]

---
# 🖥️ Open Direct

Open Direct를 통해 공급자(퍼블리셔)가 수요자에게 Open Direct의 표준에 맞춰 구축한 프로그래매틱한 인터페이스를 통해 광고 인벤토리를 제공할 수 있다. 

> **인벤토리란**? - 공급자가 판매할 수 있는 광고 지면, 매체를 뜻함
> 

각 기업 마다의 인터페이스를 통해 구매 및 판매를 전반적으로 걸쳐 재고를 관리한다. 각 공급자와 수요자의 시스템은 인터페이스가 서로 다르기 때문에, 시스템을 통합하기 위해선 많은 리소스를 소요하게 된다.

이를 위해 Open Direct를 통해 광고 인벤토리를 공급자와 수요자가 사용할 수 있게 끔 표준 인터페이스를 제공해준다.

공급자의 경우 프로그래매틱한 마켓 플레이스에서 수요자에게 다양한 *프리미엄 보장* 인벤토리를 제공할 수 있다. 또한 이 광고 인벤토리를 추적할 수 있어 가시적인 리포트를 제공하고, 노출수와 송출수를 계산할 수 있도록 기반을 제공해준다. 자동 보장 인벤토리를 제공하기 위한 인터페이스를 설계할 때 Open Direct API를 사용할 수 있다. 

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
| 수요자는 공급자로부터 OpenDirect에서 사용할 수 있는 ID를 얻는다. (ID가 계정? 혹은 다른 특정 ID?) | 수요자가 광고 인벤토리를 사용하기 위해서 ID를 제공해준다. |
| 공급자로부터 얻은 계정을 통해 광고 인벤토리를 탐색한다. | 수요자가 Open Direct 기반의 시스템에서 액세스 할 수 있는 계정을 만든다. |
| 탐색한 광고 인벤토리에 캠페인(Order)에 광고 항목(Line)을 추가한다. | 수요자가 요청하는 API를 수신받고, 이에 응답한다. |
| 캠페인(Order) 설정이 완료되면, 공급자의 승인을 위해 광고 소재(Creative)를 제출하고, 승인이 되면 캠페인(Order)이 송출 될 수 있도록 예약할 수 있다. | 수요자가 제출한 광고 소재(Creative)를 승인 혹은 반려 처리한다. |

# 🔑 Auth

OpenDriect API는 페이징 쿼리 매개 변수를 지원하고, OAuth를 사용하여 인증하는 RESTful API이다.

### 조직 (Organization)

### 수요자 (공급자)

### 광고주 (Advertiser)

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

# 🔃 Workflow

### 대행사(구매자) 등록

대행사(구매자)는 판매자(공급자)가 직접 등록한다. 해당 프로세스는 판매자(공급자)가 정의하며 판매자에 따라 다르다. 대행사에 조직이 추가되면 대행사에서 광고 고객을 위한 조직을 만들 수 있다. 조직에 존재하는 사용자들은 각자의 고유한 자격 증명을 가지고 있어야 한다.

작성중