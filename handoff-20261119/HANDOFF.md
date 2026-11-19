# C360 — handoff 2026-11-19 (gửi Claude máy server / có source)

## ⚠ VIỆC GẤP NHẤT (15:15): có 1 bài KHÔNG ĐẠT thì kết luận KHÔNG ĐẠT và GỬI SERVER

> Chủ (nhắc lần 3): "1 bài không đạt là app phải kết luận không đạt rồi gửi lên server được rồi"

Hiện trạng v1.5.6: phiên `C360_191126_0006` có IO001 **KHÔNG ĐẠT**, nhưng vì IO008 INVALID nên tổng thành "CHƯA ĐO ĐỦ — KIỂM TRA TÍN HIỆU", log ghi `[Server] Không gửi server: chỉ gửi kết quả Auto Test hoàn tất (PASS/FAIL).` Cả ngày 19/11 chưa có phiên C360 nào lên FACTORY FLOOR QA. Trạm không đổi được luật này (không có config), phải sửa code.

**Đặc tả:**
1. Tổng = **KHÔNG ĐẠT** khi có ≥ 1 bài **bắt buộc** KHÔNG ĐẠT. Bài tùy chọn (IO009) không tính và hiện **xám**.
2. Tổng = **ĐẠT** chỉ khi mọi bài bắt buộc ĐẠT.
3. Không có bài nào KHÔNG ĐẠT nhưng còn bài INVALID hoặc chưa kết luận: tổng = "CHƯA ĐO ĐỦ" (giữ như cũ, không gửi; hoặc gửi trạng thái chưa hoàn tất nếu chủ muốn).
4. Tổng KHÔNG ĐẠT thì **gửi ngay cuối phiên** (FACTORY FLOOR QA & APP TRACKER + Quy trình QA/QC). Báo cáo ghi kèm từng bài INVALID hoặc chưa kết luận và lý do (ví dụ "IO008 Pink INVALID: độ nhạy 41,9 dB < 45 dB"). Không chờ người đo bấm TIẾP TỤC.
5. Nếu sau đó bấm TIẾP TỤC và bài INVALID có kết luận: **cập nhật cùng bản ghi** (cùng serial / mã phiên), không tạo bản ghi mới.
6. Popup cuối phiên ghi rõ: "Đã gửi server: KHÔNG ĐẠT — IO001 (thiếu echo)…" kèm mã log.
7. Bài đang "tự lấy LINE CHUẨN chưa xong" (IO006) không được hiện là "KHÔNG ĐẠT" (màn hình hiện KHÔNG ĐẠT, log ghi "không kết luận"). Phải thống nhất là chưa kết luận, và không tính vào luật 1.

Sau khi phát bản mới, trạm đo lại ngay để xác nhận lên tracker.

### Kèm theo: bấm vào bài INVALID → popup "Đo lại" (tăng âm lượng) → đo lại riêng bài đó → gửi server

> Chủ: "bài rub buzz mà đo không hợp lệ thì có thể bấm lại vào và hiện lên pop up đo lại (sau khi tăng âm lượng window lên) đo lại và gửi lên server"

Hiện trạng v1.5.6: bấm vào bài trong "Danh sách bài đo tự động" chỉ mở xem lại (`[TestReview]`). Muốn đo lại thì chỉ có TIẾP TỤC chung. App tự đặt âm lượng Windows theo config ở đầu mỗi lượt (`AcousticRubBuzz-windows-levels.json`: `playbackPercent 75`, `recordingPercent 100`), nên người đo tăng tay cũng bị kéo về 75 %.

Đặc tả:
1. Bấm vào bài **ĐO KHÔNG HỢP LỆ** (hoặc chưa kết luận) thì hiện popup **"Đo lại IO008"**, gồm:
   - lý do INVALID (ví dụ "Pink: độ nhạy 41,9 dB < 45 dB");
   - mức hiện tại: Windows phát 75 % / thu 100 %, BLE MASTER/MUSIC 50 %, `Playback Level dBFS`;
   - ô chỉnh **Windows phát (%)** (và BLE %) chỉ cho lần đo lại này, mặc định gợi ý +15–25 %, có giới hạn trên để tránh clip;
   - nút **Đo lại** / **Hủy**.
