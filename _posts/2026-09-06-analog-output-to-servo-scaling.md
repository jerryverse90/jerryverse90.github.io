---
layout: post
title: "PLC 아날로그 출력 10V인데 서보가 정격 속도까지 안 올라갑니다 — 스케일이 어긋나는 5가지 이유"
series: "XG5000 ↔ 서보 연동 실전"
part: 1
date: 2026-09-06
status: draft
verified: false
tags: [XG5000, XGF-AH6A, 서보, 아날로그출력, 스케일링]
---

<!-- [초안 상태: 검증 전] — 발행 전 검증표 전 항목 체크 필수 -->

<!--
제목 3안
- 증상형: PLC 아날로그 출력 10V인데 서보가 정격 속도까지 안 올라갑니다 — 스케일이 어긋나는 5가지 이유  ← 채택
- 파라미터형: XGF-AH6A 출력 범위·분해능과 서보 속도 지령 게인(Pn300) 맞추기
- 비교형: 서보 속도 지령, 전압 출력과 전류 출력 중 무엇을 쓸까
-->

PLC 아날로그 출력 모듈에서 서보 드라이브로 속도 지령을 내려보내는 구성은 단순해 보이지만, 처음 시운전에서 "지령은 100%인데 속도가 부족하다"는 현상을 자주 만납니다. 이 글은 LS ELECTRIC XGT 계열 아날로그 출력 모듈과 야스카와 Σ-7 서보의 아날로그 속도 지령 연동을 예로, 스케일이 어긋나는 원인을 PLC 쪽과 서보 쪽으로 나누어 정리합니다. 예시는 일반 컨베이어 구동축 1개 기준입니다.

## 1. 증상

시운전 중 다음 중 하나 이상이 나타납니다.

- PLC에서 출력값을 최댓값으로 써도 서보 모니터의 속도가 정격의 60–70%에서 멈춥니다.
- 출력값 0에서 모터가 정지하지 않고 수 rpm으로 천천히 돕니다(드리프트).
- 일정 속도로 운전 중 속도 표시가 ±수십 rpm 폭으로 흔들립니다.
- 반대로 출력값의 절반만 써도 정격 속도에 도달해, 상위 절반이 낭비됩니다.

[경험: 저자가 처음 이 현상을 만났을 때 무엇을 먼저 의심했고, 실제 원인은 무엇이었는지 2–3문장]

## 2. 원인 파라미터

스케일 사슬은 네 단계입니다. 어느 한 단계의 가정이 어긋나면 끝단 속도가 어긋납니다.

```
PLC 데이터값 ──▶ 모듈 출력 전압 ──▶ 서보 입력 전압 ──▶ 모터 속도
 (디지털 범위)     (출력 범위 설정)     (배선·오프셋)      (지령 입력 게인)
```

| 단계 | 항목 | 설정 위치 | 확인할 값 | 상태 |
|---|---|---|---|---|
| ① | 디지털 입력 범위 | XG5000 I/O 파라미터 → 해당 슬롯 아날로그 출력 채널 | 0–16000 / −8000–8000 등 선택값 [값 검증 필요] | 기본값 확인 |
| ② | 출력 범위 | 같은 화면, 채널별 출력 범위 | 0–10 V / −10–+10 V / 1–5 V / 4–20 mA 중 선택 [값 검증 필요] | 서보 V-REF 입력 범위와 일치해야 함 |
| ③ | 서보 지령 입력 범위 | Σ-7S CN1 V-REF (5–6번 핀) | 최대 입력 전압 ±12 V [값 검증 필요] | 배선 극성·차폐 |
| ④ | 속도 지령 입력 게인 | SigmaWin+ → **Pn300** | 기본값 600 (= 6.00 V에서 정격 속도) [값 검증 필요] | 10 V 풀스케일이면 1000으로 변경 검토 |
| ⑤ | 지령 오프셋 | **Pn30x** 계열 자동/수동 오프셋 조정 [값 검증 필요] | 0 V에서 잔류 전압 | 드리프트 원인 |

### 어긋나는 5가지 이유

