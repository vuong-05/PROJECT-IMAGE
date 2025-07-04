# Nhập Môn Xử Lý Ảnh Số - Lab 4

## **THỰC HÀNH 4: PHÂN VÙNG ẢNH**

**Sinh viên thực hiện:** Nguyễn Hoàng Vương
**MSSV:** 2374802010574

**Môn học**: Nhập môn xử lý ảnh số

**Giảng viên**: Đỗ Hữu Quân

## Giới thiệu

Bài lab này nhằm mục đích thực hiện **các phép phân vùng ảnh và biến đổi** - là những thao tác cơ bản, nền tảng trong lĩnh vực xử lý ảnh số. Các phép phân vùng và biến đổi này giúp:

- Tách vùng cần thiết để xử lý
- Tách rõ đối tượng khỏi nền
- Xử lý nhiễu, làm sạch ảnh
- Kết nối các vùng rời rạc, lấp lỗ hổng

## Công nghệ sử dụng

- **Python**: Ngôn ngữ chính
- **Pillow (PIL)**: Đọc, chuyển đổi và lưu ảnh
- **NumPy**: Xử lý ảnh dưới dạng mảng số học
- **ImageIO**: Đọc file ảnh với định dạng hiện đại
- **Matplotlib**: Hiển thị ảnh trực quan
- **OpenCV**: Xử lý ảnh và video
- **Scipy**: Xử lý ảnh và dữ liệu đa chiều (binary, grayscale,...)
- **Threshold_local/Otsu**: Tính ngưỡng cho từng vùng nhỏ/Nhị phân toàn cục.

## Chi tiết các phép phân vùng & công thức

### 1. Phân vùng theo histogram bằng pp Otsu

**Mục đích:**

- Tự động phân tách ảnh thành các vùng sáng và tối (nền và đối tượng) dựa vào phân bố độ sáng.
- Giúp chuyển ảnh mức xám thành ảnh nhị phân (đen–trắng) để dễ xử lý hơn.

**Code chính:**

```Python
thres = threshold_otsu(a)
b = a > thres
```

---

### 2. Phương pháp Adaptive Thresholding

**Mục đích:**

- Cải tiến phân vùng chính xác hơn Otsu.
- Chia ảnh thành nhiều ảnh nhỏ và tính threshold cho từng ảnh nhỏ.

**Code chính:**

```py
a = np.array(data)
b = threshold_local(a, 39, offset=10)
```

---

### 3. Phân vùng theo region

**Mục đích:**

- Chia ảnh thành các vùng có tính chất giống nhau.
- Hiểu cấu trúc của ảnh và tách các đối tượng có ý nghĩa.

**Code chính:**

```py
a = cv2.cvtColor(data, cv2.COLOR_BGR2GRAY)
thresh, b1 = cv2.threshold(a, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU) #dùng otsu + binary đảo để phân tách foreground và background
b2 = cv2.erode(b1, None, iterations=2)
dist_trans = cv2.distanceTransform(b2, 2, 3) #tính khoảng cách đến background
thresh, dt = cv2.threshold(dist_trans, 1, 255, cv2.THRESH_BINARY) #Phân ngưỡng ảnh distance để xác định vùng foreground
labelled, ncc = label(dt)
labelled = labelled.astype(np.int32)
cv2.watershed(data, labelled)
b = Image.fromarray(labelled)
```

---

### 4. Biến đổi đối tượng trong ảnh sử dụng binay dilation

**Mục đích:**

- Dilation cho phép các pixel ở foreground của 1 ảnh có thể co giãn.
- Mở rộng biên đối tượng, kết nối các vùng trắng gần nhau, làm đầy lỗ nhỏ.

**Code chính:**

```py
b = nd.binary_dilation(data, iterations=50) #giãn nở ảnh nhị phân 50 lần
```

---

### 5. Binary opening

**Mục đích:**

- Lọc nhiễu và làm mượt biên ảnh nhị phân.

**Công thức:**

```math
opening = erosion -> dilation
```

**Lưu ý:** Erosion xóa nhiễu nhỏ → Dilation khôi phục lại hình dạng gốc sau khi làm sạch.

**Code chính:**

```py
s = [[0, 1, 0], [1, 1, 1], [0, 1, 0]] #tạo phần tử 3x3
b = nd.binary_opening(data, structure=s, iterations=25) #Xóa nhiễu trắng nhỏ và làm mịn biên ảnh nhị phân 25 lần
```

---

### 6. Binary erosion

**Mục đích:**

- Dùng để co đối tượng bằng cách loại bỏ pixels ở biên của đối tượng.

**Code chính:**

```py
binary = img > 128 #Ảnh mức xám thành ảnh nhị phân (trắng)
s = np.array([[0, 1, 0], [1, 1, 1], [0, 1, 0]])
b = nd.binary_erosion(binary, structure=s, iterations=50) #làm co và xóa vùng trắng nhỏ, tách rời các vật thể dính nhau
```

### 7. Binary closing

**Mục đích:**

- Dùng để làm kín ảnh nhị phân.

**Công thức:**

```math
closing = dilation -> erosion
```

**Code chính:**

```py
b = nd.binary_closing(binary, structure=s, iterations=50) #lấp các lỗ đen nhỏ trong vùng trắng và kết nối các vùng trắng
```

---

## Cấu trúc file

```
├── lab4.ipynb
├── README(lab4).md
```
