# 직렬화 형식

변이 페이지는 트랜잭션과 기타 데이터에 대한 XRP Ledger의 표준 바이너리 형식에 대해 설명합니다. 이 바이너리 형식은 트랜잭션 콘텐츠의 디지털 서명을 생성하고 확인하는 데 필요하며, 서버 간 P2P 통신을 비롯한 다른 곳에서도 사용됩니다. rippled API는 일반적으로 JSON을 사용하여 클라이언트 애플리케이션과 통신합니다. 그러나 JSON은 동일한 데이터를 여러 가지 다르지만 동등한 방식으로 표현할 수 있기 때문에 디지털 서명을 위해 트랜잭션을 직렬화하기 위한 형식으로 적합하지 않습니다.

트랜잭션을 JSON 또는 다른 표현에서 표준 바이너리 형식으로 직렬화하는 프로세스는 다음 단계로 요약할 수 있습니다:

1. 필수 필드를 포함하여 모든 필수 필드가 제공되었는지 확인합니다(필수이지만 '자동 입력 가능' 필드 포함).\
   트랜잭션 형식 참조는 XRP Ledger 트랜잭션의 필수 필드와 선택 필드를 정의합니다.

{% hint style="info" %}
Note:

이 단계에서는 SigningPubKey도 제공해야 합니다. 서명할 때 서명을 위해 제공된 비밀 키에서 이 키를 도출할 수 있습니다.
{% endhint %}

2. 각 필드의 데이터를 "내부" 바이너리 형식으로 변환합니다.
3. 필드를 표준 순서대로 정렬합니다.
4. 각 필드에 필드 ID를 접두사로 붙입니다.
5. 필드(접두사 포함)를 정렬된 순서대로 연결합니다.

그 결과 단일 바이너리 blob이 생성되며, 이 blob은 ECDSA(secp256k1 타원 곡선 사용) 및 Ed25519와 같은 잘 알려진 서명 알고리즘을 사용해 서명할 수 있습니다. XRP Ledger의 목적을 위해 적절한 접두사(단일 서명의 경우 0x53545800, 다중 서명의 경우 0x534D5400)를 사용하여 데이터를 해시해야 합니다. 서명 후에는 TxnSignature 필드가 포함된 트랜잭션을 다시 직렬화해야 합니다.

{% hint style="info" %}
Note:

XRP Ledger은 동일한 직렬화 형식을 사용해 ledger 개체와 처리된 트랜잭션과 같은 다른 유형의 데이터를 표현합니다. 그러나 특정 필드만 서명되는 트랜잭션에 포함하기에 적합합니다. (예를 들어, 서명 자체를 포함하는 TxnSignature 필드는 서명하는 바이너리 blob에 포함되지 않아야 합니다.) 따라서 일부 필드는 객체가 서명될 때 객체에 포함되는 "서명" 필드로 지정되고, 그렇지 않은 "비서명" 필드로 지정됩니다.
{% endhint %}

## 예시

서명된 트랜잭션과 서명되지 않은 트랜잭션은 모두 JSON 및 바이너리 형식으로 표현할 수 있습니다. 다음 샘플은 동일한 서명된 트랜잭션을 JSON 및 바이너리 형식으로 보여줍니다:

**JSON:**

```
{
  "Account": "rMBzp8CgpE441cp5PVyA9rpVV7oT8hP3ys",
  "Expiration": 595640108,
  "Fee": "10",
  "Flags": 524288,
  "OfferSequence": 1752791,
  "Sequence": 1752792,
  "SigningPubKey": "03EE83BB432547885C219634A1BC407A9DB0474145D69737D09CCDC63E1DEE7FE3",
  "TakerGets": "15000000000",
  "TakerPays": {
    "currency": "USD",
    "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B",
    "value": "7072.8"
  },
  "TransactionType": "OfferCreate",
  "TxnSignature": "30440220143759437C04F7B61F012563AFE90D8DAFC46E86035E1D965A9CED282C97D4CE02204CFD241E86F17E011298FC1A39B63386C74306A5DE047E213B0F29EFA4571C2C",
  "hash": "73734B611DDA23D3F5F62E20A173B78AB8406AC5015094DA53F53D39B9EDB06C"
}
```

**바이너리(16진수로 표시):**

