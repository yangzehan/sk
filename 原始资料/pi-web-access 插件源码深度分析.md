 pi-web-access 插件源码深度分析                                                                                                                                                                                                                                             

 一、插件定位                                                                                                                                                                                                                                                               

 pi-web-access v0.10.7 (Nico Bailon) 是一个把"网络访问能力"全套嵌入 Pi 的扩展。源码 11683 行 TS + 3359 行 curator HTML,核心目标是让 Agent "零配置" 就能搜索、抓取、读 GitHub、看 YouTube、看本地视频、看 PDF,且每条能力都有多级 fallback 链保证可用性。                     

 注册入口在 index.ts:1088-1990+,主依赖是 pi-coding-agent 的 ExtensionAPI(工具/命令/快捷键/widget 注册) + pi-ai(complete 调用) + pi-tui(widget 渲染)。                                                                                                                       

 ────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                           

 二、4 个 LLM 工具(registerTool)                                                                                                                                                                                                                                            

 ┌────────────────────┬───────────────┬────────────────────────────────────────────────────────────────┐                                                                                                                                                                    

 │ 工具              │ 行号         │ 功能                                                          │                                                                                                                                                                    

 ├────────────────────┼───────────────┼────────────────────────────────────────────────────────────────┤                                                                                                                                                                    

 │ web_search         │ index.ts:1088 │ 多查询、多 provider、可选自动打开 curator 浏览器审核           │                                                                                                                                                                    

 ├────────────────────┼───────────────┼────────────────────────────────────────────────────────────────┤                                                                                                                                                                    

 │ code_search        │ index.ts:1531 │ 编程问题专用,经 Exa MCP 拿代码片段/GitHub/SO/官方文档          │                                                                                                                                                                    

 ├────────────────────┼───────────────┼────────────────────────────────────────────────────────────────┤                                                                                                                                                                    

 │ fetch_content      │ index.ts:1574 │ 路由 + 提取: GitHub 克隆、YouTube、PDF、本地视频、网页、帧抽取 │                                                                                                                                                                    

 ├────────────────────┼───────────────┼────────────────────────────────────────────────────────────────┤                                                                                                                                                                    

 │ get_search_content │ index.ts:1819 │ 按 responseId 从 storage 取之前未取全的内容(>30k 字符会截断)   │                                                                                                                                                                    

 └────────────────────┴───────────────┴────────────────────────────────────────────────────────────────┘                                                                                                                                                                    

 每个工具都实现了 renderCall + renderResult 两个 TUI 渲染钩子,产出带颜色、进度条、图片标记([image]/[truncated]/帧数)、状态码的紧凑输出。                                                                                                                                    

 ────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                           

 三、4 个斜杠命令(registerCommand)                                                                                                                                                                                                                                          

 ┌──────────────────────────────────┬───────────────┬───────────────────────────────────────────────────────────────────┐                                                                                                                                                   

 │ 命令                            │ 行号         │ 作用                                                             │                                                                                                                                                   

 ├──────────────────────────────────┼───────────────┼───────────────────────────────────────────────────────────────────┤                                                                                                                                                   

 │ /websearch [q1, q2...]           │ index.ts:1977 │ 直接拉起 curator 浏览器(可预填查询)                               │                                                                                                                                                   

 ├──────────────────────────────────┼───────────────┼───────────────────────────────────────────────────────────────────┤                                                                                                                                                   

 │ /curator [on/off/summary-review] │ index.ts:2203 │ 切换 curator 工作流并持久化到 web-search.json                     │                                                                                                                                                   

 ├──────────────────────────────────┼───────────────┼───────────────────────────────────────────────────────────────────┤                                                                                                                                                   

 │ /google-account                  │ index.ts:2243 │ 探测 Chrome cookie,告诉你 Gemini Web 当前登录的是哪个 Google 账号 │                                                                                                                                                   

 ├──────────────────────────────────┼───────────────┼───────────────────────────────────────────────────────────────────┤                                                                                                                                                   

 │ /search                          │ index.ts:2281 │ 浏览本会话所有存储的 responseId,支持查看详情/删除                 │                                                                                                                                                   

 └──────────────────────────────────┴───────────────┴───────────────────────────────────────────────────────────────────┘                                                                                                                                                   

 ────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                           

 四、快捷键 + 实时活动 widget                                                                                                                                                                                                                                               

 ```ts                                                                                                                                                                                                                                                                      

   // index.ts:1044 / :1057                                                                                                                                                                                                                                                 

   pi.registerShortcut("ctrl+shift+s", ...)  // 打开 curator                                                                                                                                                                                                                

   pi.registerShortcut("ctrl+shift+w", ...)  // 切换 web-activity widget                                                                                                                                                                                                    

 ```                                                                                                                                                                                                                                                                        

 updateWidget()(index.ts:390)把 ActivityMonitor(activity.ts,保留最近 10 条,带 Perplexity 10/分钟限速计数)实时画成 TUI widget:                                                                                                                                               

 ```                                                                                                                                                                                                                                                                        

   ─── Web Search Activity ────────────                                                                                                                                                                                                                                     

     API  "typescript best practices"   200  2.1s ✓                                                                                                                                                                                                                         

     GET  docs.example.com/article      200  0.8s ✓                                                                                                                                                                                                                         

     GET  blog.example.com/post         404  0.3s ✗                                                                                                                                                                                                                         

   ────────────────────────────────────                                                                                                                                                                                                                                     

   Rate: 3/10                                                                                                                                                                                                                                                               

 ```                                                                                                                                                                                                                                                                        

 ────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                           

 五、最大亮点:Curator 互动审核流(自研)                                                                                                                                                                                                                                      

 这是这个插件的灵魂,跟普通 web search 最大的差异化所在。                                                                                                                                                                                                                    

 架构: 每次 web_search 都会:                                                                                                                                                                                                                                                

 1. 异步起一个 node:http 服务器 (curator-server.ts,绑定 loopback + 随机 sessionToken,只接受带 token 的 SSE 客户端)                                                                                                                                                          

 2. 拉起浏览器(macOS 优先用 glimpseui 的原生 WKWebView 窗口,否则 open/xdg-open/start)(index.ts:2122)                                                                                                                                                                        

 3. 流式推送结果到那个页面,用户可以:                                                                                                                                                                                                                                        

     - 选/取消每条结果                                                                                                                                                                                                                                                      

     - 点 "Also try" 切换 provider 重搜同 query(并排比较)                                                                                                                                                                                                                   

     - 手动 ✨ 改写查询                                                                                                                                                                                                                                                     

     - 调 timeout 倒计时(默认 20s,最大 600s)                                                                                                                                                                                                                                

 4. 用户点 Approve → 服务端把 selectedQueryIndices + summary 推回 Agent                                                                                                                                                                                                     

 5. 超时 fallback:无人工操作时,自动用 buildDeterministicSummary() 拼一份降级摘要,确保不卡死                                                                                                                                                                                 

 关键工程细节:                                                                                                                                                                                                                                                              

 - HTTP 路由:/, /events(SSE), /provider, /search, /summarize, /submit, /cancel, /heartbeat                                                                                                                                                                                  

 - STALE_THRESHOLD_MS = 30s 心跳 watchdog,断网时自动 cancel                                                                                                                                                                                                                 

 - 状态机:SEARCHING → RESULT_SELECTION → COMPLETED                                                                                                                                                                                                                          

 - 重连缓冲: SSE 断线时未发的事件在 sseBuffer 保留                                                                                                                                                                                                                          

 - summary 生成走 summary-review.ts,首选 claude-haiku-4-5 / gpt-5.3-codex-spark,model 失败会自动 retry 一次;空响应降级为确定性摘要并打 fallbackReason 标记                                                                                                                  

 - 摘要面板支持反馈文本(用 selected 文本片段追加 prompt),可预览/再生成                                                                                                                                                                                                      

 - 提交通过 pi.sendMessage({ deliverAs: "followUp", triggerTurn: true }) 注入对话                                                                                                                                                                                           

 │ 这相当于把 Perplexity 的 "Pro Search" 审核 UI 用 60KB HTML + SSE 自己复刻了。                                                                                                                                                                                            

 ────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                           

 六、搜索 provider fallback 链(gemini-search.ts:115-186)                                                                                                                                                                                                                    

 ```                                                                                                                                                                                                                                                                        

   auto mode:                                                                                                                                                                                                                                                               

     Exa(MCP 零配置 / 或 API key)  → 失败 ↓                                                                                                                                                                                                                                 

     Perplexity  (env 或 web-search.json)  → 失败 ↓                                                                                                                                                                                                                         

     Gemini API  (env 或 web-search.json)  → 失败 ↓                                                                                                                                                                                                                         

     Gemini Web  (从 Chrome/Arc/Helium/Chromium 提 cookie)  → 失败则 throw                                                                                                                                                                                                  

 ```                                                                                                                                                                                                                                                                        

 - exa.ts: 1000 req/月免费档(80% 警告),状态写入 ~/.pi/exa-usage.json                                                                                                                                                                                                        

 - perplexity.ts: 客户端 10 req/min 滑窗限速                                                                                                                                                                                                                                

 - gemini-api.ts: REST generateContent + grounding chunks                                                                                                                                                                                                                   

 - gemini-web.ts: 反向工程 gemini.google.com/_/BardChatUi/.../StreamGenerate + MODEL_HEADERS 表(gemini-3-pro / 2.5-pro / 2.5-flash 三档)                                                                                                                                    

 - chrome-cookies.ts: macOS Keychain(security find-generic-password) + Linux secret-tool + Chromium SQLite(pbkdf2Sync("saltysalt", 1003)),copyFileSync 临时拷贝避免锁库                                                                                                     

 ────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                           

 七、内容提取 fallback 链(extract.ts:212-441)                                                                                                                                                                                                                               

 ```                                                                                                                                                                                                                                                                        

   fetch_content(url):                                                                                                                                                                                                                                                      

     1. 本地文件? → 抽帧 / Gemini Files API                                                                                                                                                                                                                                 

     2. 帧参数?   → yt-dlp + ffmpeg 抽帧(支持 timestamp="23:41-25:00", frames=4)                                                                                                                                                                                            

     3. URL 合法?                                                                                                                                                                                                                                                           

        4. GitHub? → 克隆(>350MB 走 API)→ 返回文件树/内容 + 本地路径让 Agent 用 read/bash 探索                                                                                                                                                                              

        5. YouTube? → Gemini Web → Gemini API → Perplexity 兜底                                                                                                                                                                                                             

        6. PDF?     → unpdf 抽文本存到 ~/Downloads/*.md                                                                                                                                                                                                                     

        7. 普通 HTTP                                                                                                                                                                                                                                                        

           a. fetch + Readability(@mozilla/readability) + linkedom + Turndown → markdown                                                                                                                                                                                    

           b. RSC? → rsc-extract.ts 解析 Next.js `self.__next_f.push([1,"..."])`                                                                                                                                                                                            

           c. 失败? → Jina Reader(r.jina.ai, 无需 key,服务器端 JS 渲染)                                                                                                                                                                                                     

           d. 还失败? → Gemini URL Context / Gemini Web 反向提                                                                                                                                                                                                              

           e. 全部失败 → 给出可操作错误信息和所有尝试过的 provider 状态                                                                                                                                                                                                     

 ```                                                                                                                                                                                                                                                                        

 并发控制 pLimit(3),每 URL 30s 超时(p-limit 调度)。                                                                                                                                                                                                                         

 ────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                           

 八、其它增强点                                                                                                                                                                                                                                                             

 1. GitHub 真克隆(github-extract.ts, 634 行)                                                                                                                                                                                                                                

     - 识别 root / blob / tree 三种 URL,commit SHA 走 API                                                                                                                                                                                                                   

     - cloneCache session 内复用,session_shutdown 清空                                                                                                                                                                                                                      

     - 跳过 node_modules, .next, dist 等噪声目录                                                                                                                                                                                                                            

     - 巨仓库(>350MB)走 API fallback,forceClone: true 强制克隆                                                                                                                                                                                                              

     - Issues/PRs/Wiki 路径直接 fall through 走普通 HTTP 抓取                                                                                                                                                                                                               

 2. YouTube 三段处理(youtube-extract.ts, 310 行)                                                                                                                                                                                                                            

     - 识别 /watch?v=、/shorts/、/live/、/embed/、/v/、youtu.be/                                                                                                                                                                                                            

     - Gemini Web → Gemini API → Perplexity 三档 fallback                                                                                                                                                                                                                   

     - 用 yt-dlp 拿流 URL,ffmpeg 抽帧                                                                                                                                                                                                                                       

     - 时间戳接受 H:MM:SS / MM:SS / 纯秒                                                                                                                                                                                                                                    

 3. 本地视频(video-extract.ts)                                                                                                                                                                                                                                              

     - 走 Gemini Files API upload(>50MB 拒绝)                                                                                                                                                                                                                               

     - ffmpeg 抽缩略图                                                                                                                                                                                                                                                      

 4. Next.js RSC 解析器(rsc-extract.ts, 338 行)                                                                                                                                                                                                                              

     - 解析 self.__next_f.push([1,"..."]) 里的 hex-id → JSON chunk                                                                                                                                                                                                          

     - 跟踪 ref 链,递归把 RSC 树还原成 markdown                                                                                                                                                                                                                             

     - 比 Readability 在 Next.js SPA 上拿到的内容多得多                                                                                                                                                                                                                     

 5. PDF 提取(pdf-extract.ts, 192 行)                                                                                                                                                                                                                                        

     - unpdf 抽文本,存到 ~/Downloads/,Agent 后续可分段 read 不爆上下文                                                                                                                                                                                                      

 6. 会话级存储与恢复(storage.ts)                                                                                                                                                                                                                                            

     - restoreFromSession(ctx) 读 ctx.sessionManager.getBranch() 找 customType === "web-search-results" 的条目                                                                                                                                                              

     - TTL 1h,get_search_content 跨调用按 responseId 取                                                                                                                                                                                                                     

     - session_start / session_tree / session_shutdown 全生命周期管理                                                                                                                                                                                                       

 7. 可观测性(activity.ts)                                                                                                                                                                                                                                                   

     - 全局 activityMonitor 单例                                                                                                                                                                                                                                            

     - 所有 API/URL 调用都登记,onUpdate 订阅推 widget                                                                                                                                                                                                                       

     - Perplexity 限速状态也合入 widget 底部                                                                                                                                                                                                                                

 8. 模型保护(index.ts + summary-review.ts)                                                                                                                                                                                                                                  

     - getApiKeyAndHeaders()(适配 pi 0.63+ 鉴权迁移)                                                                                                                                                                                                                        

     - 关键 LLM 摘要调用支持 model provider guard retry                                                                                                                                                                                                                     

     - query rewrite 用 haiku/flash 这种快模型                                                                                                                                                                                                                              

 9. 配置系统(index.ts:35-103)                                                                                                                                                                                                                                               

     - 单一真相源:~/.pi/web-search.json,所有功能启停都在此                                                                                                                                                                                                                  

     - 缺省值: curatorTimeoutSeconds=20, maxRepoSizeMB=350, searchModel=gemini-2.5-flash                                                                                                                                                                                    

     - 改 provider 会在 curator 里实时写回配置                                                                                                                                                                                                                              

     - env vars (EXA_API_KEY / GEMINI_API_KEY / PERPLEXITY_API_KEY / PI_ALLOW_BROWSER_COOKIES=1) 优先                                                                                                                                                                       

 ────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                           

 九、CHANGELOG 显示的近期重点演进(0.10.x)                                                                                                                                                                                                                                   

 - 0.10.7: summaryModel 可选、修复 Chrome cookie 提示突兀化、修复 Exa 移除 get_code_context_exa MCP 工具后 code_search 退化、迁移 typebox 导入                                                                                                                              

 - 0.10.6: 给 4 个工具加 promptSnippet,让 Pi 0.59+ 在默认 system prompt 工具区带上它们(关键可发现性)                                                                                                                                                                        

 - 0.10.5: 把 ctx.modelRegistry.getApiKeyAndHeaders() 透传到 complete() 修鉴权                                                                                                                                                                                              

 - 0.10.4: Curator 硬切换(workflow: "none" | "summary-review",删除 result-review)、所有 web_search 默认开 curator、Exa + Exa MCP 集成、code_search 工具、Glimpse 原生窗口、按 provider 分组的重搜 ✨、摘要 panel 全套交互、query rewrite wand、per-card 重搜、SSE 缓冲不丢  

   事件等                                                                                                                                                                                                                                                                   

 ────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                           

 十、内置 Skill                                                                                                                                                                                                                                                             

 skills/librarian/SKILL.md — 教 Agent "研究开源库时,clone → grep → blame → 拼 GitHub permalink" 的工作流。配合 fetch_content clone GitHub 这个能力,变成一个完整的代码考古工具链。                                                                                           

 ────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                           

 总结:这个插件给 Pi 加了什么                                                                                                                                                                                                                                                

 1. 4 个工具 把"读外部世界"完整化: web_search / code_search / fetch_content / get_search_content                                                                                                                                                                            

 2. 4 个命令 + 2 个快捷键 + 1 个实时 widget = TUI 端的完整体验                                                                                                                                                                                                              

 3. Curator 互动审核 是杀手锏,这是 Pi 之外的默认 web search 给不出的"人类在 loop 里"能力                                                                                                                                                                                    

 4. 多 provider 链 + 多提取链 最大化"总能拿到点什么"的鲁棒性(Jina Reader 兜底常见 403,本地视频/YouTube 都有 Gemini 双通道)                                                                                                                                                  

 5. GitHub 真克隆让 Agent 拿到代码后能用 read/bash 进一步分析,不是干看 HTML                                                                                                                                                                                                 

 6. Chrome cookie 自动提让"用你正在登录的 Gemini 账号"零门槛可用                                                                                                                                                                                                            

 7. 会话级存储 + 大内容懒加载 防止长 fetch 直接撑爆上下文                                                                                                                                                                                                                   

 8. 可观测性 widget + 可配置快捷键 + 可热切换工作流 都是细节拉满的设计                                                                                                                                                                                                      

 简单说:把 Pi 从"纯本地工具"升级成"能搜、能读库、能看视频、能审结果再决定用哪条"的完整研究型 Agent。