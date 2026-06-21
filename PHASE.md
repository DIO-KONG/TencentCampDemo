# TencentCampDemo 开发阶段记录

> 最后更新：2026-06-21  
> 当前阶段：**Phase 7：提交材料完成与最终上传**  
> 状态原则：只有经过编译、运行测试或用户明确确认的结果，才可标记为已完成。Blueprint 内部 Graph 尚未审阅时，不根据资产名猜测实现。

## 1. 项目总体目标

基于 UE5 官方 First Person 模板的 Arena 变体，完成一个稳定、可运行、可录制演示并可提交的双人多人游戏 Demo。最终提交物为 Demo 视频和 PDF 技术说明；PDF 需简述实现过程并附 GitHub 仓库链接。

## 2. 作业要求

1. 实现能够移动和攻击玩家的敌人，且玩家可以击败敌人。
2. 实现基础得分和游戏胜利机制，具体玩法可自行设计。
3. 实现多人网络对战。
4. 时间允许时，再考虑渲染、AI、音效、特效或其他表现优化。

## 3. 初始化审计结果

### 3.1 已验证事实（文件只读检查或用户运行确认）

#### 文件检查确认

- 项目描述文件为 `TencentCampDemo.uproject`，`EngineAssociation` 为 `5.7`。
- 项目启用了 `ModelingToolsEditorMode`（Editor）和 `GameplayStateTree` 插件。
- 项目根目录存在 `Config`、`Content`、`DerivedDataCache`、`Intermediate`、`Saved`；不存在项目级 `Source` 和 `Plugins` 目录，因此当前可见的项目玩法实现是 Blueprint/资产驱动，而不是项目 C++ 源码驱动。
- Content 中有 453 个 `.uasset` 和 2 个 `.umap`。两张地图为：
  - `/Game/FirstPerson/Lvl_FirstPerson`
  - `/Game/Variant_Shooter/Lvl_ArenaShooter`
- Arena 变体存在以下相关 Blueprint 资产，但其内部 Graph 尚未审阅：
  - 玩家与规则：`BP_ShooterCharacter`、`BP_ShooterGameMode`、`BP_ShooterPlayerController`、`BPI_Shooter`
  - AI：`BP_ShooterAIController`、`BP_ShooterNPC`、`BP_ShooterNPCSpawner`
  - AI 逻辑资产：`ST_Shooter`、`ST_Shooter_ShootAtTarget`、EQS 和多个 StateTree Task/Condition
  - 武器与弹丸：`BP_ShooterWeaponBase`、三种武器、Bullet/Grenade Projectile 和爆炸资产
  - UI：`UI_Shooter`、`UI_ShooterBulletCounter`
- 资产路径中没有发现命名明确的 Arena `GameState` 或 `PlayerState` 资产；这只说明没有按该名称暴露的独立资产，不能证明相关逻辑不存在或未由父类承担。
- `DefaultEngine.ini` 已更新并通过文本检查确认：
  - `EditorStartupMap=/Game/Variant_Shooter/Lvl_ArenaShooter`
  - `GameDefaultMap=/Game/Variant_Shooter/Lvl_ArenaShooter`
  - `GlobalDefaultGameMode=/Game/Variant_Shooter/Blueprints/BP_ShooterGameMode`
- `DefaultEngine.ini` 设置 `bUseSplitscreen=True`；这不是网络多人功能已完成的证据。
- Arena 地图带有 World Partition 配置侧车文件。
- 根目录 `PHASE.md` 记录开发过程，`README.md` 已包含稳定玩法、运行方式、多人实现和提交材料入口。
- 项目已初始化为本地 `main` Git 仓库，并关联 `https://github.com/DIO-KONG/TencentCampDemo.git`。
- `.gitignore` 已排除 `Binaries`、`DerivedDataCache`、`Intermediate`、`Saved`、IDE 缓存及本地报告渲染临时文件。
- `.gitattributes` 已通过 Git LFS 管理 `.uasset`、`.umap`、`.mp4` 和 `.pdf`。

#### 用户单机运行观察确认

以下仅视为用户在单机运行下确认的模板行为，不外推为多人正确：

- Arena 地图可运行，并一次性生成约两个敌人。
- 敌人能够移动、寻找玩家并向玩家射击。
- 玩家能够攻击敌人。
- 玩家与敌人具有生命值、受伤和死亡逻辑。
- 存在基础武器、射击、战斗表现以及部分 HUD/战斗 UI。

#### 用户在 Unreal Editor 中确认

