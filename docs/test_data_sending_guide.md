# SECS模擬器測試數據發送指南

## 概述

SECS模擬器提供多種方式發送測試數據，主要通過Lua腳本和SML (SECS Message Language) 格式來控制SECS通信。

## 方法一: 使用Lua腳本文件

### 1. 創建Lua腳本

創建一個 `.lua` 文件，例如 `test_send_data.lua`：

```lua
-- 初始化secsgem包
secsgem = base.setup("0.1.9")

-- 建立HSMS連線 (active模式)
connection = secsgem.hsms("192.168.1.100", 5000, true, 1)

-- 連接到設備
connection.connect()

-- 發送Select請求
connection.send_select_req()
connection.wait_for_select_rsp()

-- 發送S1F1 (建立通訊)
connection.send_function("S1F1 W")

-- 發送S2F41 (主機命令發送)
connection.send_function("S2F41 W < L [2] < A \"START\" > < L > .")

-- 斷開連接
connection.disconnect()
```

### 2. 執行腳本

使用CLI工具執行腳本：

```bash
uv run sgsim run_script test_send_data.lua
```

## 方法二: 使用命令行界面

### 1. 啟動服務器

```bash
# 後台啟動服務器
uv run sgsim start_server --host 0.0.0.0 --port 4000

# 或前台運行
uv run sgsim run_server --host 0.0.0.0 --port 4000
```

### 2. 測試服務器

```bash
uv run sgsim ping_server --host localhost --port 4000
```

## 方法三: 使用Python腳本

### 創建Python腳本

```python
import secsgem_simulator.script_runner

runner = secsgem_simulator.script_runner.ScriptRunner()

lua_script = """
secsgem = base.setup("0.1.9")
connection = secsgem.hsms("192.168.1.100", 5000, true, 1)
connection.connect()
connection.send_select_req()
connection.wait_for_select_rsp()
connection.send_function("S1F1 W")
connection.disconnect()
"""

result = runner.run_string(lua_script)
print(f"執行結果: {result}")
```

### 執行Python腳本

```bash
uv run python send_test_data.py
```

## 常用SECS消息示例

### S1F1 - 建立通訊 (Are You Online)

```lua
connection.send_function("S1F1 W")
```

### S1F3 - 請求狀態

```lua
connection.send_function("S1F3 W")
```

### S2F13 - 設備常數請求

```lua
connection.send_function("S2F13 W < L [1] < U4 [1] 1001 > .")
```

### S2F14 - 設備常數發送

```lua
connection.send_function("S2F14 < L [1] < U4 [1] 1001 > < F4 [1] 25.5 > .")
```

### S2F41 - 主機命令發送

```lua
connection.send_function("S2F41 W < L [2] < A \"START\" > < L > .")
```

### S2F42 - 主機命令確認

```lua
connection.send_function("S2F42 < L [2] < B [1] 0 > < A \"ACK\" > .")
```

### S5F1 - 報警發送

```lua
connection.send_function("S5F1 < L [2] < B [1] 0 > < U4 [1] 100 > .")
```

### S6F11 - 事件數據收集

```lua
connection.send_function("S6F11 < L [2] < U4 [1] 1001 > < L [1] < U4 [1] 100 > > .")
```

## SML數據格式

### 基本數據類型

- **Boolean**: `< BOOLEAN TRUE >` 或 `< BOOLEAN FALSE >`
- **Number**: `< I1 100 >` (I1, I2, I4, I8, F4, F8, U1, U2, U4, U8)
- **String**: `< A "HELLO" >` 或 `< ASCII "TEST" >`
- **Binary**: `< B 0x01 0x02 0x03 >`
- **List**: `< L [2] < U4 [1] 100 > < A "TEST" > .`
- **Array**: `< U4 [3] 1 2 3 >`

### 複雜數據結構示例

```lua
-- 嵌套列表
connection.send_function("S2F41 W < L [3] < A \"PROCESS\" > < U4 [1] 1 > < L [2] < A \"LOT1\" > < A \"PRODUCT\" > . .")

-- 數組
connection.send_function("S6F11 < L [2] < U4 [1] 1001 > < A [5] \"A\" \"B\" \"C\" \"D\" \"E\" > .")

-- 混合數據
connection.send_function("S2F41 W < L [4] < A \"START\" > < U4 [1] 1 > < B 0x01 0x02 > < F4 [1] 25.5 > .")
```

## 連接參數說明

### HSMS連接

```lua
secsgem.hsms(address, port, active, session_id)
```

- **address**: 目標主機IP地址
- **port**: 目標端口號
- **active**: 
  - `true`: 主動模式 (設備主動連接到主機)
  - `false`: 被動模式 (主機主動連接到設備)
- **session_id**: 會話ID

## 等待回應

### 等待特定消息

```lua
-- 等待Select回應
connection.wait_for_select_rsp()

-- 等待Deselect回應
connection.wait_for_deselect_rsp()

-- 等待Linktest回應
connection.wait_for_linktest_rsp()

-- 等待特定Stream/Function
packet = connection.wait_for_function(1, 3)  -- 等待S1F3

-- 等待包回應
packet = connection.wait_for_packet_reply(sent_packet)
```

## 控制流程

### 延遲執行

```lua
-- 延遲1秒
base.sleep(1)

-- 延遲0.5秒
base.sleep(0.5)
```

### 檢查額外數據包

```lua
-- 檢查接收隊列中的額外數據包
packets = connection.extra_packets()
for _, packet in pairs(packets) do
    print(packet)
end
```

## Docker部署

### 構建鏡像

```bash
docker build -t secsgem-simulator .
```

### 運行容器

```bash
docker run -d \
  --name secsgem-sim \
  -p 4000:4000 \
  -p 5000:5000 \
  secsgem-simulator
```

### 在容器中執行腳本

```bash
docker exec -it secsgem-sim sgsim run_script test_send_data.lua
```

## 故障排除

### 連接失敗

1. 檢查IP地址和端口是否正確
2. 確認目標設備是否在線
3. 檢查防火牆設置

### 腳本執行錯誤

1. 驗證Lua語法是否正確
2. 檢查SML格式是否符合規範
3. 確認secsgem版本是否支持所需功能

### 無回應

1. 確認是否需要等待回應 (W bit)
2. 檢查超時設置
3. 使用`extra_packets()`檢查是否有意外的數據包

## 完整示例

參考項目中的 `test_send_data.lua` 和 `send_test_data.py` 文件獲取完整示例。
