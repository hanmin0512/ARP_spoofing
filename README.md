# ARP_spoofing
## 개념
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

## ARP Spoofing 구현하기
- nmap 툴을 사용한다.
- python으로 구현한다.
- scapy 모듈 사용한다.

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

## 실행 화면


