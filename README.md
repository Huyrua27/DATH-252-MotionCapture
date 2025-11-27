# Motion Capture Correction - Bài tập lọc dữ liệu 3D

## Giới thiệu
Project này thực hiện việc lọc và cải thiện dữ liệu 3D motion capture bằng các bộ lọc tín hiệu khác nhau.

## Yêu cầu hệ thống
- Python 3.10+
- Jupyter Notebook
- Các thư viện cần thiết (xem phần Installation)

## Cài đặt

### 1. Cài đặt các thư viện cần thiết
```bash
pip install numpy matplotlib scipy
pip install mokka  # Thư viện xử lý motion capture
```

### 2. Chuẩn bị dữ liệu
Đảm bảo cấu trúc thư mục như sau:
- `input/` - Chứa dữ liệu 3D raw
  - `footage_Day08_Mon10_Yr2025_Hr11_Min11_Sec50.json`
- `output/` - Chứa dữ liệu 2D và kết quả
  - `output_2d_Day08_Mon10_Yr2025_Hr10_Min57_Sec44.json`
  - Thư mục con `gif/` và `json/` cho output
- `src/` - Chứa source code
  - `test_motion_capture_correction_template.ipynb`

## Cấu trúc dữ liệu

### Input Files
- **2D Keypoints**: Shape `(9, 477, 33, 3)` 
  - 9 camera views
  - 477 frames
  - 33 keypoints
  - 3 giá trị (x, y, confidence)

- **3D Keypoints**: Shape `(477, 33, 4)`
  - 477 frames
  - 33 keypoints
  - 4 giá trị (x, y, z, confidence)

## Vấn đề cần giải quyết

Dữ liệu 3D hiện tại có các vấn đề sau:
1. **Knee jittering**: Keypoints đầu gối bị lệch bất thường trong 5-6 frames (ví dụ: frame 319-325)
2. **Foot-skating**: Hiện tượng chân trượt khi tiếp đất
3. **Varied speed motion**: Tốc độ chuyển động thay đổi (đá và đấm) → Lowpass filter đơn giản không hiệu quả

## Các bộ lọc được triển khai

### 1. Lowpass/Butterworth Filter 
- **Mô tả**: Bộ lọc đơn giản, loại bỏ tín hiệu tần số cao
- **Ưu điểm**: Dễ implement, phù hợp cho làm quen với signal filtering
- **Nhược điểm**: Không hiệu quả với motion có tốc độ thay đổi

### 2. Kalman Filter 
- **Mô tả**: Bộ lọc dự đoán và cập nhật dựa trên mô hình trạng thái
- **Ưu điểm**: Hiệu quả với noise, có thể track chuyển động
- **Nhược điểm**: Phức tạp, cần thiết kế và điều chỉnh nhiều

### 3. One Euro Filter 
- **Mô tả**: Bộ lọc thích ứng dựa trên tốc độ temporal
- **Ưu điểm**: Phù hợp với motion có tốc độ thay đổi, loại bỏ jitter tốt
- **Nhược điểm**: Cần điều chỉnh tham số

### 4. Foot-skating Correction 
- Phương pháp xử lý hiện tượng foot-skating
- Detect contact với mặt đất
- Lock position khi chân chạm đất

## Hướng dẫn sử dụng

### Bước 1: Load dữ liệu
```python
# Load 2D keypoints (từ thư mục output)
with open('../output/output_2d_Day08_Mon10_Yr2025_Hr10_Min57_Sec44.json','rb') as f:
    k2 = json.loads(json.load(f))
kpts2d = np.array([[cam['frames'][f]['a']['body'] for f in range(477)] for cam in k2])

# Load 3D keypoints (từ thư mục input)
with open('../input/footage_Day08_Mon10_Yr2025_Hr11_Min11_Sec50.json','rb') as f:
    k3 = json.load(f)
kpts3d = np.array(k3['frames'])
```

### Bước 2: Áp dụng filter
```python
# Thay thế phần filter của bạn ở đây
kpts3d_filtered = apply_your_filter(kpts3d)
```

### Bước 3: Visualize kết quả
```python
# Export ra GIF (lưu vào output/gif/)
to_gif(kpts3d_filtered, 30, POSE_CONNECTIONS, 
       '../output/gif/filtered_output.gif', div=2)

# Export ra JSON (lưu vào output/json/)
export_kpts_to_json(kpts3d_filtered, 
                    '../output/json/filtered_output.json')
```

### Bước 4: Kiểm tra kết quả
- Mở file GIF bằng **Screen2GIF** (Editor mode)
- Kiểm tra frames 319-325 để xem knee jittering
- Quan sát foot-skating có được cải thiện không


## Cấu trúc thư mục
```
project/
├── input/
│   ├── footage_Day08_Mon10_Yr2025_... .json          # Dữ liệu 3D raw
│   └── footage_Day08_Mon10_Yr2025_... .json          # Dữ liệu 3D khác
├── output/
│   ├── gif/
│   ├── json/
│   ├── example_gif_output_raw.gif                     # GIF demo output
│   └── output_2d_Day08_Mon10_... .json               # Dữ liệu 2D keypoints
├── src/
│   └── test_motion_capture_correction_template.ipynb # Notebook chính
└── README.md
```
## Ghi chú
- FPS của dữ liệu: **30 fps**
- Số lượng keypoints: **33** (MediaPipe Pose format)
- Vùng có vấn đề: Frame **319-325** (knee jittering)



---
**Lưu ý**: Đây là bài tập thực hành về signal processing cho motion capture data. Hãy thử nghiệm nhiều cách tiếp cận khác nhau!
