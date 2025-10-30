# 🗑️ Facebook Auto Unsave Script

This script automatically **unsaves all your saved posts** on [Facebook Saved](https://facebook.com/saved).  
It clicks each "More options" (⋯) button, selects **Unsave**, scrolls, and repeats until everything’s cleared.  

---

## ⚙️ How to Use

1. **Go to:** [https://facebook.com/saved](https://facebook.com/saved)
2. **Scroll a bit** so Facebook loads your saved posts.
3. **Open your browser console:**
   - Windows: `Ctrl + Shift + J`
   - Mac: `Cmd + Option + J`
4. **Paste** the script below into the console.
5. Press **Enter** and watch it go 🚀  

You can chill while it does its thing — it’ll unsave everything automatically.

---

## 💻 The Script

```js
// 👋 This script automatically "unsaves" all your saved posts on Facebook.
// Just open https://facebook.com/saved, scroll a bit so stuff loads,
// then paste this whole code into the browser console and press Enter.

async function smartUnsaveLoop() {

  // a lil helper function that lets us "wait" before doing the next action
  const delay = (ms) => new Promise((res) => setTimeout(res, ms));

  // this keeps track of items we've already touched, so we don't repeat them
  let processed = new Set();

  // keep looping until we’ve probably cleared everything
  while (true) {

    // find all those "More options" (three dots) buttons beside each saved post
    const items = [...document.querySelectorAll('div[aria-label="More options for saved item"]')]
      .filter((btn) => !processed.has(btn)); // skip ones we already did

    // if we didn’t find any new ones on screen
    if (items.length === 0) {
      // scroll down a bit to load more saved stuff
      window.scrollBy(0, 1000);

      // give Facebook a sec to load more items
      await delay(1500);

      // if still nothing shows up, we’re probably done unsaving everything
      if (document.querySelectorAll('div[aria-label="More options for saved item"]').length === 0) {
        console.log("✅ All done! Looks like you’ve unsaved everything.");
        break; // stop the loop
      }

      // otherwise, go check again (loop continues)
      continue;
    }

    // go through each saved item that’s still on the page
    for (const btn of items) {
      // mark it as already handled so we don’t repeat later
      processed.add(btn);

      // scroll that item into the center of the screen (Facebook likes that)
      btn.scrollIntoView({ block: "center" });
      await delay(200); // small chill time

      // click the "More options" (three dots)
      btn.click();
      await delay(300); // wait for the menu to show

      // look for the "Unsave" button inside the menu
      const unsave = Array.from(document.querySelectorAll('[role="menuitem"]'))
        .find((e) => e.innerText.includes("Unsave"));

      // if we found the "Unsave" option, click it
      if (unsave) {
        unsave.click();
        console.log(`🗑️ Unsave done (${processed.size})`);
      }

      // small pause before moving to the next one (keeps it smooth)
      await delay(400);
    }
  }
}

// 🚀 Run it! Just one line to start the whole thing.
smartUnsaveLoop();
```