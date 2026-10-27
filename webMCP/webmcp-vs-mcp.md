# WebMCP and MCP compared

**Status:** draft for comment, October 2026
**Comments and questions:** please open an issue or add a review comment against any section. Section and row labels are there so comments can point to them.

This note explains what WebMCP is for relative to MCP, compares the browser-side WebMCP proposal with the server-side Model Context Protocol (MCP), sets out the benefits and risks of each approach, and lists open questions where development could patch weaknesses on one side using strengths of the other.

Every claim carries a source number from the reference list at the end.

Primary texts: the WebMCP Draft Community Group Report of 20 October 2026 [24] and the MCP 2026-07-28 changelog [25]. Secondary sources are used for implementation status, Chrome behaviour and commentary.

---

## 1. Role of WebMCP relative to MCP

**Short answer.** WebMCP does not depend on MCP and does not run on it. A page registers tools with the browser, and the browser hands them to an agent; no MCP server, client or transport is involved [1][21]. The draft does not prescribe the format in which a browser exposes tools to its agent [24]. The README nonetheless describes WebMCP as designed to complement, not replace, backend protocols such as MCP [19]. These two statements fit together only if "complement" is read as a division of work. The documentation reviewed does not define the term.

**What "complement" appears to mean.** The README contrasts two separate flows: an in-browser flow, where a browser agent uses a page's WebMCP tools, and a direct backend flow, where an agent platform calls a service's MCP server [19]. Read that way, the two are alternative channels to the same service, chosen by context:

| Situation | Channel | Sources |
|---|---|---|
| User has the site open in a browser and works with an agent there | WebMCP tools on the page | [19][1] |
| Agent works headless, in the background or server-to-server | The service's MCP server | [19][1] |
| Product wants both | Site runs an MCP server and also registers WebMCP tools on its pages; each serves different agents | [1] |

On this reading the complement is at the level of a product's architecture: neither protocol calls the other.

**Where the two do touch.**

- **Shared concept.** Both define a tool as a name, a natural-language description and a JSON Schema input [2]. An early design discussion records the intention that WebMCP align closely with MCP so that developers can carry skills from one to the other [23].
- **Shared name and framing.** The draft's introduction describes pages using WebMCP as Model Context Protocol servers that implement tools in client-side script, and the README calls them "in-page MCP servers" [24][19]. The draft's security section says it adopts MCP's risk assessments and mitigations [24]. The technical notes argued in February that this framing misleads, because WebMCP does not use the MCP wire protocol [21].
- **Optional bridge.** The draft leaves open how a browser passes tools to its agent: through MCP, a vendor's own function-calling method, or any other way [24]. WebKit cites this to argue that WebMCP is not really an MCP binding [22]. Community bridges translate WebMCP tools into MCP for desktop clients; they are not official W3C or MCP projects [17].

**Functions WebMCP fulfils that a backend MCP server does not, and why**

| ID | Function | Why MCP does not cover it | Sources |
|---|---|---|---|
| R1 | Reach the long tail of websites | An MCP integration needs a server the service builds and runs; the W3C Strategy Team judges it unlikely that MCP will ever offer endpoints on the scale of websites | [20] |
| R2 | Act on the state of the open page | Logins, carts and on-screen filters exist inside the browser tab; a backend server would have to rebuild authentication and synchronisation to see them | [2] |
| R3 | Reuse the user's existing sign-in | MCP authorization is OAuth-based; WebMCP inherits the browser session. The idea began when an Amazon engineer found internal services had not implemented the OAuth flow MCP required, while users were already signed in through the browser | [21] |
| R4 | Keep the human in the loop | WebMCP targets cooperative workflows where the user works with the agent; declarative tools fill a visible form and wait for the user to submit. MCP calls run between the agent platform and the server with no page in front of the user | [19][1] |
| R5 | Use the browser as mediator | The browser already provides sandboxing, origin-based trust, discovery and a place for the human to step in, which is why service providers keep relying on websites | [20] |
| R6 | Give the site control over agent access | The site declares which actions agents may call, in place of agents scraping the page or reading screenshots | [6] |

