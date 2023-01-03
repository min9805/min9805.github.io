---
title: "[Network] MAC Basic"
excerpt: "Network Basic. 네트워크의 기본 용어들과 형태들에 대한 설명"

categories:
  - Network
tags:
  - [Network, Message, Protocol, Topology]

toc: true
toc_sticky: true

date: 2023-01-03
last_modified_at: 2023-01-03
---

# Basic Terminologies

MAC(Media Acess Control) 은 OSI 7 계층 중에서 2 계층에 해당하는 Data Link 의 일부이다. 알아보기 앞서 컴퓨터 네트워크를 이해하기 위한 최소 다섯의 기본 단어들이 존재한다. 먼저 단어를 알아보고 시작하자!

1. Message
   메세지는 통신을 위해 전달하는 정보이다. 당연히 네트워킹을 하는 목적 자체가 데이터를 주고 받기 위함이고 주고받는 데이터를 메세지라 부른다. 보통 패킷, 데이터프레임 형태이며 나중에 자세히 설명한다.
2. Sender
   위에서 데이터를 주고받는다 말했으면 데이터를 주는 주체가 필요하다. 메세지를 송신하는 쪽 혹은 장치, 디바이스를 Sender 라 한다.
3. Receiver
   말 그대로 Sender 가 보낸 데이터를 받는 쪽 혹은 장치이다.
4. Medium
   1 계층에 해당하는 통신 매체이다. 1계층은 Physical 이므로 실제 메세지가 이동할 수 있는 물리적인 경로를 말한다. 실제 유선의 전화선이나 케이블이 될 수도 있고 무선의 주파수가 될 수도 있다
5. Protocol
   메세지를 주고 받기 위해 모두가 따르는 규약이다.

## Data Flow Direction

> 컴퓨터 네트워크에서 정보를 주고 받는 방향별 명칭

1. Simplex
   A 에서 B 로 일방적으로 신호를 보내는 경우이다. TV 같은 경우가 케이블을 통해 수신을 받기만 하므로 Simplex 에 해당한다.
2. Half-duplex
   양 방향으로 메세지를 주고 받을 수 있지만 동시에 주고받을 수 없는 경우이다. 송신자와 수신자의 역할은 각 시점에 하나씩만 존재해야하고 송수신이 동시에 불가능하다. 무전기같은 경우 버튼을 눌렀을 때만 수신이 가능하기에 이에 해당한다.
3. Full-duplex
   송수신이 동시에 가능한 경우이다. 흔히 사용하는 전화가 이에 해당한다. 이외에도 현재의 게임같은 대부분 인터넷 통신은 이에 해당한다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/56664567/210301849-59c94483-0912-4a10-bf66-bb270ed1f48a.png">

## Physical Structure

> 컴퓨터 네트워크에서 Medium 연결 방식에 대한 명칭

1. Point-to-point
   점에서 점으로 한 줄을 연결한다. Station 사이의 고유의 연결 방식이다.
2. Multipoint
   줄 하나의 여러 Station 이 존재하는 경우이다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/56664567/210301836-10fd2851-0dc1-4931-8360-e84993635f1

## Physical Topology

> 컴퓨터 네트워크의 대표적인 연결 형태

### Mesh

각 Station 이 다른 Station 에 대한 고유의 줄을 가지고 있다. 모든 줄이 Point-to-point 연결 방식을 가지고 있는 것이다. 모든 Station 이 다른 모든 Station 에 대해서 연결되어있다면 Full-mesh 라 한다. 이는 전용 줄을 가지고 있기 때문에 방해도 없고 보안도 매우 좋다. 하나의 줄이 끊어졌을 경우 다른 Station 을 통해서 메세지를 전달할 수 있기 때문에 매우 안정적이다. 하지만 Station 의 개수가 많아질 수록 물리적으로 매우 힘들기 때문에 잘 사용하지 않는다. 네트워크끼리 연결하는 경우 사용할 수 있다.
<img width="300" alt="image" src="https://user-images.githubusercontent.com/56664567/210302019-b7d42cd2-08e5-4275-b75f-324bd8d2965a.png">

### Star

허브라는 중앙 장치에서 각 Station 으로 뻗어나가는 형태이다. Station 이 늘어나더라도 각 Station 은 허브로 연결하는 하나의 줄만 가지면 되기 때문에 무리없이 Station 을 늘릴 수 있다. 첫 번째 Station 의 줄이 끊어지더라도 두 번째 Station 에게는 영향이 없기 때문에 안정적이라고 할 수 있다. 단 허브가 고장나면 모든 Station 이 동작할 수 없다.
<img width="300" alt="image" src="https://user-images.githubusercontent.com/56664567/210302026-c7d95027-4dba-42c5-8298-267d6b6bcc8b.png">

### Bus

하나의 줄에 Station 이 줄을 뻗어 연결되어있는 형태이다. 설치가 용이하고 줄이 적어 비용이 적다. 하지만 메인 줄 중에서 어느 한 부분이 끊어지게 되면 전체가 동작할 수 없다.
<img width="300" alt="image" src="https://user-images.githubusercontent.com/56664567/210302036-7cc7ec05-3ba8-4500-8ede-02d8ee8fd3bc.png">

### Ring

Bus 방식에서 양 끝을 둥글게 연결한 형태이다. Bus 의 장점을 그대로 가지고 있으며 한 부분이 끊어지더라도 그 방향에 대한 방어 시스템이 존재한다면 반대 방향으로 통신할 수 있다. 하지만 결국 줄 하나를 여러 Station 이 경쟁적으로 상요하기 때문에 충돌이 발생할 가능성이 높다
<img width="300" alt="image" src="https://user-images.githubusercontent.com/56664567/210302043-eeb2621a-525f-4ecd-9a28-41cf983ecacd.png">

### Hybrid

Star + Bus 라 든지 다양하게 조합해서 네트워크의 성능을 높이고 안정성을 높일 수 있다.
<img width="300" alt="image" src="https://user-images.githubusercontent.com/56664567/210302637-3e8d114e-67d8-4f11-880b-253d480ce165.png">

## Categories of Networks

> 네트워크 규모에 대한 명칭

1. PAN/BAN
   Personal Area Network, Body Area Network. 블루투스로 사람을 기준으로 형성한 네트워크이다. 매우 짧은 연결 거리를 가지고 있다.
2. LAN
   Local Area Network. 한 방 혹은 건물에서 연결되는 네트워크이다. 아직은 짧은 거리를 연결한다. 스타크래프트에서 LAN 이 LAN 네트워크를 사용한 연결이였다..!
3. MAN
   Metropolitan Area Network. 이름 그대로 도시만한 규모에서 이루어지는 네트워크이다. LAN 들이 모여 MAN 을 형성한다.
4. WAN
   Wide Area Netwrok. LAN-LAN 혹은 MAN-MAN 을 연결하는 전용 줄이다.
