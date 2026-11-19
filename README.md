# Handoff C360: trạm đo → Claude máy server

Nhánh mồ côi, **chỉ chứa tài liệu handoff**, không có code app. Đọc theo thứ tự:

1. [`handoff-20261119/HANDOFF.md`](handoff-20261119/HANDOFF.md): mới nhất. **#11 / §5: v1.5.5 Auto Test C360 không kết nối được BLE**, làm trước tiên vì đang chặn toàn bộ trạm đo.
2. [`handoff-20261118/HANDOFF.md`](handoff-20261118/HANDOFF.md): bối cảnh ngày 18/11.

Lấy về ở máy server (repo riêng, **không phải** repo source SoncaAudioInspector):

```
git clone -b handoff/20261119 https://github.com/superzjppr0/handoff.git C360-handoff
# lần sau: cd C360-handoff && git pull
```

Mỗi đợt handoff mới sẽ có thư mục `handoff-<yyyymmdd>/` riêng; đọc thư mục ngày mới nhất trước.

Trạm đo: SoncaAudioInspector v1.5.5, config C360 rev 5.
