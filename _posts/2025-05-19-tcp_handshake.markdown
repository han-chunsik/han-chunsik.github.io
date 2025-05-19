---
layout: post
title: "3-way handshake 와 4-way Handshake"
date: 2025-05-14
categories: [network]
description: 3번 흔들어 악수하기, 4번 흔들어 악수하기
comments: true
bird_image: "crested-auklet.webp"
bird_name: "뿔바다쇠오리 (Crested Auklet)"
bird_scientific_name: "Aethia cristatella"
bird_description: "뿔바다쇠오리는 북태평양의 바위 절벽에 집단 번식을 하는 바다새로, 짙은 회색 깃털과 이마에서 위로 솟은 깃털 장식, 그리고 밝은 주황색 부리를 가진 것이 특징이다. 사회적 행동이 활발하며, 번식기에는 머리 깃털과 냄새로 짝을 유인한다."
---

TCP(Transmission Control Protocol)는 애플리케이션 간 신뢰성 있는 데이터 전송을 보장하는 연결 지향형(Connection-oriented) 프로토콜이다. 통신을 시작하기 전에 서로 연결을 설정하고, 끝나면 정상적으로 종료하는 과정이 필요하다. 이러한 연결 설정과 해제 과정에서 사용하는 절차가 각각 3-way handshake와 4-way handshake이다.

# 3-way Handshake
## 정의
3-way handshake는 TCP 연결을 수립할 때 사용하는 통신 패턴으로, 클라이언트와 서버가 서로의 존재를 확인하고, SEQ 번호를 교환하며, 연결 준비가 되었음을 확인한다.

## 단계
1. SYN (Synchronize): 클라이언트가 서버에 연결 요청을 보냄
- SEQ 번호(SEQ=X) 포함    
   → 클라이언트 상태: SYN_SENT  
   → 서버 상태: LISTEN   
  
2. SYN + ACK (Synchronize + Acknowledgement): 서버가 SYN을 받고 수락하며 응답
- 클라이언트의 SEQ+1을 ACK로 보내고, 자신의 SEQ도 함께 전송  
   → 클라이언트 상태: SYN_SENT    
   → 서버 상태: SYN_RECEIVED    
  
3. ACK (Acknowledgement): 클라이언트가 서버의 응답을 확인했다는 ACK 전송
- 서버의 SEQ+1을 ACK로 전송  
   → 클라이언트 상태: ESTABLISHED  
   → 서버 상태: ESTABLISHED 

# 4-way Handshake
## 정의
4-way handshake는 TCP 연결을 종료할 때 사용하는 종료 절차이다. TCP는 양방향(Full Duplex) 통신을 지원하기 때문에, 각 방향을 독립적으로 종료해야 하므로 4단계가 필요하다.

## 단계
1. FIN (클라이언트 → 서버): 클라이언트가 연결 종료 요청  
   → 클라이언트 상태: FIN_WAIT_1  
   → 서버 상태: ESTABLISHED  
  
2. ACK (서버 → 클라이언트): 서버가 FIN 수신 후 ACK 전송  
   → 클라이언트 상태: FIN_WAIT_2  
   → 서버 상태: CLOSE_WAIT  
  
3. FIN (서버 → 클라이언트): 서버도 종료 요청  
   → 클라이언트 상태: FIN_WAIT_2  
   → 서버 상태: LAST_ACK  
  
4. ACK (클라이언트 → 서버): 클라이언트가 서버의 FIN에 응답  
   → 클라이언트 상태: TIME_WAIT → (일정 시간 후) CLOSED  
   → 서버 상태: CLOSED  

> ### TIME_WAIT 필요성
> ```
> 클라이언트: 종료 요청할게 (FIN)  
> 서버: 알겠어! (ACK)  
> 서버: 나도 이제 종료할게 (FIN)  
> 클라이언트: 알겠어! (ACK) 그런데 혹시 내가 보낸 마지막 데이터가 아직 네트워크 어딘가에 남아 있을 수 있으니까 잠깐 기다렸다가 완전히 닫을게 (TIME_WAIT)  
> ```
> - FIN 재전송 대응: 서버가 마지막 FIN에 대한 ACK를 못 받았을 때, 다시 FIN을 보내면 클라이언트가 그에 응답할 수 있도록 하기 위함
> - 지연된 패킷 처리: 서버가 FIN 전에 보낸 지연된 데이터 패킷이 나중에 도착할 수 있는데, 이걸 기존 연결로 정상 처리하고, 새 연결과 혼동되지 않게 하기 위함
>  