**What WebMCP cannot do that MCP can.** WebMCP tools exist only while a browser has the page open; headless and background use are unsupported [1]. Account-wide, scheduled or server-to-server operations remain MCP territory, and the two can run side by side in one product [1].

**Open challenge to the premise.** WebKit questions whether a separate agent-facing tool layer is needed at all, arguing that hard-to-use pages are a gap in HTML and ARIA semantics best fixed for users, assistive technology and agents together [22].

**Points the documentation leaves open (for comment).** These are questions, not findings. Each records something the sources reviewed for this note do not settle.

| ID | Question |
|---|---|
| Q1 | What does "complement" mean in the README: product-level coexistence of two channels, or an intended technical link between WebMCP and MCP? |
| Q2 | The draft leaves the exposure format to each browser [24]. Should it name a preferred or baseline format, and should MCP be that format? |
| Q3 | If each browser vendor chooses its own format, can a site expect the same tools to behave the same way across browsers and agents? |
| Q4 | Should a site that runs both an MCP server and WebMCP tools keep names, schemas and descriptions aligned? The sources reviewed give no guidance. |
| Q5 | The draft still describes pages using WebMCP as MCP servers implemented in client-side script [24]. In what sense, given that WebMCP uses no MCP transport? |
| Q6 | Why keep "MCP" in the name of an API that does not use the protocol, and does the name lead developers and security reviewers to assume MCP's threat model? (raised in [21]) |
| Q7 | MCP Apps lets a server render interface elements inside an agent host; WebMCP lets a page offer tools to an agent. Where a service offers both, which surface does the user see, and who owns consent? |
| Q8 | Is there a case where a service should offer WebMCP tools and no MCP server, or the reverse? What criteria would decide? |


---

## 2. Changes since February 2026

| Change | Detail | Source |
|---|---|---|
| WebMCP status | Moved from a Chrome flag to a public origin trial starting in Chrome 149; Gemini in Chrome support announced as coming | [4] |
| Trial window | Chrome 149–156, extension to 162 requested | [5] |
| Current draft | Draft Community Group Report dated 20 October 2026; four editors, from Microsoft and Google | [24] |
| Entry point | `navigator.modelContext` replaced by `document.modelContext`; the navigator surface is deprecated from Chrome 150 | [2] |
| Registration model | `provideContext()` / `clearContext()` removed; tools are added one at a time with `registerTool(tool, { exposedTo, signal })` and removed by aborting the signal | [2][24] |
| Reason for removal | Issue #101 showed `provideContext()` cleared existing tools before registering, bypassing the duplicate-name guard; a third-party script could replace a tool and proxy agent-user traffic. Closed via PR #132 | [2] |
| Browser vendor positions | Mozilla: neutral. WebKit: oppose, citing duplication, internationalization, privacy and security | [3] |
| MCP governance | Donated by Anthropic to the Agentic AI Foundation under the Linux Foundation, December 2025 | [8][9] |
| MCP revision 2026-07-28 | Protocol sessions and the initialize handshake removed; `server/discover` added; Tasks moved into an official extension; extensions field added to capabilities; Roots, Sampling and Logging deprecated; authorization hardened against issuer mix-up; feature lifecycle and deprecation policy adopted | [25] |

---

## 3. Comparison table

