# 🚫 Facebook Auto Unfollow Script

This script automatically **unfollows all profiles or pages** you’re currently following on Facebook.  
It scrolls through your following list, hovers on each profile, opens the follow options, and clicks **Unfollow** — all automatically.

---

## ⚙️ How to Use

1. **Go to your following list:**  
   👉 [https://facebook.com/yourusername/following](https://facebook.com/yourusername/following)

   _(Or navigate manually: Profile → “Friends” → “Following” tab)_

2. **Scroll a bit** so Facebook loads your following list.
3. **Open your browser console:**
   - Windows: `Ctrl + Shift + J`
   - Mac: `Cmd + Option + J`
4. **Paste** the script below into the console.
5. Press **Enter** and let it run 🚀

The script will automatically unfollow everyone in view, scroll, and continue until the list is empty.

---

## 💻 The Script

```js
(async function () {
  'use strict';
  console.log('%c[Facebook Auto Unfollow Script Started]', 'color:#1877F2;font-weight:bold;');

  const INTERVAL_BETWEEN_ACTIONS = 3500;
  const SHORT_FIXED_PAUSE = 1500;
  const HOVER_WAIT_TIMEOUT = 4000;
  const MAX_SCROLL_ATTEMPTS = 3;

  let processedProfiles = new Set();
  let successCount = 0;
  let failCount = 0;
  let scrollAttemptsWithoutNew = 0;
  let processActive = true;

  const wait = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

  async function waitForElement(selector, timeout = HOVER_WAIT_TIMEOUT) {
    const start = Date.now();
    while (Date.now() - start < timeout) {
      const el = document.querySelector(selector);
      if (el) return el;
      await wait(100);
    }
    return null;
  }

  async function tryUnfollow(hoverContainer) {
    // 🟦 Case 1: Direct "Unfollow" button in hover card
    const directBtn = hoverContainer.querySelector('div[role="button"][aria-label="Unfollow"]');
    if (directBtn) {
      directBtn.click();
      console.log('%c[✅ Unfollowed]%c Directly from hover menu', 'color:green;', 'color:white;');
      return true;
    }

    // 🟨 Case 2: "Following" or "Liked" → Open dialog → Unfollow + Unlike
    const followingBtn =
      hoverContainer.querySelector('div[role="button"][aria-label="Following"]') ||
      hoverContainer.querySelector('div[role="button"][aria-label="Liked"]');

    if (followingBtn) {
      followingBtn.click();
      await wait(SHORT_FIXED_PAUSE);

      const dialog = await waitForElement('div[role="dialog"]', 6000);
      if (!dialog) {
        console.warn('[❌ Fail] Follow settings dialog not found.');
        return false;
      }

      console.log('%c[🧭 Dialog detected]%c Proceeding to unfollow + unlike...', 'color:#00B0FF;', 'color:white;');

      // Step 1: Select "Unfollow" radio
      const unfollowRadio = Array.from(dialog.querySelectorAll('div[role="radio"]'))
        .find((el) => el.textContent.includes('Unfollow'));
      if (unfollowRadio) {
        unfollowRadio.click();
        console.log('✅ Unfollow radio selected.');
        await wait(400);
      }

      // Step 2: Toggle "Unlike this Page" (handles both div + input cases)
      const unlikeButton = Array.from(document.querySelectorAll('div[role="button"], input[role="switch"]'))
        .find((el) => el.textContent.includes('Unlike this Page') || el.getAttribute('aria-label') === 'Unlike');
      if (unlikeButton) {
        unlikeButton.click();
        console.log('✅ Unlike this Page toggled.');
        await wait(400);
      }

      // Step 3: Click "Update"
      const updateBtn = Array.from(dialog.querySelectorAll('div[role="button"], button'))
        .find((el) => el.textContent.trim() === 'Update');
      if (updateBtn) {
        updateBtn.click();
        console.log('💾 Update button clicked.');
        await wait(800);
        return true;
      }

      console.warn('[⚠️ Warning] Update button not found.');
      return false;
    }

    // 🟩 Case 3: Fallback – via “Options” dropdown
    const optionsBtn = hoverContainer.querySelector('div[role="button"][aria-label*="Options"]');
    if (optionsBtn) {
      optionsBtn.click();
      await wait(SHORT_FIXED_PAUSE);

      const unfollowMenu = Array.from(document.querySelectorAll('div[role="menuitem"]'))
        .find((el) => el.textContent.includes('Unfollow'));
      if (unfollowMenu) {
        unfollowMenu.click();
        console.log('%c[✅ Unfollowed]%c via Options menu', 'color:green;', 'color:white;');
        return true;
      }
    }

    console.warn('[❌ Fail] No valid unfollow option found.');
    return false;
  }

  async function processOneProfile() {
    const profileContainers = document.querySelectorAll('div.x78zum5.x1q0g3np.x1a02dak.x1qughib > div');
    let foundProfile = false;

    for (const profile of profileContainers) {
      const link = profile.querySelector('a[href*="facebook.com/"]');
      if (!link || processedProfiles.has(link.href)) continue;

      foundProfile = true;
      scrollAttemptsWithoutNew = 0;
      console.log(`%c[⚙️ Processing]%c ${link.href}`, 'color:cyan;', 'color:white;');

      profile.scrollIntoView({ behavior: 'smooth', block: 'center' });
      await wait(500);

      // Hover to trigger profile preview
      link.dispatchEvent(new MouseEvent('mouseover', { bubbles: true, cancelable: true }));
      const hoverContainer = await waitForElement('div[aria-label="Link preview"]');

      if (!hoverContainer) {
        console.warn('[❌ Fail] Hover popup not found (timeout).');
        failCount++;
      } else {
        const success = await tryUnfollow(hoverContainer);
        if (success) successCount++;
        else failCount++;

        await wait(500);

        const closeBtn = document.querySelector('div[role="button"][aria-label="Close"]');
        if (closeBtn) closeBtn.click();
      }

      processedProfiles.add(link.href);
      break;
    }

    if (!foundProfile && profileContainers.length > 0) {
      scrollAttemptsWithoutNew++;
      console.log(`[ℹ️ Scroll] Attempt ${scrollAttemptsWithoutNew}/${MAX_SCROLL_ATTEMPTS} - no new profiles.`);
      if (scrollAttemptsWithoutNew >= MAX_SCROLL_ATTEMPTS) {
        processActive = false;
        console.log('%c[🏁 Finished] All profiles processed.', 'color:#4CAF50;font-weight:bold;');
      } else {
        window.scrollTo(0, document.body.scrollHeight);
      }
    }
  }

  while (processActive) {
    await processOneProfile();
    await wait(INTERVAL_BETWEEN_ACTIONS);
  }

  console.log(`%c[Summary] ✅ ${successCount} success | ❌ ${failCount} failed`, 'color:#FFD700;font-weight:bold;');
})();
```

---

## 🧩 Notes

- It’s safe to run — it only clicks visible "Unfollow" actions.
- Facebook’s UI might change anytime, so if it stops working, check for updated selectors.
- The script logs **success** and **fail** counts in the console.

---
