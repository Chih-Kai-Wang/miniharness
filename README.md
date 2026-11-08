# MiniHarness

**八個步驟，寫出自己的 Python agent harness。**

MiniHarness 是一套教你親手寫 agent harness 的教材：你會做出一個能呼叫工具、處理錯誤、接受評測的程式。可以直接從[八步動手營](tutorial/README.md)開始，也可以先操作網站上的[互動循環機器](https://chih-kai-wang.github.io/miniharness/#machine)。整套內容不需要 API Key，也不必先讀完全部課程。

```bash
python3 demos/demo_mock.py       # 先看一次完整的離線循環
python3 tutorial/check.py        # 查看八步工作坊的驗收規格
```

需要 Python 3.9+。harness 執行時只用標準函式庫，`pytest` 只在開發時需要；網站、測試與選修的模型訓練範例各有自己的開發依賴。

| 你現在想做什麼 | 入口 |
|---|---|
| 直接寫一個 agent harness | [八步動手營](tutorial/README.md)：循環、協議、工具、路徑守衛、修復、上下文、評測與整合 |
| 看模型數值如何改變 | [TokenScope 實驗路線](docs/tokenscope-labs.md)：attention、取樣與 BPE，可保存重播 |
| 補 Python／工程／ML 基礎 | [MiniAcademy 教材索引](academy/README.md)：依缺口選讀，不必全修才能使用 harness |

**教材狀態：** 38 個模組都已有繁體中文正文、練習與驗收指引，英文課程正文還沒有提供。模組標成 `ready` 只表示教材可以讀，不表示有學習者通過，也不表示教材經過外部教學驗證。課程清單以 `academy/manifest.json` 為準。

[自測與驗收](academy/TOOLS.md) · [作業契約](academy/submissions/README.md) · [導師手冊](academy/MENTOR.md)

## 30 秒上手

下面的指令都不需要任何 API Key。

```bash
git clone https://github.com/Chih-Kai-Wang/miniharness.git && cd miniharness

# 路線 A（推薦入口）：動手營——8 步親手寫出自己的 harness
cat tutorial/README.md
python3 tutorial/check.py          # 建議先看看你現在的進度（0/8）

# 路線 B：先跑現成的，再回頭拆
python3 demos/demo_mock.py        # 離線 demo：MockLLM 扮演大腦，完整跑一遍 harness
python3 demos/demo_from_zero.py   # 從 0 造腦：自訓語言模型 0%、手寫規則 100%、模仿學習 100%
python3 -m pytest                 # 核心、課程與失敗路徑檢查
```

如果想被一步一步帶著做，請進 [tutorial/](tutorial/README.md)。你會從空檔案開始，依序完成循環、協議、工具、安全閘、自我修復、上下文、評測，最後通過畢業考，寫出自己的 `my_harness.py`。每一步都有檢查器驗收。

下面是離線 demo 的實際輸出（`NO_COLOR=1 python3 demos/demo_mock.py`）：

```text
工作區：/var/folders/.../miniharness-demo-fge77uvn
大腦：MockLLM（腳本化，離線）

┌─ Step 1 / 10
│ 模型：Thought: 收到任務。先把計劃寫進 todo，防止漏步驟。
│ ⚙ todo {'action': 'write', 'content': '[ ] 查看目錄結構\n[ ] 搜索 TODO 標記\n[ ] 統計並寫入 REPORT.md'}
│ ↳ OK：todo 已更新。當前內容：……
┌─ Step 2 / 10
│ 模型：Thought: 先遞歸看一下工作區裡有什麼。
│ ⚙ list_dir {'path': '.', 'recursive': True}
│ ↳ .todo.md / README.md / app/core.py / app/utils.py / notes.txt / scripts/pipeline.py
┌─ Step 3 / 10
│ 模型：Thought: 結構清楚了。用 grep 在所有 .py 文件裡搜 TODO 標記。
│ ⚙ run_bash {'command': 'grep -rn "TODO" --include=*.py .'}
│ ↳ ./app/core.py:5:    # TODO: 加入邊界值校驗 …（共 4 處）
┌─ Step 4 / 10
│ ⚙ write_file {'path': 'REPORT.md', 'content': '# TODO 報告\n\n- app/core.py（2 處）：…'}
│ ↳ OK：已寫入 REPORT.md（115 字符）
┌─ Step 5 / 10  ⚙ todo {'action': 'show'}
┌─ Step 6 / 10
│ 模型：Thought: 三步計劃全部完成，可以收尾了。
└─ ✓ 完成（6 步，5 次工具調用）

── 產物 REPORT.md ──
# TODO 報告
- app/core.py（2 處）：加入邊界值校驗；參數順序容易搞混，改成關鍵字參數
- app/utils.py（1 處）：處理中文與空格
- scripts/pipeline.py（1 處）：接入日誌
```

完整未刪節的輸出見 `demos/`。可以看到誘餌檔案 `notes.txt` 被正確排除在報告之外。

## 四個 Demo

| Demo | 命令 | 說明 |
| --- | --- | --- |
| 離線循環 | `python3 demos/demo_mock.py` | MockLLM 扮演大腦，看 harness 機制本身 |
| 從 0 造腦 | `python3 demos/demo_from_zero.py` | 沒有模型也能有 agent：三種自製大腦同場對比（[09 章](docs/09-from-zero.md)） |
| 真模型 REPL | `python3 demos/demo_cli.py` | 交互式命令行 agent，把它當工具用 |
| 迷你評測 | `python3 demos/demo_eval.py [--real]` | 3 個任務的評測集，看通過率與步數 |

要接真實模型，任何 OpenAI 相容端點都可以，例如 OpenAI、DeepSeek、智譜、Moonshot 或 Ollama：

```bash
export OPENAI_API_KEY=sk-...
# 可選：export MINIHARNESS_BASE_URL=https://api.deepseek.com/v1
# 可選：export MINIHARNESS_MODEL=deepseek-chat
python3 demos/demo_cli.py
```

## 概念章節（docs/）

| 章 | 主題 | 對應代碼 | 核心問題 |
| --- | --- | --- | --- |
| [00 總覽](docs/00-overview.md) | harness 是什麼 | —— | 智能在模型，工程在 harness |
| [01 Agent 循環](docs/01-agent-loop.md) | while 循環 | `loop.py`（149 行） | agent 的心臟為什麼這麼小？ |
| [02 協議](docs/02-protocol.md) | 工具調用格式 | `protocol.py` | 模型怎麼「說」它要調工具？ |
| [03 工具](docs/03-tools.md) | 工具設計 | `tools.py` | 什麼是好工具？ |
| [04 上下文工程](docs/04-context.md) | 歷史裁剪 | `context.py` | 歷史越滾越長怎麼辦？ |
| [05 記憶與計劃](docs/05-memory.md) | todo 模式 | `todo` 工具 | 為什麼都在寫 todo 文件？ |
| [06 安全與沙箱](docs/06-sandbox.md) | 攻與防 | 安全閘 | agent 亂來怎麼辦？ |
| [07 迷你評測](docs/07-eval.md) | 評測集 | `eval.py` | 怎麼證明 harness 有用？ |
| [08 路線圖](docs/08-roadmap.md) | 生產化 | —— | 從這裡到真實系統差什麼？ |
| [09 從 0 造腦](docs/09-from-zero.md) | 自製大腦 | `brains.py` | 沒有模型，也能有 agent 嗎？ |

建議的閱讀順序是：先跑 demo，再讀 01、02 章並對照原始碼，接著讀完其餘章節，最後做 [07 章](docs/07-eval.md)的「畢業作業」，也就是替評測集加一個自己的任務。

## MiniAcademy 課程（academy/）

課程分成六個階段。下表列出每個階段的目標與練習估計時數，時數包含專案與反覆練習。

| 階段 | 主題 | 規劃目標 | 預估小時 |
| --- | --- | --- | --- |
| 0 | 零基礎起步 | 終端機/檔案/第一支程式不慌張 | 9 |
| 1 | Python 程式核心 | 獨立交付命令行工具 | 36 |
| 2 | 工程與系統基礎 | 測試、打包、協作、呼叫 API | 40 |
| 3 | 數學與機器學習 | 端到端 ML 流水線 | 60 |
| 4 | 深度學習與 LLM | 親手訓練自己的小模型 | 80 |
| 5 | Agent 工程與研究方法 | MiniHarness 全套 + 復現論文 + 公開作品 | 80 |

```bash
python3 tools/academy.py map      # 地圖 + 進度
python3 tools/academy.py next     # 現在該學什麼（嚴格按階段推進）
python3 tools/academy.py gaps     # 四面漏洞掃描：結構/內容/翻譯/學習
python3 tools/academy.py i18n     # 多語言覆蓋報告
```

每一條 Goal 都標明層級（know、do 或 build）和驗證方式，manifest 的結構由測試把關。想知道課程哪裡還有缺口，只要執行一條命令就能列出來。細節見 [academy/README.md](academy/README.md) 與 [ROADMAP.md](ROADMAP.md)。

## 互動網站（site/）

> 在線體驗：**https://chih-kai-wang.github.io/miniharness/**

網站是一個 React + Vite 的單頁應用，把課程做成可以操作的東西：

- **首屏**是一頁幾乎空白的手稿，上面會自己寫出這門課的命題：「智能在模型，工程在 harness。」
- **章貳「循環機器」**是核心裝置。它實際執行教程演算法的 JavaScript 移植版，可以單步、自動播放或重置，也能在「看模型」與「看工程」兩種視角之間切換。故障注入拉桿會送進一段壞掉的 JSON，讓你看循環如何自我修復。
- **章叁「目標星圖」**把 110 條 Goal 依 manifest 的前置依賴排成螺旋星圖。你可以懸停、釘選，或雙擊標記為達成，標記只存在本機。
- **章肆「工序單」**列出動手營的八個步驟，可以逐項打勾；全部勾滿，就是 G5.3 要求的成果。

```bash
cd site && npm install && npm run build && npm run preview   # 本地查看生產構建
```

循環機器可以用鍵盤操作：Space 單步執行，R 重置，F 切換故障。星圖則可以用方向鍵巡覽。系統設定為 `prefers-reduced-motion` 時，所有繪製與運轉動畫會直接停在最終狀態。

自動播放與單步操作共用同一套播放控制。按下暫停後，機器會在目前這一拍結束時停住；重置會取消正在進行的演出，故障開關則讓機器從新的一輪開始。系統設定減少動態時，第一次進入畫面不會自動播放。

2026-09 做過一次瀏覽器回歸檢查，涵蓋以下情境：開始後暫停、等待後步數不變、故障的第一拍顯示解析錯誤、自動恢復並完成報告，以及390px版面下的教材連結與自行記錄。

## 目錄結構

```
miniharness/
├── academy/              # ★ MiniAcademy：6 階段課程地圖（manifest + 引擎 + 正文）
│   ├── manifest.json     #   唯一事實源：38 模塊 / 110 條 Goal / 前置依賴圖
│   ├── content/          #   各語言正文（zh-TW 完整，en 骨架）
│   └── README.md         #   目標體系 / 漏洞掃描 / 貢獻流程
├── tools/academy.py      # map / next / gaps / i18n / validate / 進度管理
├── tutorial/             # ★ 動手營：8 步親手造 harness（含逐步檢查器與參考實現）
│   ├── check.py          #   python3 tutorial/check.py —— 進度總覽 / 逐步驗收
│   ├── steps/            #   9 份步驟文檔（每步：目標 → 規格 → 提示 → 常見坑）
│   └── solutions/        #   完整參考實現（卡住 30 分鐘才許看）
├── miniharness/          # 核心包（9 個模塊，約 1600 行，零依賴）
│   ├── protocol.py       #   工具調用協議：解析 / 格式化 / 系統提示詞
│   ├── llm.py            #   大腦介面：MockLLM + OpenAI 兼容適配器（純 urllib）
│   ├── tools.py          #   工具註冊表 + 5 個內置工具 + 安全閘
│   ├── context.py        #   上下文預算與歷史裁剪
│   ├── loop.py           #   AgentLoop：整個 harness 的心臟
│   ├── brains.py         #   從 0 造腦：RuleBrain / NgramBrain / PolicyBrain（09 章）
│   ├── eval.py           #   迷你評測 runner
│   └── tasks.py          #   3 個內置評測任務 + 離線 mock 劇本
├── demos/                # mock（離線）/ from_zero（造腦）/ cli（真模型）/ eval（評測）
├── docs/                 # 10 章概念文檔（第 5 階段 harness 模組的教材）
├── tests/                # 核心、課程與驗收工具測試
├── ROADMAP.md            # 專案層規劃：內容/多語言/工具/社群四條線
└── pyproject.toml
```

## 設計原則

1. **可讀比功能重要。** 每個模組的核心邏輯一個螢幕就看得完，註解說明「為什麼」，而不是「是什麼」。
2. **不用第三方依賴。** 呼叫真實 API 其實就是一個 HTTP POST，這裡用標準函式庫親手寫一次。
3. **錯誤是一等公民。** 所有錯誤都會轉成可讀的文字送回模型，讓它自我修復，而不是直接拋出例外。
4. **大腦可以互換。** MockLLM、自己訓練的小模型（09 章）和真實 API 只差一行就能切換，離線教學與真實實驗共用同一套機制。
5. **誠實面對安全問題。** [06 章](docs/06-sandbox.md)同時講防線和繞過的方法，不假裝字串過濾就等於沙箱。

## 這個專案不做什麼

- MiniHarness 不能直接用在生產環境，也不打算往那個方向發展。[08 章](docs/08-roadmap.md)列出了離生產等級還差哪些東西，以及補齊的順序。
- 工具的安全閘是教學用的字串與路徑檢查，不是沙箱。[06 章](docs/06-sandbox.md)會示範它可以怎麼被繞過。
- 英文課程目前只有標題，還沒有正文。
- 教材標成 `ready` 不代表有人學完，也不代表經過外部教學驗證。

## 相關項目對照

| 項目 | 教什麼 | 與本項目的關係 |
| --- | --- | --- |
| [minimind](https://github.com/jingyaogong/minimind) | 從零**訓練**小型 LLM | 對偶問題：它造大腦，MiniHarness 造馬具 |
| [AI for Beginners](https://github.com/microsoft/AI-For-Beginners) | AI 通識課程 | 章節組織的參照 |
| [mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) | 生產級極簡 agent | 進階閱讀：讀完本項目後它的源碼會非常好懂 |
| [smolagents](https://github.com/huggingface/smolagents) | 代碼即行動流派 | [02 章](docs/02-protocol.md)的流派對比 |
| [OpenHands](https://github.com/All-Hands-AI/OpenHands) | 完整平臺 | [08 章](docs/08-roadmap.md)的生產化參照 |

## 常見問題

**這和 LangChain 或 Agents SDK 有什麼不同？**
框架讓你跳過這一層抽象，MiniHarness 則讓你看懂它，甚至連模型都可以不用（[09 章](docs/09-from-zero.md)），全部自己動手寫。讀完之後，不管你用哪個框架，都會知道 prompt 放在哪裡、錢花在哪裡、哪裡容易出問題。

**一定要有 API Key 嗎？**
不需要。所有 demo 預設都是離線的。[09 章](docs/09-from-zero.md)更進一步，連模型都自己做：手寫規則、自己訓練的語言模型，以及模仿學習的策略。要接真實模型，只需要換一行建構函式。

## 參與

歡迎送 PR。最需要的是新的評測任務，也歡迎新增章節或修正錯字。回報問題時請開 issue，並附上用 `NO_COLOR=1` 執行的完整輸出。

## License

[MIT](LICENSE) © 2026 MiniHarness contributors

## 靈感與致謝

這個專案受到 [minimind](https://github.com/jingyaogong/minimind) 啟發。minimind 教你造大腦，MiniHarness 處理的是它的對偶問題：造馬具。章節組織參考了微軟 [AI for Beginners](https://github.com/microsoft/AI-For-Beginners) 的課程化寫法。設計想法則受到 [mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent)（"the 100-line agent"）與 Anthropic [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) 的影響。
