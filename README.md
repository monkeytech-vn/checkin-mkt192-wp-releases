# Checkin MKT192 WP - Release Channel

Kênh phát hành chính thức của plugin **Checkin MKT192 WP** (mã nguồn ở repo private
`monkeytech-vn/checkin-mkt192-wp`). Repo này **không sửa tay**: mọi thứ ở đây do GitHub Actions của repo
nguồn tự đẩy lên mỗi khi có tag `v*`.

## Cấu trúc

| Nhánh | Chứa gì | Ai ghi |
|---|---|---|
| `main` | `manifest.json` (bản mới nhất) + README này | workflow `release.yml`, commit `chore: manifest vX.Y.Z.W` |
| `dist` | Các file `checkin-mkt192-wp-<version>.zip`, giữ **2 bản gần nhất** | workflow, nhánh orphan, **force-push** mỗi lần |

- Plugin trên site khách đọc `manifest.json` ở `main` qua raw.githubusercontent để biết có bản mới và
  báo update trong WP Admin.
- Trường `package` trong manifest trỏ thẳng vào zip trên nhánh `dist`:
  `https://raw.githubusercontent.com/monkeytech-vn/checkin-mkt192-wp-releases/dist/checkin-mkt192-wp-<version>.zip`
- Mục **Releases** của GitHub (bên phải trang repo) **không còn dùng** từ bản 5.3.3.1. Khung đó đứng ở
  v5.3.3.0 là bình thường, đừng dựa vào nó để biết bản mới nhất. Muốn biết bản đang phát hành, xem
  commit `chore: manifest vX` mới nhất trên `main` hoặc mở `manifest.json`.
- Thông báo vàng *"dist had recent pushes"* xuất hiện sau mỗi lần release vì workflow vừa push nhánh
  `dist`. **Không bấm "Compare & pull request"**: `dist` không chung lịch sử với `main`.

## Quy trình phát hành (chạy ở repo nguồn)

1. Viết mục version mới vào `CHANGELOG.md` (workflow sinh HTML changelog từ đúng mục này).
2. Bump version ở header plugin `checkin-mkt192-wp.php` và `Stable tag` trong `README.txt`
   (`bin/release.sh <version>` làm bước này, hoặc làm tay khi release từ worktree).
3. Một commit `release(vX.Y.Z.W): checkin-mkt192-wp X.Y.Z.W` lên `main` + tag `vX.Y.Z.W`, push cả hai.
4. Workflow `release.yml` tự chạy (khoảng 1 phút):
   - kiểm tag khớp header plugin, sai thì dừng;
   - `git archive` ra zip, thư mục gốc trong zip là `checkin-mkt192-wp/`;
   - đẩy zip lên nhánh `dist`, xoá bản cũ chỉ giữ 2 bản;
   - sửa `manifest.json` (`version`, `package`, `last_updated`, `changelog`), commit lên `main`;
   - **tự kiểm**: tải lại manifest + zip đúng như site khách sẽ tải, giải nén đối chiếu version, sai thì fail.
5. Xem kết quả: `gh run list --repo monkeytech-vn/checkin-mkt192-wp`. Xanh là đã publish xong.

Xác thực bằng deploy key (secret `RELEASES_DEPLOY_KEY` ở repo nguồn, deploy key write ở repo này),
không dùng token cá nhân.

## Site khách nhận bản mới thế nào

- Plugin cache manifest **6 giờ** (transient `checkin_mkt192_update_manifest`). Nút *Kiểm tra lại* ở
  Dashboard → Cập nhật của WordPress **không** xoá được cache này. Bản mới sẽ hiện trong tối đa 6 giờ,
  sau đó vào Plugins bấm **Cập nhật** như bình thường, plugin vẫn giữ trạng thái kích hoạt.
- Kiểm nhanh site đang chạy bản nào không cần đăng nhập:
  `curl -s https://<site>/wp-content/plugins/checkin-mkt192-wp/README.txt | grep "Stable tag"`.

## Lưu ý

- Không commit zip vào `main`, không sửa tay `manifest.json`: lần release sau workflow sẽ ghi đè.
- raw.githubusercontent cache khoảng 5 phút, workflow đã chờ sẵn; nếu tự curl ngay sau release mà còn thấy
  version cũ thì thêm `?cb=<số ngẫu nhiên>` vào URL.
