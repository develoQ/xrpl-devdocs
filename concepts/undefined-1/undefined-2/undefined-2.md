# 출발, 데스티네이션 태그

_출발 태그_ 및 _데스티네이션 태그_는 다목적 주소로 송수신되는 [결제](../../../references/xrp-ledger/undefined-1/undefined-1/payment.md)에 대한 특정 목적지 나타낼 수 있는 XRP Ledger 결제의 기능입니다. 출발 및 데스티네이션 태그는 직접 온-ledger 기능이 없습니다. 출발 및 데스티네이션 태그는 오프-ledger 시스템이 결제를 처리하는 방법에 대한 정보만 제공합니다. 트랜잭션에서 출발 태그와 데스티네이션 태그는 모두 32비트 부호 없는 정수로 포맷됩니다.

데스티네이션 태그는 지급 데스티네이션 또는 수취인을 나타냅니다. 예를 들어, [거래소](../../../tutorials/xrp-ledger/xrp.md) 또는 [게이트웨이](../../../tutorials/xrp-ledger/undefined.md) 주소로 결제하는 경우 데스티네이션 태그를 사용하여 해당 비즈니스의 시스템에서 결제 금액에 대해 어떤 고객에게 크레딧을 제공해야 하는지 나타낼 수 있습니다. 판매자에게 지불하면 지불한 물건이나 카트를 표시할 수 있습니다.

출발 태그는 결제의 원본 또는 원본을 나타냅니다. 가장 일반적으로 출발 태그가 포함되어 있어 결제를 받는 사람이 반품 또는 "반송" 결제를 보낼 위치를 알 수 있습니다. 착신 결제를 반환할 때는 착신 결제의 출발 태그를 발신(반품) 결제의 데스티네이션 태그로 사용해야 합니다.

고객에게 다른 인터페이스를 사용하여 XRP Ledger 주소에서 트랜잭션을 보내고 받을 수 있는 기능을 제공하는 것을 호스트 계정 제공이라고 합니다. 호스트 계정은 일반적으로 각 고객에 대해 출발 및 데스티네이션 태그를 사용합니다.

{% hint style="info" %}
Tip:

[X-주소](https://xrpaddress.info/)는 출발 주소와 데스티네이션 태그를 결합하여 하나의 주소로 표시하는 것입니다. 고객에게 입금 주소를 표시할 때는 두 가지 정보를 기억하는 대신 X-주소를 사용하면 고객이 더 편리하게 사용할 수 있습니다. (X-주소의 태그는 송신 시 출발 태그로 작용하고 수신 시 데스티네이션 태그로 작용합니다.)
{% endhint %}

## 이유&#x20;

다른 분산 ledger에서는 각 고객에게 다른 입금 주소를 사용하는 것이 일반적입니다. 하지만 XRP Ledger에서는 주소가 자금을 보유하고 영구적인 계정이어야만 결제를 수신할 수 있습니다. 이 방식을 XRP Ledger에서 사용하면 네트워크의 모든 서버 리소스를 낭비하며, 각 주소마다 예약 금액을 영구히 설정해야 하므로 비용이 많이 듭니다.

출발 태그와 데스티네이션 태그는 개별 고객에게 결제와 입금을 매핑하기 위한 가벼운 방법을 제공합니다.

## 사용 사례&#x20;

비즈니스는 다음과 같은 여러 목으로 출발 태그와 데스티네이션 태그를 사용할 수 있습니다:

* 고객 계정에 직접 매핑하기.&#x20;
* 반송 결제를 원래 결제와 일치시키기.&#x20;
* 만료되는 견적에 결제를 매핑하기.&#x20;
* 특정 입금을 위해 고객이 생성할 수 있는 일회용 태그 제공하기.

개인 정보를 보호하면서 중복을 방지하기 위해 비즈니스는 사용 가능한 전체 태그 범위를 각 목적에 따라 섹션으로 나눈 다음, 해당 범위 내에서 예측할 수 없는 순서로 태그를 할당할 수 있습니다. 예를 들어 [SHA-256](https://en.wikipedia.org/wiki/SHA-2) 과 같은 암호화 해시 함수를 사용한 다음 [모듈로 연산자](https://en.wikipedia.org/wiki/Modulo\_operation)를 사용하여 출력을 관련 섹션의 크기에 매핑할 수 있습니다. 안전을 위해 새 태그를 사용하기 전에 이전 태그와 충돌하는지 확인하세요.

번호 순서대로 태그를 할당하면 고객의 개인 정보를 덜 보호합니다. 모든 XRP Ledger 거래는 공개되므로, 이러한 방식으로 태그를 할당하면 어떤 태그가 어떤 사용자의 주소와 관련되는지를 추측하거나 태그를 사용하여 사용자의 계정에 대한 정보를 유추할 수 있습니다.

## 태그 필요성&#x20;

여러 고객 계정에 대한 지불을 받을 수 있는 XRP Ledger 주소의 경우, 데스티네이션 태그 없이 지불을 받는 것은 문제가 될 수 있습니다. 즉, 어떤 고객을 신용할 것인지가 즉시 명확하지 않기 때문에, 의도된 수취인이 누구인지를 파악하기 위해 수동으로 개입하고 발송인과 논의해야 할 수 있습니다. 이와 같은 경우를 줄이려면 [RequireDest 설정을 활성화](../../../tutorials/undefined-3/undefined-5.md)할 수 있습니다. 그런 식으로, 사용자가 결제에 데스티네이션 태그를 포함하는 것을 잊어버린 경우, XRP Ledger는 당신이 무엇을 해야 할지 모르는 돈을 주는 대신 그들의 결제를 거부합니다. 그런 다음 사용자는 태그를 사용하여 결제를 다시 보낼 수 있습니다.
