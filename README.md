# fab-megascans-auto-claim-2026
A small script that runs on Brave or Chrome browsers in order to claim all Megascans asset on Fab during the FREE full library assets giveaway this Nov 2026.

Fab.com Free Asset Auto-Claimer (Guide)

Claiming all 20,000+ free assets one-by-one is impossible to do by hand before the event ends. This automated script runs in your browser, automatically clicks "Add to My Library" on every free asset, and auto-scrolls down to keep going.

📋 Prerequisites (Do this first!)
1. Open Fab.com and make sure you are logged in with your Epic Games account.
2. Filter the search to free items (or go to any seller/listing page with free items).
3. Important: Manually click "Add to My Library" on one free item yourself first. This ensures you accept any one-time Epic/Fab Terms of Service or EULA popup before automating.
   
🛠️ Step-by-Step Instructions

1. Press F12 (or Ctrl + Shift + I / Cmd + Option + I on Mac) to open Developer Tools.
2. Go to the Console tab.
3. If your browser blocks pasting:
    Type allow pasting into the console and hit Enter (or use the Sources tab ➔ Snippets ➔ + New Snippet).
4. Copy & paste the script included into the console and press Enter.
5. A floating box will appear in the bottom-left showing your live claim counter and a Pause button. Leave the tab open and let it work!

🛑 How to Pause or Stop
- To Pause / Resume: Click the red Pause button in the bottom-left corner of the webpage.
- To Stop Completely: Just hit F5 (or refresh the tab). That instantly cancels all running actions.

💻 The Script:


```javascript
(async function fabClickAllAddToLibrary() {
  const CLICK_DELAY_MS = 800;   // Delay between clicks (~1.2 items/sec, safe from rate limits)
  const SCROLL_WAIT_MS = 2500;  // Wait time after scrolling to let items load

  let isRunning = true;
  let totalClicked = 0;
  const clickedElements = new WeakSet();

  // Create floating status/control widget
  const ui = document.createElement("div");
  ui.style.cssText = `
    position: fixed; bottom: 20px; left: 20px; z-index: 999999;
    background: #18181c; color: #fff; padding: 12px 16px; border-radius: 10px;
    font-family: sans-serif; font-size: 13px; box-shadow: 0 4px 16px rgba(0,0,0,0.6);
    border: 1px solid #333; display: flex; align-items: center; gap: 12px;
  `;
  ui.innerHTML = `
    <div>
      <div style="font-weight:bold; color:#00ff88;" id="fab-bot-status">▶ Running...</div>
      <div style="margin-top:2px;">Claimed: <strong id="fab-bot-count">0</strong></div>
    </div>
    <button id="fab-bot-toggle" style="
      background:#e63946; color:#fff; border:none; padding:6px 12px;
      border-radius:6px; cursor:pointer; font-weight:bold;
    ">Pause</button>
  `;
  document.body.appendChild(ui);

  const statusEl = document.getElementById("fab-bot-status");
  const countEl = document.getElementById("fab-bot-count");
  const toggleBtn = document.getElementById("fab-bot-toggle");

  toggleBtn.onclick = () => {
    isRunning = !isRunning;
    statusEl.textContent = isRunning ? "▶ Running..." : "⏸ Paused";
    statusEl.style.color = isRunning ? "#00ff88" : "#ffcc00";
    toggleBtn.textContent = isRunning ? "Pause" : "Resume";
    toggleBtn.style.background = isRunning ? "#e63946" : "#2a9d8f";
  };

  const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

  function triggerHoverOnCards() {
    const cards = document.querySelectorAll('a[href*="/listings/"], div[class*="Card"], article, li');
    cards.forEach((card) => {
      card.dispatchEvent(new MouseEvent("mouseenter", { bubbles: true }));
      card.dispatchEvent(new MouseEvent("mouseover", { bubbles: true }));
    });
  }

  function findAddToLibraryButtons() {
    triggerHoverOnCards();
    const candidates = document.querySelectorAll('button, [role="button"]');
    const matches = [];

    for (const btn of candidates) {
      if (clickedElements.has(btn) || btn.disabled || btn.dataset.autoClaimed === "true") {
        continue;
      }

      const text = (btn.innerText || btn.textContent || "").trim().toLowerCase();
      const aria = (btn.getAttribute("aria-label") || "").trim().toLowerCase();
      const title = (btn.getAttribute("title") || "").trim().toLowerCase();
      const combined = `${text} | ${aria} | ${title}`;

      const isAddBtn =
        combined.includes("add to my library") ||
        combined.includes("add to library");

      const isAlreadyOwned =
        combined.includes("saved in my library") ||
        combined.includes("in my library") ||
        combined.includes("owned");

      if (isAddBtn && !isAlreadyOwned) {
        matches.push(btn);
      }
    }
    return matches;
  }

  let emptyScrolls = 0;

  while (true) {
    if (!isRunning) {
      await sleep(500);
      continue;
    }

    const buttons = findAddToLibraryButtons();

    if (buttons.length > 0) {
      emptyScrolls = 0;
      statusEl.textContent = `▶ Clicking (${buttons.length} in view)...`;

      for (const btn of buttons) {
        if (!isRunning) break;

        btn.scrollIntoView({ behavior: "smooth", block: "center" });
        await sleep(150);

        btn.dataset.autoClaimed = "true";
        clickedElements.add(btn);
        btn.click();

        totalClicked++;
        countEl.textContent = totalClicked;

        await sleep(CLICK_DELAY_MS);
      }
    } else {
      emptyScrolls++;
      statusEl.textContent = `⏬ Scrolling for more (${emptyScrolls})...`;
      window.scrollBy({ top: window.innerHeight * 0.85, behavior: "smooth" });
      await sleep(SCROLL_WAIT_MS);

      const nextBtn = Array.from(document.querySelectorAll("button, a")).find((el) => {
        const t = (el.innerText || el.getAttribute("aria-label") || "").trim().toLowerCase();
        return t === "load more" || t === "show more" || t === "next" || t === "next page";
      });
      if (nextBtn && !nextBtn.disabled) {
        nextBtn.click();
        await sleep(SCROLL_WAIT_MS);
      }

      if (emptyScrolls >= 15) {
        statusEl.textContent = "🎉 Finished / End of Page";
        statusEl.style.color = "#00b4d8";
        isRunning = false;
        toggleBtn.textContent = "Resume";
      }
    }
  }
})();`
```

