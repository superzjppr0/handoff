# C360 Auto Test — handoff (2026-11-18)

Load this file into Claude on the build/server computer to continue. Everything below was verified on the test station on 2026-11-18 unless marked *unverified*.

## 1. Goal (from the user)

- Run SoncaInspector Auto Test for model **C360** (BLE / BT name still **KBEATBOX**) with **all tests**, including the real KT mic test **IO006** and a **rub & buzz** test (wanted as a separate **IO008**).
- Tune loudness with **per-test Playback Level dBFS** (not model-wide BLE volume).
- Code changes + **beta release** (needs the current source — not on the station):
  1. **A/B popup**: FastTrack Pro **Analog 1/2 → LINE IN**, **Analog 3/4 → MIC IN**. If a stage gets no signal, the operator probably has to press the FastTrack **A/B** button — show a popup telling them to.
  2. **Device / port popup timing ("add 2 s")**: show the popup that lists all ports for 2 s / auto-close 2 s after all ports are confirmed. *Exact behaviour still to confirm with the user* — candidates: the "Kết nối thiết bị đo" window (lists ports, auto-closes when Windows has them) or the BleCheckAtStart popup (lists BLE inputs supported/confirmed).
  3. **Separate IO008 rub & buzz**: today both the web and the app reject it — "Mỗi ngõ C360 chỉ có một bài đang bật" (one enabled test per SpeakerStage). Needs that rule relaxed for `AcousticMic` in **both** the app validator and the web validator.

## 2. Station facts

| Item | Value |
|---|---|
| App | SoncaAudioInspector **v1.5.1**, `C:\Users\Admin\Downloads\SoncaAudioInspector-win-x64` (built 2026-11-18) |
| Base config | server serial **#1794906189** (2026-11-17), sha a4546f74…; re-downloaded and overwritten on every launch (SYS002) |
| C360 web revision | **Serial 2**, revision `cmwlsaorr000i2lkawmm8jm1b`, approved 2026-11-18 14:12 — file `C360_model_approved_rev2.json` |
| Local source on station | `C:\Users\Admin\SoncaAudioInspector` is **stale** (2026-10-16, git origin github.com/minhtruong89/SoncaAudioInspector, last fetch 2026-10-12, git not installed). Do not build from it. |
| C360 firmware id | 1075 |
| Reference lines (LINE CHUẨN) for C360 on station | **none** |

### Device mapping (approved rev 2)

| Config port | Windows device |
|---|---|
| LINE IN (play) | `Analog Connector 1/2 (7- FastTrack Pro)` |
| MIC IN (play) | `Analog Connector 3/4 (7- FastTrack Pro)` |
| BT (play) | `Headphones (5- KBEATBOX  Stereo)` (two spaces, as Windows writes it) |
| USB A (play) | `CS3600 MUSIC` (appears after BLE switches the speaker to USB A) |
| LINE OUT = "RECORDING TYPE C" (record) | `LINE OUT (DAURIS_300 usb audio)` |
| Measurement Mic (record) | `KT USB` → `Microphone (KT USB Audio)` |

The "7-"/"5-" prefixes are Windows instance numbers and can change per USB port / PC; matching appears to be partial (`KT USB` matched), so shorter names would be more portable.

### Tests (approved rev 2)

| ID | Stage | Path | Level dBFS | Notes |
|---|---|---|---|---|
| IO001 | MicEffects | MIC IN → LINE OUT | −12 | echo + reverb expected |
| IO002 | MicFrequencyShift | MIC IN → LINE OUT | −18 | FBX multitone |
| IO003 | MicDry | MIC IN → LINE OUT | −12 | |
| IO004 | LineIn | LINE IN → LINE OUT | −6 | |
| IO005 | UsbA | USB A → LINE OUT | −12 | |
| IO006 | AcousticMic | LINE IN → KT mic | −24 | FFT/THD **+ Pink rub & buzz** (`TestRubBuzz: true`, `RubBuzzMode: "PinkNoise"`), enabled |
| IO007 | Bluetooth | BT → LINE OUT | −12 | |

