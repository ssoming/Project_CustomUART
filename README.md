# Custom UART IP — SoC Integration on Basys3 FPGA

> Verilog로 설계한 커스텀 UART IP를 MicroBlaze SoC에 통합하고, 블루투스로 수신한 문자를 Dot Matrix에 실시간 표시하는 시스템

[![Notion](https://img.shields.io/badge/Notion-프로젝트%20상세%20보기-000000?style=for-the-badge&logo=notion&logoColor=white)](https://app.notion.com/p/minseokim-profile/Custom-UART-IP-359b5d65c68c8019a60ed422ddd7c67e?source=copy_link)

---

## 개요

| 항목 | 내용 |
|------|------|
| 플랫폼 | Basys3 (Xilinx Artix-7 FPGA) |
| 언어 | Verilog, C |
| 도구 | Vivado, Vitis |
| 통신 | UART (9600 bps), AXI4-Lite, SPI |
| 개발 기간 | 2025.04 |
| 팀 구성 | 3인 |

## 주요 기능

- **커스텀 UART IP 설계** — TX / RX Core를 Verilog로 직접 구현 후 AXI Slave 인터페이스로 패키징
- **SoC 통합** — UART IP를 MicroBlaze CPU에 AXI4-Lite 버스로 연결 (Vivado Block Design)
- **블루투스 수신** — HC-06 모듈을 통해 스마트폰에서 전송한 문자를 UART로 수신
- **Dot Matrix 출력** — 수신 ASCII 코드(0~9, A~Z, a~z)를 8×8 패턴으로 변환해 MAX7219 SPI로 표시

## 파일 구성

| 파일 / 경로 | 역할 |
|------|------|
| `ip_repo/myip_v1_0/src/tx.v` | UART TX Core — 병렬→직렬 송신 스테이트 머신 |
| `ip_repo/myip_v1_0/src/rx.v` | UART RX Core — 직렬→병렬 수신, 프레임 오류 감지 |
| `ip_repo/myip_v1_0/src/myip_v1_0.v` | UART IP 최상위 모듈 (TX + RX + AXI Slave 통합) |
| `uart_soc/uart_soc.bd` | Vivado Block Design — MicroBlaze + UART IP + SmartConnect |
| `vitis/src/main.c` | 문자 분류 + MAX7219 SPI 드라이버 + 메인 루프 |
| `vitis/src/char_pattern.h` | 0~9, A~Z, a~z 문자 8×8 비트 패턴 배열 |
| `sim/tb_rx_normal.v` / `tb_rx_error.v` | RX 정상 수신 / 오류 처리 테스트벤치 |

## 주요 구현

- **UART RTL 설계** — TX는 `IDLE → START → DATA(LSB first) → STOP` 스테이트 머신으로 직렬화, RX는 Baud 주기 중앙 샘플링으로 비트 복원 및 false start noise 무시 로직 포함
- **SoC 통합** — 커스텀 IP를 AXI4-Lite Slave로 패키징 후 Vivado Block Design에서 MicroBlaze — AXI SmartConnect — UART IP 구조로 연결
- **문자 출력** — 수신 ASCII를 숫자 / 대문자 / 소문자 3가지로 분기하여 Char Pattern 배열을 O(1) 인덱싱, MAX7219에 행 단위 SPI 전송

## 담당 역할

- UART RX Core RTL 설계 — 중앙 샘플링, false start 무시, 프레임 오류 감지 구현
- RX 테스트벤치 정상 / 오류 케이스 분리 설계 및 시뮬레이션 검증
- Vitis 펌웨어 — 문자 분류 로직, Char Pattern 배열 정의, MAX7219 SPI 드라이버 구현
