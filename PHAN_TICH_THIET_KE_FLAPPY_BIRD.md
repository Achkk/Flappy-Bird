# Phân tích và thiết kế chương trình game: Flappy Bird

## 1. Giới thiệu game

Flappy Bird là game 2D dạng cuộn ngang. Người chơi điều khiển một chú chim bay qua các khe hở giữa những cặp ống màu xanh. Trò chơi tập trung vào phản xạ và căn thời điểm: chim luôn bị trọng lực kéo xuống, còn người chơi phải liên tục tạo lực bay lên để tránh chướng ngại vật.

Mục tiêu là bay qua càng nhiều cặp ống càng tốt để đạt điểm cao. Game kết thúc khi chim chạm vào ống hoặc mặt đất.

## 2. Mục tiêu và đối tượng trong game

- **Người chơi:** điều khiển chim bằng phím Space hoặc nhấp chuột.
- **Nhân vật chính:** chim có hoạt ảnh vỗ cánh gồm ba hình sprite.
- **Chướng ngại vật:** các cặp ống trên và dưới, được tạo liên tục ở độ cao ngẫu nhiên.
- **Vùng tính điểm:** vùng vô hình nằm giữa mỗi cặp ống; chim đi qua vùng này sẽ được cộng điểm.
- **Môi trường:** nền cuộn tạo cảm giác chuyển động và mặt đất là ranh giới thua.

## 3. Các quy luật chơi chính

### Di chuyển

Chim chịu tác động của trọng lực và tự rơi xuống. Khi người chơi nhấn Space hoặc nhấp chuột, chim được đẩy lên. Nếu nhấn quá ít, chim rơi xuống; nếu nhấn quá nhiều, chim bay lên phía trên và có thể va vào ống.

### Chướng ngại vật

Các cặp ống xuất hiện từ phía bên phải và di chuyển sang trái. Mỗi cặp có độ cao khác nhau, nhưng giữ một khe hở để chim có thể bay qua. Các ống nằm ngoài màn hình sẽ bị xóa; Spawner tiếp tục tạo các cặp mới nên màn chơi kéo dài liên tục.

### Tính điểm

Khi chim bay qua vùng tính điểm ở giữa một cặp ống, điểm tăng thêm một. Mỗi vùng được kích hoạt khi chim đi vào vùng đó.

### Điều kiện thua

Chim thua khi chạm vào ống hoặc mặt đất. Khi đó game dừng, nút chơi lại và hình `Game Over` xuất hiện. Va chạm được nhận biết bằng các collider dạng trigger và tag `Obstacle`.

### Bắt đầu và chơi lại

Ở màn hình chờ, hình `Get Ready` và nút mũi tên được hiển thị. Nhấn nút để bắt đầu. Khi thua, nhấn nút mũi tên lần nữa để chơi lại; điểm được đặt về 0 và các ống cũ được xóa.

## 4. Thiết kế các đối tượng và thành phần

| Thành phần | Vai trò |
|---|---|
| `Player` | Nhận thao tác nhấn phím/chuột, cập nhật vị trí và độ nghiêng của chim, chạy hoạt ảnh, phát hiện va chạm. |
| `Spawner` | Tạo prefab ống theo chu kỳ và chọn độ cao ngẫu nhiên cho từng cặp. |
| `Pipes` | Di chuyển cặp ống sang trái, tạo khe hở và xóa cụm ống khi ra khỏi màn hình. |
| `Scoring Zone` | Trigger vô hình giữa hai ống, dùng để cộng điểm khi chim bay qua. |
| `GameManager` | Quản lý trạng thái chờ, đang chơi, điểm số, thua và chơi lại. |
| `Canvas` | Hiển thị điểm, hình `Get Ready`, hình `Game Over` và nút mũi tên. |
| `Parallax` | Cuộn texture nền để tạo hiệu ứng cảnh chuyển động. |
| `Main Camera` | Hiển thị màn chơi và giúp xác định vị trí ống cần xóa. |

## 5. Luồng hoạt động

1. Scene mở ở trạng thái chờ: GameManager tạm dừng thời gian, hiện `Get Ready` và nút chơi, đồng thời tắt điều khiển chim.
2. Người chơi nhấn nút mũi tên. GameManager đặt điểm về 0, ẩn giao diện chờ, xóa ống cũ, bật thời gian và cho phép điều khiển chim.
3. Spawner tạo các cặp ống liên tục. Mỗi cặp được đặt ở độ cao ngẫu nhiên và có vùng tính điểm.
4. Người chơi nhấn Space hoặc nhấp chuột để điều chỉnh độ cao của chim.
5. Khi chim đi qua vùng tính điểm, điểm tăng. Khi chim chạm vật có tag `Obstacle`, GameManager dừng game và hiện `Game Over` cùng nút chơi lại.
6. Người chơi nhấn nút mũi tên để bắt đầu lượt mới.

## 6. Sơ đồ tương tác

```text
Space / nhấp chuột
        ↓
      Player
   ↙          ↘
Ống/mặt đất   Scoring Zone
   ↓               ↓
 Game Over      Tăng điểm
   ↓
 Chơi lại

Spawner → tạo Pipes → ống di chuyển trái → xóa khi ra khỏi màn hình
Parallax → cuộn nền
```

## 7. Thông số hiện tại

Các giá trị có thể chỉnh trong Inspector:

- `spawnRate = 1`: tạo một cặp ống mỗi giây.
- `minHeight = -1`, `maxHeight = 2`: khoảng chọn độ cao ngẫu nhiên.
- `verticalGap = 3`: khoảng cách giữa ống trên và ống dưới.
- `speed = 5`: tốc độ di chuyển của ống sang trái.
- `strength`, `gravity`, `tilt`: lần lượt điều chỉnh lực bay lên, trọng lực và độ nghiêng của chim.

Trong phiên bản hiện tại, tốc độ ống và khe hở không tự tăng theo điểm. Độ khó chủ yếu đến từ việc căn thời điểm bay qua các khe có vị trí thay đổi.

## 8. Phạm vi của phiên bản hiện tại

Đây là game một người chơi với một màn chơi cuộn vô hạn và hệ thống điểm. Game không có tấn công, vũ khí, vật phẩm thu thập, mạng nhiều người chơi hoặc các màn chơi riêng. Các quy tắc trong tài liệu này mô tả đúng các tính năng đang có trong project.

## 9. Tài liệu kỹ thuật liên quan

- [Cấu trúc bên trong game](CAU_TRUC_GAME.md): mô tả thư mục, scene, prefab, script và cấu hình Unity.
- [README](README.md): hướng dẫn mở scene và chạy game.