1. **디지털 범위와 출력 범위를 다르게 가정.** 예: 코드에서 0–4000을 풀스케일로 썼는데 모듈 설정은 0–16000. 출력은 2.5 V에서 멈춥니다.
2. **Pn300 기본값을 모름.** 기본값이 6.00 V/정격속도라면 10 V를 넣었을 때 정격의 약 1.67배를 요구하게 되고, 드라이브는 최대 속도(**Pn316** 등 [값 검증 필요])로 클램프합니다. 반대로 정격 속도가 필요한데 6 V까지만 내보내면 60%에서 멈춥니다.
3. **모듈 출력 범위 선택 오류.** 1–5 V로 설정된 채널에 0–10 V 코드를 쓰면 하한에서 1 V가 남아 드리프트처럼 보입니다.
4. **오프셋 미조정.** 모듈 출력 0에서 수십 mV, 서보 입력 오프셋까지 합쳐지면 저속 회전이 남습니다.
5. **배선 문제.** 차폐 미접지, 트위스트 페어 미사용, 접지 루프. 속도가 흔들리거나 특정 장비 가동 시 튑니다.

## 3. 검증 방법

시뮬레이터에서 값의 흐름을 먼저 확인하고, 실기에서 전압과 속도를 4점 대조합니다.

| 순서 | 방법 | 도구 | 기대 결과 |
|---|---|---|---|
| 1 | 디지털 값 0 / 4000 / 8000 / 16000 을 출력 레지스터에 강제 | XG5000 시뮬레이터 → 모니터 | 채널 출력값이 선형으로 변함 |
| 2 | 같은 4점에서 모듈 단자 전압 측정 | 멀티미터 (DC V) | 0 / 2.5 / 5.0 / 10.0 V ±[허용오차 검증 필요] |
| 3 | 서보 CN1 V-REF 단자에서 같은 4점 측정 | 멀티미터 | 모듈 단자와 차이 수십 mV 이내 |
| 4 | 서보 모니터 속도 지령·피드백 읽기 | SigmaWin+ 모니터 | Pn300 계산값과 일치 (10 V × 정격/6 V 또는 변경값 기준) |
| 5 | 출력 0에서 잔류 전압·잔류 속도 기록 | 멀티미터 + 모니터 | 오프셋 조정 전후 비교 |

**계산식(검증 후 표 작성):** 목표 rpm = 입력 전압[V] ÷ (Pn300 ÷ 100) × 정격 속도[rpm]. Pn300 = 600, 정격 3000 rpm, 입력 10 V이면 5000 rpm 요구 → 최대 속도 클램프.

## 4. 도식