- 2026-06-21，用户通过 Arena 地图的 World Settings 截图确认：
  - `GameMode Override = BP_ShooterGameMode`
  - `Default Pawn Class = BP_ShooterCharacter`
  - `Player Controller Class = BP_ShooterPlayerController`
  - `HUD Class = HUD`（引擎默认类）
  - `Game State Class = GameStateBase`（引擎默认类）
  - `Player State Class = PlayerState`（引擎默认类）
  - `Spectator Class = SpectatorPawn`（引擎默认类）
- 上述结果确认 Arena 具有地图级 GameMode 绑定，也确认当前没有通过该 GameMode 配置自定义 GameState/PlayerState；这仍不能证明 GameMode、Pawn 或 PlayerController 内部已实现多人逻辑。

### 3.2 合理推断（尚未验证）

- `BP_ShooterNPCSpawner` 已确认包含初始敌人生成与死亡事件绑定；其网络复制属性和死亡后续链仍需审阅。
- StateTree/EQS 资产很可能参与敌人感知、移动、选位和射击，但无法仅凭资产名确定目标选择方式或网络执行端。
- 第三人称武器动画 Blueprint 的存在说明模板可能提供远端角色表现所需素材，但不能证明网络端已正确显示角色、武器或动画。
- 模板可能已有局部 Replication/RPC 设置；二进制 Blueprint 未审阅，当前不能确认其覆盖范围或正确性。

### 3.3 未验证内容

以下项目一律不得标记为已完成：

- Blueprint 中是否已有 Replication、RepNotify、RPC、Authority 或 Listen Server 设计。
- `BP_ShooterGameMode`、`BP_ShooterCharacter` 和 `BP_ShooterPlayerController` 内部是否具有正确多人实现。
- 未审阅的 Projectile 类型及其他武器是否遵循当前 Bullet 的 Server 权威方案。
- AI 的完整执行链是否只在 Server 运行；当前仅通过运行结果确认共享 NPC 与双方伤害一致。
- 远端玩家角色、持有武器、射击动画、枪口与命中特效是否正确可见。
- 比赛结束时如何停止 Spawner 补充及清理待执行 Delay/Timer。
- 是否存在可复用计分逻辑，以及 GameMode、GameState、PlayerState 或比赛状态流程的真实实现。
- 玩家死亡后的当前行为和适合本 Demo 的重生规则。
- 打包构建、双实例运行和最终演示启动方式。

## 4. 当前推荐玩法

当前玩法调整为两名玩家在 Listen Server Arena 中进行轻量 PvPvE 计分竞争：

- 两名玩家进入同一 Arena。
- AI 可选择并攻击任意存活玩家。
- 场上维持有限数量敌人，死亡后延迟补充。
- 每端 HUD 只显示本地玩家个人分数；击杀 NPC 得 1 分，击杀对手得 3 分。
- 比赛采用限时计分，可选达到目标分数提前结束。
- 结束时按本地玩家结果显示 Victory、Defeat 或 Draw。
- 玩家死亡后自动重生，PlayerState 个人分数跨 Pawn 重生保留。
- 已开放玩家之间的 PvP 伤害与计分；因此必须验证玩家死亡后的重生，否则 PvP 只能发生一次。

建议技术归属（待 Phase 1 审计后确认）：

- `GameMode`：仅 Server 的规则、比赛开始/结束、生成和胜负判定。
- `GameState`：复制比赛阶段、倒计时、胜者与公共状态。
- `PlayerState`：复制每名玩家的击杀数、得分和必要统计。
- `PlayerController`：本地 UI、输入模式及结果展示。
- Character/Health 组件或现有对应逻辑：玩家实体生命和死亡状态。

## 5. 范围限制

为最短时间完成稳定演示，默认不做：

- Steam、EOS 或其他 Online Subsystem 集成。
- 公网匹配、房间列表、复杂 Lobby 或 Dedicated Server。
- 多武器扩展、多种敌人类型。
- 背包、装备、存档或长期成长。
- 复杂 Behavior Tree 或大规模 AI 重构。
- 大规模地图、美术重制或与作业无关的架构重构。

优先复用 Arena 模板已有武器、伤害、生命值、敌人 AI、动画、音效和 UI。表现优化必须排在多人战斗、AI、生成、计分和胜负闭环稳定之后。

## 6. 阶段计划与验收标准

### Phase 0：Git 与项目基线

目标：建立可恢复、适合 UE 二进制资产的版本管理基线。

计划：

- 确认仓库初始化位置与 GitHub 目标仓库。
- 创建适用于 UE5 的 `.gitignore`，至少排除 `Binaries`、`DerivedDataCache`、`Intermediate`、`Saved`、IDE 缓存和本地用户文件。
- 决定并配置 Git LFS 管理 `.uasset`、`.umap`；建立 `.gitattributes` 后再添加二进制资产。
- 检查实际跟踪集合，避免把生成目录加入历史。
- 在用户确认模板基线可运行后创建基线提交；不执行破坏性 Git 操作。

