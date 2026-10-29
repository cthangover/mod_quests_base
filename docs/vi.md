# Quests Engine — nhiệm vụ, các bước và nhật ký

Quests Engine quản lý các việc người chơi phải làm. Một nhiệm vụ có tên và danh sách các bước được đánh số; kịch bản khởi đầu nhiệm vụ, đưa nó sang bước tiếp, gắn thẻ và kết thúc nó. Bảng **Nhật ký** hiển thị những nhiệm vụ người chơi đã nhận, và game lưu trạng thái của chúng.

Cthangover (tỉnh dậy trong prologue và ngôi nhà cũ) và RPG Plus (các nhiệm vụ trong rừng) đều dùng mod này.

## Nhật ký

Mở bằng biểu tượng nhật ký trên thanh công cụ. Khi người chơi chưa nhận nhiệm vụ nào, nhật ký còn trống:

![Nhật ký của RPG Plus lúc đầu: hai thẻ Đang làm và Đã xong, cả hai trang đều ghi "Chưa có nhiệm vụ".](assets/journal-before.jpg)

Trong RPG Plus, trưởng làng giao nhiệm vụ đầu tiên. Hỏi ông xem nên đi tìm ở đâu:

![Nhà trưởng làng: anh hùng đứng trước trưởng làng, hai lựa chọn bên phải — "Sói tắt được đèn sao?" và "Tìm nó ở đâu?".](assets/elder-choice.jpg)

Sau câu trả lời, nhiệm vụ xuất hiện trong nhật ký:

![Nhật ký sau cuộc trò chuyện: "Ngọn đèn trên sổ sách" nằm trong danh sách đang làm; bên phải là "Mục II trên V", bước đầu đã bị gạch, bước hiện tại là "Đánh bại con sói bên hang trong rừng phía đông. Tìm trái tim hổ phách".](assets/journal-after.jpg)

- **Đang làm** liệt kê nhiệm vụ đang tiến hành, **Đã xong** liệt kê nhiệm vụ hoàn thành. Nhiệm vụ chưa bắt đầu không bao giờ hiện ra.
- Trang bên phải hiển thị nhiệm vụ đang chọn: các bước đã làm bị gạch ngang, bước hiện tại được đánh dấu, các bước sau còn ẩn — nhật ký không tiết lộ trước câu chuyện.
- Các thẻ nhiệm vụ đã giành được hiện dưới mục nhỏ, dạng nhãn vàng nhỏ.

## Định nghĩa nhiệm vụ

Nhiệm vụ là tệp JSON kind `quest` trong thư mục `quests/` của mod. Đây là nhiệm vụ trong các hình trên (thuộc RPG Plus):

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

`Name` và từng bước đều là khóa dịch. Các con số chính là **bước** của nhiệm vụ: ở bước 1, văn bản của `1` hiện là bước hiện tại và `0` bị gạch ngang — tức "Mục II trên V" trong hình.

## Đưa nhiệm vụ đi từ kịch bản

Câu trả lời của trưởng làng trong RPG Plus khởi đầu nhiệm vụ và đặt nó ở bước 1:

```scenario
action quest.set_status quest_id=woodland_ward status=Progress
action quest.set_data_status quest_id=woodland_ward level=1
```

- `quest.set_status` — trạng thái nhiệm vụ: `NotStarted`, `Progress` (đang làm) hoặc `End` (hoàn thành).
- `quest.set_data_status` — bước hiện tại, `level` là số của nó.
- `quest.add_tag` / `quest.remove_tag` kèm `tag=…` — ghi nhớ chuyện đã xảy ra, chẳng hạn người chơi lấy vật phẩm nào trước.
- `quest.send_notification` — hiện thông báo về nhiệm vụ cho người chơi; thay đổi nhiệm vụ thì tự nó không thông báo.

## Kiểm tra nhiệm vụ

Phép kiểm nhiệm vụ dùng được ở mọi nơi game nhận điều kiện: `when`, điều kiện lựa chọn, điều kiện cảnh và sự kiện bản đồ.

```scenario
when "woodland_ward.DataStatus >= 2" -> :lantern
```

- `<quest>.IsActive` — nhiệm vụ đang tiến hành;
- `<quest>.Status` — `0` chưa bắt đầu, `1` đang làm, `2` đã hoàn thành;
- `<quest>.DataStatus` — bước hiện tại;
- `<quest>.hasTag("tag")`, `<quest>.notHasTag("tag")` — Cthangover dùng chúng để chọn kịch bản nào của cảnh sẽ chạy, ví dụ `AwakeningQuest.hasTag("book_taken")`.

## Khi có trục trặc

1. **Nhiệm vụ không có trong nhật ký.** Trạng thái của nó phải là `Progress` (hoặc `End` để vào thẻ đã xong); chỉ đặt bước là chưa đủ.
2. **Nhật ký hiện khóa thay vì văn bản.** Thêm khóa của bước vào các tệp `locale` của mod.
3. **Điều kiện luôn sai.** Id nhiệm vụ viết khác với `Id` trong tệp nhiệm vụ; log sẽ cảnh báo về nhiệm vụ lạ.