<figure>
<svg viewBox="0 0 760 300" xmlns="http://www.w3.org/2000/svg" font-family="IBM Plex Sans KR, sans-serif" font-size="12">
  <defs><marker id="a" markerWidth="8" markerHeight="8" refX="7" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" fill="#5B6675"/></marker></defs>
  <!-- boxes -->
  <rect x="20" y="40" width="150" height="70" rx="4" fill="#F5F6F8" stroke="#8A94A6"/>
  <text x="95" y="65" text-anchor="middle" font-weight="600" fill="#1B2330">PLC 데이터</text>
  <text x="95" y="85" text-anchor="middle" fill="#5B6675">D 레지스터 0–16000</text>
  <rect x="210" y="40" width="150" height="70" rx="4" fill="#FBEEDD" stroke="#C4731A"/>
  <text x="285" y="65" text-anchor="middle" font-weight="600" fill="#1B2330">아날로그 출력 모듈</text>
  <text x="285" y="85" text-anchor="middle" fill="#5B6675">출력 범위 0–10 V</text>
  <rect x="400" y="40" width="150" height="70" rx="4" fill="#FBEEDD" stroke="#C4731A"/>
  <text x="475" y="65" text-anchor="middle" font-weight="600" fill="#1B2330">서보 V-REF 입력</text>
  <text x="475" y="85" text-anchor="middle" fill="#5B6675">CN1 5(+)/6(−) · Pn300</text>
  <rect x="590" y="40" width="150" height="70" rx="4" fill="#E1F1E9" stroke="#2F7A5A"/>
  <text x="665" y="65" text-anchor="middle" font-weight="600" fill="#1B2330">모터 속도</text>
  <text x="665" y="85" text-anchor="middle" fill="#5B6675">0–정격 rpm</text>
  <line x1="170" y1="75" x2="208" y2="75" stroke="#5B6675" stroke-width="1.4" marker-end="url(#a)"/>
  <line x1="360" y1="75" x2="398" y2="75" stroke="#5B6675" stroke-width="1.4" marker-end="url(#a)"/>
  <line x1="550" y1="75" x2="588" y2="75" stroke="#5B6675" stroke-width="1.4" marker-end="url(#a)"/>
  <text x="189" y="65" text-anchor="middle" fill="#C4731A" font-size="11">①②</text>
  <text x="379" y="65" text-anchor="middle" fill="#C4731A" font-size="11">③⑤</text>
  <text x="569" y="65" text-anchor="middle" fill="#C4731A" font-size="11">④</text>
  <!-- mapping graph -->
  <line x1="60" y1="270" x2="60" y2="140" stroke="#8A94A6"/>
  <line x1="60" y1="270" x2="360" y2="270" stroke="#8A94A6"/>
  <text x="210" y="292" text-anchor="middle" fill="#5B6675" font-size="11">입력 전압 [V]</text>
  <text x="18" y="205" text-anchor="middle" fill="#5B6675" font-size="11" transform="rotate(-90 18 205)">속도 [% 정격]</text>
  <text x="60" y="284" text-anchor="middle" fill="#5B6675" font-size="10">0</text>
  <text x="240" y="284" text-anchor="middle" fill="#5B6675" font-size="10">6</text>
  <text x="360" y="284" text-anchor="middle" fill="#5B6675" font-size="10">10</text>
  <text x="50" y="273" text-anchor="end" fill="#5B6675" font-size="10">0</text>
  <text x="50" y="197" text-anchor="end" fill="#5B6675" font-size="10">100</text>
  <line x1="60" y1="193" x2="360" y2="193" stroke="#C4731A" stroke-dasharray="3 3"/>
  <text x="365" y="197" fill="#C4731A" font-size="10">최대속도 클램프</text>
  <!-- Pn300=600 -->
  <polyline points="60,270 240,193 360,193" fill="none" stroke="#C4731A" stroke-width="2"/>
  <text x="250" y="182" fill="#C4731A" font-size="11">Pn300 = 600 (6 V = 100%)</text>
  <!-- Pn300=1000 -->
  <polyline points="60,270 360,193" fill="none" stroke="#2F7A5A" stroke-width="2"/>
  <text x="290" y="240" fill="#2F7A5A" font-size="11">Pn300 = 1000 (10 V = 100%)</text>
  <!-- legend note -->
  <text x="420" y="160" fill="#1B2330" font-size="12" font-weight="600">해석</text>
  <text x="420" y="180" fill="#5B6675" font-size="11">주황: 기본값이면 6 V 이후 구간은 클램프되어</text>
  <text x="420" y="196" fill="#5B6675" font-size="11">PLC 분해능의 40%를 버립니다.</text>
  <text x="420" y="218" fill="#5B6675" font-size="11">초록: 모듈 풀스케일(10 V)과 정격을 일치시키면</text>
  <text x="420" y="234" fill="#5B6675" font-size="11">16000 스텝 전부가 속도 분해능이 됩니다.</text>
  <text x="420" y="262" fill="#C4731A" font-size="10">※ 값은 [값 검증 필요] — 매뉴얼 대조 후 확정</text>
</svg>
<figcaption>스케일 사슬과 Pn300 설정에 따른 전압–속도 매핑. 원문자는 2절 표의 단계 번호.</figcaption>
</figure>

## 5. 예제 코드 — 속도 지령 스케일링 FB

```iecst
// XG5000 v4.x, XGK-CPUH, 아날로그 출력 모듈 slot 7 (기종 확인 후 수정)
// FB_SpeedRefScale: 목표 rpm → 아날로그 출력 디지털값. 클램프 포함.
FUNCTION_BLOCK FB_SpeedRefScale
VAR_INPUT
    rTargetRPM   : REAL;          // 목표 속도 [rpm]
    rRatedRPM    : REAL := 3000.0;// 모터 정격 속도 [rpm]
    rFullScaleV  : REAL := 10.0;  // 모듈 출력 풀스케일 [V]
    rRefGainV    : REAL := 10.0;  // 서보 정격속도 도달 전압 [V] (Pn300/100)
    nDigitalMax  : INT  := 16000; // 모듈 디지털 범위 상한 [값 검증 필요]
    bEnable      : BOOL;
END_VAR
VAR_OUTPUT
    nAnalogOut   : INT;           // 출력 레지스터에 쓸 값
    bClamped     : BOOL;          // 상한 클램프 발생
END_VAR
VAR
    rVolt        : REAL;
    rDigital     : REAL;
END_VAR

IF NOT bEnable THEN
    nAnalogOut := 0;
    bClamped   := FALSE;
    RETURN;
END_IF;

// 목표 rpm → 필요한 지령 전압
rVolt := (rTargetRPM / rRatedRPM) * rRefGainV;

// 전압 → 디지털값 (모듈 풀스케일 기준 선형)
rDigital := (rVolt / rFullScaleV) * INT_TO_REAL(nDigitalMax);

// 클램프
bClamped := (rDigital > INT_TO_REAL(nDigitalMax)) OR (rDigital < 0.0);
rDigital := LIMIT(0.0, rDigital, INT_TO_REAL(nDigitalMax));

nAnalogOut := REAL_TO_INT(rDigital);
```