验收：`git status` 只包含应提交的项目文件，生成目录被忽略，LFS 规则生效，且存在用户确认可恢复的模板基线提交。

当前结果：**已完成。** 2026-06-21 建立 `main` 基线提交，482 个项目/文档文件进入版本控制，其中 461 个 Unreal、视频或 PDF 文件由 Git LFS 管理；生成目录未纳入仓库。远程 `main` 与本地提交 SHA 已核对一致。

### Phase 1：模板结构与多人能力审计

目标：查清 Arena 的实际类绑定和现有网络能力，以测试结果决定最小修改范围。

计划：

- Arena 的 World Settings/GameMode Override、Default Pawn、PlayerController 已确认；继续审计 NPC/Spawner 的放置与引用。
- 只读审阅玩家、武器、Projectile、NPC、AI Controller、Spawner、GameMode 和 UI Blueprint 的父类、Class Defaults、Replication 与 RPC 设置。
- 搜索 Blueprint 中的 Authority、Switch Has Authority、Run on Server、Multicast、RepNotify、Get Player Character/Player 0、Apply Damage、Spawn Actor、Destroy Actor 等关键节点。
- 执行两人 PIE Listen Server 人工测试并逐项记录 Host/Client 现象。

验收：明确两个玩家的生成、控制和互见情况；分别验证 Host/Client 攻击敌人、敌人攻击双方、生命/死亡/销毁同步，并确认 AI、生成和伤害的权威执行端。审计完成不等于功能通过，失败项必须单独记录。

当前结果：**已完成核心审计。已定位并修复远端 Mesh、Anim Blueprint 空引用、Spawner 客户端重复生成、Client 开火、NPC/玩家死亡表现与拥有者 UI 问题。**

### Phase 2：多人战斗闭环

目标：以最小修改修复 Client 开火、Server 权威伤害和关键状态复制。

计划：修复 Client 输入到 Server 的开火链路；由 Server 生成/判定 Projectile 或命中；复制生命与死亡状态；AI 仅在 Server 执行；保证最低可接受的远端角色和武器表现。

验收：Host 和 Client 均能击杀同一敌人，所有端看到一致的受伤、死亡与销毁结果，且没有重复弹丸或重复伤害。

当前结果：**已通过。Host/Client 对 NPC、AI 对双方玩家及死亡表现的组合回归未发现问题。**

### Phase 3：AI 多玩家目标选择

目标：消除 Player 0/唯一玩家假设，在 Server 上从存活玩家中选择有效目标。

计划：优先复用现有 StateTree/EQS/感知；采用感知事件或低频 Timer 更新，避免每帧全局扫描；目标死亡、断开或失效后能切换。

验收：敌人可攻击两名玩家，并能在目标失效或条件变化后合理切换；Host 与 Client 观察结果一致。

当前结果：**已通过。用户确认双人 PIE 中索敌、目标失效和多玩家切换机制均无问题。**

### Phase 4：敌人持续生成循环

目标：建立 Server 权威、数量受控且可停止的补充生成闭环。

计划：复用/扩展现有 Spawner；设置最大存活数与死亡补充延迟；比赛结束停止生成；防止重复计数、无限生成及 Timer 残留。

验收：连续击杀多个敌人后仍能按延迟补充，场上数量不越界，比赛结束后不再生成。

当前结果：**核心生成循环已通过。两个 Spawner 的 `Spawn Count` 分别由 1 调整为 10，每个出生点顺序生成最多 10 个、同时仅维持 1 个活动 NPC，整场合计最多 20 个。比赛结束停止生成待 Phase 5 比赛状态建立后接入。**

### Phase 5：得分与比赛状态

目标：实现可靠击杀归属、同步计分、最小比赛状态和胜负判定。

计划：采用 Waiting/Playing/Finished；防止同一敌人重复死亡/加分；限时结束，可选目标分提前结束；按玩家显示 Victory/Defeat/Draw；依据审计结果确定重生或等待。

验收：Host 或 Client 击杀均只计分一次；所有客户端显示一致的比分、倒计时、比赛状态和最终结果。

当前结果：**作业所需基础计分与胜利机制已通过验收。GameState 复制 Match Finished/Winner/Target Score，GameMode Authority-only 且防重复结束；达标后分数冻结；正式 Target Score 为 10。赛后世界完全冻结属于未完成优化，不阻断本次作业功能要求。**

### Phase 6：最小 UI 与启动流程

目标：提供可录制演示的开始、战斗 HUD 和结果反馈。

