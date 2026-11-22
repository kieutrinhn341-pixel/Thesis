# Hubs and Loops: phát hiện rửa tiền bằng quy tắc đồ thị trên IBM AML HI-Small

**Tên đề tài:** Tài khoản trung tâm và vòng tiền khép kín: phát hiện rửa tiền bằng các quy tắc đồ thị có thể giải thích trên mạng giao dịch IBM HI-Small

**English title:** *Hubs and Loops: Explainable Graph Rules for Anti-Money Laundering on the IBM HI-Small Transaction Network*
Khóa luận tốt nghiệp K65 · Nguyễn Thị Kiều Trinh

Bài này dựng mạng giao dịch từ 5,08 triệu giao dịch của IBM AML HI-Small và kiểm tra ba quy tắc đồ thị, mỗi quy tắc phát biểu được trong một câu, không dùng nhãn khi tính và không dùng học máy:

1. **Hub (RQ1):** tài khoản nhận tiền từ (*fan-in*) hoặc chuyển tiền cho (*fan-out*) nhiều đối tác trong 48 giờ có phải là tài khoản rửa tiền không?
2. **Loop (RQ2):** giao dịch nằm trên một vòng tiền 3–6 tài khoản, các chặng đúng thứ tự thời gian và khép lại trong 89 giờ, có phải là rửa tiền không?
3. **Một ngân hàng (RQ3):** một ngân hàng chỉ thấy giao dịch của khách hàng mình thì thấy trọn được bao nhiêu vòng của RQ2?

Mọi quy tắc được chốt trên đoạn dev (5,54 ngày đầu) rồi chấm một lần trên đoạn test (4,46 ngày sau). Mốc so sánh là chọn ngẫu nhiên và quy tắc **ACH-only** (gắn cờ mọi giao dịch ACH). Trên test, ACH-only tạo 262.486 cảnh báo, chỉ 0,741% là rửa tiền.

## 1. Kết quả

Số liệu trên đoạn test, chép từ cột `display` của [`reports/results_final.csv`](reports/results_final.csv), nơi chứa toàn bộ số liệu của khóa luận.

| | Quy tắc | Kết quả chính | So với mốc | Kết luận |
|---|---|---|---|---|
| **RQ1 · Hub** | Xếp hạng tài khoản theo số đối tác phân biệt trong 48h (ACH), lấy top 0,1% | Fan-out: **31,50%** tài khoản cảnh báo là tài khoản rửa tiền [Wilson 24,73–39,16]. Fan-in: 6,41% | Ngẫu nhiên 1,77% (fan-out gấp 17,8 lần). Bậc thông thường (B3) cho fan-out 33,32%, fan-in 6,41% | **Điều chỉnh.** Hub có tín hiệu, nhưng cửa sổ thời gian không thêm gì so với bậc: thắng ngẫu nhiên ở 3/3 mức ngân sách, thắng B3 ở 0/3 |
| **RQ2 · Loop** | Gắn cờ giao dịch trên vòng ACH 3–6, đúng thứ tự thời gian, khép trong δ = 89h | 122 cảnh báo, 67 đúng: precision **54,9%** [46,1–63,5], recall 2,98% | ACH-only 0,74% (gấp 74,1 lần); vòng tĩnh 43,8%; cùng SCC 10,4%; mẫu ACH ngẫu nhiên cùng cỡ 4/122 | **Đào sâu.** Chính xác nhưng hẹp. Thắng mọi mốc trên cả dev (43,3%) và test |
| **RQ3 · Một ngân hàng** | Ngân hàng thấy một chặng nếu giữ tài khoản gửi hoặc nhận | Một ngân hàng thấy trọn **5/40** vòng (12%); với vòng toàn rửa tiền: 3/19 (16%) | Độ phủ tốt nhất của một ngân hàng: trung vị 67% số chặng; 35/40 vòng cần ghép ít nhất 2 ngân hàng | **Sơ bộ.** Tín hiệu vòng chủ yếu chỉ lộ ra khi hai ngân hàng trở lên ghép dữ liệu |

Năm điều rút ra:

- **Fan-out mạnh hơn fan-in.** 19/20 tài khoản fan-out đầu bảng dính rửa tiền; chỉ 1/20 tài khoản fan-in đầu bảng như vậy. Hub nhận tiền lớn nhất là doanh nghiệp hợp pháp.
- **Cửa sổ thời gian ≈ bậc.** Cửa sổ 48h chọn gần như đúng các tài khoản mà bậc thông thường chọn (trùng 99,4–100,0% với fan-in, 96,4–98,7% với fan-out). Tài khoản fan-in đầu bảng nhận từ mọi đối tác trong trung vị 0,45 giờ (27 phút), nên cửa sổ nào cũng bằng bậc.
- **Vòng càng dài càng sạch.** Vòng 2 bước: 7,3% đúng. Vòng 3 bước: 48,5%. Vòng 4–7 bước: 77,3–100,0%.
- **Thứ tự thời gian lọc bớt nhiễu nhưng không phải tín hiệu chính.** Thêm điều kiện thời gian nâng precision từ 43,8% lên 54,9%. Tuy vậy, số vòng đúng thứ tự không cao hơn mức xáo trộn thời gian ngẫu nhiên (40 so với trung bình 35,54; p = 0,188).
- **Đơn vị độc lập là ring, không phải giao dịch.** 19 vòng toàn rửa tiền trên test đến từ 4 ring, trong đó hai ring chiếm 17 vòng. Precision 54,9% vì vậy phải đọc cùng con số "4 ring".

