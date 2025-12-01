構建下一代企業級 Agentic Service：基於 LangGraph、A2A、MCP 與全鏈路權限治理的架構藍圖
執行摘要
隨著生成式人工智慧（Generative AI）從實驗性的對話機器人（Chatbot）邁向生產級的代理服務（Agentic Service），企業正面臨著前所未有的架構挑戰。傳統的大型語言模型（LLM）應用往往局限於單次對話或簡單的檢索增強生成（RAG），缺乏處理複雜、長時程、多步驟業務流程的能力。根據最新的產業分析，約有 70% 的企業 AI 專案未能成功從原型階段過渡到生產環境，其核心原因在於系統的脆弱性、缺乏可觀察性以及整合的複雜度。
本研究報告提出了一項詳盡的企業級 Agentic Service 架構提案。該提案不依賴單一模型的能力，而是透過系統工程的方法，整合了目前業界最先進的四大技術支柱：
LangGraph：作為核心編排引擎，提供具備狀態管理（State Management）與循環執行（Cyclic Execution）能力的圖論式工作流。
Agent-to-Agent (A2A) Protocol：作為分散式通訊標準，解決異質代理（Heterogeneous Agents）之間的協作與互操作性問題。
Model Context Protocol (MCP)：作為通用工具介面，標準化代理與外部數據源及系統的連接。
Arize Phoenix 與 權限控管（RBAC）：作為維運與安全基石，提供基於 OpenTelemetry 的全鏈路追蹤與細粒度的訪問控制。
本報告將深入剖析這些技術如何協同工作，並特別針對「Claude Code CLI」這類現代化開發工具背後的多代理（Multi-Agent）建構邏輯進行解構，展示如何利用上述技術堆疊復刻並超越此類架構，為企業打造具備自主性、魯棒性與安全性的數位勞動力。
第一章 企業級代理編排的核心：LangGraph 的戰略優勢
在探討如何構建複雜的代理服務之前，必須先確立核心的編排（Orchestration）哲學。過去兩年，LangChain 的「鏈（Chain）」概念主導了市場，但隨著業務邏輯的複雜化，線性的 DAG（有向無環圖）已無法滿足需求。企業流程充滿了條件判斷、錯誤重試、循環優化與人機協作（Human-in-the-loop），這迫使架構必須向「狀態機（State Machine）」演進。
1.1 從線性鏈路到狀態圖譜的典範轉移
傳統的 Agent 框架往往將執行邏輯封裝在「黑盒子」中，開發者難以干預中間過程。LangGraph 的出現標誌著一種典範轉移：它將代理的思考過程外顯化為一張圖（Graph）。
1.1.1 狀態（State）作為一等公民
在 LangGraph 中，所有的上下文資訊、對話歷史、工具輸出與決策結果，都被封裝在一個共享的「狀態模式（State Schema）」中。這通常是一個 Python 的 TypedDict 或 Pydantic 模型。
不可變性與版本控制：雖然狀態在邏輯上是變動的，但在架構底層，每一次狀態的更新都會產生一個新的快照（Snapshot）。這使得系統具備了「時光旅行（Time Travel）」的能力，維運人員可以隨時將代理回滾到任意歷史步驟進行除錯或重試。
並行執行（Parallel Execution）：由於狀態是結構化的，LangGraph 可以輕易地將任務分發給多個節點並行處理（例如同時進行法律風險評估與財務數據檢索），最後再透過「歸約器（Reducer）」將結果合併回主狀態。
1.1.2 循環圖（Cyclic Graphs）與自我修正
企業級應用最關鍵的需求之一是「容錯」。線性工作流一旦失敗通常意味著整個流程的中斷。LangGraph 允許定義「循環邊（Cyclic Edges）」，這賦予了代理「自我反思（Self-Reflection）」的能力。 例如，在一個程式碼生成任務中，如果生成的程式碼未能通過單元測試，控制流不會終止，而是會攜帶錯誤訊息返回生成節點。代理會「看到」錯誤，並嘗試修正程式碼，直到通過測試或達到最大重試次數。這種認知架構（Cognitive Architecture）是區分玩具專案與生產級服務的分水嶺。
1.2 主流框架對比分析：為何選擇 LangGraph？
在目前的開源生態中，AutoGen、CrewAI 與 LlamaIndex Workflows 均佔有一席之地。然而，針對企業級生產環境的嚴苛要求，LangGraph 展現出了顯著的優勢。
評估維度
LangGraph
AutoGen
CrewAI
LlamaIndex Workflows
核心抽象
有狀態圖 (Stateful Graph)
對話式代理 (Conversational Agents)
基於角色的團隊 (Role-based Teams)
事件驅動工作流 (Event-driven)
控制流精細度
極高：支援低階圖操作、條件跳轉、循環
中：主要依賴對話互動，控制較鬆散
中：依賴角色定義與順序，靈活性受限
高：適合資料處理，但複雜邏輯較弱
狀態持久化
內建：支援 Postgres 等資料庫的 Checkpointing
需自行實作或依賴對話紀錄
依賴任務上下文，缺乏細粒度狀態回滾
上下文物件，主要用於 RAG 流程
人機協作 (HITL)
原生支援：可中斷 (Interrupt)、修改狀態後恢復
支援，但流程控制較為隱晦
較弱，主要關注代理間自動化
支援，透過事件觸發
企業適用性
最佳：適合複雜業務邏輯、審計與除錯
適合創意發想、模擬與多代理對話
適合快速原型驗證與明確分工場景
適合文件密集型的 RAG 應用