**테스트 시나리오 (시뮬레이터)**

| # | 입력 | 기대 nAnalogOut | 기대 bClamped |
|---|---|---|---|
| 1 | rTargetRPM=1500, Gain 10 V, Max 16000 | 8000 | FALSE |
| 2 | rTargetRPM=3000, Gain 6 V (Pn300 기본값 가정) | 9600 | FALSE |
| 3 | rTargetRPM=3600, Gain 10 V | 16000 | TRUE |

시나리오 2가 이 글의 핵심입니다. Pn300을 기본값으로 두면 정격 속도에 9600(6 V)이면 충분하고, 그 이상은 클램프 구간입니다. 반대로 Pn300을 1000으로 바꾸면 rRefGainV=10.0으로 같은 FB를 그대로 씁니다.

## 6. 출처

- LS ELECTRIC XGT 아날로그 출력 모듈 사용설명서 — 디지털 범위·출력 범위 설정 절 [출처 확인 필요: 문서번호·페이지]
- YASKAWA Σ-7S 서보팩 제품 매뉴얼 (아날로그 전압·펄스열 지령형), Pn300 속도 지령 입력 게인 절 — SIEP S800001 26 [출처 확인 필요: 개정판·페이지]
- 저자 측정값: 3절 표 (발행 전 채움)

---

## 발행 전 검증표

| # | 항목 | 유형 | 확인 방법 | 상태 |
|---|---|---|---|---|
| 1 | 사용 모듈 형식명과 채널 구성(출력 채널 수) 확정 | 저자 입력 | 실기 명판 | [ ] |
| 2 | 디지털 범위 선택값(0–16000 등) | 스케일 상수 | XG5000 I/O 파라미터 화면 + 매뉴얼 | [ ] |
| 3 | 출력 범위 선택지(0–10 V 등)와 기본값 | 파라미터 값 | 매뉴얼 | [ ] |
| 4 | Pn300 기본값 = 600 (6.00 V/정격) | 파라미터 값 | SigmaWin+ 실기 + SIEP S800001 26 | [ ] |
| 5 | V-REF 핀 번호 CN1-5/6, 최대 입력 전압 | 주소 | Σ-7S 매뉴얼 배선도 | [ ] |
| 6 | 최대 속도 클램프 관련 파라미터 번호(Pn316 여부) | 파라미터 값 | 매뉴얼 | [ ] |
| 7 | 오프셋 조정 파라미터 번호 | 파라미터 값 | 매뉴얼 | [ ] |
| 8 | 4점 전압 실측값 및 허용오차 | 타이밍/측정 | 멀티미터 | [ ] |
| 9 | 매뉴얼 문서번호·페이지 2건 | 출처 | 문서 대조 | [ ] |
| 10 | [경험: ] 문장 채움 | 저자 입력 | — | [ ] |
| 11 | 고객사·현장 식별 정보 없음 확인 | 저자 입력 | 본문 전체 재독 | [ ] |

<!--
저자에게 묻는 질문
1. 예시 모듈을 XGF-AH6A로 고정할지, 순수 출력 모듈(XGF-DV4A 계열)로 바꿀지 — 독자 보급률 기준으로 결정 필요.
2. 서보 예시를 Σ-7 하나로 갈지, 미쓰비시 MR 계열 대응 파라미터를 각주로 넣을지(시리즈 2편에서 대응표 예정).
3. 3절 실측 4점을 이번 주 실기로 확보 가능한지.
-->

*이 글의 초안과 도식은 Claude의 보조를 받아 작성했습니다. 모든 파라미터 값과 절차는 저자가 실기 또는 시뮬레이터로 확인한 뒤 발행합니다. 오류 제보는 48시간 내 확인하고 정정 이력을 공개합니다.*
