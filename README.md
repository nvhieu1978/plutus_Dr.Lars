# Plutus Pioneer Program

Đây là nhánh của tôi về Chương trình tiên phong của Plutus, nơi tôi sẽ cập nhật các ví dụ về chi nhánh chính mới nhất của Plutus.

Tôi cũng sử dụng điều này để thêm nhận xét vào mã.


## Bài giảng

- Bài giảng số 1: [Video ](https://youtu.be/CJD8ctJqDw0), [Html](docs/week1.rst)

  - Welcome
  - (E)UTxO-model
  - Chạy một hợp đồng đấu giá mẫu trên một Playground
  - Bài tập về nhà

- Bài giảng số 2: [Video ](https://youtu.be/7nDGZkUIeUQ), [Html](docs/week2.rst)

  - Kích hoạt sự thay đổi.
  - Low-level, Các tập lệnh xác thực on-chain chưa được định kiểu.
  - High-level, Các tập lệnh xác thực on-chain định kiểu.

- Bài giảng số 3: [Video ](https://youtu.be/WG3uw-TkW2k), [Html](docs/week3.rst)

  - Kịch bản context.
  - Xử lý thời gian (Time handling).
  - Hợp đồng được tham số hóa (Parameterized contracts).

-  Bài giảng số 4: [Video](https://youtu.be/6Reuh0xZDjY), [Html](docs/week4.rst)

  - Monads
  - Đơn nguyên `EmulatorTrace`.
  - Đơn nguyên `Contract`.

-  Bài giảng số 5: [Video](https://youtu.be/6VbhY162GQA), [Html](docs/week5.rst)

  - Values.
  - Mã thông báo gốc và chính sách đúc tiền
  - NFT's.

-  Bài giảng số 6: [Video](https://youtu.be/wY7R-PJn66g), [Html](docs/week6.rst)

  - Oracles.
  - Sử dụng PAB.

-  Bài giảng số 7: [Video](https://youtu.be/oJupInqvJUI), [Html](docs/week7.rst)

  - Commit schemes.
  - Máy trạng thái (State machines).

-  Bài giảng số 8: [Video](https://youtu.be/JMRwkMgaBOg), [Html](docs/week8.rst)

  - Ví dụ khác về state machine: token sale.
  - Kiểm tra tự động sử dụng dấu vết giả lập (emulator traces).
  - Chuyển tiếp (Interlude): optics.
  - Kiểm tra tài sản với `QuickCheck`.
  - Thử nghiệm hợp đồng Plutus với thử nghiệm dựa trên tài sản.

-  Bài giảng số 9: [Video](https://youtu.be/-RpCqHuxfQQ), [Html](docs/week9.rst)

  - Tổng quan về Marlowe.  [Site](https://github.com/input-output-hk/plutus-pioneer-program/blob/main/Marlowe_Plutus_Pioneers_June_2021.pdf)
  - Marlowe trong Plutus.
  - Giới thiệu Marlowe Playground.

-  Bài giảng số 10: [Video](https://youtu.be/Dg36h9YPMz4), [Html](docs/week10.rst)

  - Giới thiệu Uniswap.
  - Triển khai Uniswap trong Plutus.
  - Triển khai Uniswap với PAB.
  - Demo.
  - Sử dụng `curl` để tương tác với PAB.

## Các Code ví dụ 

- Lecture #1:  [English Auction](code/week01)
- Lecture #2:  [Simple Validation](code/week02)
- Lecture #3:  [Validation Context & Parameterized Contracts](code/week03)
- Lecture #4:  [Monads, `EmulatorTrace` & `Contract`](code/yweek04)
- Lecture #5:  [Minting Policies](code/week05)
- Lecture #6:  [Oracles](code/week06)
- Lecture #7:  [State Machines](code/week07)
- Lecture #8:  [Testing](code/week08)
- Lecture #9:  [Marlowe](code/week09)
- Lecture #10: [Uniswap](code/week10)


## Một số mô-đun Plutus

- [`Language.Marlowe.Semantics`](https://github.com/input-output-hk/plutus/blob/master/marlowe/src/Language/Marlowe/Semantics.hs), chứa các kiểu và ngữ nghĩa Marlowe.
- [`Plutus.Contract.StateMachine`](https://github.com/input-output-hk/plutus/blob/master/plutus-contract/src/Plutus/Contract/StateMachine.hs), chứa các loại và chức năng để sử dụng máy trạng thái.
- [`Plutus.Contract.Test`](https://github.com/input-output-hk/plutus/blob/master/plutus-contract/src/Plutus/Contract/Test.hs), cung cấp nhiều cách khác nhau để viết các bài kiểm tra cho các hợp đồng Plutus.
- [`Plutus.Contract.Test.ContractModel`](https://github.com/input-output-hk/plutus/blob/master/plutus-contract/src/Plutus/Contract/Test/ContractModel.hs), hỗ trợ kiểm tra dựa trên tài sản của các hợp đồng Plutus.
- [`Plutus.Contracts.Uniswap`](https://github.com/input-output-hk/plutus/blob/master/plutus-use-cases/src/Plutus/Contracts/Uniswap.hs),  một triển khai Uniswap trong Plutus.
- [`Plutus.PAB.Webserver.API`](https://github.com/input-output-hk/plutus/blob/master/plutus-pab/src/Plutus/PAB/Webserver/API.hs), chứa giao diện HTTP do PAB cung cấp.
- [`Plutus.Trace.Emulator`](https://github.com/input-output-hk/plutus/blob/master/plutus-contract/src/Plutus/Trace/Emulator.hs), chứa các loại và chức năng liên quan đến dấu vết.
- [`Plutus.V1.Ledger.Ada`](https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Ada.hs), chứa hỗ trợ cho tiền tệ Ada.
- [`Plutus.V1.Ledger.Contexts`](https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Contexts.hs), chứa định nghĩa của các kiểu liên quan đến context-related.
- [`Plutus.V1.Ledger.Interval`](https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Interval.hs), chứa định nghĩa và các hàm trợ giúp cho `Interval` .
- [`Plutus.V1.Ledger.Slot`](https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Slot.hs), c chứa định nghĩa của `Slot`.
- [`Plutus.V1.Ledger.Value`](https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Value.hs), cchứa định nghĩa và các hàm trợ giúp cho `Value`.
- [`PlutusTx.Data`](https://github.com/input-output-hk/plutus/blob/master/plutus-tx/src/PlutusTx/Data.hs), chứa định nghĩa của kiểu `Data`.
- [`PlutusTx.IsData.Class`](https://github.com/input-output-hk/plutus/blob/master/plutus-tx/src/PlutusTx/IsData/Class.hs), định nghĩa lớp `IsData`.


## Tài nguyên bổ sung

- [Kho lưu trữ Plutus](https://github.com/input-output-hk/plutus)
- [Tìm hiểu bạn một Haskell vì điều tốt!](http://learnyouahaskell.com/)
- [Khóa học về Haskell & Cryptocurrencies Mông Cổ](https://www.youtube.com/playlist?list=PLJ3w5xyG4JWmBVIigNBytJhvSSfZZzfTm)
