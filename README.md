# ste-zh：簡化技術中文規範（臺灣繁體版）

> 本專案 fork 自 [dualface/ste-zh](https://github.com/dualface/ste-zh)，改寫為繁體中文與臺灣慣用技術詞。R1–R25 規則、輸出模板與狀態詞的內容與原版相同；另外把預設輸出語言改為繁體中文（臺灣用詞），並新增臺灣慣用詞對照表（`references/terminology.md` 第 7 節）。

一個給 AI Agent 用的任務回報規範 skill。借鑑航空與工業界嚴謹的 **ASD-STE100**（Simplified Technical English，簡化技術英語）原則，約束 Agent 用清楚、嚴密且無歧義的中文回報工作結果。

Agent 的日常回報常常過程冗長、推測含糊，或者漏掉關鍵的驗證證據。啟用 `ste-zh` 後，Agent 會遵循受控中文規範回報：**結論先行、實證說話、狀態固定、一目了然**。

> 💡 本 skill 只規範 Agent 向使用者回報與討論時的表達方式，不影響專案程式碼、註解、commit message 及產生的檔案內容。

---

## 為什麼需要 ste-zh？

同一個 Bug 修復任務，常見回報與規範化回報的差異如下：

**常見 AI 回報（冗長、推測、不附實證）：**

> 「好的，我先看了一下登入相關的程式碼，發現問題應該是出在 token 更新的邏輯上，大概是並行的時候兩個請求同時去更新導致的。然後我對 `auth/refresh.ts` 進行了修改，加了一個鎖來實現循序處理，另外順手把一些日誌之類的也優化了一下。測試基本都跑過了，應該沒什麼問題，不過 iOS 那邊因為沒有環境暫時沒辦法驗證。你看要不要我再把重試邏輯也一起改了？」

**`ste-zh` 規範化回報（結論先行、實證可查、方便決策）：**

> **已完成**：修復登入後偶發自動登出的問題。`npm test auth` 已驗證，iOS 端未驗證。
>
> **原因**：兩個請求同時更新 token，後完成的更新使先取得的 token 失效。  
> **證據**：`auth/refresh.ts:42` — 更新前未檢查是否已有進行中的更新。
>
> **改動**：
>
> - `auth/refresh.ts`：同一時間只允許一個更新請求，其他請求等待該結果。
> - `auth/logger.ts`：刪除 3 處重複日誌（超出任務範圍）。
>
> **驗證**：
>
> - 已驗證：`npm test auth`（48 項測試全部通過）。
> - 未驗證：iOS 端（本機缺少建置環境）。
>
> **待確認**：  
> 是否修改重試邏輯？
>
> 1. 維持現狀（推薦）
> 2. 這次一起修改

---

## 核心特色

- **結論先行**：回覆第一句直接講結果，多個任務並行時幾秒內就能掃完。
- **狀態固定**：只使用「已完成」「部分完成」「未驗證」「阻塞」等明確狀態詞，杜絕「應該搞定了」這類模稜兩可的說法。
- **必須標明驗證**：每項改動都寫明是否驗證、怎麼驗證，杜絕虛假回報。
- **一詞一義與主動語態**：統一專用詞彙，消除虛化動詞與有歧義的長句，降低閱讀負擔。
- **選項化決策**：遇到需要人工判斷的決策點，自動整理成附推薦的編號選項，使用者輸入數字就能往下推進。
- **臺灣用詞**：輸出使用繁體中文與臺灣慣用技術詞（程式碼、設定、預設、專案），對照表見 `references/terminology.md` 第 7 節。

---

## 安裝

### 推薦：使用 `skills` CLI 安裝

[`skills`](https://github.com/vercel-labs/skills) 是 Vercel Labs 推出的 Agent Skill 管理工具，支援 Claude Code、Cursor、Codex 等主流環境。工具會依設定自動把本 skill 安裝到 `ste` 目錄。

全域安裝（所有專案通用）：

```bash
npx skills add yulin0629/ste-zh -g
```

安裝到目前專案：

```bash
npx skills add yulin0629/ste-zh
```

只為指定 Agent 安裝（例如 Claude Code）：

```bash
npx skills add yulin0629/ste-zh -g -a claude-code
```

更新與移除：

```bash
npx skills update ste -g
npx skills remove --global ste
```

### 手動安裝

把本 repo clone 到 Agent 的 skills 目錄即可。本 skill 的標準名稱是 `ste`，clone 時請確認目標目錄命名為 `ste`：

**Claude Code（全域）：**

```bash
git clone https://github.com/yulin0629/ste-zh.git ~/.claude/skills/ste
```

其他支援 `SKILL.md` 規範的 Agent，請參考各自的文件，把檔案放到對應的 skills 路徑下。

---

## 使用方式

在對話中輸入以下任一指令，即可啟用本 skill：

- `/ste`
- 「用 STE 規範輸出」
- 「按 ASD-STE100 回報」

啟用後，目前對話中的所有回報都會嚴格按照本規範組織。要退出時，傳送「停止 ste」或「stop ste」即可。

> 本 skill 與其他輸出風格衝突時，以本 skill 為優先。

### 注意事項

- **運作機制**：本 skill 靠 Agent 遵守上下文中的指令生效，沒有程式強制執行。
- **上下文衰減**：在極長的對話或發生上下文壓縮（compaction）後，Agent 可能會忘記規範。發現輸出風格退化時，隨時重新傳送 `/ste` 即可恢復。
- **全域常駐**：希望每個對話預設啟用時，可以把「每個對話開始時載入 ste skill」寫進 Agent 的全域規則（例如 Claude Code 的 `~/.claude/CLAUDE.md` 或 `~/.claude/rules/` 規則檔）。

---

## 實戰搭配：與 Kander 協同

原作者使用 [Kander](https://github.com/dualface/kander)（多 Agent 看板排程工具）並行排程多個任務時，大量依賴本 skill：

- **快速掃視**：看板上多張任務卡同時流轉，每張卡的回報第一句就是核心結論，幾秒鐘就能看完所有任務進度。
- **實證把關**：Kander 嚴格要求任務成果可核驗，搭配本 skill 強制區分「已驗證」與「未驗證」，未驗證的結論無法混進「已完成」。
- **極簡決策**：需要人工確認的阻塞點會整理成編號選項，在 Agent 對話中回覆數字就能放行。

---

## 設計背景與 ASD-STE100 的關係

- **關於標準**：[ASD-STE100](https://www.asd-ste100.org/)（Simplified Technical English）由歐洲航太、安全與國防工業協會（ASD）維護，原本是針對英文技術維修文件制定的受控語言規範，用來消除歧義與理解偏差。
- **中文改寫**：本專案提煉 ASD-STE100 的核心原則，並依 AI 互動的特性重構為一套中文實用規則。條目編號與內容均為獨立設計，並非原文直譯。
- **獨立聲明**：本專案為個人開源專案，未收錄 ASD-STE100 原文與受控詞典，與 ASD 組織無隸屬關係。ASD-STE100 與 Simplified Technical English 的權利屬於 ASD。如需研讀 ASD-STE100 英文原版標準，可到其官網免費申請。

---

## 核心檔案

| 路徑                        | 說明                                                                                          |
| --------------------------- | --------------------------------------------------------------------------------------------- |
| `SKILL.md`                  | skill 入口：生效機制、適用範圍、25 條核心規則、5 種標準化輸出模板及自我檢查清單                   |
| `references/terminology.md` | 術語字典：ASD-STE100 關鍵詞譯法、受控情態詞、虛化動詞替換表、固定狀態詞、標點規範、臺灣慣用詞 |
| `examples/result-report.md` | 單一任務結果回報改寫範例（改寫前後對照）                                                      |
| `examples/task-summary.md`  | 多項任務與迭代需求總結改寫範例（改寫前後對照）                                                |

---

## 授權

以 MIT 授權開源。詳見 [LICENSE](LICENSE)。

---

## 原作者的其他專案

歡迎試用原作者 [dualface](https://github.com/dualface) 的其他專案：

- [Kander](https://github.com/dualface/kander)：規則驅動的多 Agent 看板排程工具，內建獨立審查與自動化交付把關。
- [Ullage](https://github.com/dualface/ullage-cli)：本機 daemon 與 CLI 工具，查看 Claude、ChatGPT、Grok、Cursor 等訂閱的用量。
- [QuickTUI](https://quicktui.ai/)：給各類 coding Agent 用的手機完整終端機，支援自架直連，單臺主機免費。
