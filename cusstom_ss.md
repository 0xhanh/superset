Dưới đây là quy trình việt hóa i18n cho Apache Superset trong repo `0xhanh/superset`. Repo fork này hiện không truy cập được qua công cụ tìm kiếm mã nguồn, nên hướng dẫn dựa trên cấu trúc và tài liệu upstream Apache Superset hiện tại.

## 1. Thêm tiếng Việt vào cấu hình

Trong `superset_config.py`:

```python name=superset_config.py
LANGUAGES = {
    "en": {"flag": "us", "name": "English"},
    "vi": {"flag": "vn", "name": "Tiếng Việt"},
}

BABEL_DEFAULT_LOCALE = "vi"
BABEL_DEFAULT_TIMEZONE = "Asia/Ho_Chi_Minh"
```

- `vi` phải được dùng thống nhất trong cấu hình và thư mục bản dịch.
- `flag` thường dùng mã cờ quốc gia để hiển thị trong language picker.
- `BABEL_DEFAULT_LOCALE = "vi"` đặt tiếng Việt làm ngôn ngữ mặc định.
- Nếu muốn người dùng tự chọn ngôn ngữ, vẫn giữ cả `en` và `vi` trong `LANGUAGES`.

Superset dùng Flask-Babel cho i18n và language picker được bật khi có nhiều hơn một ngôn ngữ trong `LANGUAGES`. ([github.com](https://github.com/apache/superset/blob/master/docs/developer_docs/contributing/howtos.md))

---

## 2. Cài công cụ dịch

Ở thư mục gốc repository:

```bash
pip install -r superset/translations/requirements.txt
```

Đảm bảo môi trường backend của Superset đã được kích hoạt.

Với frontend:

```bash
cd superset-frontend
npm install
cd ..
```

---

## 3. Tạo catalog tiếng Việt

Nếu thư mục `superset/translations/vi` chưa tồn tại:

```bash
pybabel init \
  -i superset/translations/messages.pot \
  -d superset/translations \
  -l vi
```

Sau lệnh này sẽ tạo file:

```text
superset/translations/vi/LC_MESSAGES/messages.po
```

Cấu trúc thường có dạng:

```text
superset/translations/
├── messages.pot
├── vi/
│   └── LC_MESSAGES/
│       ├── messages.po
│       └── messages.mo
```

---

## 4. Trích xuất chuỗi cần dịch

### Cách khuyến nghị

Superset có script cập nhật catalog:

```bash
./scripts/translations/babel_update.sh
```

Script này xử lý các chuỗi từ cả backend Python và frontend JavaScript/TypeScript.

### Hoặc chạy thủ công

Trích xuất chuỗi Python:

```bash
pybabel extract \
  -F babel.cfg \
  -o superset/translations/messages.pot \
  -k lazy_gettext \
  superset
```

Trích xuất chuỗi frontend:

```bash
cd superset-frontend
npm run build-translation
cd ..
```

Sau đó cập nhật các file `.po`:

```bash
pybabel update \
  -i superset/translations/messages.pot \
  -d superset/translations
```

Tài liệu upstream của Superset cũng phân biệt rõ quy trình trích xuất chuỗi Python và JavaScript/TypeScript. ([github.com](https://github.com/apache/superset/blob/master/docs/developer_docs/contributing/howtos.md))

---

## 5. Dịch file `messages.po`

Mở:

```text
superset/translations/vi/LC_MESSAGES/messages.po
```

Ví dụ:

```po
#: superset-frontend/src/views/CRUD/data/database/DatabaseModal/index.tsx
msgid "Add Database"
msgstr "Thêm cơ sở dữ liệu"

msgid "Save"
msgstr "Lưu"

msgid "Cancel"
msgstr "Hủy"

msgid "Search"
msgstr "Tìm kiếm"
```

Các nguyên tắc cần lưu ý:

### Giữ nguyên placeholder

```po
msgid "Showing %(start)s to %(end)s of %(total)s results"
msgstr "Đang hiển thị từ %(start)s đến %(end)s trên tổng số %(total)s kết quả"
```

Không được đổi hoặc xóa:

```text
%(start)s
%(end)s
%(total)s
%s
%d
```

### Giữ nguyên HTML hoặc Markdown

```po
msgid "You can <a>learn more</a> here."
msgstr "Bạn có thể <a>xem thêm</a> tại đây."
```

### Không dịch tên kỹ thuật

Thông thường nên giữ nguyên:

- SQL keyword
- tên database engine
- tên API field
- enum value
- tên chart plugin
- tên icon
- tên sản phẩm hoặc thương hiệu
- placeholder mẫu

Repo hiện có cơ chế đánh dấu các chuỗi không nên dịch trong `superset/translations/do-not-translate.txt`. ([github.com](https://github.com/apache/superset/blob/master/docs/developer_docs/contributing/howtos.md))

### Xử lý entry `fuzzy`

Ví dụ:

```po
#, fuzzy
msgid "Dashboard"
msgstr "Bảng điều khiển"
```

Sau khi kiểm tra bản dịch chính xác, xóa dòng `#, fuzzy`:

```po
msgid "Dashboard"
msgstr "Bảng điều khiển"
```

Trong phiên bản hiện tại, Superset vẫn có thể hiển thị bản dịch `fuzzy`, nhưng các entry này nên được kiểm tra thủ công trước khi xem là bản dịch chính thức. ([github.com](https://github.com/apache/superset/blob/master/docs/developer_docs/contributing/howtos.md))

---

## 6. Biên dịch bản dịch backend

Sau khi chỉnh sửa `.po`:

```bash
pybabel compile \
  --use-fuzzy \
  -d superset/translations
```

Lệnh này tạo file `.mo`, được backend Flask-Babel sử dụng:

```text
superset/translations/vi/LC_MESSAGES/messages.mo
```

Có thể dùng cách tương đương:

```bash
flask fab babel-compile --target superset/translations
```

Superset có hai pipeline riêng: backend biên dịch `.po` thành `.mo`, còn frontend biên dịch `.po` thành JSON/JED. Chỉ chạy một pipeline sẽ khiến một phần giao diện vẫn hiện tiếng Anh. ([github.com](https://github.com/apache/superset/blob/master/docs/developer_docs/contributing/howtos.md))

---

## 7. Biên dịch bản dịch frontend

Chạy:

```bash
cd superset-frontend
npm run build-translation
cd ..
```

Lệnh này tạo language pack cho frontend từ file `.po`.

Nếu chạy bằng Docker, cần build image với translation được bật, ví dụ:

```bash
BUILD_TRANSLATIONS=true docker compose build
docker compose up -d
```

Từ các phiên bản Superset gần đây, translation có thể không được tạo trong Docker image nếu không bật `BUILD_TRANSLATIONS=true`. ([github.com](https://github.com/apache/superset/discussions/41282?utm_source=openai))

---

## 8. Khởi động lại Superset

Sau khi biên dịch:

```bash
docker compose restart superset
```

Hoặc nếu chạy development server:

```bash
superset run -h 0.0.0.0 -p 8088
```

Sau đó:

1. Đăng xuất rồi đăng nhập lại.
2. Xóa cookie/session cũ nếu locale vẫn không đổi.
3. Hard refresh trình duyệt:
   - Chrome/Linux: `Ctrl + Shift + R`
   - macOS: `Cmd + Shift + R`
4. Kiểm tra DevTools → Network/Console nếu language pack trả về `404`.

Ngôn ngữ được lưu theo session người dùng, vì vậy thay đổi `BABEL_DEFAULT_LOCALE` không nhất thiết thay đổi ngay session hiện tại. ([github.com](https://github.com/apache/superset/discussions/33630?utm_source=openai))

---

## 9. Kiểm tra bản dịch còn thiếu

Đếm số entry chưa dịch:

```bash
grep -c '^msgstr ""' \
  superset/translations/vi/LC_MESSAGES/messages.po
```

Liệt kê các entry chưa dịch:

```bash
awk '
/^msgid / { id=$0 }
/^msgstr ""$/ { print id }
' superset/translations/vi/LC_MESSAGES/messages.po
```

Kiểm tra lỗi cú pháp catalog:

```bash
msgfmt \
  --check \
  superset/translations/vi/LC_MESSAGES/messages.po \
  -o /tmp/messages.mo
```

Kiểm tra chuỗi `fuzzy`:

```bash
grep -n "fuzzy" \
  superset/translations/vi/LC_MESSAGES/messages.po
```

---

## 10. Dịch chuỗi mới trong code

### Backend Python

Dùng `gettext` hoặc `lazy_gettext` tùy ngữ cảnh:

```python
from flask_babel import lazy_gettext as _


title = _("Database connection")
```

### Frontend TypeScript/React

Dùng hàm `t` của Superset:

```tsx
import { t } from '@superset-ui/core';

const label = t('Database connection');
```

Khi thêm chuỗi mới, cần chạy lại:

```bash
./scripts/translations/babel_update.sh
```

Sau đó dịch entry mới trong:

```text
superset/translations/vi/LC_MESSAGES/messages.po
```

Cuối cùng chạy lại cả hai bước compile:

```bash
pybabel compile --use-fuzzy -d superset/translations

cd superset-frontend
npm run build-translation
```

---

## 11. Các lỗi thường gặp

### Language picker không hiển thị tiếng Việt

Kiểm tra `superset_config.py`:

```python
LANGUAGES = {
    "en": {"flag": "us", "name": "English"},
    "vi": {"flag": "vn", "name": "Tiếng Việt"},
}
```

Cần restart Superset sau khi sửa config.

### Có chọn tiếng Việt nhưng giao diện vẫn là tiếng Anh

Kiểm tra đủ cả ba file:

```text
superset/translations/vi/LC_MESSAGES/messages.po
superset/translations/vi/LC_MESSAGES/messages.mo
superset/translations/vi/LC_MESSAGES/messages.json
```

Nếu thiếu `.mo`, backend chưa compile. Nếu thiếu frontend JSON/language pack, frontend chưa build translation.

### Lỗi locale `vi_VN` và `vi`

Nên bắt đầu bằng mã ngắn `vi`:

```python
LANGUAGES = {
    "vi": {"flag": "vn", "name": "Tiếng Việt"},
}
BABEL_DEFAULT_LOCALE = "vi"
```

Không nên trộn lẫn:

```text
vi
vi_VN
```

Nếu dùng `vi_VN`, thư mục translation và locale frontend phải khớp. Superset có logic chuẩn hóa locale vùng, nhưng mismatch giữa `vi`, `vi-VN` và `vi_VN` có thể khiến language pack bị thiếu hoặc trả về `404`. ([github.com](https://github.com/apache/superset/blob/master/superset/views/base.py))

### Một số chữ vẫn là tiếng Anh

Đây thường là một trong các nguyên nhân:

- Chuỗi chưa có trong `messages.po`.
- `msgstr` vẫn rỗng.
- Chỉ compile backend nhưng chưa build frontend.
- Docker image được build với `BUILD_TRANSLATIONS=false`.
- Browser/session vẫn đang dùng locale cũ.
- Chuỗi đó là dữ liệu do người dùng tạo, không phải chuỗi UI tĩnh.

---

## Quy trình ngắn gọn

```bash
# 1. Tạo catalog tiếng Việt nếu chưa có
pybabel init \
  -i superset/translations/messages.pot \
  -d superset/translations \
  -l vi

# 2. Cập nhật catalog
./scripts/translations/babel_update.sh

# 3. Dịch file
$EDITOR superset/translations/vi/LC_MESSAGES/messages.po

# 4. Compile backend
pybabel compile \
  --use-fuzzy \
  -d superset/translations

# 5. Build frontend
cd superset-frontend
npm run build-translation
cd ..

# 6. Restart
docker compose restart superset
```

Các file quan trọng cần commit là:

```text
superset_config.py
superset/translations/vi/LC_MESSAGES/messages.po
superset/translations/vi/LC_MESSAGES/messages.mo
superset/translations/vi/LC_MESSAGES/messages.json
```

Tùy quy ước của branch/version, file `.mo` hoặc `.json` có thể được tạo trong quá trình build thay vì commit trực tiếp; hãy kiểm tra `.gitignore` và workflow CI của repo trước khi commit.