| Row | Dimension | MCP | WebMCP | Sources |
|---|---|---|---|---|
| C1 | Steward and status | Agentic AI Foundation (Linux Foundation); stable revision 2026-07-28 | Draft Community Group Report, W3C Web Machine Learning CG; not on the Recommendation track; draft dated 20 October 2026, editors from Microsoft and Google | [8][14] / [24] |
| C2 | Motivation | A common connection standard so any AI application can reach any tool or data source without a custom integration for each pairing | Let a site declare the actions it offers so an agent does not have to infer them from scraped UI or screenshots | [8] / [6] |
| C3 | Form | JSON-RPC 2.0 wire protocol; stdio or Streamable HTTP transports | Browser JavaScript API on `document.modelContext`, secure contexts only; no separate protocol transport | [1][8] / [24] |
| C4 | Where tools run | Separate local or remote server | Client-side script in a live page; the browser controls discovery and invocation | [1] |
| C5 | Discovery | Client connects to a configured server; servers must implement `server/discover` to advertise versions, capabilities and identity | Agent visits the page; `getTools()`, `executeTool()` and `toolchange`, `toolactivated`, `toolcancel` events; headless mode unsupported in Chrome; `/.well-known/webmcp` not defined by the draft | [25] / [24][1] |
| C6 | Identity and access | Server defines authentication; OAuth-based authorization; 2026-07-28 requires issuer validation and deprecates Dynamic Client Registration in favour of Client ID Metadata Documents | Uses the user's active session; cross-origin access closed by default; tools exposed explicitly with `exposedTo`; access gated by the `tools` Permissions Policy, default allowlist `'self'` | [1][25] / [24] |
| C7 | State | Protocol sessions removed; each request carries protocol version and client capabilities in `_meta`; servers needing cross-call state use explicit handles passed as tool arguments | Inherently tied to the tab: login session, cart and on-screen filters live in the browser | [25] / [2] |
| C8 | Primitives | Tools, resources and prompts, plus opt-in extensions such as Tasks and MCP Apps; Roots, Sampling and Logging deprecated | Tools only, via imperative `registerTool()` and declarative form attributes (`toolname`, `tooldescription`, `toolparamdescription`) | [25][10] / [1][2] |
| C9 | User interface | MCP Apps: server-shipped HTML rendered by the host in a sandboxed iframe | The page is the interface; declarative tools fill a visible form and leave submission to the user unless `toolautosubmit` is set | [12] / [1] |
| C10 | Tool description | Name, description, input and output schemas (any JSON Schema 2020-12 keywords) and annotations such as read-only | Name, description, JSON Schema `inputSchema`, `execute` callback, annotations `readOnlyHint`, `untrustedContentHint`, `consequentialHint` and `debugging` | [7][25] / [24] |
| C11 | Limits on size | Not covered in sources read | Chrome documents 500 characters per tool description, 150 per parameter description, 1.5K per tool output | [2] |
| C12 | Operations and observability | Required `Mcp-Method` and `Mcp-Name` headers on HTTP requests and OpenTelemetry trace-context conventions in `_meta` let gateways route and trace without parsing bodies | Inspection through Chrome DevTools panel and a testing interface behind a flag | [25] / [1][16] |
| C13 | Adoption | Supported by Claude, ChatGPT, Gemini and Microsoft Copilot; reported past 10,000 published servers | Chrome only, through origin trial or testing flag | [8][14] / [5][16] |

---

## 4. Shared ground

- Both describe a tool by name, natural-language description and a JSON Schema input contract. The WebMCP schema requirement mirrors server-side MCP, so an agent can handle a page tool and a server tool in the same way [2].
- Both rely on the agent reading descriptions to choose a tool, and both leave authorization, validation and business rules with the tool owner [1].
- Both carry advisory annotations about tool behaviour [1][7].
- The two can run side by side in one product: page-scoped actions through WebMCP, account-wide or background operations through an MCP server [1].
- A WebMCP tool needs no MCP server or transport. Any bridge to an MCP-speaking agent is an implementation choice [1]. Community polyfills and bridges exist that translate between the two formats; they are not official W3C or MCP projects [17].

---

## 5. Benefits and risks

### 5.1 MCP

| ID | Benefits | Sources |
|---|---|---|
| MB1 | Model-agnostic and broadly supported by the main AI assistants | [8] |
| MB2 | Vendor-neutral governance under the Linux Foundation, with competitors as co-founders | [9][14] |
| MB3 | Stateless core runs on ordinary load-balanced and serverless HTTP infrastructure | [10][11] |
| MB4 | Works headless and in the background, independent of any open browser | [1] |
| MB5 | Extensions framework adds capabilities (Apps, Tasks) outside the core; feature lifecycle policy with a minimum twelve-month deprecation window | [10][25] |
| MB6 | Routing and trace headers let gateways and observability tools see MCP traffic directly | [25] |

| ID | Risks and weaknesses | Sources |
|---|---|---|
| MR1 | Security incidents already logged, including cross-tenant data exposure in early deployments | [14] |
| MR2 | Breaking migrations: 2026-07-28 removes sessions, the handshake, `ping` and SSE resumability, and replaces the experimental Tasks design | [25] |
| MR3 | A server cannot reach state that exists only in the user's browser tab without rebuilding authentication and synchronisation | [2] |
| MR4 | Each integration needs a server to install or operate and credentials to exchange | [2] |

