# 컨센서스 소개

_컨센서스_는 분산된 결제 시스템의 가장 중요한 특성입니다. 기존의 중앙집중식 결제 시스템에서는 권위 있는 관리자가 결제가 어떻게 이루어지는지와 언제 이루어지는지에 대한 최종 결정을 내립니다. 분산 시스템은 말 그대로 관리자가 없습니다. 따라서, XRP Ledger와 같은 분산 시스템은 모든 참여자가 따르는 일련의 규칙을 정의하여 그들이 어떤 시점에서든 정확히 동일한 사건들의 연속과 결과에 동의할 수 있게 합니다. 이러한 규칙의 집합을 _컨센서스 프로토콜_이라고 합니다.

## 컨센서스  프로토콜 속성

XRP Ledger는 그 이전의 어떤 디지털 자산과도 다른 컨센서스 프로토콜을 사용합니다. 이 프로토콜은 XRP Ledger 컨센서스 프로토콜이라고 알려져 있으며, 다음과 같은 중요한 속성을 갖도록 설계되었습니다:

* XRP Ledger를 사용하는 모든 사람들이 최신 상태와 어떤 트랜잭션이 어떤 순서로 발생했는지에 대해 동의할 수 있습니다.&#x20;
* 모든 유효한 트랜잭션이 중앙 운영자 없이, 또는 단일 실패 지점 없이 처리됩니다.
* 참여자들이 일부 참여하거나, 떠나거나, 부적절하게 행동하더라도 ledger는 진행할 수 있습니다.
* 너무 많은 참여자들이 접속할 수 없거나 부적절하게 행동한다면, 네트워크는   무효한 트랜잭션을 분기하거나 확인하는 대신 진행을 실패합니다.
* 트랜잭션을 확인하는 것은 대부분의 다른 블록체인 시스템과 달리 낭비적이거나 경쟁적인 자원 사용을 요구하지 않습니다.

이러한 속성들은 때때로 다음의 원칙으로 요약됩니다: **정확성, 컨센서스, 진전.**

이 프로토콜은 아직 진화 중이며, 그 한계와 가능한 실패 케이스에 대한 우리의 지식도 진화 중입니다. 프로토콜 자체에 대한 학문적 연구를 위해서는 [컨센서스 연구](../undefined-4/undefined-9.md)를 참조하십시오.

## 배경

컨센서스 프로토콜은 _이중 지불 문제_에 대한 해결책입니다: 이중 지불이란, 같은 디지털 돈을 두 번 성공적으로 사용하는 것을 방지하는 것을 말합니다. 이 문제에서 가장 어려운 부분은 트랜잭션을 순서대로 정렬하는 것입니다: 중앙 권위 없이, 두 개 혹은 그 이상의 상호 배타적인 트랜잭션이 거의 동시에 보내졌을 때 어떤 트랜잭션이 먼저인지에 대한 분쟁을 해결하는 것이 어려울 수 있습니다. 이중 지불문제에 대한 자세한 분석, XRP Ledger 컨센서스 프로토콜이 이 문제를 어떻게 해결하는지, 그리고 관련된 타협점과 한계에 대해서는 [컨센서스 원칙과 규칙](../undefined-4/undefined-1.md)을 참조하세요.

## Ledger 역사

XRP Ledger는 "ledger versions" 또는 "ledgers"라고 불리는 블록에서 트랜잭션을 처리합니다. 각 ledger version은 세 가지 부분을 포함합니다:

