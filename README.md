# ARP_spoofing
## 개념 (ARP개념은 알고 있다고 가정하고 설명.)
- <a href="https://github.com/hanmin0512/Concept_ARP"> ARP 개념 학습하기</a>
- 피해자(victim)의 ARP Cache Table을 의도적으로 감염시켜 피해자 패킷을 훔친다(sniff)
- 기본적으로 ARP Spoofing 과 sniffing은 같이 이루어저야 시너지가 좋다.
- 하나의 네트워크에서는 호스트들 끼리 한번 이상 통신을 했다면 ARP Cache Table에 저장을하고 주기적으로 ARP요청 패킷을 보내 갱신을 한다.
- 이때 변조된 ARP 패킷을 보내서 통신을 가로챌 수 있다.

## 원리
- 호스트 A(Sender)와 B(Receiver)가 통신을 하려고한다. 호스트 A는 빠른 통신을 위해 주기적으로 ARP Cache Table을 갱신을 한다.
- 중간에 malicious(악의적인 사용자)가 호스트 A에게 변조된 ARP 패킷을 지속적으로 보낸다.
- 변조된 패킷을 받은 호스트 A의 ARP Cache Table은 변조되었다.
- 호스트 A는 호스트 B에게 데이터를 보낸다면.
- 호스트 A는 데이터 전송시 호스트 A의 ARP Cache Table을 참조하여 데이터를 전송하기 때문에 malicious에게 데이터가 전송된다.

## 어떤 방식으로 ARP 패킷을 변조하여 ARP Spoofing 공격을 할까?
- 가정 : 호스트 A, malicious는 같은 wifi에서 통신을 하고 있다.
- 1. nmap을 사용하여 같은 네트워크상에 있는 호스트들을 검색한다.
  2. Target은 A로 설정한다. Gateway(192.168.0.1) MAC 주소도 알아낸다.
  3. Target A는 외부 네트워크와 통신을 하기위해는 Gateway를 거쳐서 외부와 통신을 할 수 있다(NAT방식)
  4. 변조1 : IP : 192.168.0.1 && MAC : malicious Mac주소 를 기입한다 (Gateway의 MAC주소를 malicious의 MAC주소로 바꿔서 호스트 A가 외부와 통신할때 데이터를 모두 malicious가 받게 된다.)
  5. 변조2 : IP : Target_A_IP && MAC : malicious MAC주소 를 기입한다 (외부에서 Gateway로 들어온 데이터를 malicious가 받게된다. )
  6. 호스트 A는 변조된 패킷을 받아 ARP Cache Table을 갱신한다(Poisoning)
  7. 호스트 A는 외부와 통신을 하기위해 패킷을 전송하지만, malicious에게 패킷을 보내게 된다.
  8. malicious는 받은 패킷을 Gateway로 보낸다.
  9. Gateway는 인터넷으로 패킷을 라우팅한다.
  10. Gateway는 외부로부터 목적지가 호스트 A인 패킷을 받아 malicious에게 전달한다.

## ARP Spoofing 구현하기
- nmap 툴을 사용한다.
- python으로 구현한다.
- scapy 모듈 사용한다.
- WireShark를 이용하여 통신하는 패킷들을 확인한다.

## 실제 구현 로직
- Target_A의 IP정보를 갖고 ARP 요청 패킷을 Broadcast주소로 전송한다.
- Target_A으로 부터 유니케스트 통신으로 받은 패킷을 통해 Target_A의 MAC주소를 알아낸다.
- Gateway MAC주소를 알아내기 위해 Gateway IP정보를 갖고 ARP 패킷을 Broadcast 주소로 전송한다.
- Gateway로 부터 유니케스트 통신으로 받은 패킷을 통해 gateway MAC주소를 알아낸다.
- scapy 모듈을 이용하여 Operation Code가 2인 (reply packet) 을 변조하여 만든다
- 변조1 : ARP reply packet[src_IP = GatewayIP, src_MAC = malicious_MAC, dst_IP = Target_A_IP, dst_MAC = Target_A_MAC] <br>
  즉 Target_A에게 Gateway의 MAC주소를 malicious의 MAC주소로 변조한 패킷을 전송하는 것이다. <br>
  변조된 패킷은 Gateway가 자신의 MAC주소를 기입하여 보낸것처럼 보여진다.