## 2. Bước làm

| Bước | Notebook | Làm gì | Ghi ra |
|---|---|---|---|
| 1 | `01_eda.ipynb` | Đọc CSV, quy đổi USD, bỏ self-loop và dòng trùng, chia dev / test / đuôi theo thời gian, tính baseline ngẫu nhiên và ACH-only | `reports/tables/eda_*`, `analysis_period.json`, `fx_rates.csv`, `data/processed/hi_small_graph_transactions.parquet` |
| 2 | `02_graph.ipynb` | Dựng đồ thị sự kiện **T** và đồ thị quan hệ **R**, đồ thị con ACH; tính bậc, SCC; hình đọc đồ thị | `reports/tables/graph_*`, `reports/figures/graph_*`, `data/processed/graph/` |
| 3 | `03_rq1_hubs.ipynb` | Đếm fan-in / fan-out trong cửa sổ trượt 24/48/96h; chốt biến thể trên dev (hash `369a3997bb05980a`); chấm trên test với B1 (ngẫu nhiên) và B3 (bậc) | `reports/rq1/` |
| 4 | `04_rq2_loops.ipynb` | Tìm SCC trên R, liệt kê vòng, kiểm tra thứ tự thời gian trên T; chọn δ trên dev; chấm trên test; xáo trộn thời gian 100 lần; gom ring; typology | `reports/rq2/` |
| 5 | `05_rq3_bank_view.ipynb` | Với từng vòng của RQ2: số ngân hàng, ngân hàng nào thấy trọn, độ phủ tốt nhất, số ngân hàng tối thiểu cần ghép | `reports/rq3/` |
| 6 | `06_results_final.ipynb` | Không chạy thí nghiệm. Gom mọi con số vào một file, kiểm tra chéo giữa các notebook, lưu SHA-256 của file nguồn | `reports/results_final.csv`, `results_final_sources.csv` |

Ý tưởng thiết kế: **tìm cấu trúc trong R, kiểm tra thời gian trong T.**

## 3. Kết quả chi tiết

### 3.1. RQ1 · Hub (notebook 03)

**Chọn trên dev.** Có 8 biến thể: 2 hướng × cửa sổ 24/48h × điểm thô hoặc chuẩn hóa theo nhóm đối chiếu. Biến thể có AP cao nhất trên dev là `fan_in_48h_raw`, chốt với hash `369a3997bb05980a` trước khi tính số trên test ([`rq1_frozen_choice.json`](reports/rq1/rq1_frozen_choice.json)). Biến thể tốt nhất mỗi hướng: `fan_in_48h_raw` và `fan_out_48h_raw`.

**Precision cấp tài khoản trên test.** Có 156.284 tài khoản ứng viên, tỷ lệ dương 1,77%. Điểm là số nguyên nhỏ nên ngưỡng `k` thường cắt ngang nhóm đồng hạng (fan-out 48h, top 1%: 1.110 trong 1.563 cảnh báo được bốc thăm từ nhóm 1.518 tài khoản đồng hạng). Vì vậy bảng dùng **precision kỳ vọng khi bốc thăm ngẫu nhiên trong nhóm đồng hạng**, tính chính xác, không phụ thuộc seed.

| Điểm xếp hạng | top 0,1% (k = 156) | top 0,5% (k = 781) | top 1% (k = 1.563) |
|---|---:|---:|---:|
| Fan-in trong 48h | 6,41% | 8,39% | 8,59% |
| B3 fan-in (bậc vào cả đoạn) | 6,41% | 8,58% | 8,78% |
| Fan-out trong 24h | 23,99% | 9,84% | 7,29% |
| Fan-out trong 48h | **31,50%** [24,73–39,16] | 11,39% | 8,13% |
| Fan-out trong 96h (độ nhạy) | 33,32% | 12,01% | 8,62% |
| B3 fan-out (bậc ra cả đoạn) | **33,32%** [26,40–41,04] | 12,06% | 8,69% |
| Chọn ngẫu nhiên (B1) | 1,77% | 1,77% | 1,77% |

Nguồn: [`rq1_account_metrics.csv`](reports/rq1/rq1_account_metrics.csv), hình [`rq1_account_precision_test.png`](reports/rq1/rq1_account_precision_test.png).

