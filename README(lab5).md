# Nhập Môn Xử Lý Ảnh Số - Lab 5

## **THỰC HÀNH 5: XÁC ĐỊNH ĐỐI TƯỢNG TRONG ẢNH**

**Sinh viên thực hiện:** Nguyễn Hoàng Vương
**MSSV:** 2374802010574

**Môn học**: Nhập môn xử lý ảnh số

**Giảng viên**: Đỗ Hữu Quân

## Giới thiệu

Bài lab này nhằm mục đích thực hiện **cách xác định các đối tượng trong ảnh** - là những thao tác cơ bản, nền tảng trong lĩnh vực xử lý ảnh số. Các cách xác định này giúp:

- Đếm số vật thể, định danh vùng, tiền xử lý phân tích hình dạng
- Tách các vùng có ý nghĩa, phục vụ phân tích đối tượng hoặc học sâu
- Làm sạch, tăng cường, chuẩn hóa hoặc biến đổi cho mục đích đặc biệt

## Công nghệ sử dụng

- **Python**: Ngôn ngữ chính
- **Pillow (PIL)**: Đọc, chuyển đổi và lưu ảnh
- **NumPy**: Xử lý ảnh dưới dạng mảng số học
- **ImageIO**: Đọc file ảnh với định dạng hiện đại
- **Matplotlib**: Hiển thị ảnh trực quan
- **OpenCV**: Xử lý ảnh và video
- **Scipy**: Xử lý ảnh và dữ liệu đa chiều (binary, grayscale,...)
- **Threshold_local/Otsu**: Tính ngưỡng cho từng vùng nhỏ/Nhị phân toàn cục.

## Chi tiết các phép xác định & công thức

### 1. Gán nhãn ảnh

**Mục đích**:

- Gán nhãn cho các vùng liên thông trong ảnh nhị phân và trực quan hóa vùng đó bằng khung bao (bounding box).

**Code chính**:

```py
for i in d:
    lr, lc, ur, uc = i['BoundingBox']
    rec_width = uc - lc
    rec_height = ur - lr
    rect = mpatches.Rectangle((lc, lr), rec_width, rec_height, fill=False, edgecolor='black', linewidth=2)
    ax.add_patch(rect)
```

**Công thức gán nhãn**:

```math
L(x, y) = k ( nếu  ∈ C k​)
```

---

### 2. Dò tìm cạnh theo chiều dọc

**Mục đích**:

- Dò tìm cạnh (biên) theo chiều dọc, bằng cách sử dụng phép trừ giữa ảnh gốc và ảnh đã dịch sang phải 1 pixel.

**Công thức toán học**:

```math
bmg(x,y)=|f(x,y)−f(x,y+1)|
```

**Code chính**:

```py
bmg = abs(data - nd.shift(data, (0, 1), order=0))
```

---

### 3. Dò tìm cạnh với Sobel Filter

**Mục đích**:

- Dò tìm biên cạnh trong ảnh bằng toán tử Sobel theo cả 2 chiều (ngang và dọc) và trực quan hóa kết quả.

**Công thức**:

```math
bmg = |a| + |b|
```

**Code chính**:

```py
data = Image.open('geometric.png')
a = nd.sobel(data, axis=0)
b = nd.sobel(data, axis=1)
bmg = abs(a) + abs(b)
```

---

### 4. Xác định góc của đối tượng

**Mục đích**:

- Xác định các góc bằng thuật toán Harris Corner.

**Công thức:**

```math
detC = x1 * y1 - 2 * xy \\
trC = x1 + y1 \\
R = detC - alpha * trC**2
```

**Code chính:**

```py
def Harris(indata, alpha=0.2):
    x = nd.sobel(indata, 0)
    y = nd.sobel(indata, 1)
    x1 = x ** 2
    y1 = y ** 2
    xy = abs(x * y)
    # Làm mượt các thành phần trên bằng Gaussian filter
    x1 = nd.gaussian_filter(x1, 3)
    y1 = nd.gaussian_filter(y1, 3)
    xy = nd.gaussian_filter(xy, 3)

    detC = x1 * y1 - 2 * xy # Tính định thức ma trận C
    trC = x1 + y1 # Tính trace
    R = detC - alpha * trC**2 # Tính hàm phản hồi Harris
    return R
```

