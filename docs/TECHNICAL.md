# 技术说明 / Technical notes

记录这套 Mod 的制作流程与关键结论，供想改的人参考。

## 1. 模型侧

### 1.1 管线

```
E:\usd\output\刻晴.blend（MMD/PMX 来源）
  └─ keqing_v2.py（一次性 Blender 脚本）
       ├─ 去除 MMD 扭骨（腕捩.L/R、手捩.L/R），权重并入 Left_Lower_Arm / Left_Hand
       ├─ 腿部骨骼按魂的骨架缩放、膝盖/脚踝对位
       ├─ 描边：法线外扩 OUTLINE_THICKNESS = 0.0025（≈0.25 cm）
       ├─ 脸部：平面化法线（FACE_DOME_NORMALS，球心偏移 0.55×脸宽）
       └─ 导出 Avatar.fbx
  └─ tools/build-mod.ps1（导入 FBX → 建 AnimBP → 材质槽替换 → cook → retoc pack-raw）
```

### 1.2 关键结论

1. **材质必须借游戏原生材质实例**。自建材质打进 pak 后，发行版里着色器没被 cook，模型显示为灰模。所以只能把材质槽指向 `MI_ZeroHun_*`，靠覆盖 4 张贴图（D / N / ARM / SCMT）+ 描边贴图改外观。
2. **贴图里烘进三渲二明暗会造出改不掉的色带**。几何法线在腿/肘处突变时，烘好的明暗会沿 UV 硬切，表现为色带；调粗糙度/AO/重画 UV 都无效。解决办法是皮肤用**未烘明暗的 albedo**。
3. **手臂链不能夹未驱动的 MMD 扭骨**。`上臂 → 腕捩 → 小臂` 会让小臂看起来弯、手腕像骨折；删掉扭骨并转移权重后正常。
4. **物理抖动**：C++ 侧 `ConfigureBody` 设 `MassScale=2.0`、`SolverIterations/Joint/Collision = 8/8/4`，同时关闭裙子的模拟与侧发×头碰撞，抽搐消失。
5. **手臂长度**：`soul.arm_scale = {upperarm: 0.85, lowerarm: 0.85}`（必须写在 `soul` 段，写到 `model.adapt_humanoid` 会被忽略）。

## 2. 语音侧

### 2.1 容器

UE 5.7.4 用 IoStore。容器由三部分组成：

- `PakFile`：几乎为空（347 字节）
- `UtocFile`：TOC，手写的 v5 ContainerHeader + 压缩块索引
- `UcasFile`：数据

`FPackageId = CityHash64(lower(UTF16(path)))`；chunk id = 包 id 的 LE 字节 + 8 位 chunk 类型（hex）。TOC 偏移 56 处存容器 id 字节（原样）。

打包用 `retoc pack-raw`，Header 用自写脚本 `make_container_header.py` 生成。

### 2.2 Wwise bank

bank 结构：`BKHD / HIRC / RIFF`，媒体**内联**（没有 DIDX），本作事件 bank 的版本号是 154。

替换媒体时**总长度与单槽长度都不能变**：改长度会导致大面积无声。

| 槽位类型 | 布局 |
|---|---|
| Vorbis | `fmt(66) + hash(16) + JUNK + data`（或参考 mod 的 `fmt + JUNK + data`，不带 hash） |
| PCM | `fmt(24, tag 0xFFFE) + JUNK(4) + data` |

### 2.3 编码器

**必须用 Wwise 官方**（2023.1.19.8928）。`wav2wem` 编出来的 Vorbis 游戏解码成电音。

离线转换流程：

```powershell
WwiseConsole create-new-project <dir>\<name>.wproj     # 目录必须不存在
# 写 .wsources：<ExternalSourcesList><Source Path="x.wav" Conversion="Vorbis Quality High"/></ExternalSourcesList>
WwiseConsole convert-external-source <wproj> --source-file <wsources> --platform Windows --output Windows <outdir>
```

输出 wem = `fmt + hash + data`；`Vorbis Quality High/Medium/Low` 三档。

### 2.4 时长技巧

游戏**读 fmt 里的采样率**。所以把 PCM 槽位的采样率降到 12 kHz / 10 kHz / 8 kHz，就能在同样字节数里塞进更长音频（48k 约 0.18 s，8k 约 1.1 s）。

### 2.5 响度

先用 `mean_volume` 补到 −18 dB 的做法在短促音和长句之间听感差很大。最终改成 **EBU R128 两遍法**：

1. `loudnorm` 测量 `input_i / input_tp / input_lra / input_thresh`
2. 按测量值线性补偿到 **−16 LUFS / −1.5 dBTP / LRA 11**

### 2.6 事件语义（实测 + 文档核对）

| 事件 | 语义 | 备注 |
|---|---|---|
| `Attack_Short` / `ChuZhao_*` / `ShouZhao_*` | 普攻出手 / 出招 / 收招 | 普攻的短促出击音 |
| `Charge_*` | 蓄力过程与释放 | 会被普攻触发 |
| `BiQi_01_DuanCu` | 聚气（短） | **普攻会触发它**，早期误判成「兵技」导致普攻串大招 |
| `BiQi_02_Long` | 聚气（长）= 开大 | |
| `JinGu_*` | **筋骨 = 格挡 / 弹反** | 不是「禁锢」 |
| `Hurt*` | 受击 / 重受击 | |
| `Break` / `Weapon_Break` | 破防 | |
| `Execute_Attack_Heavy` | 处决 | |
| `Burp*` / `Snort` / `Cough` / `XiQi` | 呕血 / 哼声 / 咳嗽 / 吸气 | |
| `Stand` / `ZhongZhuan*` | 起身 / 中转 | |

**教训**：事件名按字面猜会错（`JinGu` 不是禁锢、`BiQi` 不是兵技）。确定方法是用「报数探针」——往每个事件塞不同数字的语音，进游戏做动作听编号。

### 2.7 台词库与 ASR

刻晴台词 354 条，文件名带分类前缀（`battle_hit_H_02` 等）。用 faster-whisper（small 模型）跑一遍 ASR 拿到真实文本，才能精确排除某句（例如 `battle_hit_L_02` = 「小伤而已」，出现在受击槽位会让连击收尾听起来像挨打）。

## 3. 调试经验

1. **改完填充脚本一定要检查打包脚本读的是哪个目录**——中间有一次填充输出改了目录、打包仍读旧目录，白测三版；
2. **静音比例别用哈希**：槽位少的 bank（7~16 个）用哈希会让某个事件 86% 静音。改成「按槽位序号取模 + 第 0 槽必有声」；
3. **`~mods` 的子目录也会被挂载**：把要停用的 mod 移到子文件夹没用，必须改扩展名（`.pak.off`）；
4. **装载优先级看文件名**：`zzzz_` 开头的包会压过 `zzz_` 开头的。