- **Kết luận: Điều chỉnh** cho cả hai hướng (thắng B1 ở 3/3 mức, thắng B3 ở 0/3).
- **Fan-out mạnh hơn fan-in ở đầu danh sách.** Biến thể chính được chọn theo AP (toàn bảng xếp hạng) nên là fan-in. Kết quả fan-out được báo cáo nguyên trạng, không chọn lại theo test.
- **Cửa sổ ngắn chỉ cắt bớt thông tin.** Fan-out top 0,1%: 24h 23,99% < 48h 31,50% < 96h 33,32% = B3. Trên dev cũng vậy: 48h 15,43%, B3 17,55%.
- **Vì sao cửa sổ ≈ bậc** ([`rq1_window_vs_degree.csv`](reports/rq1/rq1_window_vs_degree.csv)).
  - Tài khoản fan-in đầu bảng nhận từ mọi đối tác trong một đợt kéo dài trung vị 0,45 giờ (27 phút); cửa sổ 48h bằng bậc ở 97,2% số tài khoản này.
  - Tài khoản fan-out đầu bảng hoạt động trải trung vị 83,03 giờ, nhưng cửa sổ 48h vẫn chứa đủ đối tác ở 94,6% số tài khoản này.
  - Đoạn test chỉ dài 4,46 ngày, nên "bậc trong đoạn" vốn đã là một thước đo trong cửa sổ ngắn. Đây là đặc điểm của dữ liệu, không phải lỗi phương pháp.
- **Hai loại hub** ([`rq1_alert_profile_test.csv`](reports/rq1/rq1_alert_profile_test.csv)).
  - 20 cảnh báo fan-in đầu bảng (10 công ty hợp danh, 9 công ty, 1 hộ kinh doanh) nhận từ 73–88 người gửi, chỉ 1/20 dính rửa tiền.
  - 20 cảnh báo fan-out đầu bảng chuyển cho 7–14 người nhận trong 48h, 19/20 dính rửa tiền.
  - Loại chủ thể không phân biệt được: tỷ lệ dương của ba loại chính (công ty, công ty hợp danh, hộ kinh doanh) trong top 1% nằm trong khoảng 6,8–8,7%.

Phụ lục, không phải kết quả chính:
- **Cấp giao dịch** ([`rq1_transaction_metrics.csv`](reports/rq1/rq1_transaction_metrics.csv)). Fan-in đầu bảng gắn cờ toàn giao dịch thường (top 0,1%: 0,11%). Fan-out đầu bảng, nếu chỉ gắn cờ giao dịch trong cửa sổ đỉnh 48h, đạt 24,56% trên 1.185 cảnh báo. Phép đo cho fan-out được thêm sau khi chốt, chỉ để tham khảo.
- Biến thể chuẩn hóa theo nhóm đối chiếu (Bolton & Hand, 2001) thua bản thô ở cả hai hướng trên dev.
- Chỉ số bùng nổ Aₙ (Kim & Jo, 2016) chỉ tính được cho 2,6–6,5% ứng viên. Dùng nó để phá đồng hạng không cải thiện kết quả.

### 3.2. RQ2 · Loop (notebook 04)

**Quy tắc chính.** Gắn cờ giao dịch nằm trên một vòng **ACH dài 3–6** có các chặng theo thứ tự thời gian tăng dần và khép lại trong **δ = 89 giờ**. δ là phân vị 95% thời lượng của 16 attempt CYCLE khép trong dev (lấy từ `Patterns.txt`, chỉ dùng dev).

| Quy tắc (test) | Cảnh báo | Đúng | Precision [Wilson 95%] | Recall |
|---|---:|---:|---:|---:|
| **Loop ACH 3–6, đúng thứ tự, ≤ δ (chính)** | **122** | **67** | **54,9% [46,1–63,5]** | 2,98% |
| Như trên, không giới hạn δ | 143 | 79 | 55,2% [47,1–63,2] | 3,51% |
| Như trên, trên đồ thị mọi định dạng | 132 | 69 | 52,3% [43,8–60,6] | 3,07% |
| Vòng tĩnh 3–6 (không xét thời gian) | 210 | 92 | 43,8% [37,3–50,6] | 4,09% |
| Hai đầu giao dịch cùng một SCC | 1.826 | 189 | 10,4% [9,0–11,8] | 8,40% |
| Mẫu ACH ngẫu nhiên cùng 122 cảnh báo | 122 | 4 | 3,3% [1,3–8,1] | 0,18% |
| ACH-only | 262.486 | 1.946 | 0,74% [0,71–0,77] | 86,49% |

Nguồn: [`rq2_transaction_metrics.csv`](reports/rq2/rq2_transaction_metrics.csv).

- Precision gấp 74,1 lần ACH-only và 498 lần chọn ngẫu nhiên. Để có cùng 67 giao dịch đúng, ACH-only cần 9.037 cảnh báo.
- Trên dev: 157 cảnh báo, 43,3% [35,8–51,1]. Bỏ giới hạn δ thì dev giảm còn 36,6% nhưng test gần như không đổi (55,2%), nên kết luận không phụ thuộc δ.
- Điều kiện thời gian "<" và "≤" cho cùng kết quả (122/67): giao dịch cùng phút không ảnh hưởng.
- Độ dài khác: vòng dài 2 cho 7,3% (1.694 cảnh báo); vòng dài 7–12 cho 7/7 đúng.

**Bảng độ dài vòng** (test, ACH, đúng thứ tự, ≤ δ; [`rq2_loop_lengths.csv`](reports/rq2/rq2_loop_lengths.csv)). *Toàn rửa tiền* nghĩa là mọi giao dịch trong dãy chứng minh của vòng đều có nhãn 1.