深度洞察： AutoGen 雖然在多代理對話（如軟體開發模擬）表現優異，但其「對話即控制流」的模式導致行為難以預測，且在需要嚴格遵循 SOP（標準作業程序）的企業場景中，往往會因為代理間的「閒聊」而浪費 Token 或偏離任務。 CrewAI 則過度封裝了底層邏輯，雖然上手容易，但當企業需要客製化錯誤處理或特殊的路由邏輯時，往往會遇到「框架限制」。 LangGraph 則選擇了「低階原語（Low-level Primitives）」路線，雖然學習曲線較陡峭，但它給予了架構師對系統行為的絕對控制權，這對於金融、醫療等高監管行業至關重要。
1.3 架構模式實踐：ReAct 與 Plan-and-Execute
在 LangGraph 中，我們不僅僅是連接 LLM，而是在實踐經典的認知架構。
ReAct (Reason + Act)：這是最基礎的模式。代理在圖的一個節點進行思考（Reasoning），決定是否調用工具；如果需要，則流轉到工具節點執行（Acting），並將結果寫回狀態，再回到思考節點。LangGraph 強制將這兩個步驟拆分為不同節點，使得工具調用的權限管控與錯誤隔離變得更加容易。
Plan-and-Execute：針對複雜任務（如「撰寫一份關於半導體市場的年度報告」），單一的 ReAct 循環容易迷失方向。在 LangGraph 中，我們會設計一個「規劃者（Planner）」節點負責生成步驟清單，並將其寫入狀態；接著由「執行者（Executor）」節點（可能是一個子圖）逐一執行這些步驟。這種分層架構大幅提升了長序列任務的成功率。
第二章 突破孤島：Agent-to-Agent (A2A) 通訊協議
在企業環境中，單一的大型超級代理（Super Agent）是難以維護且違反軟體工程原則的。真實的場景往往是由不同部門、不同團隊甚至不同公司開發的異質代理組成的「代理網格（Agent Mesh）」。如何讓一個由 Python/LangGraph 構建的「訂單代理」與一個由 Java/Semantic Kernel 構建的「庫存代理」無縫協作，是 A2A 協議解決的核心問題。
2.1 異質系統的通用語言
Google 推出的 Agent-to-Agent (A2A) 協議，旨在建立一個標準化的通訊層，讓代理之間能夠在不知道對方內部實作細節（Opaque）的情況下進行協作。這類似於微服務架構中的 REST 或 gRPC，但專為 AI 代理的特性進行了優化。
2.1.1 代理名片（Agent Card）：動態發現機制
A2A 協議的核心是「代理名片」。每個符合 A2A 標準的代理都必須在一個標準端點（通常是 /.well-known/agent.json）公開一份 JSON 格式的描述文件。這份名片包含了：
身份識別：代理的名稱、版本、描述。
技能清單（Skills）：代理能執行的具體任務描述（例如 check_inventory, calculate_tax）。這些描述不僅是給人類看的，更是為了讓其他代理的 LLM 能夠理解並決定是否調用此代理。
通訊模態：支援的輸入輸出格式（文字、JSON、檔案）。
認證規範：該代理接受的認證方式（如 OAuth2 scopes, Bearer Token）。
這種設計實現了「動態綁定」。一個客戶端代理（Client Agent）可以在執行時期查詢註冊中心，獲取目標代理的名片，並根據當下的需求動態決定調用哪一個代理，而無需在程式碼中硬編碼目標代理的 API 規格。
2.2 任務（Task）生命週期管理
與傳統 API 的「請求-回應」模式不同，A2A 引入了「任務」的概念。AI 代理的處理往往是耗時的（Long-running），且可能需要多輪互動。
狀態流轉：一個 A2A 任務會經歷 submitted（已提交）、working（處理中）、input-required（需要輸入）、completed（完成）或 failed（失敗）等狀態。
非同步與中斷：當被調用的遠端代理遇到需要澄清的問題（例如「請問您指的是哪一個倉庫？」），它可以將任務狀態設為 input-required 並掛起。主調代理收到此狀態後，可以去詢問人類使用者，然後再發送一條新的訊息來恢復該任務的執行。這種機制完美契合了 LangGraph 的人機協作模式。
2.3 實作細節：將 LangGraph 封裝為 A2A 服務
為了讓 LangGraph 代理成為 A2A 生態的一員，我們需要構建一個適配層（Adapter Layer）。
Transport Layer：使用 FastAPI 或 Starlette 建立 HTTP 服務，實作 JSON-RPC 2.0 介面。
Executor Mapping：繼承 A2A SDK 的 AgentExecutor 類別。當收到 message/send 請求時，將其轉換為 LangGraph 的 graph.invoke() 或 graph.stream() 調用。
State Synchronization：將 LangGraph 的 Checkpoint 映射到 A2A 的 taskId。這確保了當遠端客戶端針對同一個 taskId 發送後續訊息時，LangGraph 能從資料庫中載入正確的歷史狀態，繼續對話。
架構洞察： 透過 A2A 協議，我們實現了「代理即服務（Agent-as-a-Service）」。企業內部的各個職能部門（HR、IT、法務）可以各自維護自己的專門代理，並透過標準協議對外提供服務。這不僅實現了關注點分離（Separation of Concerns），也讓權限控管變得更加容易——法務代理的知識庫永遠不會直接暴露給 IT 代理，雙方僅透過定義好的「技能」進行互動。
第三章 連接萬物的介面：Model Context Protocol (MCP)
如果說 A2A 解決了「代理與代理」的溝通，那麼 Anthropic 推出的 Model Context Protocol (MCP) 則解決了「代理與世界」的連接問題。在過去，要讓 LLM 讀取本地檔案、查詢資料庫或操作 GitHub，開發者需要編寫大量的膠水程式碼（Glue Code）。MCP 的出現，試圖將這些整合工作標準化為「USB-C」般的通用介面。
3.1 深度解析 Claude Code CLI 的多代理架構
針對使用者提出的「Claude Code CLI 背後的多代理是如何構建的」這一問題，雖然官方未開源其底層程式碼，但基於 MCP 的設計哲學與主流 Agentic 架構（如 LangGraph Supervisor），我們可以推導出其架構藍圖。這是一個典型的 客戶端代理編排（Client-side Agent Orchestration） 案例。
3.1.1 架構重構：Supervisor 模式與 MCP 的結合
Claude Code CLI 本質上是一個運行在使用者終端機（Terminal）上的 Orchestrator（編排者）。它並不將所有工具邏輯內建於模型中，而是透過 MCP 連接本地資源。
本地 MCP Server 群組：
FileSystem Server：這是一個 MCP Server，負責提供 list_files, read_file, edit_file 等工具。它運行在本地進程中，擁有對使用者專案目錄的存取權限。
Terminal Server：另一個 MCP Server，提供 execute_command 工具，允許代理執行 grep, git 等系統指令。
Knowledge Server：可能連接到本地的向量資料庫或文件索引，提供代碼庫的語義檢索能力。
核心編排邏輯（The Supervisor）： CLI 的核心是一個基於 LangGraph 或類似狀態機的 Supervisor Agent。
規劃（Planning）：當使用者輸入「請重構 auth.py 中的登入邏輯」時，Supervisor 首先會生成一個計劃：「1. 讀取 auth.py, 2. 分析依賴, 3. 搜尋相關測試, 4. 執行修改, 5. 運行測試」。
分派（Delegation）：Supervisor 不會自己去讀檔，而是生成一個工具調用請求（Tool Call）。由於它透過 MCP 連接了 FileSystem Server，這個請求會被轉發給該 Server 執行。
工具執行與回饋：MCP Server 執行操作後返回結果（如檔案內容或 grep 輸出）。Supervisor 接收結果，寫入狀態（State），然後決定下一步行動。
多代理協作（Multi-Agent Collaboration）： 在更複雜的場景下，CLI 內部可能運行著多個專門的 Sub-agents：
Coder Agent：專注於程式碼生成，擁有 FileSystem 的寫入權限。
Reviewer Agent：專注於代碼審查，擁有 Read-only 權限，負責在 Coder 修改後進行檢查。
Test Agent：專注於運行測試指令並解析錯誤日誌。 這些 Agent 在 CLI 內部透過共享狀態（Shared State）或 A2A 協議進行協作，共同完成使用者的指令。
3.1.2 為何這種架構是主流趨勢？
安全性隔離：工具（Tools）運行在獨立的 MCP Server 進程中。即使工具崩潰或被惡意指令影響，主 Agent 進程仍能保持穩定。
可插拔性（Pluggability）：使用者可以隨意安裝新的 MCP Server（例如連接到 PostgreSQL 的 Server），Claude Code CLI 就能立刻獲得操作資料庫的能力，而無需更新 CLI 本身。
上下文視窗優化：MCP 允許動態加載資源（Resources）和提示詞（Prompts）。Agent 不需要將整個專案的結構都塞進 Context Window，而是可以根據 MCP 的目錄結構按需讀取。
3.2 MCP 在企業架構中的實踐
在我們的提案中，LangGraph Agent 將扮演 MCP Client 的角色。我們利用 langchain-mcp-adapters 來連接企業內部的各種 MCP Server。
資源抽象化：企業的資料庫 schema、API 文件、內部 Wiki，都可以封裝成 MCP Resources。Agent 可以像瀏覽檔案系統一樣瀏覽這些資源。
傳輸層選擇：
Stdio 傳輸：適用於 Sidecar 模式。在 Kubernetes Pod 中，Agent 容器可以透過 Stdio 與同 Pod 內的 MCP Server 容器通訊，實現極低延遲且安全的本地工具調用。
SSE (Server-Sent Events) 傳輸：適用於遠端服務。Agent 可以連接到部署在其他伺服器上的 MCP Server，這對於需要集中管理的工具（如統一的身分驗證服務）非常有用。
第四章 運維智慧：Arize Phoenix 與全鏈路可觀察性
隨著 Agent 系統的複雜度增加，「黑盒子」問題變得無法忽視。當一個 Agent 陷入無限迴圈，或者在多個 Agent 之間發生「踢皮球」現象時，傳統的日誌（Logs）已不足以定位問題。我們需要的是基於 OpenTelemetry (OTEL) 的分散式追蹤（Distributed Tracing）。
4.1 全鏈路追蹤的實作
Arize Phoenix 是一個專為 LLM 應用設計的可觀察性平台。在我們的架構中，它扮演著「行車記錄器」的角色。
4.1.1 跨代理的上下文傳播（Context Propagation）
當 LangGraph Agent A 透過 A2A 協議呼叫 Agent B 時，這是一次跨進程甚至跨網路的調用。為了將這兩段執行串聯成一個完整的 Trace，我們必須利用 OTEL 的 Baggage 機制。
實作機制：在 Agent A 發起 JSON-RPC 請求時，A2A SDK 會自動攔截請求，並將當前的 traceparent ID 注入到 HTTP Header 中。
子跨度（Child Span）：Agent B 的 A2A Server 接收到請求後，會解析 Header 中的 traceparent，並以此為父 ID (Parent ID) 創建一個新的 Span。
視覺化：在 Arize Phoenix 的儀表板上，維運人員可以看到一個跨越兩個服務的完整瀑布圖（Waterfall Chart）。這讓我們能精確測量：是 Agent A 的思考耗時太久？還是網路傳輸延遲？亦或是 Agent B 的工具執行效率低落？
4.1.2 針對 Agent 的特有指標
除了傳統的延遲（Latency）與錯誤率，Phoenix 還提供了針對 Agent 的關鍵指標：
Token Usage & Cost：精確計算每一步驟、每一個工具調用的 Token 消耗，這對於成本控管至關重要。
Retrieval Quality：對於 RAG 環節，Phoenix 能追蹤檢索到的文件塊（Chunks）以及它們與問題的相關性分數。這有助於診斷「幻覺」問題——是因為檢索到了錯誤的資訊，還是模型推理錯誤？
Tool Usage Analytics：統計哪些工具最常被使用，哪些工具頻繁報錯。這有助於優化 MCP Server 的性能。
4.2 評估與持續改進（Evaluation & CI/CD）
可觀察性只是第一步，更重要的是「評估」。Phoenix 支援將生產環境的 Trace 數據轉化為測試資料集（Datasets）。
黃金資料集（Golden Datasets）：將使用者反饋良好的對話標記為黃金樣本。
回歸測試：當開發者修改了 LangGraph 的 Prompt 或流程邏輯後，可以在 CI/CD pipeline 中使用 Phoenix 的 Evaluators（通常是另一個強大的 LLM）來自動評估新版本的表現。例如，檢查「回應是否包含幻覺」、「是否正確調用了工具」。
第五章 零信任安全架構：權限控管與數據主權
在 Agentic Service 中，安全不再是外掛的防火牆，而是必須內建於圖（Graph）的每一個節點中。Agent 擁有執行工具的能力，這意味著如果沒有適當的管控，它們可能成為強大的攻擊向量。
5.1 身份驗證與授權的解耦
我們必須嚴格區分 身份驗證（AuthN） 與 授權（AuthZ）。
AuthN：在 A2A 連線建立或 API 呼叫時完成。通常使用 OIDC (OpenID Connect) 令牌。LangGraph 服務接收到請求後，驗證 Token 的簽名與時效，確認呼叫者是「User A」或「Service B」。
AuthZ：這是 LangGraph 內部的職責。
5.2 基於圖狀態的 RBAC 實作
在 LangGraph 中，權限不應該是全域變數，而應該是 狀態（State） 的一部分。
狀態注入：在工作流初始化的節點，系統會將解碼後的 User Roles 與 Permissions 注入到 State 或 Config 中（例如 config['configurable']['user_roles'] = ['hr_manager']）。
節點級防護：每一個涉及敏感操作的工具節點（Tool Node），在執行前都必須檢查 State 中的權限。
範例：一個「刪除資料庫」的工具，其 Python 函數開頭必須包含 if 'db_admin' not in state.user_roles: raise PermissionError。
動態邊（Conditional Edges）的路由控制：我們可以在圖的邊上實施權限邏輯。如果使用者沒有權限，控制流可以直接路由到一個「拒絕訪問」的回覆節點，而根本不進入工具執行的分支。
5.3 RAG 場景下的列級安全（Row-Level Security）
當 Agent 需要檢索企業知識庫時，必須防止「資料越權」。
向量庫的元數據過濾：在構建向量索引時，每一份文件都必須打上權限標籤（例如 access_group: "finance_dept"）。
查詢時的過濾注入：當 Agent 執行檢索時，不能僅僅根據語義相似度搜索。系統必須強制將使用者的權限作為過濾條件（Filter）注入到向量資料庫的查詢中。
查詢重寫：vector_store.similarity_search(query, filter={"access_group": {"$in": user.groups}})。這確保了使用者永遠不會檢索到他們無權查看的文件片段，從根本上杜絕了 RAG 系統的資料洩露風險。
5.4 隱私保護與 PII 脫敏
在將 Trace 數據發送到 Arize Phoenix（特別是當使用 SaaS 版本時）之前，必須進行個人隱私資訊（PII）的清洗。
LangChain PII Masking Middleware：在 LangGraph 的執行鏈路中，我們可以插入一個 PII 清洗層。利用 Microsoft Presidio 或正則表達式，自動偵測並遮蔽 Email、身分證號、信用卡號等敏感資訊。
替換策略：將敏感資訊替換為 `` 或雜湊值。這確保了在可觀察性平台上的數據是匿名的，同時保留了除錯所需的結構資訊（例如，我們知道這裡有一個 Email，但不知道具體內容）。
第六章 實施策略與投資回報 (ROI)
6.1 分階段導入路徑
為了降低風險，建議企業採取「三步走」的導入策略：
第一階段：核心基礎建設（The Foundation）
部署 LangGraph Supervisor 節點，實作基本的 ReAct 模式。
建立 Postgres Checkpointer 以實現狀態持久化。
導入 Arize Phoenix 進行初步的 Trace 收集。
目標：建立一個穩定、可觀測的單體 Agent。
第二階段：連接與協作（Connectivity）
將 Supervisor 封裝為 A2A Server。
開發 1-2 個專門的 A2A Remote Agents（例如「IT 支援代理」）。
部署 MCP Server 連接關鍵資料庫。
目標：驗證跨 Agent 的協作能力與 MCP 的工具抽象化。
第三階段：安全與規模化（Governance & Scale）
全面實施 RBAC 與 RAG RLS。
開啟 PII Masking。
建立基於 Phoenix Datasets 的自動化評估流水線。
目標：達到生產級的安全標準，並開始大規模推廣。
6.2 關鍵績效指標 (KPIs)
衡量 Agentic Service 成功的指標不同於傳統軟體：
圍堵率 (Containment Rate)：Agent 能夠獨立解決、無需轉接人工的任務比例。
任務成功率 (Task Success Rate)：在 A2A 協作中，任務狀態最終標記為 completed 的比例。
平均解決時間 (MTTR)：從使用者提出請求到任務完成的時間（需扣除等待使用者輸入的時間）。
Token 效率 (Token Efficiency)：完成單位任務所需的 Token 數量與成本。
6.3 結論
本提案所描繪的架構，不再是拼湊的玩具，而是一個嚴謹的企業級系統。透過 LangGraph，我們獲得了對複雜邏輯的絕對控制；透過 A2A，我們打破了孤島，實現了生態系的協作；透過 MCP，我們標準化了工具的連接；透過 Arize Phoenix，我們點亮了黑盒子；最後，透過 RBAC，我們確保了這一切都在安全的軌道上運行。
這不僅僅是一個技術堆疊的升級，更是企業數位勞動力（Digital Workforce）的基礎設施建設。隨著這些技術的成熟，Agentic Service 將成為企業自動化與智慧決策的核心引擎，釋放出前所未有的生產力。
