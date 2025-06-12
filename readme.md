_Giải thích và tóm gọn lại những gì đã học ở bài thực hành 2 (XỬ LÝ ĐIỂM ẢNH):_

- 1.1. Biến đổi cường độ ảnh (Image inverse) là phép biến đổi cường độ ảnh từ tối sang sáng và ngược lại. Những thư viện đoạn code sử dụng: + PIL (để đọc và xử lý ảnh, chuyển sang ảnh xám) + numpy dùng để xử lý ma trận của ảnh + matplotlib (hiển thị ảnh lên màn hình) + imageio (hỗ trợ đọc và ghi ảnh)

  - Ở phần 1.1 đoạn code im_2 = 255- im_1. Mỗi pixel được biến đổi bằng công thức: pixel(new) = 255 - pixel(old) -> Điều này sẽ đảo ngược màu ảnh: từ tối thành sáng và ngược lại.
  - Kết quả sẽ thấy 1 ma trận pixel (ảnh gốc) và ảnh đã được đảo màu.

- 1.2. Tăng chất lượng ảnh với Power law (Gamma-correction) dùng để tăng chất lượng của ảnh bằng cách thay đổi các giá trị gamma. Thư viện của đoạn code này như trên.

  - Ở đoạn code 1.2: ảnh được chuyển sang ảnh xám và chuẩn hóa về [0, 1] (b3 = b1 / b2) -> sau đó dùng hàm log để làm sáng vùng tối, giúp ảnh rõ hơn ở các chi tiết bị tối bằng công thức b2 = np.log(b3) \* gamma -> cuối cùng ánh xạ ngược lại thành ảnh gốc (giá trị từ 0-255) để hiển thị.
  - Kết quả thấy được ảnh gốc và ảnh đã tăng độ tương phản ở vùng tối nhờ biến đổi log.

- 1.3. Thay đổi cường độ điểm ảnh với Log Transformation là phép biến đổi logarit để điều chỉnh cường độ của ảnh xám, giúp làm nổi bật chi tiết ở vùng tối.

  - Ý tưởng: ảnh được chuyển sang xám (convert('L')) và đưa về kiểu float -> dùng công thức log có hệ số để điều chỉnh ánh xạ giá trị cường độ đầu vào về mức cường độ mong muốn c = 128 \* log(1 + b1) / log(1 + b2) -> làm tăng độ tương phản mà không làm quá sáng các vùng đã sáng.
  - Kết quả ảnh đã được thay đổi cường độ điểm ảnh, giúp cải thiện độ rõ nét ở vùng tối.

- 1.4. Histogram equalization

  - Histogram(lược đồ xám) là biểu đồ tần suất thống kê số lần xuất hiện các mức sáng trong ảnh. Mục đích là cải tiến độ tương phản hai vùng sáng tối của ảnh. Trong ảnh grayscale, giá trị intensive của ảnh nằm [0, L-1]. Ảnh càng tối khi giá trị đi vào vùng low gray, ảnh càng sáng khi giá trị đi vào vùng high gray. Hàm sử dụng xác suất, CDF(Cumulative Distribution) để tính.
  - Giải thích main code:
    - b1 = im1.flatten() #làm phẳng mảng im1 thành mảng 1 chiều.
    - hist, bins = np.histogram(im1, 256, [0, 255]): hist là 1 mảng chứa số lượng pixel tại mỗi mức cường độ (0-255), bins là ranh giới của các mức cường độ. Để đếm xem có bao nhiêu pixel có giá trị 0, 1, ..., 255.
    - dùng các công thức cdf (công thức chuẩn hóa cdf): CDFnew(i) = (CDF(i) - CDFmin / CDFmax - CDFmin) \* 255.
    - im2 = cdf[b1] #ánh xạ các giá trị pixel gốc sang các giá trị pixel mới đã được cân bằng.
    - im3 = np.reshape(im2, im1.shape) #định hình lại mảng im2 (1 chiều) về lại hình dạng ban đầu của ảnh (im1.shape).
    - plt.imshow(im4, cmap='gray') #hiển thị ảnh đã xử lý.
  - Kết quả ảnh được cân bằng lại độ sáng tối, các chi tiết dễ nhìn rõ hơn (độ tương phản và sống động tốt hơn).

