# Handoff C360: trạm đo → Claude máy server

Nhánh mồ côi, **chỉ chứa tài liệu handoff**, không có code app. Đọc theo thứ tự:

1. [`handoff-20261119/HANDOFF.md`](handoff-20261119/HANDOFF.md): mới nhất. **#11 / §5: v1.5.5 Auto Test C360 không kết nối được BLE**, làm trước tiên vì đang chặn toàn bộ trạm đo.
2. [`handoff-20261118/HANDOFF.md`](handoff-20261118/HANDOFF.md): bối cảnh ngày 18/11.

Lấy về ở máy server:

```
git fetch origin handoff/20261119
git worktree add ../C360-handoff origin/handoff/20261119   # hoặc: git show origin/handoff/20261119:handoff-20261119/HANDOFF.md
```

Trạm đo: SoncaAudioInspector v1.5.5, config C360 rev 5.