计划：建立或扩展最小开始界面；HUD 显示生命、时间和双方得分；结果显示 Victory/Defeat/Draw；客户端 UI 不读取仅存在于 Server 的 GameMode；正确切换输入模式。

验收：双实例从开始到结束可完整操作，双方 UI 数据正确，结果与权威比赛状态一致。

当前结果：**`UI_MatchResult` 已通过 Host/Client 双向获胜视觉验收：胜者显示 VICTORY、败者显示 DEFEAT，Widget 不重复创建且无 Runtime Error。**

### Phase 7：稳定性、展示与提交

目标：完成回归、打包、文档和低风险展示优化。

计划：双客户端完整回归；核对默认地图/GameMode；验证打包启动；核对 GitHub 必需资产；更新 README 稳定运行说明；整理 PDF 所需架构、多人数同步、AI、生成、计分和胜负说明；最后考虑颜色区分、命中反馈、特效、音效、灯光和 Post Process。

验收：可重复完成双人完整比赛，可打包启动并录制；GitHub 可恢复项目；README、Demo 视频和 PDF 内容齐备。

## 7. 当前阶段、阻塞与最近测试

### 当前阶段

**Phase 7：提交材料完成与最终上传。**

根据 2026-06-21 截止前评估，作业三项功能要求均已有运行测试证据，Demo 功能层面基本达标。立即停止新增玩法，优先录制 Demo、建立 GitHub 可恢复仓库并完成 PDF。

### 当前阻塞

1. Match Finished 后分数会冻结，但玩家输入、AI 伤害和 Spawner 补充尚未停止；该项为已在报告披露的非阻断限制，视频在结果出现后结束即可。
2. 项目尚未验证打包，但题面提交格式只明确要求 Demo 视频与 PDF；当前交付证据采用双窗口 PIE 录制。
3. GitHub、视频和 PDF 已准备完成，剩余外部步骤仅为用户在作业平台上传最终文件。

### 最近测试结果

