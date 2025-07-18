# 📊 이상치 거래 탐지 시스템

## 프로젝트 개요
### 구성원
<table>
  <tr>
    <td align="center">
      <a href="https://github.com/moonstone0514">
        <img src="https://github.com/moonstone0514.png" width="100px;" alt="moonstone0514"/><br />
        <sub><b>김문석</b></sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/Hyunsoo1998">
        <img src="https://github.com/Hyunsoo1998.png" width="100px;" alt="Hyunsoo1998"/><br />
        <sub><b>김현수</b></sub>
      </a>
    </td>
<td align="center">
      <a href="https://github.com/GIHYUN-LEE">
        <img src="https://github.com/GIHYUN-LEE.png" width="100px;" alt="GIHYUN-LEE"/><br />
        <sub><b>이기현</b></sub>
      </a>
    기 |
| **고객번호** | 1XHW7KY657UGWUPFR87X |
| **연령대** | 40 (40~44세) |
| **성별** | 남성 |
| **거주지역** | 전남 |
| **총 이용금액** | 50,000,000원 |
| **업종** | 자동차/연료/정비 > 수리서비스 |

**이유:**  
해당 지역 평균 대비 **수리·튜닝 과다 지출**이 확인되어 **이상치**로 탐지되었습니다.

---

## 📈 Kibana 시각화 결과

![차량 수리비](<./차량 수리비.png>)

위 그래프는 `업종 = 자동차/연료/정비 > 수리서비스` 조건을 적용하여  
**사용자 ID별 차량 수리비 사용 금액**을 시각화한 것입니다.

---

### ✅ 축 설명

- **X축 (가로)** : 사용자 ID값  
  각 고객의 고유 식별값이 나열됩니다.

- **Y축 (세로)** : 차량 수리비  
  해당 고객이 특정 기간 동안 지출한 **차량 수리·튜닝 비용**을 표시합니다.

---

### 📌 그래프에서 읽을 수 있는 포인트

- 대부분의 고객은 차량 수리비 지출이 0~2 수준으로 낮게 나타납니다.
- 그러나 일부 고객 ID에서 **수리비 지출이 급격히 증가**한 피크가 관찰됩니다.
- 특히 오른쪽 끝단의 특정 고객에서 **수십 단위의 지출**이 확인되어,
  지역 평균 대비 **비정상적 과소비(이상치)** 로 탐지됩니다.

---

## ⚡ Kibana를 통한 이상치 거래 탐지

이상치 거래 데이터를 **Kibana**에서 시각화하여 분석하고,  
특정 패턴을 실시간으로 모니터링합니다.  
이를 통해 **금융당국**과 협력하여 **위험 요소를 사전에 차단**하는 데 활용할 수 있습니다.

### 🚀 설치 및 실행

1. **데이터 준비**  
   샘플 데이터를 기반으로 **이상치 거래 데이터**를 생성하여 **Elasticsearch**에 삽입합니다.

2. **Kibana 대시보드 설정**  
   Kibana를 통해 실시간으로 거래 데이터를 시각화하고,  
   **이상치 패턴**을 탐지할 수 있는 대시보드를 구성합니다.

3. **filebeat를 이용한 데이터 수집**
- filebeat.yml 설정

| 수집할 데이터 | 출력 설정 | 필터링 |
| ------------- | ----------| --------|
| 우리카드 데이터<br> (edu_data_F)| logstash | 컬럼명 제거 |


✨ 컬럼명을 제거하는 이유
   - filebeat는 파일을 **라인 단위**(한 줄씩)로 읽어와서 message 필드에 문자열 그대로 담아 전송
   - 컬럼명이 포함된 라인은 분석에 불필요하므로, exclude_lines을 통해 컬럼명이 포함된 라인을 수집 대상에서 제외

<br>

- yml 스크립트

```yml

filebeat.inputs:
- type: log
  enabled: true
  paths:
    - C:\ce5\00.dataSet\edu_data_F.csv
  exclude_lines: ['^컬럼명']

```
<br>

4. **logstash를 이용한 데이터 필터링**

- conf 스크립트

```yml

input {
  beats {
    port => 5044
  }
}

# message 인덱스 5, 7, 8, 9 제외 
filter {
  mutate {
    split => ["message", ","]
    add_field => {
      "컬럼명1"              => "%{[message][0]}"
      "컬럼명2"              => "%{[message][1]}
        ... 
      "컬럼명54"             => "%{[message][3]}"
      "컬럼명55"             => "%{[message][4]}"
    }

# message 필드 삭제
    remove_field => ["ecs", "host", "@version", "agent", "log", "tags", "input", "message"]
  }

  mutate {
    convert => {
      "컬럼명" => "integer"
    }
    remove_field => [ "@timestamp" ]
  }
}

output {
  stdout {
    codec => rubydebug
  }

  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "cardfisa"
  }
}

```

---

## 🚀 프로젝트 목표

1. **이상치 거래 패턴 탐지**: 실시간으로 비정상적인 소비 행태를 탐지합니다.
2. **금융 리스크 관리**: 금융당국과 협력하여 사기 및 불법 자금 유입을 차단합니다.
3. **Kibana 시각화**: 데이터 분석 결과를 **Kibana** 대시보드를 통해 직관적으로 시각화하여 빠르게 대응할 수 있도록 합니다.