| Độ dài | Vòng | Toàn rửa tiền | Không rửa tiền | Giao dịch bị gắn cờ | Precision |
|---:|---:|---:|---:|---:|---:|
| 2 (A ⇄ B) | 776 | 14 | 737 | 1.694 | 7,3% |
| 3 | 27 | 9 | 16 | 97 | 48,5% |
| 4 | 7 | 6 | 1 | 33 | 87,9% |
| 5 | 4 | 2 | 0 | 22 | 77,3% |
| 6 | 2 | 2 | 0 | 8 | 100,0% |
| 7 | 1 | 1 | 0 | 7 | 100,0% |

Vòng 2 bước (hoàn tiền, đối tác thương mại) rất phổ biến và phần lớn là giao dịch thường. Vòng càng dài càng hiếm và tín hiệu càng sạch.

**Xáo trộn thời gian** ([`rq2_time_shuffle_null_summary.csv`](reports/rq2/rq2_time_shuffle_null_summary.csv), hình `rq2_time_shuffle_null.png`).
- Cách làm: giữ nguyên mọi cạnh ACH trong test, xáo trộn thời điểm giữa các giao dịch 100 lần (seed 42), rồi kiểm tra lại thứ tự thời gian trên 81 vòng tĩnh 3–6.
- Test: quan sát 40 vòng đúng thứ tự; khi xáo trộn, trung bình 35,54 (p95 = 45), p = 0,188.
- Dev: quan sát 48, trung bình khi xáo trộn 40,72, p = 0,079.
- Ý nghĩa: bản thân vòng khép kín là tín hiệu chính. Điều kiện thời gian lọc bớt vòng thường nhiều hơn vòng rửa tiền.

**Ring** ([`rq2_ring_summary.csv`](reports/rq2/rq2_ring_summary.csv)). Ring là nhóm tài khoản nối với nhau qua các vòng bị gắn cờ (union-find).
- Trên test, 40 vòng 3–6 thuộc 20 ring.
- 19 vòng toàn rửa tiền đến từ 4 ring; hai ring lớn nhất chiếm 17/19.
- Ring lớn nhất gồm 9 vòng chồng lấn trên 8 tài khoản, cả 9 vòng đều toàn rửa tiền: đây là một nhóm rửa tiền duy nhất.

**Theo kiểu rửa tiền** ([`rq2_typology.csv`](reports/rq2/rq2_typology.csv)). 67 giao dịch đúng gồm: 18 CYCLE, 17 GATHER-SCATTER, 6 FAN-OUT, 6 RANDOM, 5 FAN-IN, 5 STACK, 5 SCATTER-GATHER, 3 BIPARTITE và 2 không thuộc attempt nào. Các nhóm rửa tiền tạo vòng trong nhiều kịch bản, không chỉ kịch bản CYCLE.

**Theo attempt.** Test có 25 attempt CYCLE. Trong đó 5 nằm trong phạm vi quy tắc (dài 3–6, toàn ACH, đúng thứ tự, nằm trọn trong test), và quy tắc tìm lại trọn 3/5.

**Ba vòng minh họa** (`rq2_loop_examples.png`, chọn theo quy tắc cố định):
- Vòng 56 (kịch bản CYCLE): 3 tài khoản, 3 ngân hàng, 3 loại tiền khác nhau, khép sau 35,6 giờ, giữ lại 85% giá trị.
- Vòng 742 (ring lớn nhất): 5 tài khoản, 3 ngân hàng, 1 loại tiền, khép sau 71,3 giờ, giữ lại 38% giá trị.
- Vòng 747 (không rửa tiền): 3 tài khoản, 3 ngân hàng, 1 loại tiền, 79,1 giờ, giữ lại 11%. Số tiền giữa các chặng không khớp, giống thương mại thông thường tình cờ tạo vòng.

*Giá trị giữ lại* (số tiền USD nhỏ nhất ÷ lớn nhất dọc vòng) chỉ được mô tả trên dev. Nó không phân biệt rõ hai nhóm (trung vị 39,5% ở 18 vòng toàn rửa tiền so với 30,7% ở 28 vòng không rửa tiền), nên không dùng làm quy tắc.

**Độ đúng của thuật toán.** Cách liệt kê vòng và kiểm tra thời gian đã được so với vét cạn trên 150 đồ thị ngẫu nhiên nhỏ (có cạnh song song, cùng phút, mọi cách xoay vòng). Không lượt chạy nào chạm giới hạn thời gian hay số vòng ([`rq2_run_log.csv`](reports/rq2/rq2_run_log.csv)).

### 3.3. RQ3 · Một ngân hàng (notebook 05, sơ bộ)

**Định nghĩa.** Mỗi vòng 3–6 đúng thứ tự của RQ2 được xét qua dãy giao dịch chứng minh (một giao dịch mỗi chặng).
- Một ngân hàng **thấy** một chặng nếu giữ tài khoản gửi hoặc tài khoản nhận của chặng đó; **thấy trọn** vòng nếu thấy mọi chặng.
- *Độ phủ tốt nhất* = tỷ lệ chặng mà ngân hàng thấy nhiều nhất nhìn thấy.
- *Số ngân hàng tối thiểu* = số ngân hàng ít nhất cần ghép dữ liệu để thấy mọi chặng.

