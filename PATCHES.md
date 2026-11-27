# 本仓库相对上游的改动

上游：<https://github.com/aa1355721445-lgtm/dsh-image-studio>
本仓库：0.5.1 → **0.5.2**

## 0.5.2 · 连接失败重试与报错信息

**问题**：宿主重启（换插件、升级 DSH）时正在跑的生成任务会因为 socket 被掐断而以

```
连不上生图接口：fetch failed
```

失败，而原来的重试是"3 次、每次固定 2 秒"，重启那几十秒正好全部撞死。

**改动**（`lib/providers.js`）：

- `run65535` 的建任务循环从 3 次改为 **4 次**，间隔改为指数退避 `1.5s / 3s / 6s`。
- 最终失败的报错带上线路主机名与提示：

  ```
  连不上生图接口：fetch failed（网络抖动或被重启打断，隔十几秒再试一次；这次用的是 task-api-1-cn.65535.space）
  ```

- 429 / 503 / 409 的退避逻辑不变（仍是 `3s * (attempt + 1)`）。

**验证**：宿主重启窗口内触发一次生成会看到上面的报错文案；重试一次即成功（实测 250 秒出图，896x1200）。

## 安装

```powershell
dsh plugin --profile desktop add "dsh-image-studio@file:D:/DSH/plugins/dsh-image-studio"
```

同 `dsh-daily-schedule` 的提醒：若以前是 `link:` 到临时目录，换 `file:` 后必须重启 DSH 宿主。
