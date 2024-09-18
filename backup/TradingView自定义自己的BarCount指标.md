根据纽约的开盘时间-结束时间显示BarCount，其他时间不显示，还可以定义默认几根K线显示一个BarCount。

pip Script语言：
```pip Script
//@version=4
study("Bar Count for Market Hours", overlay=true, max_labels_count=500)

// 用户输入的标签大小
sizeOption = input(title="Label Size", type=input.string,
     options=["Auto", "Huge", "Large", "Normal", "Small", "Tiny"],
     defval="Normal")

// 选择标签的大小
labelSize = (sizeOption == "Huge") ? size.huge :
     (sizeOption == "Large") ? size.large :
     (sizeOption == "Small") ? size.small :
     (sizeOption == "Tiny") ? size.tiny :
     (sizeOption == "Auto") ? size.auto :
         size.normal

// 用户输入文本颜色
color c_labelColor = input(color.orange, "Text Color", input.color)

// 设置每隔多少根K线显示一次标签
c_contador = input(title="Display at every X bars", type=input.integer, defval=2)

// 定义纽约股市的开盘和关盘时间（9:30 AM 到 4:00 PM）
// 9:30 AM 纽约时间在 UTC 冬季是 14:30，夏季是 13:30
ny_open_hour = 9
ny_open_minute = 30

// 4:00 PM 纽约时间在 UTC 冬季是 21:00，夏季是 20:00
ny_close_hour = 16
ny_close_minute = 0

// 通过 `timenow` 判断是否在夏令时 (大约 3 月到 11 月)
is_summer_time = (timenow - timestamp("1970-01-01 00:00")) % 31536000000 >= 8280000000  // 大致为 4 月到 10 月

// 如果在夏令时，纽约时间与 UTC 相差 4 小时；否则相差 5 小时
utc_open_hour = ny_open_hour + (is_summer_time ? 4 : 5)
utc_close_hour = ny_close_hour + (is_summer_time ? 4 : 5)

// 获取当前K线的 UTC 时间的小时和分钟
utc_hour = hour(time, "UTC")
utc_minute = minute(time, "UTC")

// 判断是否在开盘时间内
is_market_open() =>
    (utc_hour > utc_open_hour or (utc_hour == utc_open_hour and utc_minute >= ny_open_minute)) and (utc_hour < utc_close_hour or (utc_hour == utc_close_hour and utc_minute <= ny_close_minute))

// 检测是否为新的一天（这里使用的是UTC天数变化）
is_new_day() =>
    dayofweek != dayofweek[1]

// 计数器
var count = 0

// 在新的一天重置计数器
if is_new_day()
    count := 0

// 如果市场开盘，递增计数
if is_market_open()
    count := count + 1

// 每隔 `c_contador` 根K线显示一次标签，但只在市场开盘时间内显示
if is_market_open() and count % c_contador == 0
    label1 = label.new(bar_index, 0, style=label.style_none, text=tostring(count))
    label.set_textcolor(label1, c_labelColor)
    label.set_yloc(label1, yloc.belowbar)
    label.set_size(label1, labelSize)
```