| ACH ([`rq3_summary.csv`](reports/rq3/rq3_summary.csv)) | dev | test |
|---|---:|---:|
| Mọi vòng một ngân hàng thấy trọn | 4/48 (8%) | 5/40 (12%) |
| Vòng toàn rửa tiền một ngân hàng thấy trọn | 1/18 (6%) | 3/19 (16%) |
| Vòng không rửa tiền một ngân hàng thấy trọn | 2/28 (7%) | 1/17 (6%) |
| Độ phủ tốt nhất (trung vị) | 67% | 67% |
| Vòng cần ít nhất 2 ngân hàng ghép dữ liệu | 44/48 | 35/40 |
| Số ngân hàng mỗi vòng (nhỏ nhất – trung vị – lớn nhất) | 2 – 3 – 6 | 2 – 3 – 5 |

- Một ngân hàng thường chỉ thấy khoảng 2/3 số chặng: với mỗi khách hàng của mình trên vòng, nó thấy chặng tiền vào và chặng tiền ra, nhưng không thấy phần còn lại. Trên test, 26/40 vòng đi qua đúng 3 ngân hàng, và gần như mọi vòng chỉ cần 2 ngân hàng ghép dữ liệu là thấy trọn.
- **Theo ring** ([`rq3_rings.csv`](reports/rq3/rq3_rings.csv)): 2/4 ring rửa tiền trên test có ít nhất một vòng mà một ngân hàng thấy trọn.
- **Những vòng một ngân hàng thấy trọn** ([`rq3_single_bank_loops.csv`](reports/rq3/rq3_single_bank_loops.csv)) không vòng nào nằm gọn trong một ngân hàng; ngân hàng đó giữ ít nhất một đầu của mọi chặng. Ngân hàng 148350 xuất hiện ở 4 vòng như vậy (dev và test, cùng một ring).
- **Độ vững:** trên đồ thị mọi định dạng, kết quả gần như không đổi (test: 3/19 vòng toàn rửa tiền; [`rq3_summary_all_formats.csv`](reports/rq3/rq3_summary_all_formats.csv)).

Số vòng và số ring nhỏ, nên đây là kết quả sơ bộ. Đường cong chia sẻ dữ liệu đầy đủ giữa nhiều ngân hàng nằm ngoài phạm vi.

## 4. Dữ liệu

