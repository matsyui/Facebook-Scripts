# 🚪 Facebook Auto Leave Groups Script

This script automatically **leaves all Facebook groups** listed on your [Joined Groups page](https://www.facebook.com/groups/joins/).  
It clicks each **More (⋯)** button → selects **Leave Group** → checks the *“Prevent people from inviting you again”* toggle → confirms → scrolls → repeats until you’re free from all groups.  

---

## ⚙️ How to Use

1. **Go to:** [https://www.facebook.com/groups/joins/](https://www.facebook.com/groups/joins/)  
2. **Scroll down a bit** so all your joined groups load.  
3. **Open your browser console:**
   - Windows: `Ctrl + Shift + J`
   - Mac: `Cmd + Option + J`
4. **Paste the entire script below** into the console.  
5. Press **Enter** — and watch it leave every group automatically.  
6. You can stop anytime by typing:
   ```js
   window.facebookAutoLeaveStop();
   ```

---

## 💻 The Script

```js
// 🚪 Facebook Auto-Leave Groups Script
// Paste this into the console on: https://www.facebook.com/groups/joins/
// To stop anytime: window.facebookAutoLeaveStop()

(function facebookAutoLeave(defaults = {}) {
  if (window._facebookAutoLeaveRunning) {
    console.warn("facebookAutoLeave: already running");
    return;
  }
  window._facebookAutoLeaveRunning = true;

  const options = Object.assign({
    maxPerMinute: 20,   // max group leaves per minute (rate-limit safety)
    baseDelay: 900,     // base ms between actions
    jitter: 600,        // random jitter ms added/subtracted
    scrollDistance: 2200,
    scrollDelayOnEmpty: 7000,
    verbose: true
  }, defaults);

  let stopRequested = false;
  window.facebookAutoLeaveStop = () => { stopRequested = true; };

  const delay = (ms) => new Promise(res => setTimeout(res, ms));
  const rand = (n) => Math.floor(Math.random() * n);
  const humanDelay = async () => {
    const jitter = Math.floor((Math.random() * 2 - 1) * options.jitter);
    const d = Math.max(250, options.baseDelay + jitter);
    if (options.verbose) console.debug(`— waiting ${d}ms`);
    await delay(d);
  };

  const actionTimestamps = [];
  function canDoAction() {
    const now = Date.now();
    while (actionTimestamps.length && now - actionTimestamps[0] > 60_000) actionTimestamps.shift();
    return actionTimestamps.length < options.maxPerMinute;
  }
  function recordAction() { actionTimestamps.push(Date.now()); }

  const processed = new WeakSet();

  function findMoreButtons() {
    const byAria = [...document.querySelectorAll('[aria-label="More"], [aria-label="More options"], div[aria-label*="More"]')];
    const byRoleBtn = [...document.querySelectorAll('div[role="button"], a[role="button"], [role="menuitem"]')];
    const candidates = (byAria.concat(byRoleBtn))
      .filter(el => {
        const listItem = el.closest('[role="listitem"], [role="article"], [data-visualcompletion]');
        return !!listItem;
      });
    return Array.from(new Set(candidates));
  }

  async function openMenuFor(btn) {
    try {
      btn.scrollIntoView({ block: "center" });
      await delay(200 + rand(300));
      btn.click();
      await delay(350 + rand(400));
      return true;
    } catch (e) {
      console.warn("openMenuFor failed", e);
      return false;
    }
  }

  function findLeaveButtonsInMenu() {
    const nodes = [...document.querySelectorAll('button, [role="menuitem"], [role="button"], a')];
    return nodes.filter(n => {
      const txt = (n.textContent || "").toLowerCase().trim();
      return txt.includes("leave group");
    });
  }

  async function clickIfVisible(el) {
    if (!el) return false;
    try {
      el.scrollIntoView({ block: "center" });
      await delay(120 + rand(250));
      el.click();
      return true;
    } catch (e) {
      console.warn("clickIfVisible failed", e);
      try { el.dispatchEvent(new MouseEvent('click', { bubbles: true, cancelable: true })); return true; } catch (_) { return false; }
    }
  }

  async function processOne(btn) {
    if (processed.has(btn)) return false;
    processed.add(btn);

    if (!canDoAction()) {
      const waitMs = 8000 + rand(6000);
      if (options.verbose) console.warn(`Rate cap reached, waiting ${waitMs}ms`);
      await delay(waitMs);
    }

    if (stopRequested) return true;

    const opened = await openMenuFor(btn);
    if (!opened) return false;

    await delay(250 + rand(300));
    const leaveCandidates = findLeaveButtonsInMenu();
    if (leaveCandidates.length === 0) {
      const alt = [...document.querySelectorAll('[aria-label]')].filter(n => /leave\s*group/i.test(n.getAttribute('aria-label') || ""));
      if (alt.length) leaveCandidates.push(...alt);
    }

    if (leaveCandidates.length === 0) {
      console.warn("No 'Leave Group' button found — maybe FB changed UI.");
      return false;
    }

    const leaveBtn = leaveCandidates[0];

    // Click Leave in the menu
    await clickIfVisible(leaveBtn); 
    await delay(500 + rand(500)); // wait a bit for modal to appear

    // Wait for the confirmation dialog
    const dialog = await new Promise(res => {
      const start = Date.now();
      const check = () => {
        const d = document.querySelector('div[role="dialog"]');
        if (d) res(d);
        else if (Date.now() - start > 5000) res(null); // timeout 5s
        else requestAnimationFrame(check);
      };
      check();
    });

    if (dialog) {
        
    // -------------------------------
    // THIS IS OPTIONAL ONLY !!!!
    // Handle the "Prevent people from inviting..." checkbox
    // -------------------------------

    //   const preventSwitch = [...dialog.querySelectorAll('input[role="switch"], input[type="checkbox"]')]
    //     .find(s => /prevent.*invite/i.test(s.getAttribute('aria-label') || (s.parentElement?.innerText || "")));

    //   if (preventSwitch && !(preventSwitch.getAttribute('aria-checked') === 'true' || preventSwitch.checked)) {
    //     if (options.verbose) console.debug("Clicking 'Prevent people from inviting...' switch in dialog");
    //     await clickIfVisible(preventSwitch);
    //     await delay(300 + rand(400));
    //   }

      // Click the Leave Group button inside the dialog
      const leaveDialogBtn = [...dialog.querySelectorAll('div[role="button"], button')]
        .find(b => /leave group/i.test(b.innerText || b.textContent || ""));
      if (leaveDialogBtn) {
        await clickIfVisible(leaveDialogBtn);
        await delay(800 + rand(500)); // wait for dialog to close
      }
    } else {
      if (options.verbose) console.warn("Dialog did not appear, skipping modal actions");
    }

    recordAction();
    if (options.verbose) console.log("✅ Left a group. Actions this minute:", actionTimestamps.length);

    await delay(900 + rand(900));
    return true;
  }

  async function mainLoop() {
    try {
      if (options.verbose) console.log("facebookAutoLeave: starting loop. To stop call window.facebookAutoLeaveStop()");

      while (!stopRequested) {
        let candidates = findMoreButtons().filter(b => !processed.has(b));
        if (candidates.length === 0) {
          window.scrollBy({ top: options.scrollDistance, behavior: 'smooth' });
          if (options.verbose) console.debug("Scrolled to load more groups...");
          await delay(options.scrollDelayOnEmpty + rand(2000));
          candidates = findMoreButtons().filter(b => !processed.has(b));
          if (candidates.length === 0) {
            const any = findMoreButtons().length;
            if (!any) {
              console.log("✅ Done — no more 'More' buttons found.");
              break;
            }
          }
        }

        const batch = candidates.slice(0, Math.max(1, Math.floor(options.maxPerMinute / 3)));

        for (const btn of batch) {
          if (stopRequested) break;
          try {
            await processOne(btn);
          } catch (err) {
            console.error("Error processing item:", err);
          }
          await humanDelay();
        }

        await delay(600 + rand(900));
      }
    } finally {
      window._facebookAutoLeaveRunning = false;
      if (options.verbose) console.log("facebookAutoLeave: stopped.");
    }
  }

  mainLoop().catch(err => {
    console.error("facebookAutoLeave fatal:", err);
    window._facebookAutoLeaveRunning = false;
  });

})();
```

---

## 🧠 Notes & Safety Tips

- Works best on **desktop browser** (not mobile).
- Scroll down first — Facebook loads groups gradually.
- Script auto-scrolls if no more visible groups are found.
- **Rate-limited:** keeps actions under ~20 per minute to avoid being flagged.
- You can tweak:
  ```js
  facebookAutoLeave({ maxPerMinute: 30, baseDelay: 500 })
  ```
  for faster runs (⚠️ higher risk of throttling).
- Always stop safely using:
  ```js
  window.facebookAutoLeaveStop();
  ```

---
