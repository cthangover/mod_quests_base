# Quests Engine — 퀘스트, 단계, 그리고 저널

Quests Engine은 플레이어의 과제를 관리합니다. 퀘스트는 이름과 번호가 붙은 단계 목록을 가지며, 시나리오가 퀘스트를 시작하고 다음 단계로 진행하고 태그를 달고 완료합니다. **저널** 패널은 플레이어가 수락한 퀘스트를 보여 주고, 게임은 그 상태를 저장합니다.

Cthangover(프롤로그의 각성과 낡은 집)와 RPG Plus(숲의 퀘스트)가 이를 사용합니다.

## 저널

툴바의 저널 아이콘으로 엽니다. 플레이어가 아직 퀘스트를 하나도 수락하지 않았다면 비어 있습니다.

![RPG Plus 시작 시점의 저널: 진행 중과 완료 탭, 두 페이지 모두 "퀘스트가 없습니다".](assets/journal-before.jpg)

RPG Plus에서는 마을 원로가 첫 퀘스트를 줍니다. 어디를 찾아야 하는지 물어 보세요.

![원로의 집: 원로 앞에 선 주인공, 오른쪽의 두 선택지 "늑대가 등불을 끌 수 있나요?"와 "어디서 찾죠?".](assets/elder-choice.jpg)

답을 고르면 퀘스트가 저널에 나타납니다.

![대화 후의 저널: 진행 중 목록의 "장부에 오른 등불"; 오른쪽은 "항목 II / V", 첫 단계는 취소선, 현재 단계는 "동쪽 숲 동굴의 늑대를 물리치고, 호박 심장을 찾으세요".](assets/journal-after.jpg)

- **진행 중**에는 진행 중인 퀘스트가, **완료**에는 끝난 퀘스트가 들어갑니다. 아직 시작하지 않은 퀘스트는 결코 나타나지 않습니다.
- 오른쪽 페이지는 선택한 퀘스트를 보여 줍니다. 끝낸 단계에는 취소선이, 현재 단계에는 표시가 붙고, 이후 단계는 감춰져 있어 저널이 이야기를 미리 까발리지 않습니다.
- 퀘스트가 얻은 태그는 항목 아래에 작은 금색 꼬리표로 표시됩니다.

## 퀘스트 정의

퀘스트는 mod의 `quests/` 폴더에 있는 kind `quest` JSON 파일입니다. 사진 속 퀘스트(RPG Plus 것)입니다.

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

`Name`과 각 단계는 번역 키입니다. 숫자가 퀘스트의 **단계**입니다. 단계 1에서는 `1`의 텍스트가 현재 단계로 표시되고 `0`에는 취소선이 그려집니다. 사진으로 말하면 "항목 II / V"입니다.

## 시나리오에서 퀘스트 진행하기

RPG Plus에서 원로의 대답이 퀘스트를 시작하고 단계 1에 놓습니다.

```scenario
action quest.set_status quest_id=woodland_ward status=Progress
action quest.set_data_status quest_id=woodland_ward level=1
```

- `quest.set_status` — 퀘스트의 상태: `NotStarted`, `Progress`(진행 중) 또는 `End`(완료).
- `quest.set_data_status` — 현재 단계, `level`이 그 번호입니다.
- `quest.add_tag` / `quest.remove_tag`와 `tag=…` — 일어난 일을 기억합니다. 예를 들어 플레이어가 어떤 아이템을 먼저 가져갔는지.
- `quest.send_notification` — 퀘스트에 대한 알림을 플레이어에게 보여 줍니다. 퀘스트를 바꾸는 것만으로는 알림이 뜨지 않습니다.

## 퀘스트 확인하기

퀘스트 판정은 게임이 조건을 받아들이는 곳이라면 어디서든 작동합니다. `when`, 선택지 조건, 장면 조건, 맵 이벤트입니다.

```scenario
when "woodland_ward.DataStatus >= 2" -> :lantern
```

- `<quest>.IsActive` — 퀘스트가 진행 중.
- `<quest>.Status` — `0` 시작 안 함, `1` 진행 중, `2` 완료.
- `<quest>.DataStatus` — 현재 단계.
- `<quest>.hasTag("tag")`, `<quest>.notHasTag("tag")` — Cthangover는 이것으로 장면의 어떤 시나리오를 재생할지 고릅니다. 예: `AwakeningQuest.hasTag("book_taken")`.

## 문제가 생겼을 때

1. **퀘스트가 저널에 없습니다.** 상태가 `Progress`(완료 탭이라면 `End`)여야 합니다. 단계만 설정해서는 부족합니다.
2. **저널이 텍스트 대신 키를 보여 줍니다.** 단계의 키를 mod의 `locale` 파일에 추가하세요.
3. **조건이 항상 거짓입니다.** 퀘스트 id의 철자가 퀘스트 파일의 `Id`와 다릅니다. 로그에 알 수 없는 퀘스트 경고가 기록됩니다.
