# tenth-man

一個適用於 [Codex](https://developers.openai.com/codex) 和 [Claude Code](https://claude.com/claude-code) 的多模型審議外掛程式（包含一個技能和數個席位代理）。適合用在設計選擇、方針審查等不應只依賴單一模型回答的決策。由不同模型擔任的席位進行投票，另有一個固定席位一律提出反對意見，因此可降低議事會在共有同一個盲點的情況下達成共識的可能性。

[English](README.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (Brasil)](README.pt-BR.md) | [Deutsch](README.de.md) | [Français](README.fr.md)

投票成員會評估每一項提案。另設的**第十人**（tenth man）不論自己的看法如何，都必須反對每一項提案，並說明什麼樣的證據可以推翻每一條反對意見。主工作階段擔任主席，主持三輪討論，處理重大疑慮，並撰寫會議紀錄，記下達成了哪些共識、理由為何，以及哪些問題仍未解決。

此技能會先提出議事會計畫，並等待你的核准。若不想等待，可在問題前加上 `fast`（例如 `/tenth-man:council fast 我們是否應該……？`）：主席提出計畫後會立即開始，自行選擇預設選項，並在會議紀錄中列出這些選擇。執行議事會不會自動將結論套用到你的專案。

## 運作方式

1. 主席提出議題、模型、規則、成本和輸出資料夾。
2. 第 1 輪：成員投票；第十人對每個議題提出質疑。
3. 第 2 輪：成員重新檢視重大疑慮，並回應最重要的反對意見。
4. 第 3 輪：成員對會議紀錄草案投票；第十人對照證據檢查結果。
5. 主席寫出 `.tenth-man/<date>-<slug>/minutes.md`。

票分為**贊成**、**有條件贊成**、**反對**和**重大疑慮**。達成共識需要投票成員過半數，且沒有尚未解決的重大疑慮。第十人沒有投票權。

## 在 Codex 中安裝

### 作為外掛程式

```sh
codex plugin marketplace add Shintaro-My/tenth-man
codex plugin add tenth-man@tenth-man
```

如果使用本機簽出的副本，請改為加入它的路徑：

```sh
codex plugin marketplace add /path/to/tenth-man
codex plugin add tenth-man@tenth-man
```

輸入 `$tenth-man:council` 並在後面寫上問題即可呼叫外掛程式的技能，也可以從 `/skills` 中選擇：

```text
$tenth-man:council 是否應該把工作階段狀態從 Redis 移到簽章 Cookie？
```

### 作為獨立技能

將 `skills/council` 複製到 Codex 的技能目錄：

```sh
cp -r skills/council ~/.agents/skills/
```

以這種方式安裝時，請用 `$council` 呼叫。

如果只在一個儲存庫中使用，請改為複製到 `<repo>/.agents/skills/`。本儲存庫的 `.codex/agents/` 下還包含不綁定角色的 Codex 席位設定檔，Codex 會在本儲存庫的簽出目錄中自動找到這些設定檔。若要在所有儲存庫中使用這些設定檔：

```sh
cp .codex/agents/seat-*.toml ~/.codex/agents/
```

即使不複製，此技能也能正常運作：它可以用同樣明確指定的模型啟動子代理，並在各輪之間延續它們的對話串。以外掛程式形式封裝時，不會自動安裝自訂代理的 TOML 檔案。

## 在 Claude Code 中安裝

### 作為外掛程式

```text
/plugin marketplace add Shintaro-My/tenth-man
/plugin install tenth-man@tenth-man
```

測試本機簽出的副本：

```sh
claude --plugin-dir /path/to/tenth-man
```

呼叫方式：

```text
/tenth-man:council 是否應該把工作階段狀態從 Redis 移到簽章 Cookie？
```

### 手動安裝

```sh
cp -r skills/council ~/.claude/skills/
cp agents/seat-*.md ~/.claude/agents/
```

手動安裝後的指令是 `/council`，席位名稱不帶 `tenth-man:` 前綴。

## 模型預設組合

問題較小時，主席會建議 `light`，其他情況則建議 `standard`。核准計畫時，你可以選擇預設組合，也可以變更任何席位。

第十人使用與推理能力最強的成員相同的模型。這個排名依據實測結果，而不是版本號碼；下表中的「第十人」欄反映的是 2026 年 10 月時的排名。新模型的排名未知時，主席會詢問你。若要使用其他模型，請在核准計畫、同意開始時說明。

### Codex

| 預設組合 | 投票成員 | 第十人 | 每輪執行次數 |
|---|---|---|---|
| `light` | GPT-6 Astra、GPT-6.1 Sol、GPT-6.1 Terra | GPT-6 Astra | 4 |
| `standard` | GPT-6 Astra、GPT-6.1 Sol、GPT-6.1 Terra、GPT-6 Sol | GPT-6 Astra | 5 |
| `no-astra` | GPT-6.1 Sol、GPT-6.1 Terra、GPT-6 Sol | GPT-6.1 Sol | 4 |

這些預設值只使用目前 Codex 主機和帳號提供的模型。主席會回報無法使用的模型，絕不會在未告知的情況下改用其他模型。如果主機的並行上限小於議事會的席位數，各席位會依據同一份未修改的任務說明分批執行；所有席位完成之前不會計票。

對應的 Codex 自訂代理是 `seat_astra`、`seat_sol`、`seat_terra`、`seat_previous_sol`，以及可選用的低成本 `seat_luna`。它們的 TOML 定義固定了模型和推理強度，並讓議事會的工作維持唯讀。

### Claude Code

| 預設組合 | 投票成員 | 第十人 | 每輪執行次數 |
|---|---|---|---|
| `light` | Fable、Opus、Sonnet | Fable | 4 |
| `standard` | Fable、Opus、Sonnet、Opus 5、Opus 4.8 | Fable | 6 |
| `no-fable` | Opus、Sonnet、Opus 5 | Opus | 4 |

即使 Fable 排名較高，每個預設組合也都保留一個 Opus 席位，因為 Opus 有時能發現 Fable 遺漏的問題。

Claude 外掛程式在 `agents/` 中包含五個不綁定角色的席位代理。每次執行的角色（成員或第十人）由任務說明指定。

### 加入 Claude 模型

可以透過代理定義中的 `model:` 欄位選擇較早的 Claude 模型。複製一個席位檔案，修改 `name`、`description` 和 `model`，然後儲存到 `~/.claude/agents/`（用於所有專案）或 `.claude/agents/`（用於單一專案）：

```markdown
---
name: seat-opus-4-7
description: A seat in a tenth-man council, on Opus 4.7.
model: claude-opus-4-7
effort: high
tools: Read, Grep, Glob, Bash, Write
---
```

內文維持與隨附席位相同、不綁定角色的內容。之後主席就可以將 `seat-opus-4-7` 指定為成員或第十人。

### 使用你自己的 Claude 代理

任何現有的 Claude 代理定義都可以擔任席位。主席會在提示詞開頭說明這次任務是擔任議事會席位，而不是該代理平常的工作，接著分配角色和共同的任務說明。對於固定在較早模型上的審查代理，這很實用。

### 固定的 Claude 模型停止提供時

如果固定的模型不再提供服務，該席位將無法啟動。主席會移除該席位並告知你，絕不會在未告知的情況下改用其他模型。請刪除或更新席位檔案，使其與你的帳號可用的模型一致。

## 成本與需求

每個席位每輪對應一次完整的子代理執行。三輪下來，Codex 的 `standard` 預設組合共執行 15 次，Claude Code 的 `standard` 預設組合共執行 18 次。

- Codex：已啟用子代理的目前版本，以及所選模型的存取權。
- Claude Code：Agent 和 SendMessage 工具、背景代理完成通知，以及所選模型的存取權。

目前不支援 ChatGPT 對話，因為它無法為每個席位使用不同的模型執行子代理。

## 為什麼叫「第十人」？

此技能的靈感來自 2013 年電影《末日之戰》（*World War Z*）中虛構的以色列規則「第十人」。當九個人根據相同的資訊得出相同的結論時，第十個人要假定他們是錯的，並調查相反的可能性。這條規則是虛構的，但現實中有類似的做法：

- 以色列軍事情報單位在 1973 年贖罪日戰爭前誤判了預警跡象，之後設立了一個專門質疑自身評估的職能，通常稱為 *ipcha mistabra*，這是亞拉姆語，意思是「很可能正好相反」。
- 《塔木德》（Sanhedrin 17a）記載，在死刑案件中，如果所有法官都投票判定有罪，被告就會獲釋；全體一致的結果表示沒有人認真為被告辯護。
- 在天主教的封聖審理中，「魔鬼代言人」負責反對候選人，並檢驗有關候選人生平與神蹟的說法中的薄弱之處。這個職位於 1587 年正式設立，1983 年大幅縮減。

## 授權條款

MIT。請參閱 [LICENSE](LICENSE)。