* ledger에 저장된 모든 잔액과 객체의 현재 상태.
* 이전 ledger에 적용된 트랜잭션 집합으로 이번 ledger가 생성됩니다.
* 현재 ledger 버전에 대한 메타데이터는 ledger 인덱스, 내용을 고유하게 식별하는 [암호화 해시](https://en.wikipedia.org/wiki/Cryptographic\_hash\_function), 그리고 이 ledger를 만들기 위한 기반으로 사용된 부모 ledger에 대한 정보 등을 포함합니다.

<figure><img src="../../.gitbook/assets/Introduction to Consensus_1.png" alt=""><figcaption></figcaption></figure>

각 ledger 버전은 _ledger 인덱스_로 번호가 매겨지고, 인덱스가 하나 작은 이전 ledger 버전을 기반으로 구축됩니다. 이는 ledger 인덱스가 1인 시작점, _제네시스 ledger_로 거슬러 올라갑니다.[`1`](undefined.md#undefined-4) 비트코인과 다른 블록체인 기술과 같이, 이는 모든 트랜잭션과 그 결과에 대한 공개 기록을 형성합니다. 그러나 많은 블록체인 기술들과 달리, XRP Ledger의 각 새 "블록"은 현재 상태의 전체를 포함하므로, 현재 무슨 일이 일어나고 있는지 알기 위해 전체 기록을 모으는 것이 필요하지 않습니다.[2](undefined.md#undefined-4)

XRP Ledger 컨센서스 프로토콜의 주요 목표는 다음 Ledger 버전에 추가할 트랜잭션 집합에 대해 동의하고, 잘 정의된 순서로 적용한 다음, 모두가 동일한 결과를 얻었는지 확인하는 것입니다. 이것이 성공적으로 이루어지면, Ledger 버전이 _유효_하고 최종적인 것으로 간주됩니다. 이후 ledger 버전을 구축함으로써 과정이 계속됩니다.

## 신뢰 기반 검증

XRP Ledger의 컨센서스 메커니즘의 핵심 원칙은 작은 신뢰가 멀리 간다는 것입니다. 네트워크의 각 참여자는 [컨센서스에 적극적으로 참여하기 위해 특별히 구성된 서버](../../tutorials/rippled/rippled-1/rippled.md)인 _검증인_들의 집합을 선택합니다. 이들 검증인은 대부분의 경우에 프로토콜에 따라 정직하게 동작할 것으로 예상되는 서로 다른 당사자들이 운영합니다. 더 중요한 것은 선택된 검증인들이 서로 공모하여 정확히 같은 방식으로 규칙을 어기려 하지 않도록 하는 것입니다. 이 목록을 _고유 노드 목록_(UNL)이라고 합니다.

네트워크가 진행됨에 따라 각 서버는 신뢰할 수 있는 검증인들의 의견을 듣습니다[3](undefined.md#undefined-4); 신뢰할 수 있는 검증인 중 충분한 비율 트랜잭션 집합이 발생해야 하며 지정된 ledger가 결과라는 데 동의하는 한 서버는 컨센서스를 선언합니다. 의견이 일치하지 않을 경우, 검증인은 자신이 신뢰하는 다른 검증인과 더 밀접하게 일치하도록 제안을 수정하고 컨센서스에 이를 때까지 여러 라운드에서 프로세스를 반복합니다.

<figure><img src="../../.gitbook/assets/Introduction to Consensus_2.png" alt=""><figcaption></figcaption></figure>

검증인의 소수 비율이 항상 제대로 작동하지 않는다 해도 괜찮습니다. 신뢰하는 검증인 중 20% 미만이 잘못되었다면, 컨센서스는 원활하게 계속될 수 있으며; 무효한 트랜잭션을 확인하려면 신뢰하는 검증인의 80% 이상이 결탁해야 합니다. 만약 신뢰하는 검증인 중 20% 이상 80% 미만이 잘못되었다면, 네트워크는 진행을 멈춥니다.

XRP Ledger 컨센서스 프로토콜이 다양한 도전, 공격, 그리고 실패 케이스에 어떻게 대응하는지에 대한 더 깊은 탐색을 원하신다면, [공격과 실패 모드에 대한 컨센서스 보호를 참조](../undefined-4/undefined-2.md)하세요.&#x20;

## 참고

* [Consensus Network Concepts](../undefined-4/): XRP Ledger 컨센서스 프로토콜의 작동 메커니즘을 자세히 설명하는 여러 기사를 참조하세요.&#x20;
* [Run rippled as a Validator](../../tutorials/rippled/rippled-1/rippled.md): 직접 검증인을 운영하는 방법에 대한 안내입니다.&#x20;
* [Decentralization Strategy Update (Ripple Dev Blog)](https://xrpl.org/blog/2017/decent-strategy-update.html): XRP Ledger의 탈중앙화를 위한 Ripple의 목표와 계획에 대한 설명을 확인하세요.

## 각주

1. XRP Ledger의 초기 역사에서의 작은 실수로 인해 [1에서 32569까지의 ledger가 손실되었습니다](http://web.archive.org/web/20171211225452/https://forum.ripple.com/viewtopic.php?f=2\&t=3613). (이 손실은 대략 첫 주의 ledger 역사를 나타냅니다.) 따라서, ledger #32570이 가장 초기의 사용 가능한 ledger입니다. XRP Ledger의 상태는 모든 ledger 버전에 기록되므로, ledger는 누락된 역사 없이도 계속될 수 있습니다. 새로운 테스트 네트워크는 여전히 ledger 인덱스 1부터 시작합니다.
2. 비트코인에서는 현재 상태를 "UTXOs" (사용하지 않은 트랜잭션 출력)라고 부르기도 합니다. XRP Ledger와 달리, 비트코인 서버는 UTXO 전체 세트를 알고 새로운 트랜잭션을 처리하기 위해 전체 트랜잭션 이력을 다운로드해야 합니다. 2018년에는 비트코인의 컨센서스 메커니즘을 수정하여 최신 UTXO를 주기적으로 요약하도록 하는 몇 가지 제안이 있었습니다. 이렇게 하면 새 서버가 이 작업을 수행할 필요가 없습니다. 이더리움은 각 블록에 현재 상태의 요약(_state root_ 라고 함)을 포함하는 XRP Ledger와 유사한 접근법을 사용하지만, 이더리움은 대량의 상태 데이터를 저장하기 때문에 동기화하는 데 더 오래 걸립니다.
3. 서버는 신뢰하는 검증인들로부터 메시지를 듣기 위해 그들과의 직접적인 연결을 필요로 하지 않습니다. XRP Ledger의 P2P 네트워크는 공개 키로 서버를 식별하고 다른 사람들의 디지털 서명 메시지를 중계하는 _gossip protocol_을 사용합니다.