```
120007220008000024001ABED82A2380BF2C2019001ABED764D55920AC9391400000000000000000000000000055534400000000000A20B3C85F482532A9578DBB3950B85CA06594D165400000037E11D60068400000000000000A732103EE83BB432547885C219634A1BC407A9DB0474145D69737D09CCDC63E1DEE7FE3744630440220143759437C04F7B61F012563AFE90D8DAFC46E86035E1D965A9CED282C97D4CE02204CFD241E86F17E011298FC1A39B63386C74306A5DE047E213B0F29EFA4571C2C8114DD76483FACDEE26E60D8A586BB58D09F27045C46
```

## 샘플 코드

여기에 설명된 직렬화 프로세스는 여러 장소와 프로그래밍 언어로 구현되어 있습니다:

* rippled 코드 베이스의 C++에서 .
* 이 저장소의 코드 샘플 섹션에 있는 JavaScript.
* 이 저장소의 코드 샘플 섹션에 있는 Python 3.

또한 많은 클라이언트 라이브러리에서 허용된 오픈 소스 라이선스에 따라 직렬화 지원을 제공하므로 필요에 따라 코드를 가져와서 사용하거나 수정할 수 있습니다.

## 내부 형식

각 필드에는 서명할 때(그리고 대부분의 다른 경우) 리플 소스 코드에서 해당 필드를 나타내기 위해 사용되는 "내부" 바이너리 형식이 있습니다. 모든 필드의 내부 형식은 SField.cpp 의 소스 코드에 정의되어 있습니다. (이 파일에는 트랜잭션 필드 이외의 필드도 포함됩니다.) 트랜잭션 형식 참조에도 모든 트랜잭션 필드에 대한 내부 형식이 나열되어 있습니다.

예를 들어 플래그 일반 트랜잭션 필드는 UInt32(32비트 부호 없는 정수)가 됩니다.

## 정의 파일

다음 JSON 파일은 XRP Ledger 데이터를 바이너리 형식으로 직렬화하고 바이너리에서 역직렬화하는 데 필요한 중요한 상수를 정의합니다:

https://github.com/XRPLF/xrpl.js/blob/main/packages/ripple-binary-codec/src/enums/definitions.json

다음 표는 정의 파일의 최상위 필드를 정의합니다:

