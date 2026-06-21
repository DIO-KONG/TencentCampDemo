# TencentCampDemo

基于 Unreal Engine 5.7 First Person Arena 模板制作的双人 Listen Server PvPvE Demo。

项目仓库：[DIO-KONG/TencentCampDemo](https://github.com/DIO-KONG/TencentCampDemo)

## 玩法

- 两名玩家在 Arena 中对抗共享 AI 敌人，也可互相攻击。
- 击杀 AI 敌人获得 1 分，击杀对手获得 3 分。
- 玩家死亡后自动重生，个人分数保存在 PlayerState 中。
- 率先达到 10 分的玩家获胜；双方分别显示 VICTORY 或 DEFEAT。

## 运行

项目默认启动地图为 `/Game/Variant_Shooter/Lvl_ArenaShooter`，默认 GameMode 为 `BP_ShooterGameMode`。

在 Unreal Editor 中使用两名玩家、Listen Server、独立窗口模式运行 PIE，即可测试多人玩法。

## 多人实现概述

- Client 通过所属 Character 的 Server RPC 请求开火。
- Projectile、伤害、生命、死亡、计分和胜负由 Server 权威处理。
- NPC/玩家死亡表现使用 Multicast，同本地玩家相关的 HUD、受击反馈、死亡视角和比赛结果使用 Owning Client RPC。
- GameState 同步比赛结束状态、胜者和目标分；PlayerState 同步个人得分。

## 提交材料

- `report/`：蓝图实现截图、Demo 视频和可打印的 HTML 技术报告。
- `output/pdf/`：正式 PDF 技术报告。

Unreal 二进制资产、视频和 PDF 使用 Git LFS 管理。`Binaries`、`DerivedDataCache`、`Intermediate` 和 `Saved` 等生成目录不纳入版本控制。
