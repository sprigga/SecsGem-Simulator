# SECS模擬器快速開始指南

## 快速開始

### 1. 安裝依賴

```bash
uv sync
```

### 2. 發送測試數據 (3步驟)

#### 步驟1: 創建Lua腳本

創建文件 `my_test.lua`:

```lua
-- 初始化secsgem包
secsgem = base.setup("0.1.9")

-- 建立連線 (修改IP和端口)
connection = secsgem.hsms("192.168.1.100", 5000, true, 1)

-- 連接
connection.connect()

-- 發送Select請求
connection.send_select_req()
connection.wait_for_select_rsp()

-- 發送測試數據 - S1F1
connection.send_function("S1F1 W")

-- 斷開連接
connection.disconnect()

print("測試完成!")
```

#### 步驟2: 執行腳本

```bash
uv run sgsim run_script my_test.lua
```

#### 步驟3: 查看結果

腳本執行時會顯示發送和接收的SECS消息。

## 常用測試場景

### 場景1: 基本連接測試

```lua
secsgem = base.setup("0.1.9")
connection = secsgem.hsms("192.168.1.100", 5000, true, 1)
connection.connect()
connection.send_select_req()
connection.wait_for_select_rsp()
connection.send_function("S1F1 W")
connection.disconnect()
```

### 場景2: 發送控制命令

```lua
secsgem = base.setup("0.1.9")
connection = secsgem.hsms("192.168.1.100", 5000, true, 1)
connection.connect()
connection.send_select_req()
connection.wait_for_select_rsp()

-- 發送START命令
connection.send_function("S2F41 W < L [2] < A \"START\" > < L > .")

-- 等待確認
packet = connection.wait_for_function(2, 42)  -- 等待S2F42

connection.disconnect()
```

### 場景3: 批量發送測試數據

```lua
secsgem = base.setup("0.1.9")
connection = secsgem.hsms("192.168.1.100", 5000, true, 1)
connection.connect()
connection.send_select_req()
connection.wait_for_select_rsp()

-- 發送多個測試數據
for i = 1, 5 do
    connection.send_function("S6F11 < L [2] < U4 [1] 1001 > < U4 [1] " .. i .. " > .")
    base.sleep(1)  -- 等待1秒
end

connection.disconnect()
```

## 參數說明

### HSMS連接參數

```lua
secsgem.hsms(address, port, active, session_id)
```

| 參數 | 說明 | 示例 |
|------|------|------|
| address | 目標IP地址 | "192.168.1.100" |
| port | 端口號 | 5000 |
| active | 連接模式 | true (主動) / false (被動) |
| session_id | 會話ID | 1 |

### SML數據格式

| 類型 | 格式 | 示例 |
|------|------|------|
| String | `< A "TEXT" >` | `< A "HELLO" >` |
| Number | `< I4 100 >` | `< I4 100 >` |
| Binary | `< B 0x01 0x02 >` | `< B 0x01 0x02 >` |
| List | `< L [2] <...> <...> .` | `< L [2] < A "A" > < A "B" > .` |
| Array | `< U4 [3] 1 2 3 >` | `< U4 [3] 1 2 3 >` |

## 常用SECS消息

| 消息 | 功能 | 示例 |
|------|------|------|
| S1F1 | 建立通訊 | `connection.send_function("S1F1 W")` |
| S1F3 | 請求狀態 | `connection.send_function("S1F3 W")` |
| S2F13 | 設備常數請求 | `connection.send_function("S2F13 W < L [1] < U4 [1] 1001 > .")` |
| S2F41 | 主機命令 | `connection.send_function("S2F41 W < L [2] < A \"START\" > < L > .")` |
| S5F1 | 報警 | `connection.send_function("S5F1 < L [2] < B [1] 0 > < U4 [1] 100 > .")` |
| S6F11 | 事件數據 | `connection.send_function("S6F11 < L [2] < U4 [1] 1001 > < L [1] < U4 [1] 100 > > .")` |

## 故障排除

### 連接失敗

```
錯誤: timeout
```

解決方案:
1. 檢查IP地址和端口是否正確
2. 確認目標設備是否在線
3. 檢查防火牆設置

### 無回應

```
錯誤: Reply not received within timeout
```

解決方案:
1. 確認消息是否需要回應 (檢查W bit)
2. 增加等待時間: `connection.wait_for_packet_reply(packet)` 或 `base.sleep(5)`
3. 檢查目標設備是否正確處理消息

### 腳本錯誤

```
錯誤: error parsing SML
```

解決方案:
1. 檢查SML格式是否正確
2. 確認數據類型是否匹配
3. 驗證列表和數組的長度

## 進階用法

詳細文檔請參考 `test_data_sending_guide.md`