- 1.5. Thay đổi ảnh với contrast stretching

  - Contrast stretching là 1 phương pháp đơn giản để cải thiện độ tương phản của ảnh bằng cách trải đều dải cường độ pixel hiện có ra toàn bộ dải cường độ có thể(0-255 cho ảnh 8bit).
  - Main code:
    - b = im1.max(), a = im1.min(): tìm giá trị cường độ pixel lớn nhất và nhỏ nhất trong mảng ảnh im1.
    - im2 = 255 \* (c - a) / (b - a): (c-a) dịch chuyển các giá trị pixel sao cho giá trị nhỏ nhất hiện tại thành 0, (c-a) / (b-a) chuẩn hóa các giá trị pixel [0, 1].
    * Công thức này sẽ cho giá trị pixel a thành 0 (đen) và b thành 255 (trắng) và tất cả các giá trị ở giữa sẽ được trải đều ra dải 0-255.
  - Kết quả đoạn code trên cho ảnh đã xử lý sẽ có độ tương phản cao hơn so với ảnh gốc.
    - Các vùng tối nhất sẽ hoàn toàn đen (0), sáng nhất sẽ hoàn toàn trắng (255). Tất cả các mức ở giữa sẽ được phân bố đều ra từ 0-255.

- Các thư viện đã sử dụng trong bài tập 1 bao gồm:
  ## code
  ```py
  import cv2 #thư viện xử lý ảnh mạnh mẽ, dùng để đọc, biến đổi và lưu ảnh.
  import os #quản lý thư mục, tên file.
  ```
  1. Mỗi hàm (inverse, gamma correction, log transformation, histogram equalization, contrast stretching) đều được định nghĩa để thực hiện 1 phép biến đổi ảnh cụ thể.
  2. > Hàm inverse: lấy `255 - pixel_value` cho mỗi pixel.
     > Gamma_correction: dùng công thức <pixel_value^gamma>:
     ### code
     ```py
     table = np.array([((i / 255.0) ** invGamma) * 255 for i in range(256)]).astype("uint8")
     ```
     - Nó sẽ tạo ra 1 bảng tra cứu (LUT) để biến đổi giá trị pixel 1 cách hiệu quả.
       > Log_transformation: áp dụng <C\*log(1+pixel_value)>. Chuẩn hóa ảnh về float, áp dụng log, sau đó chuẩn hóa lại về 0-255 và chuyển về uint8.
       > Histogram_equalization:
       # code
       ```py
       if len(img.shape) == 2: #Nếu ảnh là grayscale (2 chiều)
         return cv2.equalizeHist(img) #Nếu ảnh là grayscale, áp dụng cân bằng biểu đồ tần suất trực tiếp.
       else: #Nếu ảnh là màu (3 chiều: height, weight, channel)
         ycrcb = cv2.cvtColor(img, cv2.COLOR_BGR2YCrCb) #Chuyển đổi từ BGR sang YcrCb
         ycrcb[:, :, 0] = cv2.equalizeHist(ycrcb[:, :, 0]) #Cân bằng biểu đồ tần suất cho kênh Y (kênh độ sáng)
         return cv2.cvtColor(ycrcb, cv2.COLOR_YCrCb2BGR) #Chuyển đổi ảnh trở lại từ YCrCb sang BGR
       ```
       > Contrast stretching: chuẩn hóa trực tiếp các giá trị pixel trong ảnh về dải 0-255, kéo giãn dải độ tương phản hiện có.
  # code
  ```py
  return cv2.normalize(img, None, 0, 255, cv2.NORM_MINMAX)
  ```

3. Dùng hàm methods để gán vào các hàm để chọn các phương pháp biến đổi tương ứng.
4. Đọc ảnh từ thư mục `exercise` và lưu vào thư mục `output1`
5. Nhập I, G, L, H, C để chọn phương pháp biến đổi tương ứng.
6.

# code

```py
if key in methods: #Kiểm tra xem lựa chọn của người dùng có hợp lệ không
    for file in os.listdir(folder): #duyệt qua tất cả các mục trong thư mục đầu vào.
        if file.lower().endswith(('.jpg', '.png', '.bmp')): #Kiểm tra xem mục đó có phải là một file ảnh không.
            path = os.path.join(folder, file)
            img = cv2.imread(path) #Đọc ảnh
            result = methods[key](img) #gọi hàm xử lý
            save_path = os.path.join(output_folder, f"{key}_{file}")
            cv2.imwrite(save_path, result) #lưu ảnh đã xử lý vào thư mục đầu ra với tên file mới
            #hiển thị ảnh đã biến đổi
            cv2.imshow(f"{key} - {file}", result)
            cv2.waitKey(0)
    cv2.destroyAllWindows()
else:
    print("Phím không hợp lệ.")
```
