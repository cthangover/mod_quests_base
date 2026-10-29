# Quests Engine — 任务、任务步骤与日志

Quests Engine 替玩家记账。每个任务都有名称和编号的步骤列表；剧本负责开启任务、推进到下一步、打上标记并完成任务。**日志**面板显示玩家已领取的任务，游戏会保存它们的状态。

Cthangover（序章的苏醒和旧宅）与 RPG Plus（森林任务）都用到了它。

## 日志

用工具栏上的日志图标打开。玩家还没领取任何任务时，日志是空的：

![RPG Plus 开局时的日志：「进行中」和「已完成」两个页签，两页都写着「暂无任务」。](assets/journal-before.jpg)

在 RPG Plus 里，村庄的长老给出第一个任务。向他问问去哪里找：

![长老的家：主角站在长老面前，右侧有两个选项——「狼会熄灯吗？」和「去哪儿找它？」。](assets/elder-choice.jpg)

回答之后，任务就出现在日志里：

![对话后的日志：「账簿上的一盏灯」出现在进行中列表；右侧是「第 II 条，共 V 条」，第一步已划掉，当前步骤为「击退东部森林洞窟旁的狼。找到琥珀之心」。](assets/journal-after.jpg)

- **进行中**列出正在做的任务，**已完成**列出做完的。还没开始的任务永远不会出现。
- 右页显示选中的任务：已完成的步骤划掉，当前步骤做了标记，后续步骤保持隐藏——日志不会剧透后面的故事。
- 任务获得的标记会以小小的金色标签显示在条目下方。

## 任务的定义

任务是 mod 的 `quests/` 文件夹里类型为 `quest` 的 JSON 文件。这就是截图中的任务（来自 RPG Plus）：

```json
{
  "Kind": "quest",
  "Items": [
    {
      "Id": "woodland_ward",
      "Name": "rpg_plus/quest/ward/name",
      "StatusToDescription": {
        "0": "rpg_plus/quest/ward/0",
        "1": "rpg_plus/quest/ward/1",
        "2": "rpg_plus/quest/ward/2",
        "3": "rpg_plus/quest/ward/3",
        "4": "rpg_plus/quest/ward/4"
      }
    }
  ]
}
```

`Name` 和每个步骤都是翻译键。数字就是任务的**步骤**：在第 1 步时，`1` 的文本显示为当前步骤，`0` 则划掉——即截图中的「第 II 条，共 V 条」。

## 从剧本推进任务

RPG Plus 中长老的回答会开启任务并把它放到第 1 步：

```scenario
action quest.set_status quest_id=woodland_ward status=Progress
action quest.set_data_status quest_id=woodland_ward level=1
```

- `quest.set_status` — 任务的状态：`NotStarted`、`Progress`（进行中）或 `End`（已完成）。
- `quest.set_data_status` — 当前步骤，`level` 是它的编号。
- `quest.add_tag` / `quest.remove_tag` 配合 `tag=…` — 记下发生过的事，比如玩家先拿了哪个物品。
- `quest.send_notification` — 向玩家弹出任务通知；改变任务本身不会触发通知。

## 检查任务

凡是游戏接受条件的地方都能用任务检查：`when`、选项条件、场景条件和地图事件。

```scenario
when "woodland_ward.DataStatus >= 2" -> :lantern
```

- `<quest>.IsActive` — 任务正在进行；
- `<quest>.Status` — `0` 未开始，`1` 进行中，`2` 已完成；
- `<quest>.DataStatus` — 当前步骤；
- `<quest>.hasTag("tag")`、`<quest>.notHasTag("tag")` — Cthangover 用它们决定场景播放哪段剧本，例如 `AwakeningQuest.hasTag("book_taken")`。

## 出问题时

1. **任务不在日志里。** 它的状态必须是 `Progress`（或在「已完成」页签显示则用 `End`）；只设置步骤是不够的。
2. **日志显示的是键而不是文本。** 把步骤的键加进 mod 的 `locale` 文件。
3. **条件永远为假。** 写的任务 id 与任务文件里的 `Id` 不一致；日志会警告存在未知任务。
