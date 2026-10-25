# Data Directory

> **Mục đích:** Lưu trữ toàn bộ dữ liệu của pipeline MSAD qua 3 tầng `raw → interim → processed`,
> cộng với nhãn sự kiện UBCKNN tách riêng để chống rò rỉ nhãn.

## Cấu trúc thực tế (đang commit trong repo)

```
data/
├── raw/                       # Dữ liệu CSV thô, KHÔNG sửa thủ công
│   ├── market/                #   16 CSV thị trường (15 mã + VNI.csv benchmark)
│   ├── control/               #   9 CSV rổ đối chứng (8 mã đơn + control_universe.csv panel gộp)
│   └── news/                  #   5 CSV tin tức thô (Baodautu, CafeF, DNSE, Fireant, Vneconomy)
├── interim/                   # Tái tạo được từ raw qua Bước 1 pipeline
│   ├── market/                #   *_clean.csv / *_synthetic_clean.csv (đã ffill, cắt đuôi, fuse VNI + sentiment)
│   ├── control/               #   *_control_clean.csv
│   ├── news/                  #   news_daily_sentiment.csv
│   └── cleaning_summary.csv   #   Tóm tắt số dòng raw → cleaned theo từng mã
├── processed/                 # Tái tạo được từ interim qua Bước 2 pipeline
│   ├── market/                #   21 đặc trưng/mã (order flow, price, market context, sentiment)
│   ├── control/               #   21 đặc trưng/mã đối chứng
│   └── feature_engineering_summary.csv
└── labels/                    # Nhãn sự kiện UBCKNN + nhãn tổng hợp nội bộ (9 CSV)
    ├── market_events.csv
    ├── market_events_expanded.csv
    ├── events_master_normalized.csv
    ├── event_daily_labels_normalized.csv
    ├── event_windows_normalized.csv
    ├── pseudo_labels.csv
    ├── synthetic_events.csv
    ├── synthetic_extreme_events.csv
    └── news_weak_events.csv
```

## Quy ước

- **Dữ liệu thô (`raw/`)**: KHÔNG chỉnh sửa thủ công. Mọi biến đổi phải đi qua pipeline `src/data/`.
  Panel gộp `control_universe.csv` KHÔNG được clean/model như một mã đơn (xem `is_pooled_control_panel()`).
- **Dữ liệu trung gian (`interim/`, `processed/`)**: Có thể tái tạo bằng cách chạy lại pipeline từ `raw/`.
- **Nhãn (`labels/`)**: Sự kiện UBCKNN + sự kiện tổng hợp nội bộ. Tách biệt khỏi features để tránh rò rỉ nhãn.

## Cách tái tạo dữ liệu từ đầu

```powershell
# Bước 1: Làm sạch — data/raw -> data/interim
uv run python src/data/run_cleaning.py --input-dir data/raw --output-dir data/interim

# Bước 2: Trích xuất 21 đặc trưng — data/interim -> data/processed
uv run python src/features/run_feature_engineering.py --input-dir data/interim --output-dir data/processed
```

> ⚠️ Ghi chú paths: defaults trong code/YAML cũ (`data/raw_data`, `data/data_cleaning`,
> `data/data_features`) **không khớp** cây `raw/interim/processed` ở trên — luôn truyền
> `--input-dir/--output-dir` tường minh. Chưa đổi defaults để giữ zero behavior change.

## Liên hệ pipeline

| Bước | Runner | Config |
|------|--------|--------|
| 1. Làm sạch | `src/data/run_cleaning.py` | `configs/cleaning.yaml` |
| 2. Đặc trưng | `src/features/run_feature_engineering.py` | `configs/features.yaml` |
| 3. Huấn luyện | `src/models/run_pipeline.py --data-dir data/processed` | `configs/training.yaml`, `configs/config.yaml` |
