# 02_Timer_Interrupt_and_Timeout (Non-RTOS)

## 要約
1msタイマー割込みを基準時間として利用し、FSM構造に基づく制御ロジックを実装した。
RUN状態で3秒間センサー入力が発生しない場合、FAULT状態へ遷移するTimeout検知を実装した。

---

## 1. 背景と目的

組込み制御では、一定時間イベントが発生しない場合に安全側へ制御を移行するTimeout設計が重要である。

本実習の目的は以下の通りである。

- タイマー割込みによる1ms基準時間の生成
- Δt（現在時刻 − 最終イベント時刻）方式によるTimeout判定
- 状態（IDLE / RUN / FAULT）に基づく制御設計
- ISR最小化設計の理解

---

## 2. 状態遷移表

| Current | Event   | Next  | 備考 |
|----------|---------|-------|------|
| IDLE     | START   | RUN   | タイマー基準初期化 |
| RUN      | SENSOR  | RUN   | lastSensorMs更新 |
| RUN      | TIMEOUT | FAULT | 3秒以上入力なし |
| FAULT    | RESET   | IDLE  | 手動復帰 |

---

## 3. Timeout設計

Timeout判定は以下のΔt方式で実装した。

now - lastSensorMs > 3000

Δt方式を用いることで、タイマーオーバーフローが発生しても安全に時間計算が可能である。

---

## 4. 実装ポイント

- 1MHzタイマー生成（1tick = 1µs）
- 1000tickごとに割込み（1ms周期）
- ISR内では g_ms++ のみ実行（処理最小化）
- ポーリングと割込みの役割分離

---

## 5. 動作確認

![Timer Demo](./timer_demo.gif)

- IDLE: LED低速点滅
- RUN: LED高速点滅
- 3秒間センサー入力なし → FAULT
- RESETによりIDLEへ復帰

---

## 6. 考察

- ISR内処理は最小化するべきである
- Δt方式は組込み設計の基本パターンである
- 状態分離により可読性と安全性が向上した