---

### 5. Dò tìm đường thẳng trong ảnh

**Mục đích:**

- Nhằm phát hiện các đường thẳng có mặt trong ảnh nhị phân.
- Phát hiện đường biên, vạch kẻ đường, lề đường, mép vật thể.

**Code chính:**

```py
def LineHough(data, gamma):
    V, H = data.shape
    R = int(np.sqrt(V * V + H * H))
    ho = np.zeros((R, 90), float) # Khởi tạo ma trận Hough
    w = data + 0
    ok = 1
    theta = np.arange(90)/180.0 * np.pi # Tạo mảng gốc theta từ 0° đến 89° (rad)
    tp = np.arange(90).astype(float)
    while ok:
        mx = w.max() # Tìm gtri pixel lớn nhất
        if mx < gamma:
            ok = 0
        else:
            v, h = divmod(w.argmax(), H) #Lấy tọa độ điểm sáng nhất
            y = V - v #Tính lại y theo hệ tọa độ gốc dưới
            x = h
            rh = x * np.cos(theta) + y * np.sin(theta) # tính rho cho mọi góc theta
            for i in range(len(rh)):
                if 0 <= rh[i] < R and 0 <=tp[i] < 90:
                    ho[int(rh[i]), int(tp[i])] += mx
            w[v, h] = 0
    return ho

data = np.zeros((256, 256))
data[128, 128] = 1
bmg = LineHough(data, 0.5)
```

---

### 6. Dò tìm đường tròn trong ảnh

**Mục đích**:

- Xử lý để phát hiện các đường tròn trong ảnh bằng cách xác định các điểm có thể là tâm hoặc nằm trên đường tròn.

**Code chính**:

```py
image_gray = rgb2gray(data)
# Áp dụng thuật toán Harris để phát hiện các điểm góc
coordinate = corner_harris(image_gray, k = 0.001)
```

---

### 7. Image matching

**Mục đích**:

- Xác định các điểm tương đồng giữa hai ảnh bằng cách tìm các điểm góc (corner), trích xuất đặc trưng quanh các điểm đó và so sánh với nhau.
- Phục vụ cho các ứng dụng như nhận diện, ghép ảnh, hay theo dõi chuyển động.

**Code chính**:

```py
def harris_corners(imgGray, threshold=0.01):
    imgFloat = np.float32(imgGray) # chuyển sang ảnh xám
    dst = cv2.cornerHarris(imgFloat, 2, 3, 0.04)
    dst = cv2.dilate(dst, None) #dùng phép giãn làm nổi bật góc
    points = np.argwhere(dst > threshold * dst.max())
    return [tuple(p[::-1]) for p in points] #đổi thứ tự hàng và cột thành x, y


def get_patch(img, point, size=11):
    x, y = point
    r = size #bán kính
    if x - r < 0 or y - r < 0 or x + r >= img.shape[1] or y + r >= img.shape[0]:
        return None #nếu vượt biên ảnh thì bỏ qua
    return img[y - r:y + r + 1, x - r:x + r + 1] #cắt ra vùng vuông quanh điểm x,y


def compute_descriptors(img, keypoints, size=11):
    descriptors = [] #ds chứa vector
    valid_pts = []
    for pt in keypoints:
        patch = get_patch(img, pt, size) #lấy patch quanh điểm góc
        if patch is not None:
            desc = patch.flatten().astype(np.float32)
            desc = (desc - np.mean(desc)) / (np.std(desc) + 1e-8) # chuẩn hóa về trung bình 0, độ lệch chuẩn 1
            descriptors.append(desc)
            valid_pts.append(pt)
    return valid_pts, descriptors


def match_feature(desc1, pts1, desc2, pts2, ratio=0.75):
    matches = [] #ds các cặp điểm ghép đúng
    for i, d1 in enumerate(desc1):
        dists = np.linalg.norm(desc2 - d1, axis=1)
        if len(dists) < 2:
            continue #bỏ qua nếu không đủ điểm để so sánh
        idx = np.argsort(dists) #sắp xếp theo khoảng cách tăng dần
        best, second = dists[idx[0]], dists[idx[1]]
        if best / (second + 1e-8) < ratio:
            matches.append((pts1[i], pts2[idx[0]]))
    return matches
```