Rub & buzz was merged into IO006 because the separate IO008 was rejected (see §1.3). Sine rub & buzz is blocked on C360; Pink is only allowed on `AcousticMic`.

## 3. Run 1 — serial C360_181126_0001 (14:14–14:17)

Session folder: `C:\Users\Admin\Downloads\SoncaAudioInspector-win-x64\fail data\20261118\C360_181126_0001_141434370\` (images copied to `images/`).

| Test | Path | Result | Measured | Reason |
|---|---|---|---|---|
| IO001 MIC echo/reverb | FastTrack 3/4 | **FAIL** | −11 dBFS, SNR 46 dB, THD@1k 1.0 % | reverb detected, **echo NOT detected**; polarity inverted |
| IO002 FBX | 3/4 | **PASS** | SNR 41–74 dB | shift −3.000 Hz with FBX on and off (no change caused by FBX) |
| IO003 MIC dry | 3/4 | **FAIL** | −11 dBFS, SNR 47 dB, **THD@1k 0.78 %** (limit 0.5) | dominant H3 → MIC IN likely overdriven by line-level FastTrack output |
| IO004 LINE IN | FastTrack 1/2 | **INVALID** | **−77 dBFS, SNR ≈ 0** | no signal — **A/B not pressed** (BLE confirmed speaker on LINE IN, reg 6242 = 4) |
| IO005 USB A | CS3600 | **INVALID** | −34 dBFS, SNR 30 dB, THD@1k 0.03 % | acquisition OK; most likely **no LINE CHUẨN** for this config |
| IO006 KT mic FFT + R&B | 1/2 | **INVALID** | KT mic −48 dBFS, SNR 1.5 dB | same A/B cause — speaker never played |
| IO007 BT | BT | not run | — | session stopped 4 s into IO007 and restored BLE state |

App state after run: button **TIẾP TỤC ĐO TỰ ĐỘNG** (resume). Resume re-runs only IO004–IO007 and keeps IO001–IO003. One A/B press between IO003 and IO004 covers IO004 and IO006; IO005/IO007 do not use the FastTrack.

Images: `FRA-IO001.png`, `IO002-frequency-shift.png`, `FRA-IO003.png`, `FRA-IO004.png`, `FRA-IO005.png`, `FRA-IO006.png`.

## 4. Next steps

1. Operator presses FastTrack **A/B** (to 1/2) → press **TIẾP TỤC ĐO TỰ ĐỘNG**.
2. Config rev 3 (web `/config`, needs approval): lower MIC IN levels — IO001/IO003 −12 → about **−24 dBFS**, IO002 −18 → about **−30 dBFS** — then re-check THD@1k and echo detection. *Starting values, unverified.*
3. Run **Tìm LINE CHUẨN** with a known-good C360; until then IO003–IO007 cannot PASS (D13: no matching reference → INVALID).
4. Pink rub & buzz shows **CẦN DUYỆT** until ≥3 good-speaker runs are saved as reference (and ≥3 faulty for threshold validation).
5. Code (needs current source): A/B popup, popup 2 s timing, IO008 one-test-per-stage relaxation (app + web), then beta release.

## 5. Reporting to the server

`https://inventory.acnos.store/report` is a **read-only** QA table (Date, Unit ID, Model, QA Result, App Result, Notes; "Dữ liệu thử", "Xuất Excel"). There is no upload form. Results and images reach it only through the app's QA upload at the end of a completed session. The app blocks upload while any step is INVALID / not concluded (D16), so run 1 has **not** been uploaded.

## 6. Files in this folder

- `HANDOFF.md` — this file
- `C360_model_approved_rev2.json` — the approved C360 model JSON (paste into web "JSON nâng cao" for edits)
- `model-used-in-run.json`, `run-policy.json` — exactly what run 1 used
- `images/*.png` — result graphs from run 1