- 2026-06-21：完成只读初始化审计；未执行 Unreal Editor 编译、PIE 多人运行或打包测试。
- 用户此前确认单机 Arena 基础 PvE 可运行；多人结果为空。
- 2026-06-21：用户通过 World Settings 截图确认 Arena 使用 `BP_ShooterGameMode`、`BP_ShooterCharacter`、`BP_ShooterPlayerController`，并使用默认 `GameStateBase`、`PlayerState` 和 `HUD`。
- 2026-06-21：完成第一次两人 PIE Listen Server 测试：
  - 已确认 Server 与 Client 均生成可控制玩家，且双方能够看见对方。
  - 远端玩家出现两个身体：一个第三人称身体正常，另一个随视角发生明显扭曲。
  - Server 日志持续出现 `ABP_FP_Weapon` Blueprint Runtime Error：`Accessed None trying to read property CallFunc_GetController_ReturnValue`，出错节点涉及 `Set` 与 `Set Pitch`。
  - PIE 日志有 `Not enough login credentials to launch all PIE instances, change editor settings` 警告；两个实例仍成功登录并启动，因此当前将其记录为非阻断警告，不视为上述玩法错误的根因。
  - Client 无法攻击 AI 敌人。
  - Server 与 Client 观察到的 AI 状态不一致。
  - 初始约两个敌人之外，Client 窗口获得焦点后又出现约两个不会移动的敌人。
  - 用户检查 `BP_ShooterCharacter` 组件后确认：`Mesh (CharacterMesh0)` 与 `FirstPersonMesh` 的 `Owner No See`、`Only Owner See` 当前均为关闭。
  - 用户将 `FirstPersonMesh.Only Owner See` 设为启用、`CharacterMesh0.Owner No See` 设为启用，编译保存后重新执行双人 PIE；双方观察对方时均不再看见第一人称 Mesh，双身体问题已验证修复。
  - 用户定位到第一人称武器 Anim Blueprint 的 `Event Blueprint Update Animation`：现有 Graph 只用 `Is Valid` 检查 `Try Get Pawn Owner`，随后 Sequence 分支直接执行 `Get Controller -> Get Control Rotation`；远端模拟代理 Pawn 有效但没有本地 Controller，错误最终在使用这些纯节点结果的两个 `Set` 上触发。用户决定先只在手枪 Anim Blueprint 验证修复方案。
  - 用户在 `ABP_FP_Pistol` 的有效 Pawn 检查与 Sequence 之间增加 `Is Locally Controlled -> Branch`，仅 True 执行更新；双人 PIE 中手枪不再报该空引用，Host/Client 动画与瞄准均正常。手枪闭环已验证完成，其他武器未标记完成。
  - 用户审阅 `BP_ShooterNPCSpawner` Event Graph 后确认：`Event BeginPlay -> Should Spawn Enemies Immediately Branch -> Delay(Initial Spawn Delay) -> Spawn Enemy`；`Spawn Enemy` 自定义事件直接以 `Spawn Capsule` 的 World Transform 执行 `SpawnActor(NPC Class)`，随后绑定 NPC 的 `On Death`。截图范围内未见 Server/Client、Authority 或 Net Mode 判断。
  - 用户确认 `BP_ShooterNPCSpawner` Class Defaults：`Replicates=false`、`Replicate Movement=false`、`Net Load on Client=true`。这意味着关卡中的 Spawner 会在 Client 加载，但当前不是由 Server 复制的 Actor。
  - 用户确认 `BP_ShooterNPC` Class Defaults：`Replicates=true`、`Replicate Movement=true`、`Net Load on Client=true`；Arena 地图中放置了两个 `BP_ShooterNPCSpawner`。这与 Server 生成两个可复制活动 NPC、Client 另由两个本地 Spawner 生成两个静止 NPC 的运行现象完全吻合。
  - 用户关闭 `BP_ShooterNPCSpawner.Net Load on Client` 后重新执行双人 PIE，确认两端敌人数量与活动状态基本一致，Client 不再报告额外两个静止敌人。Spawner 客户端重复生成闭环已验证修复。
  - 用户将 `ABP_FP_Weapon` 按 `ABP_FP_Pistol` 相同方式增加 `Is Locally Controlled` 保护并确认修复；已检查的两个第一人称武器 Anim Blueprint 不再作为当前阻塞。
  - 用户回传 `BP_ShooterCharacter` 开火输入节点文本：`IA_Shoot.Triggered -> Validated Get Current Weapon -> BP_ShooterWeaponBase.Start Firing`；`IA_Shoot.Completed/Canceled -> Validated Get Current Weapon -> Stop Firing`。该段是普通函数调用，未见 Server RPC、Authority 或复制事件。
  - 用户回传 `BP_ShooterWeaponBase` 下游文本：`Start Firing` 是 Custom Event，设置 `Is Firing`、执行射速检查并直接调用/定时调用 `Fire`；`Fire` 检查 `Is Firing`，通过 Weapon Owner 的 `Get Weapon Target Location` 获取目标，调用 `Fire Bullet(Target Position)`，随后更新 `Time of Last Shot`、调用 `MakeNoise` 并安排后续 Timer。已回传范围仍未见 RPC 或 Authority；实际 Projectile 生成预计位于尚未审阅的 `Fire Bullet`。
  - 用户确认 `Fire` Custom Event 为 `Not Replicated`；`BP_ShooterProjectile_Bullet` Class Defaults 为 `Replicates=false`、`Replicate Movement=false`。
  - 用户回传 `Fire Bullet(Target Position)`：函数直接按 `Bullet Class` 执行 `SpawnActor`，Spawn Transform 由目标位置计算，Projectile Owner 使用 Weapon 的 Owner，Instigator 使用 `Pawn Owner`；同一函数还包含第一人称开火 Montage、后坐力、弹药减少/装填与 HUD 更新。因此当前 Gameplay Spawn 与本地表现/状态耦合在同一普通函数中。
  - 用户确认 `BP_ShooterWeaponBase` Class Defaults：`Replicates=false`、`Replicate Movement=false`、`Net Load on Client=true`、`Net Use Owner Relevancy=false`。`BP_ShooterCharacter.Add Weapon Class` 在本地直接 Spawn Weapon，Spawn Owner 与 Instigator 均为 Character Self，再加入 `Owned Weapons` 并激活；因此各网络实例持有独立的非复制 Weapon，不能直接依赖 Weapon Actor Server RPC。
  - 用户按 Character Server RPC、Authority-only Spawn 和 Bullet 复制方案完成初步修改后测试：Client 射击场景靶子时，靶子在 Server 与 Client 两端均产生摇动。该结果证明请求/碰撞/靶子响应链至少有一部分已成为双方可见状态，但尚未证明 NPC 的 Server 权威伤害、生命或死亡同步通过。
  - 用户回传 `BP_ShooterProjectileBase.Process Hit`：命中链包含物理对象的 `AddImpulseAtLocation`，解释了靶子摇动；可伤害 Character 分支调用 `Apply Damage`，Damaged Actor 为 Hit Actor，Base Damage 为 Projectile Damage，Event Instigator 为 Projectile `Get InstigatorController`，Damage Causer 为 Projectile Self，Damage Type 使用配置变量。该归属链可支持后续 Server 计分，但应优先保留 Instigator Controller/PlayerState，而非长期保存即将销毁的 Projectile 引用。
  - 用户回传 `BP_ShooterNPC.ReceiveAnyDamage`：当 `Current HP > 0` 时执行 `Current HP = Current HP - Damage`，随后在 `Current HP <= 0` 时调用 `Die`。该链未见 Authority 判断，且用户确认 `Current HP.Replication=None`；`Instigated By` 与 `Damage Causer` 当前未接入 NPC 的计分/死亡归属逻辑。
  - 用户应用 Authority-only ProcessHit 与 NPC HP 复制后测试：Client 仍无法击杀 AI，但现在能够击杀 Server 玩家。该结果验证 Character RPC、Server Bullet 和对玩家 Apply Damage 的基础链已工作，问题进一步收缩到 Server Bullet 对 NPC 的实际命中或 NPC `ReceiveAnyDamage`/HP 处理。
  - 用户进一步观察后判断，Client 对 NPC 的实际伤害/死亡逻辑可能已经生效，之前被误判为“无法击杀”的主要原因是 Client 不显示死亡布娃娃。当前明确表现缺陷：Client 看不到 NPC 与 Server 玩家死亡时的布娃娃；Client 自己受伤时没有受伤特效，自己死亡时不切换死亡摄像机。该项仍需用 NPC 停止行为/销毁/补充生成等非布娃娃证据完成伤害闭环验收。
  - 用户确认 `BP_ShooterNPC.Die` 是 `Not Replicated` Custom Event。其单一执行链混合了 Server 规则与表现：DoOnce、广播 `On Death`、调用 GameMode `IncrementTeamScore(Team Byte)`、添加 Dead Tag、停止 Movement、关闭 Capsule Collision、将 Mesh 设为 Ragdoll Collision、启用 Simulate Physics/Physics Blend、Delay 后 Destroy Actor。
  - 用户按拆分方案新增 NPC Reliable Multicast 死亡表现并测试，确认 NPC 布娃娃在 Client 正常显示。NPC 共享死亡表现闭环已验证修复。
  - 用户回传 `BP_ShooterCharacter.ReceiveAnyDamage`：Server 伤害链直接修改 `Current HP`，并在同一链上通过 Server 侧 `Player Controller` 调用 `Update Health UI`；HP 小于等于 0 后先停用当前武器，再调用 `Die`。这解释了远端拥有者 Client 收不到本地 UI/武器表现更新。
  - 用户回传 `BP_ShooterCharacter.Die`：单一事件混合了 GameMode 队伍计分、Dead Tag、停止/禁用移动、禁用输入、子弹 UI、Mesh 布娃娃、第一人称/死亡摄像机切换以及 Delay 5 秒后 Destroy Actor。该结构尚未通过多人死亡表现验收。
  - 用户新增 Reliable Multicast `Multicast Play Player Death` 执行玩家 Mesh Ragdoll/Simulate Physics，并新增 Reliable Owning Client `Client Enter Death View` 切换 First Person/Death Camera；双人 PIE 已确认 Client 端玩家布娃娃与死亡摄像机问题修复。
  - 用户确认 Client 玩家受到伤害时权威 `Current HP` 会正确减少，HP 清空后也会死亡；当前缺陷仅为 Client 本地生命 UI 不更新且不播放受击特效。Host 侧 `AnyDamage -> Update Health UI` 的直接本地调用不能跨网络驱动远端拥有者 UI。
  - 用户新增 Owning Client 受伤反馈 RPC，在 Client 本地调用 `Update Health UI`；用户确认该接口实现同时负责血条和受击特效，双人测试中 Client 血条与受击特效均已恢复。HP 与死亡仍由 Server 权威处理，本闭环已验证完成。
  - 用户完成多人战斗组合回归：Client/Host 分别击杀 NPC、AI 分别攻击双方、生命 UI、受击特效、玩家/NPC 死亡与销毁均未发现问题，也未报告新的 Runtime Error。Phase 2 多人战斗闭环验收通过。
  - 用户完成 AI 多玩家索敌测试并确认机制无问题；NPC 能处理两名玩家与目标失效场景。Phase 3 验收通过，未进行 AI 重构。
  - Phase 4 首次生成测试失败：开局数秒后只生成两个 NPC，击杀后没有任何补充。已确认问题不是 Client 额外副本，而是 Server Spawner 没有形成死亡补充循环。
  - 用户找到现有重生逻辑：`Spawn Count` 为每个 Spawner 实例的总生成额度，默认值 1。改为 10 后，两个出生点都能在各自 NPC 死亡后继续生成，直到各自累计 10 个 NPC 全部死亡，整场合计最多 20 个。有限持续生成循环已验证可用。
  - 用户回传 `BP_ShooterGameMode.EventGraph`：`Increment Team Score(TeamByte)` 在 `TeamScores : Map<Byte, Int>` 中查找同一 Key 的值、加 1、写回 Map，然后广播 `Score Updated(TeamByte)` Event Dispatcher。该链没有 Killer/Instigator 输入、没有 PlayerState/GameState、没有 RPC/复制或胜负判断。
  - 同一 GameMode 的 `BeginPlay` 直接 `Create Widget(UI_Shooter) -> Add to Viewport`，Owning Player 未连接。GameMode 仅存在于 Server，因此这不是普通 Client 可依赖的 HUD 创建路径；当前 Client 已工作的生命 UI 来自此前审阅的 PlayerController 本地链，不能据此推断 Score UI 已同步。
  - 用户运行确认当前模板语义：NPC 死亡固定使 `Team2Score +1`，任意 `BP_ShooterCharacter` 死亡固定使 `Team1Score +1`；它按死亡对象阵营计数而非按击杀玩家归属。Client 端不显示这套 Score UI。
  - 用户已将计分显示简化为每端只显示本地玩家自己的一个分数，并将规则调整为击杀 NPC +1、击杀对手玩家 +3。该方案取代此前计划的双分数栏与“默认关闭 PvP”；跨方向归属、AI 击杀不加分和玩家重生仍需专项测试后才能完成验收。
  - 用户完成个人计分/重生专项测试：Host 与 Client 击杀 NPC 均只给击杀者 +1；PvP 双方向只给击杀者 +3；AI 击杀不错误加分；双方玩家死亡后均能重生并继续比赛，个人分数跨 Pawn 重生保持，测试未发现问题。个人计分核心闭环已通过。
  - 用户回传当前加分入口：`BP_ShooterNPC.Die(Killer Controller)` 经 DoOnce/OnDeath 后校验 Killer Controller，取得其 PlayerState、Cast `BP_ShooterPlayerState`，调用统一 `Add Kill Score(Score=1)`；`BP_ShooterCharacter.Die(Killer Controller)` 使用同样链调用 `Add Kill Score(Score=3)`。因此 +1/+3 已集中到 PlayerState 的同一函数，目标分判断无需写入两个死亡事件。
  - 用户回传 `BP_ShooterPlayerState.Add KillScore`：Custom Event 先以 `Has Authority` Branch 限制 Server，True 时执行 `Kill Score = Kill Score + Score`；当前没有比赛结束检查或 GameMode/GameState 调用。该 Set 之后是唯一正确的目标分判断插入点。
  - 用户完成目标分比赛结束后端：新增 `BP_ShooterGameState`，包含 `Match Finished`（Bool RepNotify）、`Winner Player State`（`BP_ShooterPlayerState` 引用）和 `Target Score`（测试值 3）；GameMode `Finish Match(Winner)` 仅 Authority 执行，检查未结束后先写 Winner、再写 Match Finished，防止重复结束。
  - `BP_ShooterPlayerState.Add KillScore` 已增加 Match Finished 门禁，使用加分后的 New Score 与 Target Score 比较并调用 GameMode Finish Match(Self)；用户确认达标后双方分数冻结。
  - GameMode Finish Match 遍历所有 `BP_ShooterPlayerController`，按 Controller 的 PlayerState 是否等于 Winner 计算 Victory Bool，并调用 Reliable Owning Client RPC `Client Show Match Result(Victory)`；双端调试验证胜者收到 Victory、败者收到 Defeat。
  - 用户删除了父类错误的普通 Widget，重新创建 `UI_MatchResult`（父类 UserWidget）：全屏 Canvas、中央 ResultText；`Set Result(Victory)` 将文字设为 VICTORY 或 DEFEAT。PlayerController RPC 已连接 Create Widget、Set Result、Add to Viewport。正式 Widget 的双向视觉 PIE 尚未验证。
  - 用户完成 `UI_MatchResult` 双向视觉测试：Host 获胜与 Client 获胜时，两端分别正确显示 VICTORY/DEFEAT，Widget 未重复创建、无 Runtime Error；正式 Target Score 已从测试值 3 恢复为 10。
  - 用户在 Project Settings 中修正默认地图与 GameMode；随后只读检查 `Config/DefaultEngine.ini` 确认 Editor/Game 默认地图均为 Arena，Global Default GameMode 为 `BP_ShooterGameMode`。默认启动配置风险已解除。

