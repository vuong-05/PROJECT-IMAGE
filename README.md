# THỰC HÀNH 3: BIẾN ĐỔI HÌNH HỌC

## 1. Viết chương trình biến đổi ảnh:

    1.1. Chọn đối tượng trong ảnh: là phép trích ảnh nhỏ trong ảnh lớn hơn ban đầu.
    - Đều sử dụng những thư viện giống như lab 2 nên em k giải thích nữa.
    - Main code và cách hoạt động:

```python
data = iio.imread('fruit.jpg') #Đọc ảnh
bmg = data[800:1200, 570:980] #Lấy tọa độ quả cam
print(data.shape)
iio.imsave('orange.jpg', bmg) #Lưu ảnh
```

    1.2. Tịnh tiến đơn là 1 phép biến đổi hình học dịch chuyển toàn bộ ảnh theo 1 vector(dx, dy).
    - Cách thức hoạt động ở đoạn code:

```python
nd.shift(data, (100, 25)) #Dịch toàn ảnh theo vector(dy=100, dx=25)
```

    1.3. Thay đổi kích thước ảnh là giảm không gian của ảnh (phóng to hoặc thu nhỏ).
    - Main code:

```python
bdata = nd.zoom(data, 2) #phóng to toàn bộ ảnh lên 2 lần ở cả 3 chiều (cao, rộng, channels)
data2 = nd.zoom(data, (2, 2, 1)) #phóng to theo (chiều cao nhân 2, rộng x2, channels giữ nguyên)
data3 = nd.zoom(data, (0.5, 0.9, 1)) #cao 50%, rộng 90% và channels giữ nguyên
```

    1.4. Xoay ảnh: dùng hàm rotate(image, degree) để xoay 1 ảnh với
        + Image: là ảnh trong bộ nhớ
        + Degree: là góc xoay
    - Main code và cách thức hoạt động:

```py
d1 = nd.rotate(data, 20) #data là ảnh trong bộ nhớ và xoay ngược chiều kim đồng hồ 20 độ
d2 = nd.rotate(data, 20, reshape=False) #giữ nguyên kích thước gốc của ảnh
```

    1.5. Dilation và Erosion dùng để loại bỏ những pixel nhiễu.
    - Dilation thay thế pixel tọa độ (i, j) bằng giá trị lớn nhất của những pixel lân cận (kề).
    - Erosion thay thế pixel tọa độ (i, j) bằng giá trị nhỏ nhất của những pixel lân cận (kề).
    - Cách thức hoạt động:

```py
d1 = nd.binary_dilation(data) #giãn nhị phân 1 phép biến đổi hình thái
plt.imshow(d1)
plt.show()
d2 = nd.binary_dilation(data, iterations=3) #lặp phép giãn 3 lần liên tiếp
plt.imshow(d2)
#binary_dilation làm nở rộng các vùng trắng trong ảnh nhị phân
```

    1.6. Coordinate Mapping cho phép tạo hàm mới do người dùng định nghĩa ngoài các hàm có sẵn như shifting, rotate.
        - Tạo 1 coordinate map chứa các pixel mới
        - Dùng hàm map_coordinate để ánh xạ vị trí mới cho ảnh đầu vào
    - Code:

```py
V, H = data.shape #Lấy kích thước ảnh
M = np.indices((V, H)) #tạo ma trận 2 chiều chứa y và x của điểm ảnh
d = 5 #Đặt biên độ nhiễu tối đa
q = 2 * d * np.random.ranf(M.shape) - d #tạo ma trận ngẫu nhiên [-d, d] và cùng kích thước với M
mp = (M + q).astype(int) #cộng nhiễu vào chỉ số gốc, tạo lưới tọa độ mới bị nhiễu
d1 = nd.map_coordinates(data, mp) #Lấy giá trị ảnh từ tọa độ đã bị nhiễu bằng ánh xạ tọa độ
```

    1.7. Biến đổi chung dùng khi ta muốn biến đổi các ảnh chung phép toán do người dùng định nghĩa.
    - Code và giải thích:

```py
#tạo hàm biến đổi hình học: dịch và làm gợn sóng tọa độ ảnh theo hàm cos
def GeoFun(outcoord):
    #outcoord[0] = y, outcoord[1] = x
    #a: tọa độ y mới sau biến đổi
    a = 10 * np.cos(outcoord[0] / 10.0) + outcoord[0]
    #b: tọa độ x mới sau biến đổi
    b = 10 * np.cos(outcoord[1] / 10.0) + outcoord[1]
    # Trả về tọa độ trong ảnh gốc để lấy giá trị màu
    return a, b
data = iio.imread('world_cup.jpg', mode='F')
d1 = nd.geometric_transform(data, GeoFun)
```