Nguồn: [IBM Transactions for Anti Money Laundering (AML)](https://www.kaggle.com/datasets/ealtman2019/ibm-transactions-for-anti-money-laundering-aml) (Altman et al., 2023). Đây là dữ liệu **tổng hợp** mô phỏng ngân hàng, cá nhân và doanh nghiệp giao dịch với nhau, trong đó một số tác nhân rửa tiền; không có dữ liệu người thật. Bản dùng: **HI-Small** (tỷ lệ rửa tiền cao hơn, cỡ nhỏ nhất).

| File | Nội dung | Dùng ở |
|---|---|---|
| `HI-Small_Trans.csv` | 5.078.345 giao dịch, 11 cột, thời điểm chính xác tới phút | Mọi notebook |
| `HI-Small_Patterns.txt` | 370 lần rửa tiền (*attempt*) theo 8 kiểu, gồm 3.209 giao dịch | Mô tả (01), chọn δ trên dev và chấm theo attempt (04) |
| `HI-Small_accounts.csv` | Loại chủ thể của tài khoản | Nhóm đối chiếu và hồ sơ cảnh báo (03), bảng top hub (02) |

### 4.1. Quy mô và nhãn

| Chỉ số | Giá trị |
|---|---:|
| Giao dịch | 5.078.345 |
| Giao dịch rửa tiền | 5.177 (0,102%, khoảng 1/981) |
| Tài khoản (mã ngân hàng dạng số + số tài khoản) | 515.088 |
| Ngân hàng (sau khi gộp số 0 đầu, `010` = `10`) | 30.470 |
| Thời gian | 01/09/2022 00:00 → 18/09/2022 16:18 (17,68 ngày) |

Với tỷ lệ dương thấp như vậy, đoán mọi giao dịch là hợp lệ vẫn có accuracy gần tuyệt đối, nên bài đánh giá bằng precision, recall, lift và AP (mục 5.2).

### 4.2. Theo định dạng thanh toán

| Định dạng | Giao dịch | Rửa tiền | Tỷ lệ |
|---|---:|---:|---:|
| ACH | 600.797 | 4.483 | 0,746% |
| Bitcoin | 146.091 | 56 | 0,038% |
| Cash | 490.891 | 108 | 0,022% |
| Cheque | 1.864.331 | 324 | 0,017% |
| Credit Card | 1.323.324 | 206 | 0,016% |
| Reinvestment | 481.056 | 0 | 0,000% |
| Wire | 171.855 | 0 | 0,000% |

ACH chiếm **11,8%** số giao dịch nhưng chứa **86,6%** giao dịch rửa tiền. Vì vậy ACH-only là baseline cấp giao dịch, và đồ thị con ACH là phạm vi chính của RQ1–RQ2.

![Tỷ lệ rửa tiền theo định dạng và tiền tệ](reports/figures/eda_rate_by_format_currency.png)

### 4.3. Số tiền và tiền tệ

| Nhóm (USD) | Q1 | Trung vị | Q3 |
|---|---:|---:|---:|
| Hợp lệ | 158 | 891 | 5.274 |
| Rửa tiền | 2.161 | 5.879 | 13.759 |

- Tỷ giá được ước lượng từ chính dữ liệu: trung vị `amt_received / amt_paid` trên các giao dịch khác tiền tệ có số tiền trả từ 100 trở lên, rồi nối các cặp về USD ([`fx_rates.csv`](reports/tables/fx_rates.csv)).
- Không giao dịch rửa tiền nào đổi tiền tệ trong cùng một giao dịch (hợp lệ: 1,42%). Nhưng vòng rửa tiền thường đổi tiền tệ **giữa các chặng**, nên so số tiền dọc vòng phải quy USD.

![Phân phối số tiền USD](reports/figures/eda_amount_distribution.png)

### 4.4. Tám kiểu rửa tiền trong `Patterns.txt`

| Kiểu | Mô tả | Attempt | Giao dịch |
|---|---|---:|---:|
| GATHER-SCATTER | Gom từ nhiều nguồn về một tài khoản, rồi phân tán | 51 | 716 |
| SCATTER-GATHER | Chia qua nhiều trung gian, rồi gom về một đích | 44 | 626 |
| STACK | Nhiều lớp chuyển tiếp nối tiếp | 43 | 466 |
| FAN-OUT | Một tài khoản chuyển cho nhiều tài khoản | 48 | 342 |
| FAN-IN | Nhiều tài khoản chuyển về một tài khoản | 40 | 318 |
| CYCLE | Tiền đi qua chuỗi tài khoản và quay về | 54 | 287 |
| BIPARTITE | Hai nhóm chuyển qua lại | 49 | 263 |
| RANDOM | Chuỗi nhiều bước qua trung gian ngẫu nhiên | 41 | 191 |
| **Tổng** | | **370** | **3.209** |

Các giao dịch trong `Patterns.txt` chỉ chiếm **62,0%** số giao dịch rửa tiền; 38% còn lại mang nhãn 1 mà không thuộc attempt nào. 369/370 attempt chỉ gồm giao dịch ACH.

### 4.5. Làm sạch và chia đoạn

| Bước | Loại bỏ | Còn lại |
|---|---:|---:|
| CSV gốc | — | 5.078.345 |
| Bỏ self-loop (gửi và nhận cùng tài khoản) | 591.212 | 4.487.133 |
| Bỏ dòng trùng hoàn toàn | 9 | **4.487.124** |

Các dòng bị loại vẫn nằm trong **mẫu số** khi tính recall.

| Đoạn | Thời gian | Giao dịch | Rửa tiền | Tỷ lệ | Vai trò |
|---|---|---:|---:|---:|---|
| **dev** | 01/09 00:00 → 06/09 13:00 (5,54 ngày) | 3.035.199 | 2.272 | 0,075% | Chọn biến thể, chọn δ |
| **test** | 06/09 13:00 → 11/09 00:00 (4,46 ngày) | 2.042.038 | 2.250 | 0,110% | Chấm một lần, sau khi đã chốt |
| **đuôi** | 11/09 → 18/09 16:18 | 1.108 | 655 | 59,1% | Loại khỏi đánh giá chính |

- **Đuôi** bắt đầu từ ngày mà ngày đó và mọi ngày sau đều có dưới 5% trung vị số giao dịch/ngày. Đây là phần mô phỏng chạy nốt các pattern sau khi dữ liệu nền đã dừng: chỉ 0,022% số giao dịch nhưng chứa 12,7% số giao dịch rửa tiền, nên giữ lại sẽ làm kết quả đẹp giả.
- **Dev** là 60% số giao dịch đầu của kỳ chính, làm tròn xuống đầu giờ; **test** là phần còn lại. Chia theo thời gian để mô phỏng việc đặt quy tắc trên quá khứ rồi áp cho tương lai, và không xé một vòng ra hai tập.
- Các mốc lưu trong `reports/tables/analysis_period.json`; mọi notebook sau đọc từ file này.

![Số giao dịch và tỷ lệ rửa tiền theo thời gian](reports/figures/eda_activity_over_time.png)

### 4.6. Baseline cấp giao dịch

| Tập | Baseline | Cảnh báo | Đúng | Precision | Recall | Lift |
|---|---|---:|---:|---:|---:|---:|
| Cả file | Ngẫu nhiên (kỳ vọng) | — | — | 0,102% | — | 1,0× |
| Cả file | ACH-only | 600.797 | 4.483 | 0,746% | 86,6% | 7,32× |
| dev | ACH-only | 337.203 | 1.882 | 0,558% | 82,8% | 7,46× |
| **test** | Ngẫu nhiên (kỳ vọng) | — | — | 0,110% | — | 1,0× |
| **test** | **ACH-only** | **262.486** | **1.946** | **0,741%** | **86,5%** | **6,73×** |

Nguồn: [`eda_transaction_baselines.csv`](reports/tables/eda_transaction_baselines.csv). Để so ở cùng ngân sách, RQ1 và RQ2 còn lấy thêm một mẫu ACH ngẫu nhiên có đúng bằng số cảnh báo của quy tắc.

## 5. Phương pháp

### 5.1. Đồ thị

| Dạng | Là gì | Số lượng |
|---|---|---:|
| Giả đồ thị (*pseudograph*) | File CSV thô, có self-loop | 5.078.345 giao dịch |
| Đa đồ thị (*multigraph*) = **đồ thị sự kiện T** | Bỏ self-loop và dòng trùng; mỗi giao dịch một cạnh | 4.487.124 cạnh, 422.734 tài khoản |
| Đồ thị đơn (*simple graph*) = **đồ thị quan hệ R** | Mỗi cặp gửi → nhận một cạnh | 647.939 cạnh (ACH: 208.522) |

$$T = (V, E, s, d, t, a), \qquad R = (V, E_R),\; E_R = \{(s(e), d(e)) : e \in E\}$$

- $V$: tài khoản (một tài khoản tại một ngân hàng).
- $E$: giao dịch sau làm sạch, mỗi giao dịch một cạnh, kể cả khi cùng cặp và cùng phút.
- $s(e)$, $d(e)$: tài khoản gửi và nhận.
- $t(e)$: thời điểm (phút).
- $a(e)$: số tiền. Đây là **thuộc tính** của cạnh, không phải trọng số khoảng cách.

| Ký hiệu | Tên trong code | Trả lời câu hỏi | Dùng để |
|---|---|---|---|
| **T** (*event graph*) | `graph_edges` | Ai trả ai, **lúc nào**, bao nhiêu | Kiểm tra thứ tự thời gian của vòng, đếm cửa sổ của hub |
| **R** (*relationship graph*) | `directed_pairs` | Ai **từng** trả cho ai | Bậc, SCC, liệt kê vòng tĩnh |

| Chỉ số | Mọi định dạng | Chỉ ACH |
|---|---:|---:|
| Tài khoản | 422.734 | 238.922 |
| Giao dịch (cạnh của T) | 4.487.124 | 529.446 |
| Cặp gửi → nhận (cạnh của R) | 647.939 | 208.522 |
| Giao dịch lặp lại một cặp đã có | 85,6% | 60,6% |
| Ngân hàng | 23.378 | 16.280 |
| Bậc vào / ra lớn nhất | 545 / 14.230 | 93 / 17 |
| SCC lớn nhất | **17.075** | **25** (dev 10, test 12) |
| Tài khoản trong SCC ≥ 2 | 23.247 | 6.513 |

Nguồn: [`graph_summary.csv`](reports/tables/graph_summary.csv), [`graph_degree_summary.csv`](reports/tables/graph_degree_summary.csv), [`graph_components.csv`](reports/tables/graph_components.csv).

- Bậc có đuôi dày: phần lớn tài khoản chỉ có 1 đối tác, một số ít có hàng trăm đến hàng chục nghìn. Vì vậy đầu bảng xếp hạng bậc chủ yếu là doanh nghiệp lớn hợp pháp.
- Mọi chu trình có hướng nằm trọn trong một SCC. Trên mọi định dạng có một SCC khổng lồ, nên vòng ở đó phần lớn là nhiễu nền. Trong ACH, SCC lớn nhất chỉ 25 tài khoản, nên vòng hiếm và nổi bật. SCC của ACH chỉ tính trên các tài khoản có giao dịch ACH.

| RQ | Dùng đồ thị thế nào |
|---|---|
| **RQ1** | Trên T (ACH): đếm số đối tác phân biệt theo từng hướng trong cửa sổ trượt 24h, 48h (96h để kiểm tra độ nhạy). B3 là bậc trên R trong cả đoạn |
| **RQ2** | Trên R (ACH, từng đoạn): tìm SCC, liệt kê chu trình đơn (Johnson, 1975). Trên T: kiểm tra có một dãy giao dịch thật với thời điểm tăng dần dọc vòng, khép lại trong δ giờ |
| **RQ3** | Ngân hàng thấy giao dịch nếu giữ tài khoản gửi **hoặc** nhận (`bank_edge_view`). Vòng được thấy trọn khi một ngân hàng chạm vào mọi chặng |

### 5.2. Đánh giá

- **Tách nhãn.** Đồ thị và mọi quy tắc được tính khi chưa đọc cột `Is Laundering`; nhãn chỉ ghép vào ở bước chấm điểm, theo `transaction_id`.
- **Chọn trên dev, chấm trên test.** Biến thể của RQ1 và δ của RQ2 được chốt trên dev, lưu kèm hash trước khi tính số trên test.
- **Đơn vị.** RQ1 chấm theo **tài khoản**: tài khoản dương nếu gửi hoặc nhận ít nhất 1 giao dịch rửa tiền (bất kỳ định dạng nào) trong đoạn. RQ2 chấm theo **giao dịch**: mẫu số là mọi giao dịch của đoạn trong file gốc.
- **Ngân sách cảnh báo.** RQ1 báo cáo tại top 0,1%, 0,5% và 1% số ứng viên.
- **Chỉ số.**
  - Precision kèm khoảng Wilson 95% (Wilson, 1927). Một quy tắc thắng baseline khi cận dưới Wilson lớn hơn precision của baseline.
  - Recall, tính trong tập ứng viên và trên cả đoạn.
  - Lift so với baseline cùng cấp.
  - Average Precision (Saito & Rehmsmeier, 2015).
- **Đồng hạng.** Khi ngưỡng cắt qua nhóm cùng điểm, precision được tính theo kỳ vọng khi bốc thăm ngẫu nhiên trong nhóm đó.

## 6. Hạn chế

1. δ được chọn từ `Patterns.txt` (thông tin nhãn) trên dev. Ảnh hưởng nhỏ: test gần như không đổi khi bỏ δ.
2. Mỗi vòng phải nằm trọn trong một đoạn, nên vòng vắt qua mốc 06/09 13:00 không được tính.
3. Các giao dịch đúng không độc lập: 19 vòng toàn rửa tiền trên test đến từ 4 ring. Khoảng Wilson giả định độc lập nên có thể hẹp hơn thực tế; precision luôn đi kèm số ring.
4. Phép xáo trộn thời gian không bác bỏ được giả thuyết "thứ tự thời gian là tình cờ" (p = 0,188 trên test).
5. Nhãn tài khoản là "chạm ít nhất 1 giao dịch rửa tiền", nên tài khoản vô can nhận tiền một lần cũng là dương.
6. Biến thể chính của RQ1 chọn theo AP (toàn bảng xếp hạng), trong khi câu hỏi quan tâm đầu danh sách.
7. Dữ liệu tổng hợp: các tỷ lệ (ví dụ 86,6% rửa tiền đi qua ACH) phản ánh bộ mô phỏng, không suy rộng ra thực tế.


### Cấu trúc thư mục

```
Notebook/
  01_eda.ipynb             Làm sạch, chia dev/test, baseline
  02_graph.ipynb           Dựng T và R, bậc, SCC, đọc đồ thị
  03_rq1_hubs.ipynb        RQ1 · Hub
  04_rq2_loops.ipynb       RQ2 · Loop
  05_rq3_bank_view.ipynb   RQ3 · Góc nhìn một ngân hàng
  06_results_final.ipynb   Gom số liệu cuối cùng, kiểm tra chéo
reports/
  results_final.csv          Mọi số liệu dùng trong khóa luận
  results_final_sources.csv  File nguồn, số dòng, SHA-256
  figures/   eda_*.png, graph_*.png
  tables/    eda_*.csv, graph_*.csv, analysis_period.json, fx_rates.csv
  rq1/       metrics, cảnh báo, lựa chọn đã chốt, hình
  rq2/       danh sách vòng, metrics, xáo trộn thời gian, ring, hình minh họa, nhật ký chạy
  rq3/       từng vòng theo góc nhìn ngân hàng, theo ring, hình
requirements.txt
.gitignore
```

Quy ước đặt tên:
- Notebook là `NN_chủ_đề.ipynb`.
- File đầu ra là `<tiền tố>_<nội dung>`, tiền tố theo notebook: `eda_`, `graph_`, `rq1_`, `rq2_`, `rq3_`.
- Không ghi ngày hay số phiên bản vào tên file; phiên bản do git quản lý.

## 8. Tài liệu tham khảo

- Altman, E., Blanuša, J., von Niederhäusern, L., Egressy, B., Anghel, A., & Atasu, K. (2023). Realistic synthetic financial transactions for anti-money laundering models. *NeurIPS 2023, Datasets and Benchmarks Track*. arXiv:2306.16424.
- Bolton, R. J., & Hand, D. J. (2001). Unsupervised profiling methods for fraud detection. *Conference on Credit Scoring and Credit Control VII*, Edinburgh.
- Goh, K.-I., & Barabási, A.-L. (2008). Burstiness and memory in complex systems. *EPL*, 81(4), 48002.
- Johnson, D. B. (1975). Finding all the elementary circuits of a directed graph. *SIAM Journal on Computing*, 4(1), 77–84.
- Kim, E.-K., & Jo, H.-H. (2016). Measuring burstiness for finite event sequences. *Physical Review E*, 94(3), 032311.
- Kumar, R., & Calders, T. (2018). 2SCENT: An efficient algorithm to enumerate all simple temporal cycles. *Proceedings of the VLDB Endowment*, 11(11), 1441–1453.
- Needham, M., & Hodler, A. E. (2019). *Graph Algorithms: Practical Examples in Apache Spark and Neo4j*. O'Reilly.
- Saito, T., & Rehmsmeier, M. (2015). The precision-recall plot is more informative than the ROC plot when evaluating binary classifiers on imbalanced datasets. *PLOS ONE*, 10(3), e0118432.
- Starnini, M., Tsourakakis, C. E., Zamanipour, M., Panisson, A., Allasia, W., Fornasiero, M., Li Puma, L., et al. (2021). Smurf-based anti-money laundering in time-evolving transaction networks. *ECML PKDD 2021*, LNAI 12978, 171–186.
- Tarjan, R. (1972). Depth-first search and linear graph algorithms. *SIAM Journal on Computing*, 1(2), 146–160.
- Wilson, E. B. (1927). Probable inference, the law of succession, and statistical inference. *Journal of the American Statistical Association*, 22(158), 209–212.