### 当前诊断推断（等待 Blueprint 审阅确认）

- 双身体现象的直接配置原因已确认并修复：第一人称与第三人称 Mesh 原本均未设置 Owner 可见性；标准可见性组合已通过双端 PIE 验证。
- 第一人称武器 Anim Blueprint 空引用已在 `ABP_FP_Pistol` 和 `ABP_FP_Weapon` 使用本地控制保护处理；已完成项后续仍需纳入回归。
- Client 重复静止敌人的根因已确认并修复：两个非复制但会在 Client 加载的 Spawner 分别执行本地 Spawn；关闭 Spawner 的 `Net Load on Client` 后双端状态基本一致。
- Client 无权威伤害的直接机制和 RPC 边界已确认：Weapon 是各端独立非复制实例，RPC 应放在由 Client 拥有的复制 Character 上；Client 保留本地 Montage/后坐力/HUD，但本地禁止 Spawn，Character RPC 让 Server 端对应 Weapon Spawn 可复制 Bullet。Client 传入的 Target Position 暂不做反作弊校验，限于当前两人 Listen Server Demo。
- 修改后 Client 射击靶子的双端响应已部分通过，说明 RPC 或复制 Bullet 路径可能已工作；AI 仍异常时应优先审阅 Projectile 命中/Apply Damage 与 NPC Health/AnyDamage，而不是立即推翻整个开火 RPC 方案。
- Projectile 的 Apply Damage 参数链已确认具备击杀归属基础；当前待确认点转为 `BP_ShooterNPC` 是否只在 Server 修改 HP、HP 是否复制，以及死亡/销毁是否由 Server 触发。
- NPC HP 同步缺口已确认：`Current HP` 未复制；同时复制 Projectile 的 `Process Hit` 没有 Authority 保护，可能在各端重复执行本地碰撞响应。计划先以 Authority-only ProcessHit + `Current HP` RepNotify 建立一致生命/死亡闭环，再接计分。
- Character RPC 与 Server Bullet 已有强证据工作；当前问题更符合“Server 执行伤害/死亡，但表现函数只在 Server 本地运行”。预计需要把共享死亡表现拆为 Multicast/复制死亡状态，把拥有者专属受伤反馈与死亡摄像机拆为 Run on Owning Client，但必须先审阅现有 `Die`/受伤 Graph。
- NPC `Die` 拆分边界已确认：OnDeath、GameMode 计分、Dead Tag、Movement 停止、Delay/Destroy 必须只在 Server；Capsule 关闭与 Mesh Ragdoll/Simulate Physics 可由 Server 调用 Reliable Multicast 在所有端执行。不能把整个 `Die` 直接改成 Multicast。
- 玩家 `Die` 的拆分边界已由截图确认：GameMode 计分、Dead Tag、Movement 与 Delay/Destroy 属于 Server 规则；Mesh 布娃娃属于全端共享表现；本地武器停用、输入/UI 与摄像机切换属于 Owning Client 表现。不能把整个 `Die` 改成 Multicast，也不能指望 Server 侧 PlayerController 调用更新远端 Client 的本地 UI/摄像机。

