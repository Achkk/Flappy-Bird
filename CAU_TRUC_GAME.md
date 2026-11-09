# Cấu trúc bên trong game Flappy Bird

Tài liệu này mô tả các thư mục, tài nguyên, scene và script chính của project, cùng luồng hoạt động từ lúc bắt đầu đến khi thua.

## 1. Tổng quan

Game dùng Unity 2D. `Player` là chim do người chơi điều khiển. `Spawner` tạo các cặp ống từ prefab `Pipes`; các ống di chuyển sang trái. Người chơi ghi điểm khi đi qua vùng tính điểm nằm giữa hai ống. Va chạm với chướng ngại vật làm trò chơi dừng lại.

Luồng kết nối chính:

```text
Người chơi nhấn phím/chuột
          ↓
       Player
          ↓ va chạm
  ┌───────┴────────┐
  ↓                ↓
Ống/mặt đất     Vùng tính điểm
  ↓                ↓
GameManager.GameOver()   GameManager.IncreaseScore()
  ↓                       ↓
Hiện Game Over, dừng game    Cập nhật chữ điểm

Spawner → Pipes.prefab → di chuyển sang trái → bị xóa ngoài màn hình
Parallax → cuộn họa tiết nền
```

## 2. Cây thư mục

```text
Project root/
├── .gitignore
├── .vsconfig
├── README.md
├── CAU_TRUC_GAME.md
├── PHAN_TICH_THIET_KE_FLAPPY_BIRD.md
├── .vscode/
├── Assets/
├── Packages/
└── ProjectSettings/

Assets/
├── Fonts/
│   └── bit5x3.ttf
├── Materials/
│   ├── Background.mat
│   └── Ground.mat
├── Prefabs/
│   └── Pipes.prefab
├── Scenes/
│   └── Flappy Bird.unity
├── Scripts/
│   ├── GameManager.cs
│   ├── Parallax.cs
│   ├── Pipes.cs
│   ├── Player.cs
│   └── Spawner.cs
└── Sprites/
    ├── Background.png
    ├── Bird_01.png
    ├── Bird_02.png
    ├── Bird_03.png
    ├── GameOver.png
    ├── GetReady.png
    ├── Ground.png
    ├── Pipe.png
    └── PlayButton.png

Packages/
├── manifest.json
└── packages-lock.json

ProjectSettings/
└── các thiết lập Unity của project
```

Các file `.meta` bên cạnh tài nguyên được Unity tạo để lưu GUID và thiết lập nhập file. Scene và prefab dùng GUID này để tham chiếu đúng script, hình, font hoặc material. Không xóa riêng các file `.meta` khi vẫn giữ tài nguyên tương ứng.

Các file ở thư mục gốc:

- `.gitignore`: quy định những file/cache/build nào Git bỏ qua.
- `.vsconfig`: danh sách workload Visual Studio được khuyến nghị cho phát triển game Unity.
- `.vscode/extensions.json`: gợi ý extension Visual Studio Tools for Unity.
- `.vscode/launch.json`: cấu hình VS Code gắn debugger vào Unity Editor.
- `.vscode/settings.json`: thiết lập VS Code, gồm các loại file bị ẩn trong cây thư mục.
- `Assembly-CSharp.csproj` và `unity-flappy-bird-tutorial-main.slnx`: file project/solution tạo cho IDE để mở và chỉnh sửa C#; Unity có thể tạo lại.
- `README.md`: giới thiệu và cách mở/chạy game.
- `PHAN_TICH_THIET_KE_FLAPPY_BIRD.md`: phần phân tích đề tài, luật chơi và thiết kế tương tác của Flappy Bird.

## 3. Scene chính

Scene duy nhất trong project là `Assets/Scenes/Flappy Bird.unity`. Scene này cũng được bật trong `ProjectSettings/EditorBuildSettings.asset`, nên đây là scene được đưa vào build.

Các GameObject quan trọng trong scene:

- **Main Camera**: hiển thị màn chơi và được script `Pipes` dùng để tính mép trái cần xóa ống.
- **Player**: chim, sprite renderer, collider và script `Player`.
- **Ground**: mặt đất; được gắn tag `Obstacle` để va chạm làm kết thúc lượt.
- **Background**: hình nền/cuộn cảnh; script `Parallax` dịch chuyển offset texture của material.
- **Spawner**: điểm bắt đầu sinh ống; script `Spawner` giữ prefab và thông số tạo ống.
- **Game Manager**: đối tượng quản lý duy nhất, điều khiển tạm dừng, bắt đầu, điểm và game over.
- **Canvas**: giao diện người chơi.
  - **Score**: chữ số điểm.
  - **GetReady**: UI Image hiển thị `GetReady.png` ở màn hình chờ.
  - **GameOver**: UI Image hiển thị `GameOver.png`; bị ẩn lúc bắt đầu và chỉ bật khi chim thua.
  - **PlayButton**: nút có hình mũi tên để bắt đầu hoặc chơi lại.
