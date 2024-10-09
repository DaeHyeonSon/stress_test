# stress_test
Spring Boot로 만든 간단한 **티켓팅 페이지**에서 **각종 Tool**을 사용하여 티켓팅 서버의 **stress 테스트**를 진행하고자 한다.

## Ticketing API 구현 💡
티켓 생성 및 조회 기능을 제공하는 간단한 RESTful API를 구현한다.

<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/0b4a55c4-a13b-4ed4-bdda-856fb85422fc' width=50%>
</div>

<Br>

application.properties는 다음과 같이 설정해준다.
```java
spring.application.name=Ticketing

spring.datasource.url=jdbc:mysql://localhost:3306/ticketing_db
spring.datasource.username=ID
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```


Controller의 Request를 다음과 같이 설정해준다.
```java
package com.example.ticketing.controller;

import com.example.ticketing.model.Ticket;
import com.example.ticketing.service.TicketService;![스크린샷 2024-10-09 오후 11 59 21](https://github.com/user-attachments/assets/89c1115b-0dd2-4c5f-bc4c-3fca4b721a9d)

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/tickets")
public class TicketController {
    @Autowired
    private TicketService ticketService;

    @GetMapping
    public List<Ticket> getTickets() {
        return ticketService.getAllTickets();
    }

    @PostMapping
    public Ticket createTicket(@RequestBody Ticket ticket) {
        return ticketService.saveTicket(ticket);
    }
}
```

<br>

## JMeter 설정 및 실행 💻
위와 같이 셋팅되어있는 환경에서 부하 테스트를 위해 Jmeter를 설치해준다.

```bash
brew install jmeter
```

부하 테스트를 위한 시나리오는 다음과 같다.

- Thread Group을 설정하여 여러 사용자가 동시에 API에 요청을 보내도록 설정한다.
- HTTP Request 샘플러를 추가하고 API 경로와 요청 데이터를 설정한다.
- View Results Tree를 사용해 테스트 결과를 시각적으로 확인할 수 있다.

테스트를 위한 순서 : <br>

Thread Group 추가 -> HTTP Request 추가 -> HTTP Header Manager 설정 -> Sampler 추가 -> Listener 추가 -> 실행 및 분석

<br>

### Thread Goup 설정

`Number of Threads`
스레드 개수이며, 가상 유저 수라고 생각하면 된다.
HTTP Request Sampler 가 2개일 경우, 스레드를 1로 설정해도, 2번의 request 가 발생한다. 가상 유저수가 HTTP Request Sampler 에 비례한다.

`Ramp-Up`
스레드 당 생성시간으로, 만약 Number of Threads = 100이고, Ramp-Up = 10 라면, 100명의 유저를 생성할 때 까지 10초가 걸린다는 말이다. 즉, 1초 동안 10명의 유저가 요청을 하는것이고, 만약 Ramp-Up = 0 으로 설정하면, 동시 접속 자 수는 100명이 된다.

`Loop Count`
하나의 스레드가 수행할 작업 수이며, 만약 Number of Threads = 100이고, Loop Count = 10 이면, 100명의 유저는 동일한 작업을 10번 수행하게 되며, 총 1000번 (총 요청횟수) 이 수행된다.

<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/b6fabd32-35fb-4905-8285-3ee292c61ebe' width=70%>
</div>

그림과 같이 설정했을 경우 1000명의 유저가 1초동안 3번의 요청을 수행한다는 의미이다.

<br>

※ 초기에는 10-20개 정도의 스레드로 시작해 서버 반응을 확인하고, 점진적으로 늘려가는 것이 좋다. 이렇게 하면 서버의 성능을 평가할 수 있고, 시스템이 과부하되는 것도 예방할 수 있다.

<br>

### HTTP Request 설정

상세 테스트 정보(예: 프로토콜, 서버 IP, 포트, 경로 등)를 입력해야 한다 정보는 다음과 같다.

- 프로토콜: http
- 서버 ip: loaclhost
- port: 8080
- Path: /api/tickets (테스트할때 지정한 PostMapping 경로)

<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/69ed9e0f-f0b7-4771-b099-4ae64b5aa659' width=70%>
</div>

다음 Body data에 정보를 실어 테스트를 진행한다.

### HTTP Header Manager 설정

POST로 요청시에는 HTTP Header 설정을 추가로 해주어야 한다. <br>
**HTTP Request - Config Element - HTTP Header Manager**를 추가하고 아래와 같이 입력 한다.
키:값 으로 `content-type : application/json` 입력

<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/8830ccc0-1040-406a-bfe2-5227178c1aab' width=50%>
</div>

### Listener 추가
sampler가 받아오는 리턴 값을 바탕으로 그래프, Reporting을 만들어주는 listener를 그림과 같이 추가해 준다. 

<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/6a271d83-2fa0-4871-9c01-0fa97bfd3bda' width=50%>
</div>

### 테스트 결과 

- View Results Tree
<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/7e0a2282-1fc1-46e5-94cc-2a084ab22322' width=50%>
</div>

- Summary Report
<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/c224c8a1-caa9-4fe6-86bc-c07151ff5003' width=50%>
</div>

- TPS(Transaction Per Second) 확인
<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/359d3d05-15cd-4a53-a1b4-fc2f30a2fce5' width=50%>
</div>

## stress-ng 도구를 사용한 부하 테스트 🤨
시스템에 부하를 주면서 JMeter로 API 부하 테스트를 병행하고자 한다.

먼저 stress-ng tool 설치를 진행한다.
```bash
brew install stress-ng
```

시스템에 부하를 주는 명령어를 백그라운드에 실행 
```bash
# 4개의 CPU 코어를 120초 동안 부하
stress-ng --cpu 4 --timeout 120s &
```

<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/b9367ee9-a175-4535-b2ab-ea0f5730124e' width=50%>
</div>

그림 같이 임시적으로 부하가 이뤄지는 것을 확인 할 수 있다. 

동시에 JMeter 부하 테스트를 실행합니다
```bash
jmeter -n -t Ticketing.jmx -l Ticetkingresult1.jtl
```

TicketingResult1.jtl에 대한 결과는 다음과 같다.

- View Results Tree
<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/5b2f42e7-e2d5-44a8-9db8-77741545138e' width=50%>
</div>

- Summary Tree
<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/0dbf23a5-d79e-49c0-aaa8-36d50ad655ab' width=50%>
</div>

- TPS
<div align='center'>
  <img src = 'https://github.com/user-attachments/assets/cfb8e369-c0e2-4a9b-b208-e265d2235289' width=50%>
</div>


<br>

## 결론 📖
해당 프로젝트는 Spring Boot 애플리케이션에서 JMeter를 통해 부하 테스트를 진행하는 방법과 함께, stress 및 stress-ng를 활용하여 시스템의 리소스를 의도적으로 사용하여 성능을 극한으로 진행하는 테스트를 진행하였다. 이를 통해 서버 성능 및 안정성을 더 철저히 평가할 수 있을 것이라 기대한다.