## 8. 当前提交成果与下一步

### 已完成成果

- GitHub 仓库：`https://github.com/DIO-KONG/TencentCampDemo`，本地与远程 `main` SHA 已核对一致。
- 正式 PDF：`output/pdf/黄文灏-香港中文大学（深圳）-开局一课客户端大作业.pdf`。
- 可打印 HTML：`report/黄文灏-香港中文大学（深圳）-Demo技术报告.html`。
- Demo 视频：`report/PVP.mp4` 与 `report/Vectory.mp4`。
- PDF 已验证为 7 页 A4，逐页渲染无裁切/重叠；姓名、学校、GitHub 文本和链接注释均可解析。

### 用户下一步

1. 在作业平台上传正式 PDF 与两个 MP4（共 3 个文件，低于 9 个文件上限）。
2. 上传前各播放一次视频，并在 PDF 中点击 GitHub 链接确认浏览器可访问。
3. 不再进行非必要 Blueprint 修改；如必须修改项目，需重新保存、测试、提交并推送。

## 9. 后续 Phase 1 人工测试清单（首测通过后逐项执行）

- Host 开火：两端是否看见弹丸/动画/特效，敌人是否受伤并同步死亡。
- Client 开火：Server 是否接收，敌人是否发生权威伤害，两端结果是否一致。
- 敌人分别攻击 Host/Client：生命值、受伤、死亡是否同步。
- 多玩家目标：敌人是否只盯 Host/Player 0，目标失效后能否切换。
- Spawner：由哪一端生成，是否重复生成，数量是否在两端一致。
- 远端表现：角色、武器、射击动画、枪口和命中特效是否最低可接受。

每次只执行并记录一个可验证闭环；失败项不与其他功能同时修改。
