# PPO_HEMS
PPO 用電分配最佳化家庭能源管理系統

## 專案簡介

本專案實作 近端策略優化(Proximal Policy Optimization, PPO) 結合 PMV (Predicted Mean Vote) 指標，模擬家庭能源管理系統（HEMS）的溫濕度調控，達成能源消耗與室內舒適度的最佳平衡。

系統會依據室內環境條件（溫度、濕度）、外部氣候與電價時段，動態決策：

 - 冷氣模式、冷氣風扇模式與設定溫度

 - 除濕機開關與設定濕度

 - 電扇開關

並輸出最佳化結果、用電成本與舒適度指標。

## 核心功能

#### 1.PPO 智能體

 - 使用策略網路 (Policy Network) 和價值網路 (Value Network) 進行學習，透過強化學習不斷優化設備運行策略以降低電費並保持舒適。

#### 2.訓練與測試

 - 訓練階段進行多輪模擬，逐步更新策略網路。測試階段應用訓練好的模型，生成設備控制參數並計算成本與舒適度。

#### 3.PMV 舒適度評估

 - 使用 pythermalcomfort 計算室內舒適度。

#### 4.電費計算

 - 根據台灣分時電價計算不同時段的耗電成本。

## 主要程式架構

```
PPO_HEMS/
│
├─ ppo_price_online.py                # 主程式，包含訓練與測試流程
├─ config/
│   └─ ppo_price_checkpoint.pt  # 訓練好的模型存檔
├─ data/
│   ├─ 紅外線遙控器冷氣調控指令集.csv  # 冷氣遙控指令集
│   └─ nilm_data_ritaluetb_hour.csv     # 歷史用電數據
└─ result/
    ├─ PPO用電分配參數測試結果.csv
    └─ PPO用電分配參數應用結果.csv
```

## 執行流程

#### 1.初始化環境與智能體

```
env = HEMSEnvironment(pmv_up=0.5, pmv_down=-0.5)
agent = PPOAgent(state_dim=2, action_dim=4)
isa = pd.read_csv("紅外線遙控器冷氣調控指令集.csv")
```

#### 2.訓練循環

對每個 episode：

 - 環境重置取得初始狀態。

 - 24 小時逐步模擬：

    - 智能體選擇動作

    - 環境更新狀態、計算獎勵與能源消耗

    - 保存環境狀態、設備狀態、PMV 指標

 - 根據回報更新策略網路。

 - 記錄總能耗與平均能耗。

訓練完成後保存模型。

#### 3.模型測試

 - 使用訓練好的模型控制家電。

 - 記錄每小時環境變化與設備運行狀態。

 - 計算總電費成本，與歷史用電數據進行節能效果分析。

#### 4.比較與分析

 - 匯出訓練與測試結果表格

 - 與實際 NILM 測試數據比較節能率

## 安裝需求

請先安裝所需套件：
```
pandas == 1.2.4
numpy == 1.24.4
pythermalcomfort == 2.10.0
matplotlib == 3.7.4
scikit-fuzzy==0.5.0
scikit-learn==1.2.2
joblib == 1.3.2
Flask == 3.0.3
psycopg2 == 2.8.6
pymssql == 2.2.1
sshtunnel == 0.4.0
torch == 2.2.0
torchvision == 0.17.0
```

另外需提供：
 
 - 模型 `ppo_price_checkpoint.pt`
 - 測試數據檔案 `nilm_data_ritaluetb_hour.csv` (用於計算與實際用電差異)

## 範例輸出

程式執行後會輸出：

 - 訓練模型所需時間
 - 測試結果資料表
 - 運算所需時間
 - 與原始數據比較的節能率
 - 環境變化模擬與電器調整結果資料表
