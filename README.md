<div align="center">

[![diting stars](https://img.shields.io/github/stars/yinnho/diting?style=flat-square&labelColor=161b22)](https://github.com/yinnho/diting/stargazers)
[![License](https://img.shields.io/github/license/yinnho/diting?style=flat-square&labelColor=161b22)](https://github.com/yinnho/diting/blob/main/LICENSE)
[![Release](https://img.shields.io/github/v/release/yinnho/diting?style=flat-square&labelColor=161b22)](https://github.com/yinnho/diting/releases)

</div>

I build Rust infrastructure for the agent internet — the browser agents browse with, and the systems that let agents reach each other the way people reach websites over HTTP.

## Products

**[diting](https://github.com/yinnho/diting)** — a browser engine built for AI agents: one Rust binary, own V8 + own CSS/layout/paint stack, **no Chromium**. Fetch live pages as markdown, render JS/SPAs, screenshots, interactive login sessions that survive restarts, 20-engine meta-search, deterministic document generation (PDF/PPTX/DOCX). Every response is a receipt — tier, redirect trail, content hash — so agents act on what actually happened. Apache-2.0, self-host in one line:

```bash
curl -fsSL https://browser.aginx.net/install.sh | sh
```

Measured against headless Chrome on the same pages: **7.6× faster** to agent-usable text (p50 532 ms vs 4 053 ms), **~10× less memory**, zero hard failures. Onboarding for agents is zero-touch: point Codex / Claude Code at the SKILL.md and the browser becomes their tool.

**[aginxos-next](https://github.com/yinnho/aginxos-next)** — a Rust OS for AI agents on a real phone: Linux kernel for drivers, Rust userspace for the system. Pixel 5: voice-first agent, camera as peer input, screen as HTML canvas. 给 Agent 的操作系统。

**[AginxBrain](https://github.com/yinnho/AginxBrain)** — local-first AI gateway with bidirectional protocol conversion (Anthropic ↔ OpenAI ↔ Responses). Route Claude Code / Codex CLI to any provider by quality tag, with failover. Rust + Tauri.

**[aginx](https://github.com/yinnho/aginx)** — the Agent Protocol: route messages to agents as easily as HTTP routes them to servers. A pure server — which models an agent runs is the agent's own business.

Also: **[homebrew-diting](https://github.com/yinnho/homebrew-diting)** (macOS tap), **[model-router](https://github.com/yinnho/model-router)** (protocol-translating model proxy with failover), **[opencarrier](https://github.com/yinnho/opencarrier)** (agent OS: WeChat / Feishu / DingTalk access, 24/7 scheduling).

## How I work upstream

When something breaks, I fix it in our tree first, verify the fix against a repro, and then hand the root cause and the patch data back upstream. Every claim below links a commit and a test. Recent highlights:

- **[obscura#1108](https://github.com/h4ckf0r0day/obscura/issues/1108)** — SSRF denials that don't name the opt-in: eight separate denial sites, zero mentions of `--allow-private-network`. All eight now share one hint constant; the operator learns the knob at the moment they hit it ([7cfeb1f](https://github.com/yinnho/aginxbrowser/commit/7cfeb1f)).
- **[agent-browser#2000](https://github.com/vercel-labs/agent-browser/issues/2000)** — the date/time input family stored whatever you handed it (`2026-02-30` reflected verbatim). Shipped the HTML value sanitization algorithm for date/month/week/time/datetime-local, with a composing-typing draft so agent keystrokes aren't eaten mid-date; 44-assertion parity harness plus CDP e2e ([42840ea](https://github.com/yinnho/aginxbrowser/commit/42840ea)).
- **[blitz#887](https://github.com/DioxusLabs/blitz/pull/887) gap — sticky table rows and groups** — boxless `thead`/`tbody`/`tfoot` got a geometry-preserving flex-column stand-in; the sticky containing-block walk skips table-internal stand-ins, and positioned z>0 hoists past non-stacking-context parents ([b1875f9](https://github.com/yinnho/aginxbrowser/commit/b1875f9)).
- **[moli#514](https://github.com/lexmount/moli/issues/514)** — non-HTML responses walked the HTML pipeline; now `text/*`/JSON/XML render as Chrome's escaped `<pre>` and `contentType` rides the response header end to end ([f963306](https://github.com/yinnho/aginxbrowser/commit/f963306)).
- **[obscura#940](https://github.com/h4ckf0r0day/obscura/issues/940)** — `location.href` assignment moved the realm URL at queue time: synchronous JS between assignment and navigation could read the *target* origin's cookies (SOP bypass). URL now moves on commit, both halves pinned by regression tests ([9bd14ff](https://github.com/yinnho/aginxbrowser/commit/9bd14ff)).
- **[obscura#852](https://github.com/h4ckf0r0day/obscura/issues/852)** — two SSRF deny-sets drifting: navigation validated the full IANA special-purpose list while page fetch used a hand-rolled loopback check — a page could `fetch()` addresses navigation refuses. One shared `is_forbidden_ip` across fetch, XHR, scripts, redirects ([0df75f4](https://github.com/yinnho/aginxbrowser/commit/0df75f4)).
- **[blitz#349](https://github.com/DioxusLabs/blitz/issues/349)** — box-shadow was absent; shipped v1 as a spec-shaped parser + SDF paint (outer, then inset with inverted geometry and Chrome-matched falloff), sidestepping the non-uniform-radius wall upstream traces to ([6df1ecf](https://github.com/yinnho/aginxbrowser/commit/6df1ecf), [8f08298](https://github.com/yinnho/aginxbrowser/commit/8f08298)).
- **[obscura#935](https://github.com/h4ckf0r0day/obscura/issues/935)** — text-decoration painted underline only; added line-through/overline with the CSS 2.1 §16.3.1 propagation rule, probed against a Chrome positive control ([6049614](https://github.com/yinnho/aginxbrowser/commit/6049614)); upstream closed it with a matching fix.
- **[obscura#828](https://github.com/h4ckf0r0day/obscura/issues/828)** — `innerText` returned raw inline `<script>` source (~100× vs Chromium); rebuilt as a rendered-text walker — juejin.cn homepage went 38 775 → 295 chars ([18e131c](https://github.com/yinnho/aginxbrowser/commit/18e131c)).
- **[obscura#817](https://github.com/h4ckf0r0day/obscura/issues/817)** — webpack style-loader "couldn't find a style target": aliased constructors made `head instanceof HTMLIFrameElement` true for every element. Rebuilt with `Symbol.hasInstance` keyed on tag name — bilibili's 2 MB player core boots past it ([f709e17](https://github.com/yinnho/aginxbrowser/commit/f709e17)).
- **[obscura#967](https://github.com/h4ckf0r0day/obscura/issues/967)** — `Authorization` leaked across cross-origin redirects; pinned with raw TCP listeners on two ports. Fixed per spec, and upstream merged the same fix two days after our data point ([7fbcb51](https://github.com/yinnho/aginxbrowser/commit/7fbcb51)).

Full ledger: 60+ upstream issues across obscura, blitz, moli, servo, agent-browser, takumi and the modelcontextprotocol spec — each reported with a repro and a fix already running in our build.
