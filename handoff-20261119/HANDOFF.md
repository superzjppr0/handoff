# C360 — handoff 2026-11-19 (gửi Claude máy server / có source)

Đọc kèm handoff trước: `..\handoff-20261118\HANDOFF.md`. File này ghi những gì mới từ trạm đo sáng 19/11. Mọi việc bên dưới đều cần **source hiện tại** (trạm đo chỉ có source cũ 16/10) hoặc **web `/settings` + duyệt** (config có chữ ký, sửa tại trạm sẽ bị ghi đè).

## TỔNG HỢP YÊU CẦU (đọc phần này trước)

| # | Yêu cầu của chủ | Loại việc | Trạng thái | Chi tiết |
|---|---|---|---|---|
| 1 | Các bài sau MIC IN phải tắt MIC qua BLE | — | **Đã có trong app v1.5.4**, readback xác nhận; không cần sửa | §1.1 |
| 2 | Bài mic thực tế chỉ dùng MIC KT USB, không dùng MIC IN | — | **Đã đúng** (IO006/IO008 phát LINE IN, thu mic KT, MIC tắt); riêng IO009 cố ý dùng MIC IN | §1.1 |
| 3 | Đổi tên cổng `Measurement Mic` → `MIC KT USB` | Config web (+ kiểm tra source) | Cần làm. Chuỗi cứng trong dll, **kiểm tra source trước** | §1.2 |
| 4 | Hình Rè IO008 giống Ctrl+Shift+I (ngoài màn hình chính hiện chirp cũng được) | Code app | Cần làm | §1.3 |
| 5 | IO009 Gain trước hú phải có hình; thông số ghi vào ô DISTORTION như các bài khác | Code app (+ config StartGain) | Cần làm | §1.4 |
| 6 | Ctrl+Shift+M: mỗi phép đo có ô tick để **tự động tạo thành bài IO mới** | Code app + web `/settings` + validator | Cần làm | §1.5 |
| 7 | IO006/IO008 INVALID dù mic KT có thu (chủ xác nhận đồ thị có đáp tuyến) | Code app (cổng nhận sweep) + config mức phát | Cần làm. **Nguyên nhân chính khiến IO006 INVALID** | §2.1 |
| 8 | Bài mic thực tế vẫn cần A/B (LINE IN = FastTrack 1/2) | Code app (popup A/B) | Cần làm | §2.2 |
| 9 | Bấm gửi server mà FACTORY FLOOR QA & APP TRACKER không hiện gì | Code app + API + `/report` (tùy chủ) | App **không gửi** phiên chưa hoàn tất (D16); cần quyết định | §3 mục 4 |
| 10 | IO001 không phát hiện echo | Điều tra | Còn mở | §3 mục 5 |
| 11 | **v1.5.5: Auto Test C360 không kết nối được BLE** ("tên bắt đầu bằng KBEATBOX"). Chủ: "không có thì chuyển qua tên Bluetooth âm thanh đo tiếp chứ" | Code app | **CHẶN TOÀN BỘ trạm đo C360**, làm trước tiên | §5 |

