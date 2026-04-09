# Vibe Coding - 台股分析期中報告

本專案為「生成式 AI」課程之期中分組報告，核心目標在於實踐 **Vibe Coding** 的開發模式。

## 專案簡介
透過 AI 應用程式，我們將開發職責從傳統的「撰寫邏輯」轉向「定義意圖」與「審核產出」，在極短時間內完成台股數據蒐集、類股研究及視覺化分析。

## AI 工具鏈與系統環境
為了落實代理開發（Agentic Workflow），本專案整合了以下工具：

* **核心 LLM**：ChatGPT / Claude 3.5（負責產業研究與意圖定義）
* **輔助 IDE**：Cursor / VS Code（負責 AI 代碼生成與 Git 上傳）
* **系統環境**：Python 3.10+
* **關鍵套件**：
    * `yfinance` / `FinMind`：數據抓取
    * `mplfinance`：專業級 K 線圖繪製
    * `pandas`：數據結構處理

## 任務執行紀錄
1.  **數據蒐集**：定義意圖要求 AI 撰寫腳本，抓取台股上市股票之每日開、收、高、低價（OHLC）數據。
2.  **類股分類**：依靠 AI 進行快速研究，將台股劃分為「半導體巨人」與「AI 基礎建設」兩大族群，每類各 5 支代表性股票。
3.  **視覺化實踐**：下達口語化指令，要求 AI 處理時間序列數據並生成包含均線（MA）與成交量的 K 線圖。

## 2026 台股趨勢分析與結論
根據產出的 K 線圖，本組得出以下結論：

<img width="1167" height="793" alt="下載 (1)" src="https://github.com/user-attachments/assets/6b2cba98-9ea1-47e7-b744-43cf4640d234" />

* **趨勢判讀**：2026 年初大盤呈現強勢多頭排列（5MA > 20MA > 60MA），反映 AI 伺服器與先進封裝產能達標的市場預期。
* **量價觀察**：在 34,000 點壓力區出現高檔震盪，但整體支撐力道強勁，多頭格局穩固。
* **最終結論**：AI 與半導體仍為台灣經濟命脈，技術面上只要未跌破長期均線，後市依舊看好。

## Vibe Coding 流程優化與創新
本專案不只是單純的程式碼撰寫，更著重於開發流程的實驗與優化：

* **流程簡化**：透過「模糊意圖」驅動 AI 處理複雜的金融維度，開發者僅需負責邏輯審核，省去了手寫代碼與手動排除 Git 報錯的數天開發時間。
* **Prompt 創新**：我們開發了一套具備「角色扮演」與「數據審核」機制的 Prompt 鏈，確保 AI 在生成圖表時能精確遵循金融專業格式（如：紅漲綠跌、英文標籤處理）。

## 核心程式碼實作
基於上述開發流程，我們實作了以下 Python 腳本。該腳本特別針對 `yfinance` 的多重索引（Multi-Index）問題進行了自動化修正，並將圖表標籤統一為英文以確保跨平台顯示正常。

<details>
<summary>點擊展開 Python 原始碼</summary>

```python
import yfinance as yf
import mplfinance as mpf
import pandas as pd

def fetch_twii_data(start_date="2025-10-01"):
    """
    抓取台股大盤數據並修正 Multi-Index 與型別錯誤。
    """
    data = yf.download("^TWII", start=start_date, auto_adjust=True)
    
    if data.empty:
        raise ValueError("無法取得數據，請檢查網路連線。")

    # 處理 yfinance v0.2.x 以上版本可能產生的 Multi-Index
    if isinstance(data.columns, pd.MultiIndex):
        data.columns = data.columns.get_level_values(0)
    
    data.index = pd.to_datetime(data.index)
    data = data.dropna()

    # 強制轉換 OHLC 為數值型別，避免繪圖錯誤
    cols = ['Open', 'High', 'Low', 'Close', 'Volume']
    for col in cols:
        if col in data.columns:
            data[col] = pd.to_numeric(data[col], errors='coerce')

    return data

def plot_stock_chart(data):
    """
    繪製符合台灣市場習慣的 K 線圖 (紅漲綠跌)。
    """
    mc = mpf.make_marketcolors(up='red', down='green', inherit=True)
    s  = mpf.make_mpf_style(marketcolors=mc, gridstyle='--', y_on_right=True)

    mpf.plot(
        data,
        type='candle',
        style=s,
        title='2026 TAIEX (^TWII) Trend Analysis',
        ylabel='Price (TWD)',
        ylabel_lower='Volume',
        volume=True,
        mav=(5, 20, 60),
        figsize=(12, 8),
        tight_layout=True
    )

if __name__ == "__main__":
    df = fetch_twii_data()
    plot_stock_chart(df)