# 취약점: SYN Flood 공격
3-way handshake의 구조적 특성을 악용한 공격으로 공격자가 SYN 요청만 보내고 ACK를 보내지 않을 경우, 서버는 연결을 대기 상태(SYN_RECEIVED)로 유지하며 메모리와 큐를 점유
위와 같은 상황이 반복되면 정상 연결도 못 받는 DoS 상태가 발생한다.

## 대응 방법
- SYN Cookie: 서버가 연결 요청을 메모리에 저장하지 않고, 응답 시 SEQ 번호에 암호화된 연결 정보를 포함시켜 저장공간을 사용하지 않도록 하는 기술
- Connection Rate 제한: 초당 SYN 요청 수 제한 설정 (IPTables의 --limit 모듈, Nginx의 limit_conn 등)
- TCP Proxy 도입: 요청 앞단에서 먼저 handshake 처리

# TCP에서 SEQ / ACK 번호가 필요한 이유   
**SEQ(SEQ 번호)**: 내가 보내는 데이터가 전체 중 어디서부터 시작되는지를 나타냄   
**ACK(응답 번호)**: 내가 상대방의 데이터를 어디까지 받았는지를 알려줌 (상대 SEQ + 1)  
  
- 데이터 순서 보장: 패킷이 뒤죽박죽 와도 순서대로 정렬 가능  
- 손실 감지 및 재전송: ACK가 안 오면 TCP가 자동으로 재전송  
- 중복 제거: 같은 데이터가 여러 번 와도 구분 가능  
- 정상적인 연결 수립/종료 관리: 3-way / 4-way handshake에서도 정확한 번호 교환이 필수  

> <div style="overflow-x: auto;">
>   <h3><strong>SEQ / ACK 번호 변화 예시</strong> (임의의 숫자이며 실제로는 ISN(Initial Sequence Number)이 랜덤하게 생성됨)</h3>
>   <ul>
>     <li>SYN/FIN의 경우 데이터는 없지만 1바이트 데이터처럼 간주하여 <code>상대 SEQ + 1</code> (클라이언트가 SEQ=100으로 SYN을 보내면, 서버는 ACK=101로 응답)</li>
>     <li>실제 데이터 전송의 경우, 전송한 바이트 수만큼 <code>상대 SEQ + 전송한 바이트 수</code> (아래 표 HTTP 데이터 전송 4번의 SEQ 참고)</li>
>   </ul>
> 
>   <h4>TCP 3-way Handshake</h4>
>   <table border="1" cellspacing="0" cellpadding="5" style="width: auto;">
>     <thead>
>       <tr>
>         <th>단계</th>
>         <th>출발지 → 목적지</th>
>         <th>Flags</th>
>         <th>SEQ</th>
>         <th>ACK</th>
>         <th>설명</th>
>       </tr>
>     </thead>
>     <tbody>
>       <tr>
>         <td>1</td>
>         <td>클라이언트 → 서버</td>
>         <td>[S]</td>
>         <td>3113377296</td>
>         <td>-</td>
>         <td>클라이언트가 연결 요청 (SYN)</td>
>       </tr>
>       <tr>
>         <td>2</td>
>         <td>서버 → 클라이언트</td>
>         <td>[S.]</td>
>         <td>1171478793</td>
>         <td>3113377297</td>
>         <td>서버가 SYN+ACK 응답</td>
>       </tr>
>       <tr>
>         <td>3</td>
>         <td>클라이언트 → 서버</td>
>         <td>[.]</td>
>         <td>3113377297</td>
>         <td>1171478794</td>
>         <td>서버의 SEQ+1에 대한 ACK 전송</td>
>       </tr>
>     </tbody>
>   </table>
> 
>   <h4>HTTP 데이터 전송</h4>
>   <table border="1" cellspacing="0" cellpadding="5" style="width: auto;">
>     <thead>
>       <tr>
>         <th>단계</th>
>         <th>출발지 → 목적지</th>
>         <th>Flags</th>
>         <th>SEQ</th>
>         <th>ACK</th>
>         <th>설명</th>
>       </tr>
>     </thead>
>     <tbody>
>       <tr>
>         <td>4</td>
>         <td>클라이언트 → 서버</td>
>         <td>[P.]</td>
>         <td>3113377297 → 3113378141</td>
>         <td>1171478794</td>
>         <td>844 바이트 HTTP 요청 전송 (PSH+ACK)</td>
>       </tr>
>       <tr>
>         <td>5</td>
>         <td>서버 → 클라이언트</td>
>         <td>[.]</td>
>         <td>1171478794</td>
>         <td>3113378141</td>
>         <td>요청 데이터 수신 확인 (ACK=마지막+1)</td>
>       </tr>
>     </tbody>
>   </table>
> 
>   <h4>TCP 4-way Handshake</h4>
>   <table border="1" cellspacing="0" cellpadding="5" style="width: auto;">
>     <thead>
>       <tr>
>         <th>단계</th>
>         <th>출발지 → 목적지</th>
>         <th>Flags</th>
>         <th>SEQ</th>
>         <th>ACK</th>
>         <th>설명</th>
>       </tr>
>     </thead>
>     <tbody>
>       <tr>
>         <td>6</td>
>         <td>클라이언트 → 서버</td>
>         <td>[F.]</td>
>         <td>3113378141</td>
>         <td>1171478794</td>
>         <td>클라이언트가 연결 종료 요청</td>
>       </tr>
>       <tr>
>         <td>7</td>
>         <td>서버 → 클라이언트</td>
>         <td>[.]</td>
>         <td>1171478794</td>
>         <td>3113378142</td>
>         <td>서버가 FIN 수신 확인 (ACK 전송)</td>
>       </tr>
>       <tr>
>         <td>8</td>
>         <td>서버 → 클라이언트</td>
>         <td>[F.]</td>
>         <td>1171478795</td>
>         <td>3113378142</td>
>         <td>서버도 종료 요청 (FIN 전송)</td>
>       </tr>
>       <tr>
>         <td>9</td>
>         <td>클라이언트 → 서버</td>
>         <td>[.]</td>
>         <td>3113378142</td>
>         <td>1171478796</td>
>         <td>서버 FIN 수신 확인 (최종 ACK 전송)</td>
>       </tr>
>     </tbody>
>   </table>
> </div>