**Thứ tự đề xuất:**
0. **#11 trước tiên** (v1.5.5 không đo được C360 nào, nên không có phiên nào gửi lên server).
1. Kiểm tra source về `Measurement Mic` (#3).
2. Code beta: #7 → #8 → #4 → #5 → #6.
3. Config C360 rev 5 trên web: #3, mức phát IO006/IO008, `StartGainDb` IO009.
4. Trạm đo nhận bản mới (đăng nhập lại) → đo lại.

## 0. Trạng thái trạm đo (đã kiểm tra 19/11)

| Mục | Giá trị |
|---|---|
| App | SoncaAudioInspector **v1.5.4** (tự cập nhật 19/11 08:51), `C:\Users\Admin\Downloads\SoncaAudioInspector-win-x64` |
| Base config | server #1794906189 (17/11), sha a4546f74 |
| C360 web revision | **Serial 4**, revision `cmwm90qxy000034p7evf38tpd`, AutomationProfile `C360Electronic` v9, IO001–IO009 |
| Thiết bị Windows của mic đo | chủ đã đổi tên thành `MIC KT USB (KT USB Audio)` trong lúc đo (trước đó `MIC KT (KT USB Audio)`, hôm qua `Microphone (KT USB Audio)`); config `"KT USB"` vẫn khớp nhờ so khớp một phần |
| Phiên đo | `C360_191126_0001`, thư mục `C:\Users\Admin\Downloads\SoncaAudioInspector-win-x64\fail data\20261119\C360_191126_0001_085703591` |

## 1. Yêu cầu chi tiết

### 1.1 Tắt MIC qua BLE sau các bài MIC IN → **ĐÃ CÓ, không cần sửa**

> "tất cả các bài sau MIC IN phải tắt MIC đi bằng BLE đã có điều kiện đó chưa, bài đo mic thực tế cần gì MIC IN chỉ cần MIC KT USB thôi"

Logic nằm trong code app (CONFIG_TESTS.vi.md: "trước ngõ nhạc, app mute MIC, đưa mức MIC về 0, tắt effect/FBX"; IO006: "MIC IN, FBX, echo và reverb trên loa đều tắt"). Readback BLE của phiên này (`control-readbacks.jsonl`, mỗi dòng chụp **đầu** bài sau khi app đặt xong):

| Giờ VN | Đầu bài | `6447` (mute MIC) | `6400` (mức MIC) | `6242` (ngõ) |
|---|---|---|---|---|
| 08:57:34 / 08:57:55 / 08:58:15 | IO001 / IO002 / IO003 | 0 | 4096 (50 %) | 0 |
| 08:58:51 | IO004 LINE IN | **1** | **0** | 4 |
| 08:59:45 | IO005 USB A | 1 | 0 | 1 |
| 09:00:42 | IO006 Mic KT | **1** | **0** | 4 |
| 09:06:58 | IO007 BT | 1 | 0 | 0 |
| 09:07:31 | IO008 Rè | **1** | **0** | 4 |
| 09:10:13 | IO009 Gain trước hú | 0 | 4096 | 4 |
| 09:10:22–27 | khôi phục cuối phiên | 0 | 4096 | 0 (MASTER/MUSIC về 8160) |

Ý nghĩa `6447`/`6400` suy ra từ hành vi (`6400` 4096 = 50 % giống MASTER/MUSIC `6102`/`6103`; `6447` lật đúng lúc tài liệu nói app mute MIC). Nên xác nhận lại trong source.

Ngoại lệ có chủ đích: **IO009 MicFeedbackGain** phát ra `MIC IN` (vòng mic KT → MIC IN → loa → mic KT) nên phải bật MIC.

Đề nghị (tùy chủ): ghi vào `auto-test-log.txt` một dòng kiểu "BLE: MIC mute ✓ (6447=1, 6400=0) trước IO004", để người đo khỏi phải đọc jsonl.

### 1.2 Đổi tên cổng `Measurement Mic` → `MIC KT USB` → **CONFIG WEB (cần duyệt), kiểm tra source trước**

> "đổi tên Measurement Mic luôn trên config luôn nha"

JSON model C360 (rev 4 → rev 5):

```jsonc
"Devices": {
  "Output": {
    "LINE OUT": "LINE OUT (DAURIS_300 usb audio)",
    "MIC KT USB": "KT USB"          // trước: "Measurement Mic": "KT USB"
  }
}
// IO006, IO008, IO009:
"Recording In": "MIC KT USB"        // trước: "Measurement Mic" (3 chỗ)
```

**Kiểm tra trong source trước khi duyệt:** chuỗi `"Measurement Mic"` nằm cứng trong `SoncaAudioInspector.dll` v1.5.4, cạnh `C360ModelEndpoint` và seed IO006 ("Mic thực tế · Đáp tuyến / THD (cấu hình tạm)"). Cần chắc chắn:
1. Nâng cấp profile `C360Electronic` (v9) không tự thêm lại cổng `Measurement Mic` hoặc ép `Recording In` về tên cũ.
2. Stage `AcousticMic` / `AcousticRubBuzz` / `MicFeedbackGain` tìm mic theo `Recording In` (tên tự do), không theo hằng `Measurement Mic`.
3. Validator web chấp nhận tên cổng output mới.
4. Dấu vân tay LINE CHUẨN có chứa tên cổng không. Hiện IO006/IO008 chưa có chuẩn nào, nên đổi ngay bây giờ không mất gì.

Nếu code đang cứng tên thì sửa app trước (beta) rồi mới đổi config. Làm ngược lại thì IO006/IO008/IO009 sẽ mất thiết bị thu.

### 1.3 Hình Rè (Rub & Buzz) IO008 giống Ctrl+Shift+I → **CODE APP**

> "rub buzz phải hiện hình giống ở ctrl shift I chứ ở ngoài thì hiện hình chirp cũng được"

Hiện tại `FRA-IO008.png` và khung đồ thị chính khi bấm IO008 chỉ là **đáp tuyến dBFS 1/24 octave**. Yêu cầu:
- Khi xem IO008 (bấm bài trong danh sách Auto Test) **và** ảnh xuất `FRA-IO008.png`: hiện giao diện Ctrl+Shift+I → **Rè Pink · v1.2** (bảng HF / P95 / sự kiện/s / peak-crest, chọn lượt, đồ thị envelope).
- Màn hình chính hiện hình **chirp** cũng được.
- Tiền lệ đã có: IO001 "dùng cùng đồ thị Impulse/ETC/Schroeder với Ctrl+Shift+I, kể cả ảnh xuất kết quả" (CONFIG_TESTS.vi.md dòng 89), nên làm IO008 theo cùng cơ chế.

### 1.4 IO009 Gain trước hú: có hình + thông số vào ô DISTORTION → **CODE APP (+ config)**

> "gain trước hú không có hình ảnh à, thông số thì có thể ghi ở DISTORTION giống bài đo ở trên"

Hiện trạng: bấm IO009 thì khung đồ thị chính **trống**; ô DISTORTION chỉ có thiết bị/rate kèm badge "CHƯA CÓ TÍN HIỆU". Ảnh xuất `FRA-IO009.png` có vẽ đỉnh/nền theo gain, nhưng mỗi gain nhiều bước nên thành vạch đứng, khó đọc.

Yêu cầu:
- Khung đồ thị chính hiện hình IO009 dễ đọc: X = gain vòng (dB), Y = mức thu dBFS và/hoặc đỉnh/nền dB. Hai đường FBX TẮT (cam) và FBX BẬT (xanh), mỗi gain một điểm, đánh dấu điểm hú, kẻ ngang `HowlLevelDbFs` / `HowlMarginDb`. Ảnh xuất dùng cùng hình.
- Ô DISTORTION ghi thông số giống các bài trên, ví dụ:
  ```
  IO009 — MIC IN · Gain trước hú (FBX tắt/bật)
  PHÁT → MIC IN: Analog Connector 3/4 · THU ← MIC KT USB
  FBX TẮT: hú ở −28,0 dB · 118 Hz · đỉnh −26,8 dBFS
  FBX BẬT: hú ở −29,0 dB · 118 Hz · đỉnh −34,3 dBFS
  FBX thêm được: −1,0 dB (chưa có MinimumAddedDb → chỉ báo số)
  Cài đặt: start −30 dB, bước 1 dB / 0,25 s, ngưỡng hú −35 dBFS & đỉnh/nền ≥ 25 dB
  ```
- Dữ liệu có sẵn trong `MicFeedbackGain-result.json` (settings + Steps của Off/On; bản sao trong `session-C360_191126_0001\`).

**Số liệu IO009 phiên này khó tin được:**
- FBX BẬT: ngay bước đầu (−30 dB) đã có đỉnh 118 Hz cao hơn nền 49 dB, mức −36,5 dBFS. Tức là vòng đang hú sẵn, hoặc tiếng hú của lượt TẮT chưa tắt hẳn.
- FBX TẮT: ở −30 dB đỉnh/nền đã 30–38 dB.
- Vậy `StartGainDb = −30` quá sát điểm hú, và con số "FBX thêm −1 dB" chưa nói lên gì.
- Đề xuất: config hạ `StartGainDb` xuống khoảng −50; code thêm khoảng lặng hoặc chờ hết hú giữa lượt TẮT và BẬT, và cho FBX thời gian thích nghi trước khi tăng gain. *Chưa đo kiểm.*

### 1.5 Ctrl+Shift+M: ô tick "tạo bài IO mới" cho từng phép đo → **CODE APP + WEB + VALIDATOR**

> "với các bài đo trong ctrl shift M nên có ô tick để tự động điền vào thành bài test IO mới"

**Bối cảnh:** trong Ctrl+Shift+M, một bài IO (một `SpeakerStage`) đang gánh nhiều phép đo qua tab **Bật/tắt kiểm tra**, **Rè (Rub & Buzz)**, **Tone cơ khí**, **Echo / Reverb**… (TestRubBuzz/Pink Rè, TestSoundLevel, TestAirLeak, TestLooseParticle, THD 80/4000, BandAverage, FeedbackGain…). Hôm 18/11, muốn tách Rè thành IO008 riêng thì phải sửa JSON bằng tay, và còn bị chặn bởi luật "Mỗi ngõ C360 chỉ có một bài đang bật" ở cả app lẫn web. Rev 4 mới tách được bằng stage riêng `AcousticRubBuzz`.

**Yêu cầu:** cạnh mỗi phép đo trong Ctrl+Shift+M có ô tick **"Tạo thành bài IO riêng"**. Tick vào thì app tự điền một bài mới vào danh sách bài đo:

| Thuộc tính bài mới | Cách tự điền (đề xuất) |
|---|---|
| `id` | mã IO kế tiếp (`IO010`…), theo luật đánh số hiện có (mã theo vị trí, `ReferenceTestId` giữ identity gốc) |
| `name` | "<tên bài nguồn rút gọn> · <tên phép đo>", ví dụ "Mic thực tế · Rè (Rub & Buzz)" |
| `SpeakerStage` | stage riêng của phép đo (như `AcousticRubBuzz`, `MicFeedbackGain`); phép nào chưa có stage riêng thì source cần thêm |
| Đường tín hiệu | chép từ bài nguồn: `Playback Out`, `Recording In`, `Playback Level dBFS`, Windows Levels, kênh, calibration mic |
| Thông số phép đo | **chuyển** (không nhân đôi) từ bài nguồn sang bài mới, rồi tắt phép đó ở bài nguồn để khỏi chạy hai lần |
| `IsEnabled` / số lượt | bật; `MaximumRuns` / `RequiredPasses` mặc định của model |
| Vị trí | ngay sau bài nguồn (hoặc cuối danh sách, chủ chọn) |

Bỏ tick: hỏi xác nhận, rồi gộp thông số về bài nguồn và xóa (hoặc tắt) bài IO đã tạo.

**Ràng buộc:**
- Trạm đo: Ctrl+Shift+M chỉ để xem ("config do server quản lý"). Tính năng phải có ở **chế độ admin** (`SONCA_CONFIG_ADMIN=1`) **và** trên **web `/settings`** (cạnh mỗi phép đo, giống "+ Thêm bài"). Kết quả đi qua **Lưu & gửi duyệt** như mọi revision.
- **Validator app + web** phải chấp nhận nhiều bài cùng ngõ phát/thu khi bài tách ra mang stage riêng. Luật "một bài đang bật mỗi SpeakerStage" giữ nguyên cho stage gốc.
- LINE CHUẨN: tách phép đo ra khỏi bài nguồn không được làm đổi identity chuẩn của bài nguồn. Kiểm tra phép nào nằm trong dấu vân tay.
- Thứ tự chạy và A/B: bài mới chạy cùng ngõ FastTrack với bài nguồn, nên đặt ngay sau bài nguồn để khỏi phải bấm A/B thêm lần (xem §2.2).

**Cần hỏi lại chủ:** (a) phép đo nào có ô tick (tất cả trong Bật/tắt kiểm tra, cộng Rè, Tone cơ khí, Echo/Reverb, Gain trước hú?); (b) bài mới đặt ngay sau bài nguồn hay cuối danh sách; (c) tick là **chuyển** phép đo hay **chép** (giữ ở cả hai bài).

## 2. Kết quả phiên C360_191126_0001 (08:57 → 09:10, 783,6 s)

Kết quả tổng trên app: **CHƯA ĐO ĐỦ — KIỂM TRA TÍN HIỆU**, nút **TIẾP TỤC ĐO TỰ ĐỘNG** sáng. **Chưa upload** (log 09:10:22: "Không gửi server: chỉ gửi kết quả Auto Test hoàn tất (PASS/FAIL)").

| Bài | Kết quả | Số liệu chính |
|---|---|---|
| IO001 MIC Echo/Reverb | **FAIL** | có reverb (đuôi 210 ms), **không phát hiện echo**; 0/2 lượt |
| IO002 Di tần FBX | PASS | −2,998 Hz (FBX bật) / −3,000 Hz (tắt): FBX không làm đổi di tần; config chưa có `ExpectedFbxChange` nên chỉ xác nhận đo được |
| IO003 MIC khô | PASS | THD@1k **0,374 %** (rev 3 hạ mức đã sửa được lỗi 0,78 % hôm qua), −12,2 dBFS, SNR 47,4 dB |
| IO004 LINE IN | PASS | sau 1 lần bấm A/B; THD@1k 0,221 %, −32,6 dBFS, SNR 46,3 dB |
| IO005 USB A | PASS | THD@1k 0,035 %, −33,9 dBFS |
| IO006 Mic KT FFT/THD | **INVALID** | app báo "không nhận ra sweep", SNR 2,9–9,3 dB (xem §2.1: thực ra có sweep) |
| IO007 Bluetooth | PASS | THD@1k 0,015 %, −24,0 dBFS |
| IO008 Mic KT Rè Pink+Chirp | **INVALID** | lượt 1 có tín hiệu nhưng bị chặn; lượt 2 mất tín hiệu (§2.1, §2.2) |
| IO009 Gain trước hú | PASS (chỉ báo số) | hú −28 dB (TẮT) / −29 dB (BẬT) @118 Hz → FBX thêm −1 dB; số liệu khó tin (§1.4) |

PASS của IO003/004/005/007 là PASS so với **LINE CHUẨN tự lấy từ chính loa này** (người đo chọn "tự lấy LINE CHUẨN từ loa đang đo"; 4 file mới `%LOCALAPPDATA%\SoncaAudioInspector\save standards\c360-*`). Chỉ có giá trị nếu loa này là loa tốt.

### 2.1 IO006/IO008 INVALID: mic KT CÓ thu được sweep, cổng "nhận sweep" của app chặn sai → **CODE APP**

> chủ: "MIC KT có nhận và vẽ được đáp tuyến mà"

Chủ đúng. Đối chiếu bản thu với excitation (matched filter; cùng thuật toán chạy trên IO004 để so):

| Bản thu | Matched filter peak/rms | Sweep cao hơn nền theo dải (60 Hz–16 kHz) | Kết luận |
|---|---|---|---|
| IO004 LINE IN → LINE OUT (điện tử) | 70,5 dB | 44–51 dB | chuẩn so sánh |
| IO006 lượt 1 | **32,2 dB** | 14–20 dB (30–60 Hz: 7 dB) | **có sweep**, đồ thị là thật |
| IO006 lượt 2 | **32,6 dB** | 10–17 dB | **có sweep** |
| IO008 lượt 1 (09:07:31) | **28,1 dB** | 14–20 dB | **có tín hiệu** |
| IO008 lượt 2 (09:10:03) | 9,7 dB (đỉnh ngẫu nhiên) | 0–3 dB | **không có tín hiệu** |

App báo "SNR 2,9–9,3 dB · không nhận ra sweep" có vẻ vì lấy RMS toàn băng của **cả 5 s** bản thu so với nền (−41 vs −48 dBFS), trong khi sweep chỉ dài 1,49 s. Cách này ổn cho đường điện tử (nền −79 dBFS) nhưng **chặn oan bài âm học**.

**Sửa code:** với `AcousticMic` / `AcousticRubBuzz`, xét có sweep bằng matched filter / deconvolution SNR (ví dụ ≥ 20 dB) thay cho SNR RMS toàn băng.

**Config rev 5 (chưa đo kiểm):** tăng mức phát, để **THD có nghĩa** chứ không phải để nhận sweep (THD 0,5 % = −46 dB, mà SNR theo dải hiện chỉ 10–20 dB):
- IO006 `Playback Level dBFS` −24 → khoảng **−6** (bằng IO004; đường LINE IN đã xác nhận THD 0,22 % ở mức này).
- IO008: tăng tương tự. Kiểm tra mức pink lấy từ `Playback Level dBFS` hay `RubBuzzLevelDbFs` (mặc định −18 dBFS **RMS**).
- Nền mic KT −43…−48 dBFS là cao (Pink rè cần SNR LF ≥ 20 dB). Xem vị trí mic, ồn phòng, Windows enhancements/AGC của `KT USB Audio`.

| File (`continue-1\attempt-*\`) | RMS |
|---|---|
| excitation IO006 (FastTrack 1/2, −24 dBFS) | −27,3 dBFS |
| nền mic KT `AcousticMic-quiet.wav` | −46…−48 dBFS |
| mic KT trong đoạn sweep | −39…−40 dBFS |
| so sánh IO004 (−6 dBFS) → LINE OUT | −32,7 dBFS, nền −79 dBFS |

### 2.2 A/B ở bài âm học: CẦN (LINE IN = FastTrack 1/2), popup phải dựa trên phát hiện đúng → **CODE APP**

> chủ: "bài Mic thực tế vẫn cần A/B vì có cổng LINE IN cần đo"

Chuỗi C360 cần **A/B hai lần** mỗi phiên:
- IO001–IO003 dùng 3/4 (MIC IN).
- **Chuyển sang 1/2** trước IO004. IO004, IO006, IO008 dùng 1/2.
- **Chuyển về 3/4** trước IO009 (MIC IN).

Diễn biến phiên này (suy từ matched filter, chưa kiểm chứng bằng mắt):
- IO006 cả hai lượt có sweep, tức A/B lúc đó đúng 1/2. Bốn popup A/B là do cổng SNR ở §2.1.
- IO008 lượt 1 có tín hiệu, sau đó 5 popup A/B, lượt 2 mất tín hiệu. Nhiều khả năng A/B đã bị bấm sang 3/4 trong lúc trả lời popup.
- IO009 (3/4) sau đó chạy được, khớp với điều trên.

**Sửa code:**
1. Chỉ hỏi A/B khi matched filter **thật sự** không thấy excitation, không hỏi khi chỉ có SNR RMS thấp.
2. Popup ghi rõ **chiều cần chuyển** theo ngõ FastTrack của bài: "chuyển A/B sang 1/2 (LINE IN)" / "chuyển về 3/4 (MIC IN)". Có thể nhắc chủ động trước IO004 và IO009.
3. (Tùy) thu song song LINE OUT trong bài âm học: LINE OUT có sweep nghĩa là A/B đúng, lỗi nằm ở phía âm học.

## 3. Việc cần làm bên server (thứ tự)

1. **Source:** kiểm tra 4 điểm §1.2 về chuỗi cứng `Measurement Mic`; sửa nếu cần.
2. **Code beta:** cổng nhận sweep âm học (§2.1, nguyên nhân chính) → popup A/B (§2.2) → hình Rè IO008 (§1.3) → IO009 hình + DISTORTION + khoảng lặng TẮT/BẬT (§1.4) → ô tick tạo bài IO mới trong Ctrl+Shift+M, admin + web + validator (§1.5) → (tùy) log MIC mute (§1.1).
3. **Config C360 rev 5 (web, duyệt):** `Measurement Mic` → `MIC KT USB` (§1.2); IO006/IO008 tăng mức phát (§2.1); IO009 `StartGainDb` −30 → khoảng −50 (§1.4).
4. **Tracker:** chủ hỏi "bấm gửi lên server rồi mà sao ở FACTORY FLOOR QA & APP TRACKER không hiện gì cả". App không upload phiên còn INVALID (D16), server không nhận gì. Nếu chủ muốn tracker thấy cả phiên chưa hoàn tất: thêm upload trạng thái `CHƯA HOÀN TẤT / INVALID` (app + API + cột QA Result trên `/report`). Tối thiểu, app phải hiện popup "chưa gửi — lý do" khi người đo bấm gửi.
5. **Sau khi trạm nhận bản mới:** đo lại. IO001 thiếu echo vẫn còn mở: echo bật (`6404=1`) trong lúc IO001 chạy mà vẫn không phát hiện echo; cần xem kỳ vọng echo trong config/thuật toán.

## 5. v1.5.5 (19/11 10:47): Auto Test C360 không kết nối được loa BLE → **CODE APP, chặn trạm đo**

App tự cập nhật lên v1.5.5 lúc 10:47 (config C360 rev 5, `"BleNamePrefix": "KBEATBOX"`, `FirmwareModelId` 1075). Phiên `C360_191126_0002`: bấm đo 4 lần (10:49:53, 10:50:05, 10:50:17, 10:56:44), lần nào cũng dừng trước IO001:

```
[Auto Test] Chưa kết nối được loa BLE có tên bắt đầu bằng "KBEATBOX" và đúng model 1075. Bật loa, đặt gần máy đo rồi bấm đo lại. (InvalidOperationException)
```

Loa đúng model, đặt sát máy (RSSI −44 dBm). Mixer Console (Ctrl+Shift+H) cùng lúc đó vẫn kết nối được và đọc INFO: `KBEATBOX · ID 1075 · SCA_MAIN48_C360.C360 · VER A:1.1.5, C:1.6.7-AT`.

### 5.1 Nguyên nhân (đã kiểm chứng)

1. **C360 không phát tên trên BLE.** Gói quảng bá của `42:3E:3F:75:2E:7D` (quét Active, Windows) chỉ có:
   `01:18` · `03:4353` (dịch vụ Sonca 0x5343) · `FF:5343 423E3F752E7D 0433 01FB`. **ScanResponse rỗng**, không có AD type 0x08/0x09.
   `KBEATBOX` là tên **Bluetooth âm thanh** (classic, `43:3E:3F:75:2E:7D` = BLE XOR 0x01, Windows: `Headphones (5- KBEATBOX  Stereo)`). Mixer Console đã biết ghép hai địa chỉ ("Bluetooth cùng loa (địa chỉ XOR 0x01): KBEATBOX", "Tên BLE: Sonca BLE 423E3F752E7D"), nhưng đường Auto Test thì không.
2. **Auto Test chỉ so tiền tố với tên quảng bá BLE.** Binary v1.5.5 chỉ có nhánh `[BLE] Không thấy loa BLE nào có tên bắt đầu bằng "{0}" trong 4 s quét.`, không có nhánh dự phòng.
3. **Loa đã kết nối BLE thì không quảng bá.** App mở lúc 10:47 là Mixer Console tự nối BLE luôn (RSSI "mẫu cũ" tăng dần từ 10:47). Bước quét 4 s không thấy loa đang kết nối, và Auto Test cũng không xét loa đang nối sẵn. Đã thử ngắt BLE (loa quảng bá lại ngay) rồi đo: vẫn lỗi, do điểm 1–2.
4. Gói quảng bá **có sẵn mã model**: `0x0433` = **1075** (sau 6 byte địa chỉ trong manufacturer data). Loa khác gần đó: `432D3F33C4B8 … 0422` (= 1058), `75063F0A0A0A` (header `4D56`).

### 5.2 Yêu cầu sửa (theo ý chủ: "ưu tiên lọc theo config trước", "không có thì chuyển qua tên Bluetooth âm thanh đo tiếp")

Thứ tự chọn loa khi bắt đầu Auto Test:
1. **Loa đang kết nối BLE** (Mixer Console / phiên trước): đọc INFO, khớp `FirmwareModelId` của config (1075) thì dùng luôn, **không quét**.
2. **Quét 4 s, lọc theo config**: thiết bị có dịch vụ 0x5343, coi là khớp `BleNamePrefix` khi **một trong các tên sau** bắt đầu bằng tiền tố:
   - tên quảng bá BLE (nếu có);
   - **tên Bluetooth âm thanh của cùng loa** (địa chỉ XOR 0x01; lấy từ thiết bị Windows đã ghép đôi hoặc đang kết nối, như Mixer Console đang làm);
   - (tùy chọn) bỏ qua tên khi mã model trong manufacturer data (`0x0433`) khớp `FirmwareModelId`.
   Sau đó chọn RSSI mạnh nhất rồi xác nhận INFO = 1075 như hiện tại.
3. Không khớp gì thì báo lỗi, ghi rõ trong log từng loa đã thấy: địa chỉ, RSSI, tên BLE, tên Bluetooth, mã model. Người đo khỏi phải đoán.

**Chưa kiểm chứng:** bỏ trống `BleNamePrefix` trên web có làm v1.5.5 quay về cách nối theo `BleControllerAddress` đã lưu (như v1.5.4) hay không. Chủ chọn sửa code, không đổi config.

**Không dùng được ở trạm:** Ctrl+Shift+M ở v1.5.5 là "cấu hình thử". Sửa tiền tố ở đó thì kết quả thành DỮ LIỆU THỬ, phải lấy lại LINE CHUẨN thử, và không vào FACTORY FLOOR QA.

Bằng chứng: `ble-v155-autotest-connect\` (`auto-test-log.txt` + `model.json` của phiên `C360_191126_0002`, `ble-scan.txt` quét BLE lúc 11:01).

## 4. File trong thư mục này

- `HANDOFF.md`: file này
- `ble-v155-autotest-connect\`: bằng chứng §5 (log + model phiên `C360_191126_0002`, kết quả quét BLE)
- `session-C360_191126_0001\`: `auto-test-log.txt`, `summary.json`, `model.json`, `run-policy.json`, `control-readbacks.jsonl`, `MicFeedbackGain-result.json`, ảnh `FRA-IO00x.png` / `IO002-frequency-shift.png`, `wav\` (bản thu, nền, excitation của IO006/IO008)