2. Đo lại **chỉ bài đó** với mức vừa chỉnh (không bị config ghi đè trong lần này). Ghi vào log và báo cáo: "đo lại IO008 với Windows phát 95 % (config 75 %)".
3. Có kết luận thì tính lại tổng theo luật ở trên rồi **gửi server** (hoặc cập nhật bản ghi đã gửi).
4. Áp dụng cho mọi bài INVALID, không riêng Rè. Bài ĐẠT/KHÔNG ĐẠT thì bấm vào vẫn chỉ xem lại.
5. (Tùy chủ) Lần đo lại thành công với mức mới thì gợi ý "Lưu mức này vào config?" và đi qua luồng gửi duyệt web như thường.

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

### 5.3 Cập nhật 13:30: v1.5.5.1 + config C360 **rev 6** (`BleNamePrefix` = trống), phiên `C360_191126_0004`

Chủ đã bỏ trống tiền tố trên web (rev 6, `cmwn5wt7k000932mbwj28ulk4`). v1.5.5.1 chỉ đổi chiều cao cửa sổ, **chưa có sửa §5.2**.

**BLE khi tiền tố trống: chưa tự nối loa mạnh nhất.**
```
13:24:05.050 Chưa kết nối được điều khiển BLE của loa. Mở Ctrl+Shift+H để kiểm tra đúng loa trước khi đo.   ← 0,2 s sau khi bấm, không quét
13:24:28.804 Chưa khôi phục được ngõ BLE trước đo; dừng Auto Test.
13:24:47.551 Tiếp tục C360_191126_0004 …   ← lần 3 mới chạy (nhiều khả năng do người đo tự nối ở Ctrl+Shift+H)
```
Chủ yêu cầu: **"nếu bài đo không có tên BLE thì tự động kết nối với thiết bị có mức mạnh nhất"**. Tiền tố trống thì quét 4 s, lấy các thiết bị có dịch vụ Sonca 0x5343, xếp theo RSSI, nối loa mạnh nhất và đọc INFO. Khớp `FirmwareModelId` (C360 = 1075) thì đo; không khớp thì thử loa mạnh kế tiếp. Có thể lọc sẵn theo mã model trong manufacturer data (`…0433…`) để khỏi phải nối thử. Loa đang nối sẵn mà INFO khớp thì dùng luôn (§5.2 bước 1).

**IO006 Mic thực tế: vẫn INVALID vì lỗi §2.1, phiên này xác nhận lại.** IO001–IO005 chạy xong (IO005 USB A FAIL: lệch line chuẩn 4,88 dB, 2/408 điểm). IO006 hỏi A/B 8 lần.

| Bản thu (`continue-2\attempt-*\AcousticMic-*`) | Matched filter peak/rms | SNR app (RMS cả 5 s so với nền) | Trong đoạn sweep so với nền |
|---|---|---|---|
| IO004 LINE IN (−6 dBFS → LINE OUT) để so | 74,2 dB | 39,3 dB | 44,6 dB |
| IO006 lượt 1, lần thử cuối | **33,7 dB, có sweep** | **7,1 dB** (app báo 7,2–8,7) | 11,4 dB |
| IO006 lượt 2, lần thử cuối 13:29:21 | 11,0 dB, không có sweep | 0,7 dB | 1,2 dB |

