# STM32 Low-Power Modes: Sleep, Stop và Standby

Đây là dự án nhóm khảo sát các chế độ tiết kiệm năng lượng Sleep, Stop và Standby trên bo mạch **STM32F103C8T6 Blue Pill**. Repository gồm ba project firmware Keil độc lập. Nút nhấn điều khiển việc vào chế độ công suất thấp và đánh thức vi điều khiển; LED hiển thị trạng thái chương trình, còn đồng hồ vạn năng được dùng để ghi nhận dòng tiêu thụ ở cấp bo mạch.

Đây là dự án nhóm có sự tham gia phát triển của tôi và [@NguyenVo-09](https://github.com/NguyenVo-09). Repository portfolio này giữ lại lịch sử Git gốc và ghi nhận repository nguồn: [NguyenVo-09/stm32-low-power-modes](https://github.com/NguyenVo-09/stm32-low-power-modes).

## Mục tiêu

- Triển khai và quan sát hoạt động của Sleep, Stop và Standby trên STM32F1.
- Thử nghiệm các nguồn đánh thức đã cấu hình và xử lý cần thiết trong firmware sau khi thức dậy.
- Ghi nhận dòng tiêu thụ, đồng thời mô tả ảnh hưởng của bo mạch phát triển và mạch thử nghiệm đến kết quả đo.

## Phần cứng

- Bo mạch STM32F103C8T6 Blue Pill
- Bộ nạp/lập trình và gỡ lỗi ST-LINK
- Breadboard, nút nhấn và LED
- Đồng hồ vạn năng để đo dòng điện

Schematic Proteus đi kèm ghi vi điều khiển là STM32F103R6, trong khi bo mạch thực tế và các project firmware sử dụng STM32F103C8T6. Chỉ nên dùng schematic làm tài liệu tham khảo kết nối GPIO; không dùng số chân trên package trong schematic để đấu nối bo C8T6. Chưa xác minh các giá trị linh kiện trong schematic có trùng với mạch thử nghiệm thực tế hay không.

## Phần mềm và công cụ phát triển

| Hạng mục | Công cụ và thành phần |
|---|---|
| Firmware | C, STM32 HAL, CMSIS, GPIO, EXTI, SysTick, PWR và TIM1 trong các project Stop, Standby |
| Cấu hình | Các file project STM32CubeMX; file `.ioc` khai báo STM32Cube FW_F1 V1.8.6 |
| IDE và build | Keil µVision / MDK-ARM, device pack Keil.STM32F1xx_DFP 2.4.1 |
| Cấu hình compiler | Sleep chọn Arm Compiler 5 (`uAC6=0`), không chỉ rõ phiên bản; Stop và Standby khai báo Arm Compiler 6.24 |
| Schematic | Tài liệu Proteus được cung cấp cùng dự án dưới dạng PDF |

## Các chế độ tiết kiệm năng lượng đã triển khai

Mỗi chế độ nằm trong một project firmware riêng; firmware không có menu để chuyển đổi chế độ khi chạy.

| Chế độ | Điều khiển vào chế độ và đánh thức | Hoạt động của firmware |
|---|---|---|
| Sleep | PA0 yêu cầu vào Sleep; nhấn PA0 lần nữa để đánh thức MCU | Tạm dừng SysTick, thực thi `WFI` và khôi phục SysTick trong callback EXTI sau khi thức dậy |
| Stop | PA0 yêu cầu vào Stop; PA1 đánh thức MCU | Tạm dừng SysTick, thực thi `WFI` với bộ điều áp chính đang bật; sau khi thức dậy, khôi phục cấu hình clock và SysTick |
| Standby | PA1 yêu cầu vào Standby; PA0/WKUP đánh thức MCU | Xóa cờ đánh thức, bật chân WKUP rồi vào Standby; khi được đánh thức, chương trình khởi động lại từ đầu |

Sleep sử dụng HSI/2 với PLL ×16. Stop và Standby sử dụng HSE 8 MHz với PLL ×8. Cả ba project đều cấu hình xung hệ thống 64 MHz.

## Nguồn đánh thức và hoạt động của GPIO

| GPIO | Project Sleep | Project Stop | Project Standby |
|---|---|---|---|
| PA0 | Vào/đánh thức khỏi Sleep, EXTI0 cạnh lên | Vào Stop, EXTI0 cạnh lên | Ngõ vào đánh thức WKUP |
| PA1 | Ngõ vào điều khiển LED PA4 | Đánh thức khỏi Stop, EXTI1 cạnh lên | Vào Standby, EXTI1 cạnh lên |
| PA3 | LED được đảo trạng thái trong vòng lặp chính | LED được đảo trạng thái bởi TIM1 | LED được điều khiển trong vòng lặp chính theo bố trí thí nghiệm trong tài liệu |
| PA4 | LED phản ánh trạng thái PA1 | LED được đảo trạng thái trong vòng lặp chính | LED được đảo trạng thái bởi TIM1 theo bố trí thí nghiệm trong tài liệu |

**Lưu ý về đấu nối Standby:** `main.c` của project Standby gán PA4 cho `blink_led()` và PA3 cho callback TIM1; cách gán này ngược với vai trò LED trong bố trí thí nghiệm được tài liệu mô tả. Hãy kiểm tra cách đấu dây và ánh xạ LED dự kiến trước khi nạp firmware.

## Đo dòng tiêu thụ

Các giá trị dưới đây được ghi trong báo cáo dự án và thể hiện trong ảnh đo đi kèm. Đây là số đo của bo mạch và mạch thử nghiệm, không phải dòng tiêu thụ riêng của MCU. Tải thử nghiệm và dòng ở trạng thái Run không giống nhau giữa các thí nghiệm, vì vậy không nên dùng các số liệu này để xếp hạng trực tiếp ba chế độ theo cùng một điều kiện chuẩn hóa.

| Thí nghiệm | Trạng thái | Dòng điện được báo cáo |
|---|---|---:|
| Sleep | Run | ≈ 25.67 mA |
| Sleep | Sleep | ≈ 7.97 mA |
| Sleep | Sleep với một GPIO được giữ ở mức HIGH | ≈ 10.90 mA |
| Stop | Trước khi vào Stop | 30.41 mA (theo ảnh đo) |
| Stop | Trong Stop | ≈ 2 mA (ảnh đo ghi 2.02 mA) |
| Stop | Sau khi đánh thức | 29.42 mA (theo ảnh đo) |
| Standby | Run | ≈ 32.62 mA |
| Standby | Standby | ≈ 1.67 mA |

Các báo cáo lưu ý LED nguồn trên Blue Pill và những thành phần khác của bo mạch/mạch thử nghiệm góp phần vào dòng đo được. Xem [báo cáo Sleep](Sleep_mode/docs/sleep_mode_report.pdf#page=38), [báo cáo Stop](Stop_mode/docs/Stop_mode_report.pdf#page=29) và [báo cáo Standby](Standby_mode/docs/Standby_mode_report.pdf#page=30) để tham khảo kết quả gốc và bối cảnh đo.

<details>
<summary>Ảnh đo dòng điện</summary>

| Sleep: Run | Sleep | Sleep với GPIO ở mức HIGH |
|---|---|---|
| <img src="Images/results/sleep-run.jpg" alt="Dòng điện ở trạng thái Run trong thí nghiệm Sleep" width="240"> | <img src="Images/results/sleep-mode.jpg" alt="Dòng điện ở chế độ Sleep" width="240"> | <img src="Images/results/sleep-gpio-high.jpg" alt="Dòng điện ở chế độ Sleep khi GPIO được giữ ở mức HIGH" width="240"> |

| Stop: trước khi vào chế độ | Trong Stop | Sau khi đánh thức |
|---|---|---|
| <img src="Images/results/stop-run.jpg" alt="Dòng điện trước khi vào Stop" width="240"> | <img src="Images/results/stop-mode.jpg" alt="Dòng điện trong chế độ Stop" width="240"> | <img src="Images/results/stop-wakeup.jpg" alt="Dòng điện sau khi đánh thức khỏi Stop" width="240"> |

| Standby: Run | Standby |
|---|---|
| <img src="Images/results/standby-run.jpg" alt="Dòng điện ở trạng thái Run trước khi vào Standby" width="320"> | <img src="Images/results/standby-mode.jpg" alt="Dòng điện trong chế độ Standby" width="320"> |

</details>

## Độ trễ đánh thức

Repository và các báo cáo không cung cấp số đo độ trễ đánh thức. README này không đưa ra số liệu độ trễ. Project Stop khôi phục cấu hình clock sau khi thức dậy, còn khi đánh thức khỏi Standby, chương trình khởi động lại từ đầu; ảnh hưởng về thời gian của các thao tác này chưa được định lượng trong kết quả hiện có.

## Cấu trúc dự án

```text
Images/
  hardware/          Ảnh schematic đã trích xuất
  results/           Ảnh đo dòng điện và mạch thử nghiệm
Sleep_mode/
  SLEEP_MODE/         Firmware Sleep, cấu hình CubeMX và project Keil
  docs/               Báo cáo Sleep và schematic PDF
Stop_mode/
  Stop_Mode/          Firmware Stop, cấu hình CubeMX và project Keil
  docs/               Báo cáo Stop
Standby_mode/
  Standby_Mode/       Firmware Standby, cấu hình CubeMX và project Keil
  docs/               Báo cáo Standby
```

Mỗi project firmware bao gồm các file HAL/CMSIS cần thiết và cấu hình đi kèm. Các project Keil:

| Chế độ | Project Keil | Cấu hình CubeMX | Mã nguồn chính |
|---|---|---|---|
| Sleep | [SLEEP_MODE.uvprojx](Sleep_mode/SLEEP_MODE/MDK-ARM/SLEEP_MODE.uvprojx) | [SLEEP_MODE.ioc](Sleep_mode/SLEEP_MODE/SLEEP_MODE.ioc) | [main.c](Sleep_mode/SLEEP_MODE/Core/Src/main.c) |
| Stop | [stopmode.uvprojx](Stop_mode/Stop_Mode/MDK-ARM/stopmode.uvprojx) | [stopmode.ioc](Stop_mode/Stop_Mode/stopmode.ioc) | [main.c](Stop_mode/Stop_Mode/Core/Src/main.c) |
| Standby | [stanby_mode.uvprojx](Standby_mode/Standby_Mode/MDK-ARM/stanby_mode.uvprojx) | [stanby_mode.ioc](Standby_mode/Standby_Mode/stanby_mode.ioc) | [main.c](Standby_mode/Standby_Mode/Core/Src/main.c) |

## Hướng dẫn build và nạp firmware

Các bước dưới đây được biên soạn dựa trên cấu hình project; chưa được thực thi trong quá trình chuẩn bị README này.

1. Cài đặt Keil µVision / MDK-ARM và device pack Keil.STM32F1xx_DFP 2.4.1.
2. Mở một trong các project `.uvprojx` ở trên. Kiểm tra target, compiler và device pack được chọn. Project Sleep sử dụng target Arm Compiler 5; Stop và Standby khai báo Arm Compiler 6.24.
3. Nếu cần, mở file `.ioc` tương ứng trong STM32CubeMX để xem cấu hình. Các project hiện có sẵn mã nguồn được sinh và các dependency HAL/CMSIS đi kèm.
4. Đối chiếu cách đấu nút nhấn và LED với bảng GPIO. Đặc biệt, hãy kiểm tra lưu ý về ánh xạ LED của Standby ở trên.
5. Kết nối bo mạch qua SWD bằng ST-LINK, chọn cấu hình target/debug phù hợp, sau đó build và nạp project đã chọn.
6. Thao tác nút theo bảng chế độ. Khi lặp lại phép đo dòng, hãy ghi nhận trạng thái bo mạch, cách cấp nguồn, kết nối debugger và số đo trên đồng hồ.

Mỗi lần chỉ nạp một project chế độ lên bo mạch.

## Bằng chứng thực nghiệm

Thư mục `Images/results/` chứa các ảnh đo được trích dẫn trong báo cáo. Báo cáo đầy đủ của nhóm được lưu dưới dạng PDF trong thư mục `docs/` của từng chế độ. Kết quả được trình bày trong bối cảnh dự án nhóm và giữ nguyên ngữ cảnh của báo cáo gốc.

## Thành viên và đóng góp cá nhân

Đây là dự án nhóm thuộc học phần Hệ thống nhúng, có sự tham gia phát triển của [@haquanghuy1230-wq](https://github.com/haquanghuy1230-wq) và [@NguyenVo-09](https://github.com/NguyenVo-09). Phần đóng góp cá nhân của tôi gồm:

- Cấu hình các project STM32CubeMX cho Sleep, Stop và Standby.
- Triển khai hoạt động firmware để vào chế độ công suất thấp, đánh thức và quan sát trạng thái hệ thống.
- Chạy firmware trên phần cứng thực tế, đo dòng tiêu thụ và ghi nhận kết quả cho từng chế độ.

Các báo cáo, số đo và sản phẩm tổng thể được thực hiện trong khuôn khổ dự án nhóm. Tác giả của các commit Git gốc được giữ nguyên. Xem [repository nguồn](https://github.com/NguyenVo-09/stm32-low-power-modes) để tham khảo dự án và lịch sử ban đầu.
