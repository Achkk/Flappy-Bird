# Flappy Bird 2D

Đây là dự án game Flappy Bird 2D làm bằng Unity. Người chơi điều khiển chim bay qua các khe giữa những cặp ống. Chạm vào ống hoặc mặt đất thì lượt chơi kết thúc; bay qua khe ống sẽ được cộng điểm.

## Chạy game trong Unity

1. Mở dự án trong Unity Hub.
2. Mở scene `Assets/Scenes/Flappy Bird.unity`.
3. Nhấn **Play** trong Unity Editor.
4. Nhấn **Space** hoặc nhấp chuột để làm chim bay lên.

Project hiện dùng Unity `6000.6.6f1`, theo thông tin trong `ProjectSettings/ProjectVersion.txt`.

## Nội dung thư mục

- `Assets/Scenes`: scene của game.
- `Assets/Scripts`: mã điều khiển người chơi, ống, điểm số và nền.
- `Assets/Prefabs`: mẫu các cặp ống được sinh trong lúc chơi.
- `Assets/Sprites`, `Assets/Materials`, `Assets/Fonts`: hình ảnh, vật liệu và font.
- `Packages`: các package Unity mà project sử dụng.
- `ProjectSettings`: thiết lập project, tag, input và scene build.

Xem [CAU_TRUC_GAME.md](CAU_TRUC_GAME.md) để biết chi tiết từng thành phần và cách chúng phối hợp.

Xem [PHAN_TICH_THIET_KE_FLAPPY_BIRD.md](PHAN_TICH_THIET_KE_FLAPPY_BIRD.md) để tham khảo phần phân tích và thiết kế game theo chủ đề Flappy Bird.