- 변조2 : ARP reply packet[src_IP = Target_A_IP, src_MAC = malicious_MAC, dst_IP = Gateway_IP, dst_MAC = Gateway_MAC] <br>
  즉 외부로부터 Target_A가 목적지인 패킷을 받을 때 Gateway를 거쳐야 하므로 Gateway의 ARP Cache Table에 Target_A의 MAC주소를 malicious MAC주소로 변조시켜 <br>
  외부로부터 들어온 데이터를 malicious가 받도록 하기위한 변조 패킷이다.

## 실행 환경
- victim computer : window11
- malicious computer : macOS
- gateway : iptime(wifi)

## 실행 화면
> 피해자(victim)컴퓨터 IP 및 ARP Cache Table
![windowsIP](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/d0e344fa-71ef-4fe5-a3f7-97a1392f23e2)
![windows_arp_cache_table](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/e00d88a7-8ba9-4b1a-b6de-da39c577b7e6)

> 파이썬 스크립트 arp_spoofing.py 실행 <br>
![스크린샷 2023-07-23 오전 1 54 00](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/8dde4388-c2ec-4c51-a77f-55980ead4cbb)

> 피해자(victim)의 감염된 ARP Cache Table <br>
![poisoning_arp_cache_table_win](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/e64714c6-d1a0-4953-84bd-04f0d20b817e)

> 피해자(victim)가 받은 자신의 MAC주소를 묻는 패킷 구현로직1,2 해당
![who_is_20_victim](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/ade67e69-2dd3-47d0-af14-2ad8840a78f1)

> 피해자(victim)가 자신의 MAC주소를 기입하여 공격자에게 유니캐스팅 
![unicasting_vicitim_to_malicious](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/fe25498e-779f-4db4-9615-3ddb40a7a973)

> Gateway MAC주소를 알기위한 요청 패킷
![who_is_gateway](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/681368c1-5fe4-434a-80c7-c5d7928068f5)

> Gateway MAC주소를 malicious(공격자)에게 전송
![스크린샷 2023-07-23 오전 2 07 54](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/d938d722-a53c-4059-bead-293dc0f4470b)

> 피해자 컴퓨터가 받은 Gateway의 MAC주소가 공격자 MAC주소로 변조한 패킷
 ![스크린샷 2023-07-23 오전 2 09 03](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/17a8a438-4f80-41a4-a9cf-a110d4b6f79e)

> Gateway가 받은 피해자 컴퓨터 MAC주소가 공격자 MAC주소로 변조된 패킷
![스크린샷 2023-07-23 오전 2 09 37](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/b3146e8c-8baf-450d-add2-e8eb575da78b)

## 그림으로 표현

> 감염시킬 Gateway와 victim의 MAC주소를 요청하는 ARP request요청 패킷이 담긴 프레임을 Broadcast에 전송
![ARP_request](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/83a627d5-3c92-45e6-8dfe-73bce798936f)

> Broadcast는 같은 네트워크에 존재하는 호스트들에게 ARP 요청 패킷을 Broadcasting한다.
![Arp_reply](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/e1a76db2-502a-492d-9e97-55d6995ee923)

> ARP요청 패킷에 해당하는 호스트들은 자신의 MAC주소를 유니캐스팅으로 알려준다.
![getMAC](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/f8d613fe-69dc-483e-9c74-29558b0a5c08)

> Malicious 호스트는 Victim에 변조된 ARP패킷이 담긴 프레임을 전송한다.
![Victim_poisoning](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/d66d5315-75c1-40d9-aa68-e6b44b6b5da1)

> Malicious 호스트는 Gateway에 변조된 ARP패킷이 담긴 프레임을 전송한다.
![Gateway_poisoning](https://github.com/hanmin0512/ARP_spoofing/assets/37041208/3ff48402-0fbb-411f-b276-fb55bd3aa0a0)



## 결론
- ARP Spoofing은 MITM공격하기전 사전작업에 해당하는 네트워크 공격 기법이다.
- ARP Spoofing은 컴퓨터간의 통신을 막을 수 있어 Dos공격이 될 수도 있다.


