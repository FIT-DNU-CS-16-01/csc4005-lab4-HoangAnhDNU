# CSC4005 Lab 4 Report – CRNN for UrbanSound8K

## 1. Thông tin sinh viên

- Họ tên: Nguyễn Hoàng Anh
- Mã sinh viên: 1771040002
- Lớp: KHMT 17-01
- Link GitHub repo: [csc4005-lab4-HoangAnhDNU](https://github.com/FIT-DNU-CS-16-01/csc4005-lab4-HoangAnhDNU)
- Link W&B project: https://wandb.ai/hoanganhdnu-dainam-vietnam/csc4005-lab4-urbansound8k-crnn

## 2. Mục tiêu thí nghiệm

Mục tiêu Lab 4 là xây dựng mô hình CRNN kết hợp CNN và RNN để phân loại âm thanh môi trường UrbanSound8K. Log-mel spectrogram được dùng làm input vì nó giữ cấu trúc thời gian–tần số, giúp CNN trích đặc trưng cục bộ và RNN học diễn biến theo thời gian. Khác với 1D-CNN ở Lab 3 chỉ học pattern cục bộ trên chuỗi đặc trưng, CRNN bổ sung khả năng mô hình hóa chuỗi thời gian. Đánh giá dựa trên accuracy, learning curves, confusion matrix và so sánh với Lab 3.

## 3. Cấu hình dữ liệu

| Thành phần | Giá trị |
|---|---|
| Dataset | UrbanSound8K |
| Số lớp | 10 |
| Train folds | 1–8 |
| Validation fold | 9 |
| Test fold | 10 |
| Feature | log-mel spectrogram |
| Sampling rate | 16 kHz |
| Duration | 4 giây |

## 4. Cấu hình mô hình

| Thành phần | Giá trị |
|---|---|
| Model | CRNN (crnn_small) |
| CNN blocks | (16, 32, 64) channels |
| RNN type | GRU |
| Hidden size | 96 |
| Dropout | 0.3 |
| Optimizer | AdamW |
| Learning rate | 0.001 (giảm xuống 0.0005) |
| Batch size | 32 |
| Epochs | 25 |

## 5. Kết quả huấn luyện

Điền kết quả tốt nhất từ W&B hoặc `metrics.json`.

| Run | best_val_acc | test_acc | Ghi chú |
|---|---:|---:|---|
| logmel_crnn_gru_baseline | 0.7463 | 0.7551 | Epoch 24, LR giảm tại epoch 21 |
| extension_bilstm_crnn | - | - | Chưa chạy |

## 6. Learning curves

![Learning curves](outputs/logmel_crnn_gru_baseline/curves.png)

Nhận xét:

- Mô hình không có overfitting nghiêm trọng - train và val loss đi gần nhau
- Validation loss giảm ổn định từ 1.92 → 0.93 (epoch 1-24), tăng nhẹ epoch 25
- Scheduler giảm LR từ 0.001 → 0.0005 tại epoch 21 giúp val loss giảm tiếp
- Train accuracy tăng đều từ 0.25 → 0.78, validation từ 0.33 → 0.73

## 7. Confusion matrix

![Confusion matrix](outputs/logmel_crnn_gru_baseline/confusion_matrix.png)

Nhận xét:

- Lớp phân loại tốt nhất: gun_shot (32/32, 100% recall, f1=0.98)
- Lớp dễ bị nhầm nhất: siren (44/83, 53% recall, f1=0.56) - nhầm với children_playing (30 mẫu)
- Các cặp nhầm lẫn đáng chú ý:
  - drilling ↔ jackhammer: âm thanh cơ khí có tần số tương tự
  - engine_idling ↔ jackhammer: engine_idling nhầm 20 sang jackhammer
  - children_playing ↔ dog_bark: đều có background phức tạp
- Việc nhầm hợp lý về mặt âm thanh: các âm thanh cùng loại (cơ khí, background phức tạp) khó phân biệt

## 8. So sánh với Lab 3 1D-CNN

| Tiêu chí | Lab 3 MFCC | Lab 3 log-mel | Lab 4 CRNN |
|---|---|---|---|
| Feature chính | MFCC | log-mel | log-mel |
| Kiến trúc | 1D-CNN | 1D-CNN | CRNN (CNN+GRU) |
| Test accuracy | 55.5% | 63.9% | 75.5% |
| Avg epoch time | 4.95s | ~5s | 56.50s |
| Params | 137,930 | 137,930 | 71,338 |
| Overfitting | Nghiêm trọng (train 99% vs val 58%) | Nhẹ hơn | Không overfit |
| Nhận xét | Pattern cục bộ đủ mạnh nhưng overfit | Tốt hơn MFCC +8.4% | Tốt hơn Lab 3 MFCC +20%, tốt hơn Lab 3 log-mel +11.6% |

## 9. Kết luận

CRNN đạt test accuracy 75.5% với log-mel spectrogram, kết quả ổn định và không overfit nghiêm trọng. Mô hình học được pattern cục bộ qua CNN và quan hệ thời gian qua GRU. Lớp gun_shot phân loại hoàn hảo, siren gặp khó khăn. So với 1D-CNN, CRNN có lợi thế khi âm thanh cần quan sát diễn biến theo thời gian. Nếu làm tiếp, sẽ thử BiLSTM, tăng CNN depth, hoặc data augmentation mạnh hơn để cải thiện các lớp khó.

## 10. Link minh chứng

- GitHub commit cuối: [điền sau khi push]
- W&B run baseline: https://wandb.ai/hoanganhdnu-dainam-vietnam/csc4005-lab4-urbansound8k-crnn/runs/mqskm0iz
- W&B run mở rộng: [nếu có]