### 5.2 WebMCP

| ID | Benefits | Sources |
|---|---|---|
| WB1 | Agent acts inside the session the user already has open; no app install, no separate API key exchange | [2] |
| WB2 | Site owner defines the action surface in place of DOM scraping or screenshot reading | [1][6] |
| WB3 | Progressive enhancement: the visual page stays the product and the agent gets a second interface to the same logic | [1] |
| WB4 | Low adoption cost for form-based sites through declarative attributes | [1][2] |
| WB5 | Visible user control: declarative tools pre-fill forms and wait for the user to submit by default | [1] |
| WB6 | Cross-origin access closed by default and gated by explicit exposure and Permissions Policy | [1] |

| ID | Risks and weaknesses | Sources |
|---|---|---|
| WR1 | The draft names prompt injection (description poisoning, output injection, tools as targets), misrepresentation of intent, privacy leakage through over-parameterisation, same-origin violations and private-browsing interactions as threats | [24] |
| WR2 | Annotations, including the new `consequentialHint`, are hints only; a tool can claim to be read-only and still change data | [1][24] |
| WR3 | The agent inherits the user's session authority, so a misused tool acts with the user's full rights | [1] |
| WR4 | Shared-document risk: several scripts share one page; the removed `provideContext()` showed how a third-party script could take over tools | [2] |
| WR5 | Single-engine implementation, with WebKit opposed and Mozilla neutral | [3] |
| WR6 | API churn: the surface changed twice within the trial; many tutorials show deprecated calls | [2][18] |
| WR7 | Tools exist only while a browser has the page open; no headless or background use | [1] |
| WR8 | The declarative section of the July draft was marked TODO while Chrome already shipped it; WebKit lists the consent hook and cross-origin analysis as TODO too | [1][22] |
| WR9 | Chrome's own documentation states that safety inside an LLM cannot be guaranteed | [2] |

---

## 6. Directions for development

Each item pairs a weakness with a strength on the other side. These are proposals for discussion, offered to the WebMCP and MCP communities; none is drawn from either group's agenda.

| ID | Weakness addressed | Strength drawn on | Proposal for discussion |
|---|---|---|---|
| D1 | WR2 advisory annotations | MCP extensions framework (MB5) | Could annotations gain a verifiable form, with the browser or host checking declared behaviour against observed effects? |
| D2 | WR3 inherited session authority | MCP OAuth scopes (C6) | Could a WebMCP tool declare a narrower scope than the user's session, granted per agent and revocable? |
| D3 | WR4 shared-document takeover | WB6 origin gating | Should tool registration be restricted to first-party scripts or bound to a declared origin by default? |
| D4 | WR7 browser-only reach and MR3 server blindness to tab state | MB4 headless operation and WB1 live session | A documented pattern for pairing a page's WebMCP tools with the same site's MCP server, with shared identifiers and consistent descriptions |
| D5 | WR5 single-engine implementation | MB2 neutral multi-vendor governance | Engage the concerns in the WebKit position (duplication, internationalization, privacy, security) issue by issue before further shipping |
| D6 | WR6 and MR2 churn | MB5 written deprecation policy | Adopt a written deprecation and versioning policy for WebMCP during the trial |
| D7 | WR1 and MR1 injection and data exposure | WR1 threat taxonomy | A shared threat model and test suite covering both page tools and server tools |
| D8 | Observability gap on the WebMCP side (C12) | MB6 routing and trace headers | Standard audit logging of agent-invoked actions, comparable to MCP trace context |
| D9 | Description quality drives tool choice in both | C10 shared schema | Common guidance on writing and evaluating tool descriptions, testing correct and incorrect tool selection |

---

## 7. Open questions

Questions on how WebMCP relates to MCP are listed as Q1–Q8 in section 1. The questions below concern design and governance.

