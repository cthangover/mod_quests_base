# Quests Engine — quests, their steps and the journal

Quests Engine keeps track of the player's tasks. A quest has a name and a numbered list of steps; scenarios start it, move it to the next step, mark things with tags and finish it. The **Journal** panel shows the quests the player has taken, and the game saves their state.

It is used by Cthangover (the prologue's awakening and the old house) and by RPG Plus (the woodland quests).

## The journal

Open it with the journal icon on the toolbar. Before the player has taken any quest, it is empty:

![The Journal in RPG Plus at the start: Active and Completed tabs, "No quests" on both pages.](assets/journal-before.jpg)

In RPG Plus the elder of the village gives the first quest. Ask him where to look:

![The elder's house: the hero in front of the elder, two answers on the right — "Can a wolf put out lamps?" and "Where do I find it?".](assets/elder-choice.jpg)

After the answer the quest is in the journal:

![The Journal after the talk: "A light on the ledger" in the Active list; on the right "Entry II of V", the first step struck through and the current step "Defeat the wolf by the cave in the eastern forest. Find the amber heart."](assets/journal-after.jpg)

- **Active** lists quests in progress, **Completed** the finished ones. Quests not yet started never appear.
- The right page shows the selected quest: steps already done are struck through, the current step is marked, and later steps stay hidden, so the journal never gives the story away.
- Tags the quest has earned are shown as small gold labels under the entry.

## A quest definition

Quests are JSON files of kind `quest` in a mod's `quests/` folder. This is the quest from the pictures (from RPG Plus):

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

`Name` and every step are translation keys. The numbers are the quest's **step**: step 1 shows the text of `1` as current and `0` struck through — "Entry II of V" in the picture.

## Moving a quest from a scenario

The elder's answer in RPG Plus starts the quest and puts it on step 1:

```scenario
action quest.set_status quest_id=woodland_ward status=Progress
action quest.set_data_status quest_id=woodland_ward level=1
```

- `quest.set_status` — the quest's state: `NotStarted`, `Progress` (active) or `End` (completed).
- `quest.set_data_status` — the current step, `level` is its number.
- `quest.add_tag` / `quest.remove_tag` with `tag=…` — remember something that happened, for example which item the player took first.
- `quest.send_notification` — shows the player a notification about the quest; changing a quest does not do this by itself.

## Checking a quest

Quest checks work wherever the game takes a condition: `when`, answer conditions, scene conditions and map events.

```scenario
when "woodland_ward.DataStatus >= 2" -> :lantern
```

- `<quest>.IsActive` — the quest is in progress;
- `<quest>.Status` — `0` not started, `1` in progress, `2` completed;
- `<quest>.DataStatus` — the current step;
- `<quest>.hasTag("tag")`, `<quest>.notHasTag("tag")` — Cthangover uses these to pick which scenario of a scene plays, for example `AwakeningQuest.hasTag("book_taken")`.

## If something goes wrong

1. **The quest is not in the journal.** Its state must be `Progress` (or `End` for the Completed tab); setting only the step is not enough.
2. **The journal shows a key instead of text.** Add the step's key to the mod's `locale` files.
3. **A condition is always false.** The quest id is spelled differently from the `Id` in the quest file; the log warns about an unknown quest.