| 필드                    | 내용물                                                                                                                                                                                                      |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TYPES`               | 필드 ID를 구성하고 필드를 정식 순서로 정렬하기 위해 데이터 유형을 해당 "유형 코드" 에 매핑합니다. 1 미만의 코드는 실제 데이터에 나타나지 않아야 합니다. 10000 이상의 코드는 다른 개체 내에서 직렬화할 수 없는 "트랜잭션"과 같은 특별한 "고수준" 개체 유형을 나타냅니다. 각 유형을 직렬화하는 방법에 대한 자세한 내용은 유형 목록을 참조하세요. |
| `LEDGER_ENTRY_TYPES`  | 원장 개체를 해당 데이터 유형에 매핑합니다. 이는 원장 상태 데이터와 처리된 트랜잭션 메타데이터 의 "영향을 받는 노드" 섹션에 나타납니다.                                                                                                                           |
| `FIELDS`              | 거래, 원장 개체 또는 기타 데이터에 나타날 수 있는 모든 필드를 나타내는 정렬된 튜플 배열입니다. 각 튜플의 첫 번째 멤버는 필드의 문자열 이름이고 두 번째 멤버는 해당 필드의 속성을 가진 개체입니다. (해당 필드에 대한 정의는 아래의 "필드 속성" 표를 참조하세요.)                                                  |
| `TRANSACTION_RESULTS` | [거래 결과 코드를](https://xrpl.org/transaction-results.html) 숫자 값으로 매핑합니다. 원장에 포함되지 않은 결과 유형은 음수 값을 갖습니다. `tesSUCCESS`숫자 값이 0입니다. [`tec`-클래스 코드는](https://xrpl.org/tec-codes.html) 원장에 포함된 오류를 나타냅니다.          |
| `TRANSACTION_TYPES`   | 모든 [거래 유형을](https://xrpl.org/transaction-types.html) 숫자 값으로 매핑합니다.                                                                                                                                       |

서명 및 제출을 위해 트랜잭션을 직렬화하기 위해서는 FIELDS, TYPES 및 TRANSACTION\_TYPES 필드가 필요합니다.

FIELDS 배열의 필드 정의 객체에는 다음과 같은 필드가 있습니다:

| 필드               | 유형      | 내용물                                                                                                          |
| ---------------- | ------- | ------------------------------------------------------------------------------------------------------------ |
| `nth`            | 숫자      | 필드 ID를 구성 하고 동일한 데이터 유형의 다른 필드와 함께 정렬하는 데 사용되는 이 필드의 필드 코드 입니다.                                              |
| `isVLEncoded`    | boolean | 인 경우 `true`가 필드에는 길이 접두사가 붙습니다.                                                                              |
| `isSerialized`   | boolean | 인 경우 `true`가 필드는 직렬화된 이진 데이터로 인코딩되어야 합니다. 이 필드가 인 경우 `false`필드는 일반적으로 저장되지 않고 요청 시 재구성됩니다.                   |
| `isSigningField` | boolean | `true`서명을 위해 트랜잭션을 준비할 때 이 필드를 직렬화해야 하는지 여부입니다. 이면 `false`서명할 데이터에서 이 필드를 생략해야 합니다. (전혀 거래의 일부가 아닐 수도 있습니다.) |
| `type`           | 문자열     | 이 필드의 내부 데이터 유형입니다. 이는 이 필드에 대한 유형 코드를`TYPES` 제공하는 맵의 키에 매핑됩니다.                                              |

## 필드 ID

필드의 유형 코드와 필드 코드를 결합하면 필드의 고유 식별자를 얻을 수 있으며, 이 식별자는 최종 직렬화된 blob에서 필드 앞에 접두사로 붙습니다. 필드 ID의 크기는 결합하는 유형 코드와 필드 코드에 따라 1\~3바이트입니다. 다음 표를 참조하십시오:

|                      | Type Code < 16                                                                                                                                                               | Type Code >= 16                                                                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Field Code < 16**  | ![1 byte: high 4 bits define type; low 4 bits define field.](https://xrpl.org/img/field-id-common-type-common-field.png)                                                     | ![2 bytes: low 4 bits of the first byte define field; next byte defines type.](https://xrpl.org/img/field-id-uncommon-type-common-field.png)       |
| **Field Code >= 16** | ![2 bytes: high 4 bits of the first byte define type; low 4 bits of first byte are 0; next byte defines field](https://xrpl.org/img/field-id-common-type-uncommon-field.png) | ![3 bytes: first byte is 0x00, second byte defines type; third byte defines field](https://xrpl.org/img/field-id-uncommon-type-uncommon-field.png) |

디코딩할 때 첫 번째 바이트의 비트가 0이면 필드 ID가 몇 바이트인지 알 수 있습니다. 이는 위 표의 경우에 해당합니다:

| <p><br>상위 4비트는 0이 아닙니다.</p> | 상위 4비트는 0입니다.                                                            |                                                                     |
| --------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------- |
| **낮은 4비트는 0이 아닙니다.**        | 1바이트: 상위 4비트는 유형을 정의합니다. 낮은 4비트는 필드를 정의합니다.                              | 2바이트: 첫 번째 바이트 정의 필드의 하위 4비트; 다음 바이트는 유형을 정의합니다.                    |
| **하위 4비트는 0입니다.**           | 2바이트: 첫 번째 바이트 정의 유형의 상위 4비트; 첫 번째 바이트의 하위 4비트는 0입니다. 다음 바이트는 필드를 정의합니다. | 3바이트: 첫 번째 바이트는 `0x00`이고, 두 번째 바이트는 유형을 정의합니다. 세 번째 바이트는 필드를 정의합니다. |

{% hint style="info" %}
Caution:

필드 ID가 필드를 정렬하는 데 사용되는 두 요소로 구성되어 있더라도 필드 ID의 바이트 구조에 따라 정렬 순서가 변경되므로 직렬화된 필드 ID 자체로 정렬해서는 안 됩니다.
{% endhint %}

## 길이 접두사

일부 유형의 가변 길이 필드에는 길이 표시기가 접두사로 붙습니다. blob 필드(임의의 이진 데이터 포함)가 이러한 유형 중 하나 입니다. 길이 접두사가 붙는 유형 목록은 유형 목록 표를 참조하십시오.

{% hint style="info" %}
Note:

길이가 다양한 일부 필드 유형은 길이 접두사가 붙지 않습니다. 이러한 유형에는 콘텐츠의 끝을 나타내는 다른 방법이 있습니다.
{% endhint %}

길이 접두사는 유형 접두사 바로 뒤와 콘텐츠 앞에 있는 필드의 길이를 나타내는 1\~3바이트로 구성됩니다.

* 필드에 0\~192바이트의 데이터가 포함되어 있는 경우 첫 번째 바이트가 콘텐츠의 길이를 정의하고, 길이 바이트 바로 뒤에 해당 바이트만큼의 데이터가 이어집니다.
*   필드에 193\~12480바이트의 데이터가 포함된 경우 처음 두 바이트는 다음 수식을 사용하여 필드의 길이를 나타냅니다:

    ```
    193 + ((byte1 - 193) * 256) + byte2
    ```
*   필드에 12481\~918744바이트의 데이터가 포함된 경우 처음 세 바이트는 다음 수식을 사용하여 필드의 길이를 나타냅니다:

    ```
    12481 + ((byte1 - 241) * 65536) + (byte2 * 256) + byte3
    ```
* 길이 접두사가 붙은 필드는 918744바이트를 초과하는 데이터를 포함할 수 없습니다.

디코딩할 때 첫 번째 길이 바이트의 값에서 추가 길이 바이트가 0, 1 또는 2인지 여부를 알 수 있습니다:

* 첫 번째 길이 바이트의 값이 192 이하인 경우 이 길이 바이트가 유일한 길이 바이트이며 필드 콘텐츠의 정확한 길이(바이트)를 포함합니다.
* 첫 번째 길이 바이트의 값이 193\~240이면 길이 바이트가 두 개 있습니다.
* 첫 번째 길이 바이트의 값이 241\~254이면 길이 바이트가 3개입니다.

## 표준 필드 순서

트랜잭션의 모든 필드는 먼저 필드 유형(특히 각 유형에 할당된 숫자 '유형 코드')을 기준으로 특정 순서로 정렬된 다음 필드 자체('필드 코드')를 기준으로 정렬됩니다. (성, 이름 순으로 정렬한다고 생각하면 됩니다. 여기서 성은 필드의 유형이고 이름은 필드 자체입니다.)

## 유형 코드

각 필드 유형에는 임의의 유형 코드가 있으며, 낮은 코드가 먼저 정렬됩니다. 이러한 코드는 SField.h 에 정의되어 있습니다.

예를 들어 UInt32는 유형 코드가 2이므로 모든 UInt32 필드는 유형 코드가 6인 모든 Amount 필드 앞에옵니다.

정의 파일에는 TYPES 맵의 각 유형에 대한 유형 코드가 나열되어 있습니다.

## 필드 코드

각 필드에는 서로 동일한 유형을 가진 필드를 정렬하는 데 사용되는 필드 코드가 있으며, 코드가 낮은 필드부터 정렬됩니다. 이러한 필드는 SField.cpp에 정의되어 있습니다.

예를 들어 결제 트랜잭션의 계정 필드는 정렬 코드가 1이므로 정렬 코드가 3인 목적지 필드보다 앞에옵니다.

필드 코드는 필드 유형이 다른 필드에 재사용되지만 같은 유형의 필드에는 동일한 필드 코드를 가질 수 없습니다. 유형 코드를 필드 코드와 결합하면 해당 필드의 고유 필드 ID를 얻을 수 있습니다.

## 유형 목록

트랜잭션 지침에는 다음 유형의 필드가 포함될 수 있습니다:

| 유형 이름                                                         | 유형 코드 | 비트 길이     | 길이 접두사 | 설명                                                                                      |
| ------------------------------------------------------------- | ----- | --------- | ------ | --------------------------------------------------------------------------------------- |
| [계정 ID](https://xrpl.org/serialization.html#accountid-fields) | 8     | 160       | 예      | 계정의 고유 식별자입니다.                                                                          |
| [양](https://xrpl.org/serialization.html#amount-fields)        | 6     | 64 또는 384 | 아니요    | RP 또는 토큰의 금액입니다. 필드의 길이는 XRP의 경우 64비트, 토큰의 경우 384비트(64+160+160)입니다.                     |
| [얼룩](https://xrpl.org/serialization.html#blob-fields)         | 7     | 변수        | 예      | 임의의 바이너리 데이터. 이러한 중요한 필드 중 하나는 트랜잭션을 승인하는 서명인 TxnSignature입니다.                          |
| [해시128](https://xrpl.org/serialization.html#hash-fields)      | 4     | 128       | 아니요    | 128비트 임의의 바이너리 값입니다. 유일한 필드는 이메일 해시로, 그라바타를 가져올 목적으로 계정 소유자 이메일의 MD-5 해시를 저장하기 위한 것입니다. |
| [해시160](https://xrpl.org/serialization.html#hash-fields)      | 17    | 160       | 아니요    | 160비트 임의의 바이너리 값입니다. 통화 코드 또는 발행자를 정의할 수 있습니다.                                          |
| [해시256](https://xrpl.org/serialization.html#hash-fields)      | 5     | 256       | 아니요    | 256비트 임의의 바이너리 값입니다. 일반적으로 트랜잭션, 원장 버전 또는 원장 데이터 개체의 "SHA-512Half" 해시를 나타냅니다.           |
| [경로 집합](https://xrpl.org/serialization.html#pathset-fields)   | 18    | 변수        | 아니요    | 교차 통화 결제를 위한 가능한 결제 경로 집합입니다.                                                           |
| [스타레이](https://xrpl.org/serialization.html#array-fields)      | 15    | 변수        | 아니요    | 필드에 따라 다른 유형이 될 수 있는 가변 멤버 수를 포함하는 배열입니다. 여기에는 다중 서명에 사용되는 메모와 서명자 목록이 포함됩니다.           |
| [ST객체](https://xrpl.org/serialization.html#object-fields)     | 14    | 변수        | 아니요    | 중첩된 필드가 하나 이상 포함된 객체입니다.                                                                |
| [UInt8](https://xrpl.org/serialization.html#uint-fields)      | 16    | 8         | 아니요    | 8비트 부호 없는 정수입니다.                                                                        |
| [UInt16](https://xrpl.org/serialization.html#uint-fields)     | 1     | 16        | 아니요    | 16비트 부호 없는 정수입니다. 트랜잭션 유형은 이 유형의 특수한 경우로, 특정 문자열이 정수 값에 매핑됩니다.                          |
| [UInt32](https://xrpl.org/serialization.html#uint-fields)     | 2     | 32        | 아니요    | 32비트 부호 없는 정수입니다. 모든 트랜잭션의 플래그 및 시퀀스 필드가 이 유형의 예입니다.                                    |

위의 모든 필드 유형 외에도 다음 유형은 ledger 객체 및 트랜잭션 메타데이터와 같은 다른 컨텍스트에서 나타날 수 있습니다:

| 유형 이름                                                     | 유형 코드 | 길이 접두사 | 설명                                                                               |
| --------------------------------------------------------- | ----- | ------ | -------------------------------------------------------------------------------- |
| 거래                                                        | 10001 | 아니요    | 전체 트랜잭션을 포함하는 "상위 수준" 유형입니다.                                                     |
| 원장항목                                                      | 10002 | 아니요    | 전체 원장 개체를 포함하는 "상위 수준" 유형입니다.                                                    |
| 확인                                                        | 10003 | 아니요    | 합의 프로세스에서 유효성 검사 투표를 나타내기 위해 P2P 통신에서 사용되는 "상위 수준" 유형입니다.                        |
| 메타데이터                                                     | 10004 | 아니요    | 하나의 트랜잭션에 대한 메타데이터를 포함하는 "상위 수준" 유형입니다.                                          |
| [UInt64](https://xrpl.org/serialization.html#uint-fields) | 3     | 아니요    | 64비트 부호 없는 정수입니다. 이 유형은 트랜잭션 명령에 나타나지 않지만 여러 원장 객체에서 이 유형의 필드를 사용합니다.            |
| 벡터256                                                     | 19    | 예      | 이 유형은 트랜잭션 지침에 나타나지 않지만 수정 내역 원장 객체의 수정 내역 필드에서 이 유형을 사용하여 현재 활성화된 수정 내역을 나타냅니다. |

## AccountID 필드

이 유형의 필드에는 XRP Ledger 계정에 대한 160비트 식별자가 포함됩니다. JSON에서 이러한 필드는 오타로 인해 유효한 주소가 생성되지 않을 수 있도록 추가 체크섬 데이터와 함께 base58 XRP 레저 "주소"로 표시됩니다. (이 인코딩은 "Base58Check"라고도 하며, 실수로 잘못된 주소로 송금하는 것을 방지합니다.) 이러한 필드의 바이너리 형식에는 체크섬 데이터가 포함되지 않으며 주소 Base58 인코딩에 사용되는 0x00 "유형 접두사"도 포함되지 않습니다. (그러나 바이너리 형식은 주로 서명된 트랜잭션에 사용되므로 서명된 트랜잭션을 기록할 때 오타나 기타 오류가 발생하면 서명이 무효화되어 송금할 수 없게 됩니다.)

stand-alone 필드(예: 계정 및 대상)로 표시되는 AccountID는 길이가 160비트로 고정되어 있음에도 불구하고 길이 접두사가 붙습니다. 따라서 이러한 필드의 길이 표시기는 항상 바이트 0x14입니다. 특수 필드(금액 발행자 및 경로 설정 계정)의 하위 필드로 표시되는 계정 ID는 길이 접두사가 붙지 않습니다.

## 금액 필드

"금액" 유형은 XRP 또는 토큰과 같은 화폐 금액을 나타내는 특수 필드 유형입니다. 이 유형은 두 가지 하위 유형으로 구성됩니다:

* XRP\
  XRP는 64비트 부호 없는 정수(빅 엔디안 순서)로 직렬화되지만, 가장 중요한 비트는 항상 0으로 XRP임을 나타내고 두 번째로 중요한 비트는 1로 양수임을 나타냅니다. 최대 금액(1017드롭)에는 57비트만 필요하므로 표준 64비트 부호 없는 정수를 가져와 0x4000000000000000으로 비트단위-OR을 수행하여 XRP 직렬화 형식을 계산할 수 있습니다.
* 토큰\
  토큰은 순서대로 세 개의 세그먼트로 구성됩니다:\
  \
  1\. 토큰 금액 형식의 금액을 나타내는 64비트. 첫 번째 비트는 1로 XRP가 아님을 나타냅니다.\
  \
  2\. 화폐 코드를 나타내는 160비트. 표준 API는 표준 화폐 코드 형식을 사용하여 "USD"와 같은 3자 코드를 160비트 코드로 변환하지만 사용자 지정 160비트 코드도 사용할 수 있습니다.\
  \
  3\. 발급자의 계정 ID를 나타내는 160비트입니다. (참고: 계정 주소 인코딩)

첫 번째 비트를 기준으로 두 가지 하위 유형 중 어떤 유형인지 알 수 있습니다: 0은 XRP, 1은 토큰입니다.

다음 다이어그램은 XRP 금액과 토큰 금액 모두에 대한 직렬화 형식을 보여줍니다:

<figure><img src="https://xrpl.org/img/serialization-amount.svg" alt=""><figcaption></figcaption></figure>

## 토큰 금액 형식

XRP Ledger은 64비트를 사용해 (대체 가능한) 토큰의 숫자액을 직렬화합니다. (JSON 형식에서 숫자 금액은 화폐 금액 개체의 값 필드입니다). 바이너리 형식에서 숫자 금액은 "XRP가 아님" 비트, 부호 비트, 유효 자릿수, 지수로 순서대로 구성됩니다:

1. 토큰 금액의 첫 번째(가장 중요한) 비트는 1로 XRP 금액이 아님을 나타냅니다. (XRP 금액은 이 형식과 구분하기 위해 항상 최상위 비트가 0으로 설정되어 있습니다.)
2. 부호 비트는 금액이 양수인지 음수인지를 나타냅니다. 표준 2의 보수 정수와 달리, XRP Ledger 형식에서 1은 양수를 나타내고 0은 음수를 나타냅니다.
3. 다음 8비트는 지수를 부호 없는 정수로 나타냅니다. 지수는 -96에서 +80(포함) 범위의 스케일(유효 자릿수에 10의 어떤 거듭제곱을 곱해야 하는가)을 나타냅니다. 그러나 직렬화할 때는 지수에 97을 더하여 부호 없는 정수로 직렬화할 수 있도록 합니다. 따라서 직렬화된 값 1은 -96의 지수를 나타내고, 직렬화된 값 177은 80의 지수를 나타내는 등의 방식으로 표시됩니다.
4. 나머지 54비트는 부호 없는 정수로 유효 자릿수(맨티사라고도 함)를 나타냅니다. 직렬화할 때 이 값은 0이라는 특수한 경우를 제외하고 1015(1000000000000000)에서 1016-1(99999999999999)을 포함하는 범위로 정규화됩니다. 0의 특수한 경우 부호 비트, 지수 및 유효 자릿수가 모두 0이므로 64비트 값은 0x80000000000000000000000000000000으로 직렬화됩니다.

숫자 금액은 화폐 코드 및 발행자와 함께 직렬화되어 전체 토큰 금액을 구성합니다.

## 화폐 코드

프로토콜 수준에서 XRP Ledger의 화폐 코드는 특별한 의미가 있는 다음 값을 제외하고는 임의의 160비트 값입니다:

* 화폐 코드 0x000000000000000000005852500000000000은 항상 허용되지 않습니다. ('표준 형식'의 "XRP" 코드입니다.)
* 화폐 코드 0x000000000000000000000000000000000000(모두 0)은 일반적으로 허용되지 않습니다. 일반적으로 XRP 금액은 화폐 코드로 지정되지 않습니다. 그러나 이 코드는 드물게 필드에 XRP에 대한 화폐 코드를 지정해야 하는 경우 XRP를 나타내는 데 사용됩니다.

rippled API는 다음과 같이 3자 ASCII 코드를 160비트 16진수 값으로 변환하는 표준 형식을 지원합니다:

<figure><img src="https://xrpl.org/img/currency-code-format.svg" alt=""><figcaption></figcaption></figure>

1. 처음 8비트는 0x00이어야 합니다.
2. 다음 88비트는 예약되어 있으며 모두 0이어야 합니다.
3. 다음 24비트는 3문자의 ASCII를 나타냅니다. 리플은 ISO 4217 코드 또는 "BTC"와 같이 널리 사용되는 유사 ISO 4217 코드를 사용할 것을 권장합니다. 그러나 모든 대문자와 소문자, 숫자, 기호 ?, !, @, #, $, %, ^, &, \*, <, >, (, ), {, }, \[, ], | 등 모든 문자의 조합은 허용됩니다. 화폐 코드 XRP(모두 대문자)는 XRP 전용으로 예약되어 있으며 토큰에서 사용할 수 없습니다.
4. 다음 40비트는 예약되어 있으며 모두 0이어야 합니다.

비표준 형식은 처음 8비트가 0x00이 아닌 한 160비트 데이터입니다.

## 배열 필드

서명자 항목(서명자 목록 집합 트랜잭션의 서명자 항목) 및 메모와 같은 일부 트랜잭션 필드는 객체 배열입니다("STArray" 유형이라고 함).

배열은 기본 바이너리 형식의 여러 객체 필드를 특정 순서로 포함합니다. JSON에서 각 배열 멤버는 멤버 객체 필드의 이름인 단일 필드를 가진 JSON "래퍼" 객체입니다. 해당 필드의 값은 ("내부") 객체 자체입니다.

이진 형식에서 배열의 각 멤버는 필드 ID 접두사(래퍼 객체의 단일 키에 기반)와 내용(내부 객체로 구성되며 객체로 직렬화됨)을 갖습니다. 배열의 끝을 표시하려면 "필드 ID"가 0xf1(필드 코드가 1인 배열의 유형 코드)이고 내용이 없는 항목을 추가합니다.

다음 예제는 배열의 직렬화 형식(SignerEntries 필드)을 보여줍니다:

<figure><img src="https://xrpl.org/img/serialization-array.svg" alt=""><figcaption></figcaption></figure>

## blob 필드

Blob 유형은 임의의 데이터를 포함하는 길이 접두사가 붙은 필드입니다. 이 유형을 사용하는 두 가지 일반적인 필드는 각각 트랜잭션 실행을 승인하는 공개 키와 서명을 포함하는 SigningPubKey 및 TxnSignature입니다.

blob 필드는 콘텐츠에 대한 추가 구조가 없으므로 필드 ID와 길이 접두사 뒤에 가변 길이 인코딩에 표시된 바이트 수로 정확하게 구성됩니다.

## 해시 필드

XRP Ledger에는 여러 가지 "해시" 유형이 있습니다: 해시128, 해시160, 해시256. 이러한 필드에는 주어진 비트 수의 임의 바이너리 데이터가 포함되며, 이는 해시 연산 결과를 나타낼 수도 있고 아닐 수도 있습니다.

이러한 모든 필드는 길이 표시 없이 특정 비트 수로 직렬화되며, 빅 엔디안 바이트 순서로 표시됩니다.

## 객체 필드

서명자 항목(서명자 목록 집합 트랜잭션의 서명자), 메모(메모 배열의 메모)와 같은 일부 필드는 객체("STObject" 유형이라고 함)입니다. 객체의 직렬화는 배열의 직렬화와 매우 유사하지만 한 가지 차이점이 있는데, 배열 필드에는 이미 명시적인 순서가 있는 반면 객체 멤버는 객체 필드 내에 표준 순서로 배치해야 한다는 점입니다.

객체 필드의 표준 필드 순서는 모든 최상위 필드에 대한 표준 필드 순서와 동일하지만 객체의 멤버는 객체 내에서 정렬되어야 합니다. 마지막 멤버 뒤에는 콘텐츠가 없는 0xe1의 "개체 끝" 필드 ID가 있습니다.

다음 예는 객체(메모 배열의 단일 메모 객체)에 대한 직렬화 형식을 보여줍니다.

<figure><img src="https://xrpl.org/img/serialization-object.svg" alt=""><figcaption></figcaption></figure>

## PathSet 필드

화폐 간 결제 트랜잭션의 Paths 필드는 JSON에서 배열 배열로 표시되는 "PathSet"입니다. 경로가 사용되는 용도에 대한 자세한 내용은 경로를 참조하세요.

PathSet은 1\~6개의 개별 경로가 순서대로 직렬화됩니다\[Source]. 각 완전한 경로 뒤에는 다음에 오는 경로를 나타내는 바이트가 이어집니다:

* 0xff는 다른 경로가 이어짐을 나타냅니다.
* 0x00은 패스셋의 끝을 나타냅니다.

각 경로는 1\~8개의 경로 단계로 순서대로 구성됩니다\[Source] . 각 단계는 유형 바이트에서 시작하여 경로 단계를 설명하는 하나 이상의 필드가 이어집니다. 유형은 비트 단위 플래그를 통해 해당 경로 단계에 어떤 필드가 있는지를 나타냅니다. (예를 들어 0x30 값은 화폐와 발행자를 모두 변경함을 나타냅니다.) 필드가 두 개 이상 있는 경우 필드는 항상 특정 순서로 배치됩니다.

다음 표에서는 사용 가능한 필드와 이를 나타내기 위해 유형 바이트에 설정할 비트별 플래그에 대해 설명합니다:

| 유형 플래그 | 필드 존재      | 필드 유형                                                                  | 비트 크기  | 오더  |
| ------ | ---------- | ---------------------------------------------------------------------- | ------ | --- |
| `0x01` | `account`  | [AccountID](https://xrpl.org/serialization.html#accountid-fields)      | 160 비트 | 첫번재 |
| `0x10` | `currency` | [Currency Code](https://xrpl.org/currency-formats.html#currency-codes) | 160 비트 | 두번째 |
| `0x20` | `issuer`   | [AccountID](https://xrpl.org/serialization.html#accountid-fields)      | 160 비트 | 세번째 |

일부 조합은 유효하지 않으므로 자세한 내용은 경로 사양을 참조하세요.

계정 및 발행자 필드의 계정 ID는 길이 접두사 없이 표시됩니다. 화폐가 XRP인 경우 화폐 코드는 160비트의 0으로 표시됩니다.

각 단계 뒤에는 경로의 다음 단계가 바로 이어집니다. 위에서 설명한 것처럼 경로의 마지막 단계 뒤에는 0xff(다른 경로가 이어지는 경우) 또는 0x00(마지막 경로가 끝나는 경우)이 뒤따릅니다.

다음 예제는 PathSet의 직렬화 형식을 보여줍니다:

<figure><img src="https://xrpl.org/img/serialization-pathset.svg" alt=""><figcaption></figcaption></figure>

## UInt 필드

XRP Ledger에는 몇 가지 부호 없는 정수 유형이 있습니다: UInt8, UInt16, UInt32, UInt64입니다. 이들 모두는 지정된 비트 수를 가진 표준 빅 엔디안 이진 부호 없는 정수입니다.

이러한 필드를 JSON 객체로 표현할 때 대부분은 기본적으로 JSON 숫자로 표현됩니다. 한 가지 예외가 있는데, 일부 JSON 디코더는 이러한 정수를 64비트 "배정밀도" 부동 소수점 숫자로 표현하려고 시도할 수 있기 때문에 UInt64는 문자열로 표현되며, 이는 모든 고유한 UInt64 값을 전체 정밀도로 표현할 수 없기 때문에 예외입니다.

또 다른 특별한 경우는 트랜잭션 유형 필드입니다. JSON에서 이 필드는 일반적으로 트랜잭션 유형의 이름이 포함된 문자열로 표시되지만, 바이너리에서는 이 필드가 UInt16입니다. 정의 파일의 TRANSACTION\_TYPES 객체는 이러한 문자열을 특정 숫자 값에 매핑합니다.