# 실습 - 터미널에서 확인하기(ephemeral port)
- OS: MacOS
- 클라이언트: `127.0.0.1:60661`
- 서버: `127.0.0.1:8080`

### 1. port 열기
```shell
nc -l 8080 &
```

### 2. port 확인
```shell
lsof -i :8080
```
```
COMMAND   PID     FD   TYPE   SIZE/OFF  NODE NAME
nc        12345   3u   IPv4   0t0       TCP *:8080 (LISTEN)
```

### 3. tcpdump 확인(브라우저에서 `localhost:8080` 접속)
> Loopback 인터페이스(127.0.0.1와 같은 IPv4 루프백 주소를 사용해서 자기 자신에게 네트워크 요청을 하는 것)는 송신과 수신이 모두 캡처되기 때문에 클라이언트와 서버가 동일 호스트일 경우, 동일 세션의 양방향 패킷이 모두 관측됨

```shell
sudo tcpdump -i any tcp port 8080 -n
```

```shell
# 3-way handshake
14:33:34.647984 IP 127.0.0.1.60661 > 127.0.0.1.8080: Flags [S]     # 1. SYN
14:33:34.648219 IP 127.0.0.1.8080 > 127.0.0.1.60661: Flags [S.]    # 2. SYN-ACK  
14:33:34.648268 IP 127.0.0.1.60661 > 127.0.0.1.8080: Flags [.]     # 3. ACK

# 4-way handshake
14:33:36.361347 IP 127.0.0.1.60661 > 127.0.0.1.8080: Flags [F.]    # 1. FIN
14:33:36.361385 IP 127.0.0.1.8080 > 127.0.0.1.60661: Flags [.]     # 2. ACK
14:33:36.361403 IP 127.0.0.1.8080 > 127.0.0.1.60661: Flags [F.]    # 3. FIN
14:33:36.361427 IP 127.0.0.1.60661 > 127.0.0.1.8080: Flags [.]     # 4. ACK
```

> ### 플래그 설명
> #### 3-way handshake
> - [S]   → SYN (클라이언트 → 서버)
> - [S.]  → SYN+ACK (서버 → 클라이언트)
> - [.]   → ACK (클라이언트 → 서버)
> 
> #### 4-way handshake
> - [F.]  → FIN (종료 요청)
> - [.]   → ACK (종료 요청 확인)
> - [F.]  → FIN (상대방 종료 요청)
> - [.]   → ACK (종료 응답)
