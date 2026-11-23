# DevKart demo: runbook (8 minutes)

The demo matches the deck's running example: a laptop for React work under ₹70,000.

## One-time setup (do it the day before)

1. Chrome 155 or newer. You have 158.
2. Open `chrome://flags/#enable-webmcp-testing`, set it to **Enabled**, and relaunch Chrome.
3. Install **WebMCP – Model Context Tool Inspector** from the Chrome Web Store:
   https://chromewebstore.google.com/detail/webmcp-model-context-tool/gbpdfapgefenggkahomfgkhfehlcenpd
4. Get a Gemini API key at https://aistudio.google.com/apikey and paste it into the Inspector. This is the agent in the demo.
5. Optional, for the on-page Built-in AI step: the Prompt API is available on the web from Chrome 148. Click **Understand** once at home so Gemini Nano downloads, then check `chrome://on-device-internals` until the model shows as ready. If it isn't there, the page falls back to a simple parser and says so in the log, and the demo still works. Gemini Nano needs **at least 22 GB free disk space**; with less, it stays on "downloadable" forever. Until the model is ready, the page uses the fallback straight away and logs download progress (⬇️) in Agent activity.
7. In the WebMCP extension, keep **Settings → Model → On-device model** OFF, so it uses the local server (Gemini) and requests appear on the logs page.
6. Use a separate Chrome profile for the talk: no other extensions, no bookmarks bar, zoom at 125%.

## Start the site (on the day)

```bash
cd ~/Downloads/DevFest-WebMCP/demo
python3 -m http.server 8765
```

Open http://localhost:8765. It must be `localhost`, because WebMCP needs a secure context. The header pill should be green: **WebMCP: tools registered**.

## Script

**1. The human way (30 s).** Point at the grid, search form and cart. Say: "This is a normal store. Nothing about it is AI-specific yet."

**2. What the page exposes (1 min).** Open the Inspector side panel. It lists the tools:
- `search_products` is the declarative one. It is just the `<form>` with `toolname` and `tooldescription` attributes.
- `get_product_details`, `compare_products`, `add_to_cart` and `view_cart` are imperative, registered with `document.modelContext.registerTool()`.
- Point out that `checkout` is **missing**.
- On the page, the "Tools this page exposes" panel tags each tool with a blue **DECLARATIVE** or purple **IMPERATIVE** badge, and every Agent activity line carries the same badge. Point at them as the agent works.

**3. Built-in AI turns language into criteria (1 min).** In the "Describe what you need" box, click **Understand**. The Prompt API with a JSON `responseConstraint` extracts `{maxPrice, useCase}`. The page validates the result and fills the form. Show the 🧠 line in Agent activity: raw output, then the validated version. Say: "The model interprets. My code decides what's valid."

**4. The agent takes over (3 min).** In the Inspector chat, type:

> I need a laptop for React and full-stack development. My budget is ₹70,000. Compare the two best options and add the lighter one to my cart.

Narrate as it runs:
- The search form gets a yellow outline. That's the `:tool-form-active` CSS pseudo-class, so a human can see the agent using the form.
- 🤖 entries show up in Agent activity, one per tool call with its arguments.
- Product cards flash as the agent reads them, and the grid switches to the comparison.
- The cart updates. **`checkout` now appears in the tool list.** The page registers it only when it can be used, and unregisters it when the cart is empty again.

**5. A human stays in the loop (1.5 min).** Type:

> Go ahead and place the order.

`checkout` has `consequentialHint: true`, so the Inspector should ask before it runs. Then the page itself opens a **Place this order?** dialog. Click **Cancel** first, and the agent is told "The user cancelled". Then ask again and approve. Say: "The agent can do the work. It can't spend my money without me."

**6. Wrap (30 s).** Go back to slides: "Five tools, one form and about 150 lines of JavaScript, sitting on logic the UI already had."

## Backup plan

- Record the full run as a screen recording the night before (QuickTime → New Screen Recording) and keep it on the desktop.
- If the Inspector or the Gemini key fails, open DevTools and drive the tools from the console. Everything still shows up in Agent activity:

```js
const mc = document.modelContext;
const t = async n => (await mc.getTools()).find(x => x.name === n);
await mc.executeTool(await t("search_products"), { maxPrice: 70000, useCase: "development" });
await mc.executeTool(await t("compare_products"), { productIds: ["lp-101", "lp-106"] });
await mc.executeTool(await t("add_to_cart"), { productId: "lp-106" });
(await mc.getTools()).map(x => x.name); // checkout is now listed
```

- The official demos are a second backup: https://googlechromelabs.github.io/webmcp-tools/demos/explainer/ (scraping and WebMCP side by side).

## Venue Wi-Fi

The site itself runs fully offline. Only the Inspector's Gemini calls need the internet, so bring a phone hotspot.