- Mic KT thu được sweep. Nhưng IO006 phát **−24 dBFS** (IO004 phát −6), và `MinimumSnrDb` = 20 lại tính trên RMS cả bản thu 5 s, trong khi sweep chỉ dài 1,48 s. Vì vậy cổng chặn oan.
- Popup A/B **gây hại**: lần thử cuối mất sweep, tức người đo đã bấm A/B rời khỏi 1/2 (đúng như §2.2 cảnh báo).
- **Config rev 7 (đề xuất cho chủ):** IO006 và IO008 `Playback Level dBFS` −24 → **−6** (bằng IO004, loa không to hơn bài LINE IN). SNR dự kiến khoảng 25 dB, qua ngưỡng 20. Không nên hạ `MinimumSnrDb`: ở SNR 7–11 dB, THD bị nền ồn chi phối nên không có nghĩa.
- **Code (§2.1, §2.2):** cổng nhận tín hiệu cho `AcousticMic`/`AcousticRubBuzz` dùng matched filter, hoặc ít nhất SNR trong đoạn sweep. Popup A/B chỉ hiện khi matched filter không thấy excitation.

Bằng chứng: `session-C360_191126_0004\` (log, model.json, các wav IO004/IO006).

## 6. Lần đo 13:41 do Claude tự chạy (`C360_191126_0004_134133109`, v1.5.5.1, config rev 6): danh sách lỗi

Chủ: "bạn tự đo lại và tìm lỗi". BLE nối được ngay lần đầu (chuẩn bị BLE 8,9 s). Tổng 247 s.

| Bài | Kết quả | Ghi chú |
|---|---|---|
| IO001 Echo/Reverb | KHÔNG ĐẠT (`ECHO_MISSING`) | reverb 210 ms, không có echo |
| IO002 Di tần | ĐẠT | FBX không đổi di tần (−2,995 / −3,001 Hz) |
| IO003 MIC khô / IO004 LINE IN / IO005 USB A / IO007 BT | ĐẠT | IO004 không cần A/B |
| IO006 Mic thực tế | ĐO KHÔNG HỢP LỆ | có sweep, bị cổng SNR chặn (§6.2) |
| IO008 Rè | ĐO KHÔNG HỢP LỆ | lượt 1 có sweep nhưng bị chặn, lượt 2 mất tín hiệu (§6.3) |
| IO009 Gain trước hú | KHÔNG ĐẠT | FBX tắt/bật đều hú ở −27 dB, 118 Hz; FBX thêm +0,0 dB (yêu cầu ≥ 6) |
| **Tổng** | **CHƯA ĐO ĐỦ, không gửi server** | dù đã có 2 bài FAIL (§6.5) |

### 6.1 IO001: thiếu echo phải là FAIL, không phải "chưa xác định" → CODE APP

> Chủ: "echo không có mà sao lại ghi chưa xác định phải báo fail chứ"

- Lần 13:24 (rev 6): màn hình ghi "Echo chưa xác định: **nền IR cao**", kết luận **ĐO KHÔNG HỢP LỆ**. Lần 13:41: nền IR −62,9 / −62,7 dB, ra **FAIL `ECHO_MISSING`**. Cùng loa, cùng cấu hình, mà kết luận lúc INVALID lúc FAIL. Dữ liệu lần 13:24 đã bị app xóa ("Đã xóa dữ liệu phiên đo sai"), chỉ còn chữ trên màn hình.
- Binary có nhánh `… chưa xác định: {nền IR cao | đuôi thu quá ngắn | đuôi thu bị cắt}`.
- Config IO001: `ExpectEcho: true`, `MinimumEchoLevelDb: −45`, `MinimumEchoProminenceDb: 10`.
- **Sửa:** `ExpectEcho` = true mà không thấy echo thì **FAIL** (ghi kèm "nền IR cao" trong finding nếu có), không xếp INVALID. Nếu muốn chắc thì đo lại tối đa `MaximumRuns`, hết lượt vẫn không thấy echo thì FAIL. Bằng chứng phụ: cùng IR đó app **vẫn phát hiện được reverb** (đuôi 168–210 ms, SNR đuôi 16–17 dB), nên echo ở mức −45 dB cũng phải thấy được nếu có.
- Còn mở (§3 mục 5): echo bật (`6404=1`) mà vẫn không có echo. Mixer Console hiện ECHO **Wet = "—"** (không đọc được), nên xem lại tham số Wet của echo trên firmware C360.

### 6.2 IO006 (cổng "Measurement Mic" = KT USB): mic thu tốt, cổng SNR chặn oan → CODE + CONFIG

| Bản thu | Matched filter | SNR app (cả 5 s) | Trong đoạn sweep |
|---|---|---|---|
| IO006 lượt 1, lần thử cuối | **32,0 dB** | 4,7 dB | 9,0 dB |
| IO006 lần thử 13:43:26 | 13 dB (không có sweep) | 1,3 dB | 2,2 dB |

App kết luận: "SNR thu 12,0 dB < ngưỡng cấu hình 20,0 dB". Đây là §2.1, chưa sửa. Đề xuất config rev 7 (IO006/IO008 phát −24 → −6 dBFS) và sửa code cổng nhận tín hiệu bằng matched filter.

### 6.3 Popup A/B làm người đo bấm sai, mất tín hiệu thật → CODE

Trong IO006 người đo bấm A/B theo popup: 13:43:26 SNR 3,3 dB (mất sweep), 13:43:42 SNR 7,0 dB (có lại). IO008 lượt 1 (13:44:30) có sweep (matched filter 34 dB). **IO008 lượt 2 (13:44:55) mất hẳn** (matched filter 9 dB, −48,8 dBFS = nền). Sau đó IO009 (MIC IN 3/4) hú được, tức A/B lúc đó đang ở 3/4. Sáng nay IO008 lượt 2 cũng mất tín hiệu y hệt. Sửa theo §2.2: chỉ hỏi A/B khi matched filter không thấy excitation, popup ghi rõ chiều cần chuyển, không hỏi ở bài âm học khi có sweep.

### 6.4 IO008 Rè: vẫn hiện hình FFT, và phần Rè chưa từng chạy → CODE

> Chủ: "sao rè vẫn để hình FFT ???"

- `FRA-IO008.png` và khung đồ thị vẫn là **đáp tuyến dBFS 1/24 octave** (§1.3 chưa làm trong v1.5.5.1).
- Nặng hơn: stage `AcousticRubBuzz` phát **sweep 1,48 s** để kiểm tra tín hiệu (excitation giống IO006, 262 202 byte) và bị cổng SNR chặn, nên **pink + chirp rè không được phát**, log ghi "Chưa tính THD cho lượt này". Ảnh xuất lấy từ lượt 2 không có tín hiệu nên toàn nhiễu.
- **Sửa:** (a) cổng nhận tín hiệu như §6.2; (b) IO008 xuất ảnh và hiện giao diện Ctrl+Shift+I → Rè Pink (bảng HF / P95 / sự kiện/s / peak-crest + envelope), màn hình chính hiện chirp; (c) nếu bước kiểm tra tín hiệu fail thì ảnh phải ghi rõ "KHÔNG CÓ TÍN HIỆU", đừng vẽ nhiễu như đáp tuyến.

### 6.5 Phiên có FAIL vẫn không lên server → CODE + API (§3 mục 4)

IO001 và IO009 KHÔNG ĐẠT, nhưng IO006/IO008 INVALID khiến tổng thành "CHƯA ĐO ĐỦ — KIỂM TRA TÍN HIỆU", và log ghi `[Server] Không gửi server: chỉ gửi kết quả Auto Test hoàn tất (PASS/FAIL)`. Chủ muốn thấy báo cáo trên server. Đề xuất: **có bài FAIL thì tổng là FAIL và gửi luôn** (liệt kê bài INVALID trong báo cáo); hoặc thêm trạng thái gửi "CHƯA HOÀN TẤT" lên `/report`.

### 6.6 "Bấm đo lại có lúc bị ẩn màn hình": chưa tái hiện

Bộ ghi trạng thái cửa sổ mỗi 0,7 s suốt lần chạy: cửa sổ chính luôn `visible=1, minimized=0`, 0,0 1366×728. Popup "Kết quả QA · ID tự tạo khi đo · C360" (13:41:23) và "Thông báo" hiện đúng. Lúc 13:45:29 có cửa sổ Sound (`rundll32 mmsys.cpl,,sounds`, cha là **explorer**, tức người dùng tự mở) lên trước. Cần chủ mô tả lại: bấm nút nào thì ẩn, ẩn là thu nhỏ, mất hẳn hay popup nằm sau cửa sổ chính. Nghi vấn: sau thay đổi kích thước cửa sổ ở v1.5.5.1, popup có thể không còn Owner hoặc Topmost nên bị cửa sổ chính toàn màn hình che.

Bằng chứng: `session-C360_191126_0004-run2\` (log, model.json, control-readbacks, ảnh FRA, wav IO006/IO008 hai lượt, `IO001-MicEffects-excerpt.json`).

## 7. v1.5.6 + config rev 7: lần đo 14:47 (`C360_191126_0005_144705053`)

v1.5.6 đã làm được: tự nối BLE (8,8 s), không còn popup A/B, IO001/IO002/IO008/IO009 bỏ LINE CHUẨN. Rev 7: IO006/IO008 phát −6 dBFS, nên **cổng SNR đã qua** (IO006 SNR 27,3 dB; sweep kiểm của IO008 SNR 22 dB, matched filter 48 dB). Tổng 155 s, nhưng **vẫn không gửi server**.

| Bài | Kết quả | Lý do |
|---|---|---|
| IO001 | KHÔNG ĐẠT | `ECHO_MISSING` (lần này không bị "chưa xác định") |
| IO002, IO004, IO005, IO007 | ĐẠT | |
| IO003 MIC khô | **KHÔNG ĐẠT** | THD 1 kHz **0,700 %** > 0,5 %. Đáp tuyến khớp LINE CHUẨN 0/408 điểm. Cùng loa: 0,443 % (13:25), 0,213 % (13:42), 0,700 % (14:48) |
| IO006 Mic thực tế | "KHÔNG ĐẠT" trên màn hình, log ghi "không kết luận" | "Tự lấy LINE CHUẨN chưa xong (lượt 1/3)", THD 1 kHz 0,568 % |
| IO008 Rè | ĐO KHÔNG HỢP LỆ (log lại ghi "1/1 lượt đạt") | Pink/Chirp: `0x8889000A` thiết bị đang bị giữ, `FRA-IO008.png` trống |
| IO009 | KHÔNG ĐẠT (màu đỏ) | FBX thêm −1,0 dB (yêu cầu ≥ 6) |
| **Tổng** | **CHƯA ĐO ĐỦ, không gửi** | dù có 3 bài KHÔNG ĐẠT |

### 7.1 Có bài KHÔNG ĐẠT thì tổng là KHÔNG ĐẠT và gửi server → CODE (ưu tiên 1)

> Chủ: "có không đạt thì kết quả chung là không đạt và gửi lên server chứ"

Log: `[Server] Không gửi server: chỉ gửi kết quả Auto Test hoàn tất (PASS/FAIL).` Đổi luật: có ít nhất 1 bài **bắt buộc** KHÔNG ĐẠT thì tổng = **KHÔNG ĐẠT** và **gửi ngay** (báo cáo kèm danh sách bài INVALID hoặc chưa kết luận). Chỉ khi không có FAIL nào mà còn INVALID mới là "CHƯA ĐO ĐỦ". v1.5.6 hứa "đo hết các bài rồi báo kết quả gửi server" nhưng chưa đúng trong trường hợp này.

### 7.2 IO009 tùy chọn: hiển thị **màu xám**, không đỏ/xanh → CODE (+ config)

> Chủ: "gain trước hú thì để màu xám thôi vì không xét"

Bài tùy chọn (IO009) luôn hiện xám (badge, danh sách, LblVerdict, ảnh, báo cáo server), chỉ ghi số đo, không dùng chữ "KHÔNG ĐẠT". Config rev 5 có `MinimumAddedDb: 6`; nếu không xét thì bỏ trống trên web.

### 7.3 Sai số LINE CHUẨN 10 %: đã có trong config nhưng đang TẮT → CONFIG web

> Chủ: "mấy bài có line chuẩn cho phép sai số 10% chưa"

Mọi bài: `AllowLineOutliers: false`, `AllowedLineOutlierPercent: 10`, nên log ghi "cho phép ≤0%". Chiều 13:26, IO005 FAIL chỉ vì 2/408 điểm (0,49 %) lệch tối đa 4,88 dB. Bật `AllowLineOutliers: true` cho IO003–IO007 thì cho phép 10 % điểm ngoài ±3 dB.

### 7.4 IO003 THD MIC IN dao động 0,21–0,70 % → ĐIỀU TRA (code)

> Chủ: "mic in không effect đẹp thế mà không đạt hả"

FAIL do **THD**, không phải đáp tuyến. Bản thu sạch (matched filter 69 dB, SNR 39–44 dB). **Giả thuyết, chưa kiểm chứng:** IO002 đo được đường MIC luôn bị **dịch tần ≈ −3 Hz kể cả khi FBX TẮT**. Dịch tần làm hệ thống không còn bất biến theo thời gian, nên THD tính bằng log-sweep deconvolution bị rò năng lượng sang cửa sổ hài, ra số ngẫu nhiên theo pha. Đề xuất: với MIC IN, đo THD bằng sine 1 kHz ổn định (FFT, cửa sổ dài) thay cho sweep, hoặc tắt hẳn dịch tần trước bài khô nếu firmware cho phép. Kiểm lại readback xem bài "Không effect" có còn FBX hoặc shifter không.

### 7.5 IO008 Rè không có hình: `0x8889000A` (AUDCLNT_E_DEVICE_IN_USE) → CODE + trạm

Sweep kiểm tín hiệu chạy được (shared mode), nhưng Pink/Chirp mở thiết bị (nhiều khả năng **exclusive**) thì lỗi "đang bị giữ". Trên trạm lúc đó còn một tiến trình **`rundll32 mmsys.cpl,,sounds` mồ côi** (mở 13:45, không còn cửa sổ nào, vẫn nạp AudioSes/MMDevApi). Chưa xác nhận đây là thủ phạm (Claude bị chặn đóng tiến trình). Code nên:
(a) thử lại ở shared mode khi exclusive bị `0x8889000A`;
(b) ghi rõ **thiết bị nào** (mic KT hay ngõ phát FastTrack) và **tiến trình nào** đang giữ;
(c) log không được ghi "1/1 lượt đạt" khi Pink/Chirp chưa chạy (màn hình thì ghi ĐO KHÔNG HỢP LỆ, hai chỗ mâu thuẫn).

### 7.6 IO006 tự lấy LINE CHUẨN không bao giờ xong trong một phiên → CODE

Popup đầu phiên: "Chưa có LINE CHUẨN… IO006… CÓ: mỗi bài thiếu chuẩn đo thêm 2–3 lượt để lưu, rồi đo và kết luận ngay". Thực tế: "Chưa lưu LINE CHUẨN (lượt 1/3)", `MaximumRuns` = 2, nên IO006 "không kết luận" và làm phiên không hoàn tất. Hoặc đo đủ 3 lượt chuẩn ngay trong phiên như popup nói, hoặc không tính bài đang lấy chuẩn là INVALID. Ngoài ra màn hình ghi "KHÔNG ĐẠT" trong khi log ghi "không kết luận", lại mâu thuẫn. Popup này được trả lời "Có" chỉ 1,5 s sau khi hiện (14:47:16 → 14:47:17,9): cần xác nhận có phải mặc định hoặc phím Enter từ popup QA trước đó lọt sang không.

Bằng chứng: `session-C360_191126_0005\` (log, model.json, ảnh FRA, wav IO003/IO006/IO008).

### 7.7 Lần đo 15:04 (`C360_191126_0006_150358242`, v1.5.6, config **rev 8**)

Rev 8: `AllowLineOutliers` = true cho 5 bài có LINE CHUẨN, IO009 `MinimumAddedDb` = null. Trước khi đo, tiến trình `rundll32 mmsys.cpl` mồ côi đã được đóng.

| Bài | Kết quả |
|---|---|
| IO001 | KHÔNG ĐẠT (`ECHO_MISSING`) |
| IO002, IO004, IO007 | ĐẠT |
| IO003 | ĐẠT. THD 1 kHz **0,290 %** (cùng loa: 0,21 / 0,44 / 0,70 / 0,29 %, xem §7.4) |
| IO005 | ĐẠT, 1/408 điểm lệch, "cho phép ≤10%" (ngưỡng 10 % có hiệu lực) |
| IO006 | "KHÔNG ĐẠT" / không kết luận: **vẫn "LINE CHUẨN lượt 1/3"** như phiên trước, THD 1 kHz 0,686 % (SNR 20,3 dB) |
| IO008 | ĐO KHÔNG HỢP LỆ: **Pink INVALID "Độ nhạy 41,9 dB < 45 dB: phòng ồn hoặc mic xa loa"**, Chirp PASS (đỉnh rè 0,0 dB @ 23 Hz). Không còn `0x8889000A`, nên **xác nhận rundll32 mmsys.cpl mồ côi là nguyên nhân §7.5** |
| IO009 | Màn hình ghi **"ĐẠT" màu xanh**, trong khi log ghi "chưa đặt ngưỡng MinimumAddedDb nên chỉ báo số" → phải **xám** (§7.2) |
| Tổng | CHƯA ĐO ĐỦ. Popup: "Chưa gửi server: còn bài chưa kết luận PASS/FAIL — IO008 INVALID", dù IO001 KHÔNG ĐẠT (§7.1) |

Phát hiện thêm:
- **IO006 tự lấy LINE CHUẨN không tiến được qua các phiên** (14:49 và 15:05 đều "lượt 1/3"). Có vẻ lượt chỉ được tính vào chuẩn khi THD qua ngưỡng, mà THD âm học 0,57–0,69 % luôn > 0,5 %, nên không bao giờ có chuẩn. Chủ quyết: **IO006 cho phép THD < 0,7 %** (config). Code: tách điều kiện lưu chuẩn khỏi ngưỡng THD, hoặc báo rõ "lượt không được tính vì THD".
- **IO008 Pink cần to hơn:** chủ quyết **tăng BLE của IO008 lên 80 %** (config). Nếu thang BLE tuyến tính, 50 % → 80 % là +4,1 dB, đủ để 41,9 → khoảng 46 dB > 45.
- **Ảnh chirp IO008 là một đường phẳng 0,0 dB** từ 23 Hz đến khoảng 1,8 kHz, không thấy các lượt màu xám. Kiểm tra lại: hoặc giá trị đang bị kẹp ở 0 (dư rè ≤ 0 thì vẽ 0), hoặc chưa có dữ liệu mà vẫn vẽ.
- **Popup "Chưa có LINE CHUẨN" lại được trả lời "Có" sau 1,4 s** (15:04:09,5 → 15:04:10,9). Ảnh chụp màn hình không có đếm ngược, nút "Có — lấy chuẩn rồi đo" đang được focus. Hai lần liên tiếp khoảng 1,5 s: nghi phím Enter của popup QA trước đó lọt sang, hoặc tự chọn mặc định.
- **Chọn model C360 trong ComboModels hình như tự bắt đầu Auto Test** (15:03:58, Claude chưa bấm nút; cũng có thể người ở trạm bấm đúng lúc đó). Cần kiểm tra trong code.

Bằng chứng: `session-C360_191126_0006\`.

## 4. File trong thư mục này

- `HANDOFF.md`: file này
- `ble-v155-autotest-connect\`: bằng chứng §5 (log + model phiên `C360_191126_0002`, kết quả quét BLE)
- `session-C360_191126_0001\`: `auto-test-log.txt`, `summary.json`, `model.json`, `run-policy.json`, `control-readbacks.jsonl`, `MicFeedbackGain-result.json`, ảnh `FRA-IO00x.png` / `IO002-frequency-shift.png`, `wav\` (bản thu, nền, excitation của IO006/IO008)
