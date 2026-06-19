# Custom UART IP — SoC Integration on Basys3 FPGA

> Verilog로 설계한 커스텀 UART IP를 MicroBlaze SoC에 통합하고, 블루투스로 수신한 문자를 Dot Matrix에 실시간 표시하는 시스템

<!--
[![Notion](https://img.shields.io/badge/Notion-프로젝트%20상세%20보기-000000?style=for-the-badge&logo=notion&logoColor=white)](https://app.notion.com/p/minseokim-profile/Custom-UART-IP-359b5d65c68c8019a60ed422ddd7c67e?source=copy_link)
-->
[![YouTube](https://img.shields.io/badge/YOUTUBE-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%20%EC%98%81%EC%83%81%20%EB%B3%B4%EA%B8%B0-555555?style=for-the-badge&logo=youtube&logoColor=white&labelColor=FF0000)](https://youtube.com/shorts/AjUsJadkgzk)

---

## 1. Overview

| 항목 | 내용 |
|------|------|
| 플랫폼 | Basys3 (Xilinx Artix-7 FPGA) |
| 언어 | Verilog, C |
| 도구 | Vivado, Vitis |
| 통신 | UART (9600 bps), AXI4-Lite, SPI |
| 개발 기간 | 2026.04.13 - 05.21 |
| 팀 구성 | 3인 팀 프로젝트 |

---

## 2. 주요 기능

- **SoC 통합**: Verilog 기반 커스텀 UART IP를 AXI4-Lite로 패키징해 MicroBlaze SoC에 통합, CPU 메모리 맵 방식으로 레지스터 직접 제어
- **블루투스 문자 수신**: HC-06 모듈로 스마트폰에서 전송한 문자를 UART로 수신하고 ASCII 코드로 파싱
- **Dot Matrix 실시간 출력**: 수신 문자(0~9, A~Z, a~z)를 8×8 비트 패턴으로 변환해 MAX7219에 SPI로 전송하여 동적 표시

---

## 3. 담당 역할

**UART RX IP 설계 및 Vitis 펌웨어를 통한 Dot Matrix 문자 출력 제어**

- **UART RX Core RTL 설계 (rx.v)**  
  Baud 주기 중앙 샘플링으로 비트를 복원하고, false start noise 무시 및 Stop bit 기반 프레임 오류 감지 로직 구현

- **RX 시뮬레이션 검증 (rx_tb.v)**  
  정상 수신(Normal receive, Back-to-back)과 오류 처리(Frame error, False start) 케이스를 독립 테스트벤치로 분리하여 4가지 조건 검증

- **Vitis 펌웨어 — 문자 분류 및 MAX7219 SPI 드라이버 구현**  
  수신 ASCII 코드를 숫자 / 대문자 / 소문자로 분기하여 Char Pattern 배열을 O(1) 인덱싱 후 행 단위 SPI 전송

---

## 4. 시스템 아키텍처 및 핵심 구현

### 핵심 구현

![block_diagram](CustomUartDotmatrix_Blockdiagram.png)

| 구현 | 설명 |
|---|---|
| **① RX Core 중앙 샘플링** | Start bit 감지 후 Baud 주기 절반 지점에서 rx 재확인 → false start noise 무시, Stop bit ≠ High 시 `rx_error` 플래그 출력 |
| **② 문자 분류 및 패턴 출력** | 수신 ASCII 코드를 0~9 / A~Z / a~z 범위로 분기, Char Pattern 배열 O(1) 인덱싱 후 MAX7219에 행 단위 SPI 전송 |
| **③ RX 테스트벤치 분리 설계** | 정상 수신과 오류 케이스를 독립 테스트벤치로 분리하여 스테이트 머신 잔류 상태 간섭 없이 각각 검증 |

---

## 5. 트러블슈팅

| 발생 문제 | 발생 원인 | 해결 방안 | 결과 |
|-----------|-----------|-----------|------|
| RX 시뮬레이션에서 정상·오류 케이스 결과 혼재 | 오류 케이스 후 스테이트 머신이 IDLE 복귀 전 다음 케이스 시작 | 테스트벤치를 정상 / 오류 케이스로 분리, 독립 초기화 후 실행 | 4가지 조건 독립 검증 완료 |
| Dot Matrix 일부 LED 미점등 및 밝기 불균일 | Basys3 GPIO(3.3V)가 MAX7219 신호 요구 레벨(3.5V+) 미달 | MC74HCT245 레벨 시프터 추가 | 전체 LED 정상 점등 확인 |
| 62가지 문자 패턴 수작업 정의로 작업 시간 과다 | 자동화 도구 없이 8×8 픽셀을 비트 단위로 직접 설계 | 그리드 스케치 → 이진수 → 16진수 변환 워크플로 정립 | 전체 문자 패턴 체계적 정의 완료 |

---

## 6. 디렉토리 구조

| 파일 / 경로 | 역할 |
|------|------|
| `SoC/ip_repo/myip_rxtx_1_0/` | UART TX/RX Core + AXI Slave 커스텀 IP |
| `SoC/ip_repo/myip_uart_1_0/` | UART IP 패키징 최상위 모듈 |
| `SoC/ip_repo/rx.v` | RX Core — 중앙 샘플링, 프레임 오류 감지 |
| `SoC/ip_repo/tx.v` | TX Core — 병렬→직렬 송신 스테이트 머신 |
| `SoC/ip_repo/rx_tb.v` | RX 테스트벤치 |
| `SoC/project_rxtx/` | Vivado 프로젝트 — Block Design, 소스, 제약 파일 |
| `Vitis/dot/src/` | Vitis 펌웨어 소스 — 문자 분류, SPI 드라이버 |
| `Vitis/dot_matrix/microblaze_riscv_0/` | MicroBlaze 보드 지원 패키지 |