### 2. BÀI TẬP

> bt1

```py
translated_img = Image.new('RGB', img.size) #Tạo ảnh mới cùng kích thước ảnh gốc
translated_img.paste(img, (30, 0)) #lệch sang phải 30 pixel (x=30, y=0)
translated_img.save('kiwi.jpg') #lưu ảnh với tên kiwi.jpg
```

> bt2

```py
dudu = img[300:820, 100:750] #Cắt ảnh quả đu đủ
duahau = img[200:1200, 1650:2500] #Cắt ảnh quả dưa hấu
dudu[:, :, [0, 2]] = dudu[:, :, [2, 0]] #đổi kênh màu của đu đủ
duahau[:, :, 2] = np.clip(duahau[:, :, 0] + 100, 0, 255) #làm dưa hấu ngả sang xanh
fig, axes = plt.subplots(1, 2, figsize=(12, 6)) #hiển thị ảnh đu đủ và dưa hấu sau khi xử lý
axes[0].imshow(dudu)
axes[1].imshow(duahau)
plt.tight_layout()
```

> bt3

```py
nui = img[0:330, 410: 700] #Cắt vùng ảnh chứa núi
thuyen = img[420:580, 500:700] #Cắt vùng ảnh chứa thuyền
xoay_ngon_nui = nd.rotate(nui, 45, reshape=True) #xoay ngọn núi 45(reshape=True để mở rộng kích thước)
xoay_con_thuyen = nd.rotate(thuyen, 45, reshape=True) #xoay con thuyền 45 độ
iio.imsave('ngon_nui.jpg', xoay_ngon_nui) #lưu 2 ảnh đã xoay
iio.imsave('con_thuyen.jpg', xoay_con_thuyen)
fig, axes = plt.subplots(1, 2, figsize=(12, 6))
axes[0].imshow(xoay_ngon_nui)
axes[1].imshow(xoay_con_thuyen)
```

> bt4

```py
pagoda = img[130:210, 0:600] #cắt ảnh chứa ngôi chùa từ ảnh gốc
upsizePagoda = nd.zoom(img, (5, 5, 1)) #phóng to ảnh lên 5 lần theo chiều cao và chiều rộng
iio.imsave('pagodaZoomX5.jpg', upsizePagoda) #lưu ảnh
plt.imshow(upsizePagoda)
```

> bt5

```py
#Hàm tịnh tiến ảnh theo dx, dy
def tinhTien(img, shift_x=30, shift_y=30):
    return nd.shift(img, shift=(shift_y, shift_x, 0))  #shift=(trục y, trục x, màu)

#Hàm xoay ảnh 1 góc 45 độ, reshape=True để không bị cắt góc
def xoay(img, angle=45):
    return nd.rotate(img, angle, reshape=True)

#Hàm phóng to hoặc thu nhỏ ảnh theo hệ số vector
def phongto_thunho(img, vector):
    return nd.zoom(img, (vector, vector, 1))

#Hàm biến dạng ảnh bằng cách làm nhiễu ngẫu nhiên tọa độ (hiệu ứng sóng)
def coordinate_map(img):
    if img.ndim == 3:
        data = img.mean(axis=2)  #nếu ảnh màu, chuyển về ảnh xám để xử lý
    else:
        data = img

V, H = data.shape
M = np.indices((V, H))  #Tạo lưới tọa độ gốc (y, x)
d = 5
q = 2 * d * np.random.ranf(M.shape) - d  #Sinh nhiễu ngẫu nhiên [-d, d]
mp = (M + q).astype(int)
mp[0] = np.clip(mp[0], 0, V - 1)  #Giới hạn tọa độ y
mp[1] = np.clip(mp[1], 0, H - 1)  #Giới hạn tọa độ x
d1 = nd.map_coordinates(data, mp)

#Đọc ảnh được chọn
file_path = os.path.join(folder, files[index])
img = iio.imread(file_path)
#Hiển thị menu lựa chọn xử lý ảnh
print("\nChọn thao tác:")
print("T - Tịnh tiến")
print("X - Xoay ảnh")
print("P - Phóng to")
print("H - Thu nhỏ")
print("C - Coordinate Map")
#nhập lựa chọn từ người dùng
choice = input("Nhập lựa chọn của bạn (T/X/P/H/C): ").strip().upper()
#dùng if, elif, else để người dùng nhập vào các giá trị sau khi chọn thao tác
```
