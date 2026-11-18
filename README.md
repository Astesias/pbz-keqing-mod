# Keqing Mod for Phantom Blade Zero / 刻晴 Mod · 影之刃零

把《影之刃零》主角「魂」的外观替换为《原神》的刻晴，战斗语音同步换成刻晴语音。

Replace the protagonist **Hun** in *Phantom Blade Zero* with **Keqing** from *Genshin Impact*, with Keqing's combat voice lines.

## 下载 / Download

**[⬇ Keqing_Mod_2026.11.19.zip](https://github.com/Astesias/pbz-keqing-mod/releases/download/v2026.11.19/Keqing_Mod_2026.11.19.zip)** （8.75 MB）

历史版本见 [Releases](https://github.com/Astesias/pbz-keqing-mod/releases)。

## 截图 / Screenshots

![screenshot](docs/screenshot_1.png)
![screenshot](docs/screenshot_2.png)
## 特性 / Features

| 项目 | 说明 |
|---|---|
| 模型 | 刻晴全身模型，含描边、布料物理、表情与眨眼 |
| 外观 | 刻晴配色与描边风格，随游戏场景光照变化 |
| 语音 | 39 个战斗事件（普攻、格挡、受击、破防、处决、呼吸、开大）为刻晴语音 |
| 保留原声 | 剧情对白、死亡、待机等保持游戏原版 |

- Model: full Keqing mesh with outline, cloth physics, facial animation & blinking
- Look: Keqing palette and outline style, lit by the scene's existing lighting
- Voice: 39 combat events use Keqing lines (attack, parry, hurt, break, execute, breath, ult)
- Untouched: story dialogue, death, idle — still the original voice

## 安装 / Install

1. 关闭游戏 / Close the game.
2. 找到游戏 Pak 目录 / Locate the game's pak folder:
   ```
   <steam>\steamapps\common\PBZ\ProjectPBZ\Content\Paks\
   ```
3. 把压缩包里的 `~mods` 文件夹整个复制进去 / Copy the whole `~mods` folder into it:
   ```
   ...\Paks\~mods\zzz_Keqing_P.{pak,utoc,ucas}
   ...\Paks\~mods\zzz_KeqingVoice_P.{pak,utoc,ucas}
   ```
4. 启动游戏 / Launch the game.

## 卸载 / Uninstall

删除 `~mods` 里这 6 个文件即可完全还原。
Delete those 6 files — the game returns to vanilla.

## 已知限制 / Known limits

1. 语音只覆盖 39 个战斗事件，剧情/死亡/待机为原版；
2. 同一句语音可能在多个动作中重复出现；
3. 极短语音位置保留原声；
4. 外观随场景光照变化，暗处会偏暗；
5. 脸部为平涂风格，强侧光下立体感弱；
6. 与其他替换主角外观 / 战斗语音的 mod 冲突，需二选一（把对方文件改扩展名为 `.off` 即可停用）。

## 兼容性 / Compatibility

- 游戏版本：UE 5.7.4（IoStore），2026-11 测试通过
- 与其他 `~mods` 子目录 mod 可共存（GoldenTiger、PBZAdditiveWardrobe 已验证）
- 与其他战斗语音替换 mod 冲突（需二选一）

## 文件校验 / Checksums

见发布包内 `checksums.txt`（SHA256）。

## 许可 / License

刻晴角色与语音版权归 miHoYo / HoYoverse 所有；本项目为非商业粉丝作品，仅供个人使用。
影之刃零版权归 S-GAME 所有。

Keqing character and voice © miHoYo / HoYoverse. This is a non-commercial fan project for personal use.
Phantom Blade Zero © S-GAME.