- **EventSystem**: nhận sự kiện tương tác UI, chẳng hạn nhấn nút.

## 4. Prefab ống

`Assets/Prefabs/Pipes.prefab` là mẫu được `Spawner` tạo ra nhiều lần. Prefab có:

- **Pipes** (GameObject gốc): mang script `Pipes`, điều khiển cả cụm ống và khoảng cách giữa hai ống.
- **Top Pipe** và **Bottom Pipe**: hai hình ống; mỗi ống có tag `Obstacle` và `BoxCollider2D` dạng trigger.
- **Scoring Zone**: vùng trigger nằm giữa cặp ống, có tag `Scoring`. Khi Player đi qua vùng này, điểm tăng một lần.

Các giá trị mặc định trong prefab/script gồm tốc độ ống `5` và khe dọc `3`. `Spawner` có thể ghi đè khe dọc khi tạo mỗi cụm.

## 5. Vai trò từng script

### `Player.cs`

- Lấy `SpriteRenderer` của chim.
- Lặp qua các sprite `Bird_01`, `Bird_02`, `Bird_03` để tạo hoạt ảnh vỗ cánh.
- Khi nhấn Space hoặc chuột trái, đặt vận tốc hướng lên theo `strength`.
- Mỗi frame cộng trọng lực vào hướng bay, cập nhật vị trí và nghiêng chim theo vận tốc dọc.
- Khi trigger chạm tag `Obstacle`, gọi `GameManager.GameOver()`.
- Khi trigger chạm tag `Scoring`, gọi `GameManager.IncreaseScore()`.

### `GameManager.cs`

- Giữ `Instance` tĩnh để các script khác gọi cùng một GameManager.
- `Start()` hiện `GetReady`, ẩn `GameOver`, giữ nút chơi và tạm dừng game.
- `Pause()` đặt `Time.timeScale` về `0` và tắt điều khiển Player.
- `Play()` đặt điểm về `0`, ẩn `GetReady`, `GameOver` và nút chơi, chạy lại thời gian, bật Player và xóa các ống cũ.
- `GameOver()` ẩn `GetReady`, hiện lại nút chơi và ảnh `GameOver`, sau đó dừng game.
- `IncreaseScore()` tăng điểm và cập nhật Text điểm.

### `Spawner.cs`

- Khi được bật, gọi `Spawn()` lặp lại theo `spawnRate`.
- Mỗi lần tạo prefab `Pipes` tại vị trí Spawner.
- Chọn độ lệch dọc ngẫu nhiên trong khoảng `minHeight` đến `maxHeight`.
- Gán khoảng cách ống từ `verticalGap`.
- Khi bị tắt, hủy lịch gọi lặp.

### `Pipes.cs`

- Khi bắt đầu, dịch ống trên và dưới ra hai phía để tạo khe.
- Mỗi frame di chuyển cả cụm sang trái theo `speed`.
- Xóa cụm khi vị trí của nó vượt khỏi mép trái camera.

### `Parallax.cs`

- Lấy `MeshRenderer` của nền.
- Mỗi frame tăng offset texture theo `animationSpeed`, tạo hiệu ứng nền chuyển động.

## 6. Thông số thường chỉnh

Các thông số này có thể được chỉnh trong Inspector của GameObject mang script tương ứng:

- **Player**: `strength` quyết định độ bật lên; `gravity` quyết định chim rơi nhanh thế nào; `tilt` quyết định độ nghiêng theo hướng bay; `sprites` là các khung hoạt ảnh.
- **Spawner**: `spawnRate` là thời gian giữa hai lần sinh ống; `minHeight` và `maxHeight` giới hạn độ cao ngẫu nhiên; `verticalGap` là khoảng trống giữa hai ống; `prefab` chọn mẫu ống.
- **Pipes**: `speed` điều khiển tốc độ trôi sang trái; `gap` là khoảng cách dọc giữa ống trên và dưới.
- **Parallax**: `animationSpeed` điều khiển tốc độ cuộn họa tiết nền.
- **GameManager**: các trường `player`, `spawner`, `scoreText`, `getReady`, `playButton` và `gameOver` là liên kết đến các đối tượng trong scene.

## 7. Luồng một lượt chơi