1. Is the overlap between WebMCP and MCP tool descriptions close enough to justify a single shared vocabulary for tool metadata?
2. Who is accountable when a browser agent invokes a page tool with the user's session: the site, the browser, or the agent provider?
3. Should declarative WebMCP tools ever auto-submit for state-changing actions?
4. How should knowledge representation of tool capability (what a tool does, its preconditions and effects) be expressed beyond a natural-language description and an input schema?
5. What evidence would satisfy the concerns behind the WebKit opposition?

---

## References

1. Nnorukam, C. *What Is WebMCP?* chudi.dev, updated July 2026. https://chudi.dev/blog/what-is-webmcp
2. Kim, J. *WebMCP's Origin Trial: provideContext Is Already Gone.* DEV Community, July 2026. https://dev.to/jangwook_kim_e31e7291ad98/webmcps-origin-trial-providecontext-is-already-gone-54j8
3. web-platform-dx. *Declarative WebMCP*, web-features data (standards positions). https://web-platform-dx.github.io/web-features-explorer/features/declarative-webmcp.json
4. Chrome for Developers. *15 updates from Google I/O 2026.* https://developer.chrome.com/blog/chrome-at-io26
5. seunghan91. *webmcp-go* package documentation. https://pkg.go.dev/github.com/seunghan91/webmcp-go
6. Dubois, S. *WebMCP.* https://www.dsebastien.net/webmcp/
7. Bisetty, S. *The 2026 agent protocol stack: MCP, A2A, gateways and identity.* DEV Community. https://dev.to/thasha/part-4-the-2026-agent-protocol-stack-mcp-a2a-gateways-and-identity-12n5
8. Forkast. *Model Context Protocol (MCP)* glossary. https://forkast.news/glossary/model-context-protocol/
9. IT Brief Australia. *Anthropic donates MCP to new Agentic AI Foundation.* https://itbrief.com.au/story/anthropic-donates-mcp-to-new-agentic-ai-foundation
10. Karlsson, J. *The 2026-07-28 MCP Release Explained.* CData. https://www.cdata.com/blog/mcp-2026-07-28-release
11. Dash, S. K. *The MCP 2026-07-28 Update.* Composio. https://composio.dev/content/mcp-2026-07-28-update-statelessness-apps-auth
12. agentpedia. *MCP 2026 spec: stateless, Tasks, Apps.* https://agentpedia.codes/blog/mcp-2026-spec-stateless-tasks-apps
13. mcp.directory. *MCP 2026-07-28: The Stateless Release Candidate, Explained.* https://mcp.directory/blog/mcp-2026-07-28-release-candidate
14. 200OK Solutions. *Best Guide to the Model Context Protocol (MCP) in 2026.* https://www.200oksolutions.com/blog/best-guide-to-the-model-context-protocol-mcp-in-2026/
15. web-platform-dx. *document.modelContext*, web-features data. https://web-platform-dx.github.io/web-features-explorer/features/document-modelcontext.json
16. angeo. *module-webmcp* package notes. https://packagist.org/packages/angeo/module-webmcp
17. WebMCP-org (MCP-B) GitHub organisation. https://github.com/webmcp-org
18. MCP Playground. *WebMCP in 2026: The API Moved and Most Guides Are Wrong.* https://mcpplaygroundonline.com/blog/what-is-webmcp
19. Web Machine Learning Community Group. *WebMCP* repository README (explainer). https://github.com/webmachinelearning/webmcp
20. Le Hegaret, P. *W3C and the State of the Web in the AI Era.* W3C Strategy Team, October 2026. https://www.w3.org/2026/10/w3c-and-state-of-web-in-ai-era.html
21. W3C AI Knowledge Representation Community Group. *WebMCP Technical Notes.* February 2026. https://w3c-cg.github.io/aikr/webMCP/webmcp-technical-notes.html
22. WebKit. *WebMCP* standards position, issue #670. https://github.com/WebKit/standards-positions/issues/670
23. Web Machine Learning Community Group. *WebMCP for Service Workers* explainer discussion, pull request #19. https://github.com/webmachinelearning/webmcp/pull/19
24. Web Machine Learning Community Group. *WebMCP.* Draft Community Group Report, 20 October 2026. https://webmachinelearning.github.io/webmcp/
25. Model Context Protocol. *Key Changes*, specification revision 2026-07-28. https://modelcontextprotocol.io/specification/2026-07-28/changelog