1. Khi mở scene, GameManager hiện `GetReady` và nút chơi, ẩn `GameOver`, đặt `Time.timeScale = 0` và tắt điều khiển chim.
2. Người chơi nhấn nút mũi tên. `Play()` đặt điểm về 0, dọn các ống cũ, bật thời gian và bật Player.
3. `Spawner` tạo các cặp ống theo chu kỳ. Mỗi cặp có vị trí dọc ngẫu nhiên và vùng tính điểm.
4. Người chơi nhấn Space hoặc nhấp chuột để bay lên; trọng lực kéo chim xuống giữa các lần nhấn.
5. Đi qua `Scoring Zone` thì điểm tăng. Chạm vật có tag `Obstacle` thì hiện ảnh `GameOver`, hiện nút chơi lại, ẩn `GetReady` và dừng thời gian.
6. Nhấn nút lần nữa để bắt đầu lại từ điểm 0.

## 8. Hình ảnh, font và vật liệu

- `Background.png`: hình nền.
- `Bird_01.png`, `Bird_02.png`, `Bird_03.png`: ba khung hoạt ảnh chim.
- `Ground.png`: hình mặt đất.
- `Pipe.png`: hình ống dùng trong prefab.
- `PlayButton.png`: hình nút bắt đầu/chơi lại.
- `GameOver.png`: ảnh chữ Game Over, chỉ hiện sau khi chim thua.
- `GetReady.png`: ảnh chữ Get Ready, hiện ở màn hình chờ trước khi bắt đầu.
- `bit5x3.ttf`: font pixel cho giao diện.
- `Background.mat`, `Ground.mat`: vật liệu dùng cho các phần nền/mặt đất.

## 9. Cấu hình project

- `Packages/manifest.json`: danh sách package trực tiếp, gồm các gói 2D và UI.
- `Packages/packages-lock.json`: phiên bản package được giải quyết, gồm cả phụ thuộc gián tiếp.
- `ProjectSettings/ProjectVersion.txt`: phiên bản Editor ghi nhận hiện tại là Unity `6000.6.6f1`.
- `ProjectSettings/TagManager.asset`: khai báo tag `Obstacle`, `Scoring` và các layer.
- `ProjectSettings/EditorBuildSettings.asset`: danh sách scene đưa vào build.
- `ProjectSettings/ProjectSettings.asset`: cấu hình chung và cấu hình build của trò chơi, như tên ứng dụng và nền tảng.
- `ProjectSettings/InputManager.asset`: trục và nút của hệ thống Input Manager cũ; script Player hiện tự đọc Space và chuột trái.
- `ProjectSettings/Physics2DSettings.asset` và `PhysicsCoreProjectSettings2D.asset`: thiết lập vật lý 2D.
- `ProjectSettings/TimeManager.asset`: bước thời gian và thời gian cố định của mô phỏng.
- `ProjectSettings/GraphicsSettings.asset` và `QualitySettings.asset`: thiết lập hiển thị, shader và chất lượng.
- `ProjectSettings/AudioManager.asset`: cấu hình âm thanh.
- `ProjectSettings/EditorSettings.asset`, `PresetManager.asset`, `VersionControlSettings.asset`, `PackageManagerSettings.asset`: thiết lập Editor, preset, quản lý phiên bản và package.
- `ProjectSettings/MemorySettings.asset`: cấu hình bộ nhớ.
- `ProjectSettings/ClusterInputManager.asset`: cấu hình đầu vào cho chế độ nhiều màn hình/máy trong cluster; game hiện không dựa vào tính năng này.
- `ProjectSettings/MultiplayerManager.asset` và `NetworkManager.asset`: cấu hình multiplayer/network; game hiện không dùng luồng mạng.
- `ProjectSettings/NavMeshAreas.asset`: các vùng điều hướng NavMesh; game hiện không dùng NavMesh.
- `ProjectSettings/UnityConnectSettings.asset`: thiết lập kết nối dịch vụ Unity.
- `ProjectSettings/ProjectAuditorSettings.asset`: thiết lập công cụ kiểm tra project.
- `ProjectSettings/VFXManager.asset` và `XRSettings.asset`: cấu hình VFX và XR; game hiện không cần các tính năng này.

Trong `Packages/manifest.json`, các package đáng chú ý gồm 2D Animation, 2D Pixel Perfect, 2D PSD Importer, Sprite Shape, 2D Tilemap, UI (UGUI), AI Navigation và tích hợp Visual Studio. `packages-lock.json` lưu phiên bản đã giải quyết cho các package cùng phụ thuộc của chúng.

Hãy dùng phiên bản ghi trong `ProjectVersion.txt` khi chọn Unity Editor để mở project.

## 10. File do công cụ tạo

Khi Unity mở project, nó tạo các thư mục như `Library/`, `Temp/`, `Logs/` và `UserSettings/`. Đây là cache, file tạm, log và thiết lập riêng trên máy. Chúng không phải nội dung logic của game; Unity có thể tạo lại chúng. `.gitignore` đã loại các thư mục này cùng file build và file IDE sinh tự động khỏi Git.
