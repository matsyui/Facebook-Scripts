# 👋 Facebook Auto Unfriend Script

This script automatically **unfriends all your Facebook friends** from the [Friends List](https://facebook.com/{YOUR_USERNAME}/friends).  
It clicks each “Friends” button ➜ selects **Unfriend** ➜ confirms it ➜ scrolls ➜ and repeats until your list’s empty.  

---

## ⚙️ How to Use

1. **Go to:** [https://www.facebook.com/{YOUR_USERNAME}/friends](https://facebook.com/{YOUR_USERNAME}/friends)  
2. **Scroll a bit** so Facebook loads more friends.  
3. **Open your browser console:**
   - Windows: `Ctrl + Shift + J`
   - Mac: `Cmd + Option + J`
4. **Paste** the script below into the console.  
5. Press **Enter** and let it handle the rest ⚡  

You can just chill while it does the unfriending one-by-one — it even auto-confirms the popup.  

---

## 💻 The Script

```js
// 👋 This script automatically unfriends all your friends on Facebook.
// Just open https://facebook.com/{YOUR_USERNAME}/friends, scroll a bit to load your friends,
// then paste this code into your browser console and press Enter.

async function smartUnfriendLoop() {
  // lil helper so we can wait between actions
  const delay = (ms) => new Promise((res) => setTimeout(res, ms));

  // keep track of processed friends
  let processed = new Set();

  while (true) {
    // find all "Friends" buttons
    const buttons = [...document.querySelectorAll('div[aria-label="Friends"]')]
      .filter((btn) => !processed.has(btn));

    // if none found, scroll and wait
    if (buttons.length === 0) {
      window.scrollBy(0, 1000);
      await delay(1500);

      // check if we’re really done
      if (document.querySelectorAll('div[aria-label="Friends"]').length === 0) {
        console.log("✅ All done! Looks like you’ve unfriended everyone.");
        break;
      }

      continue;
    }

    // loop through visible friends
    for (const btn of buttons) {
      processed.add(btn);
      btn.scrollIntoView({ block: "center" });
      await delay(300);

      // open the dropdown
      btn.click();
      await delay(500);

      // find “Unfriend” in the menu
      const unfriend = Array.from(document.querySelectorAll('[role="menuitem"]'))
        .find((e) => e.innerText.includes("Unfriend"));

      if (unfriend) {
        unfriend.click();
        await delay(500);

        // confirm the popup if it appears
        const confirm = Array.from(document.querySelectorAll('[aria-label="Confirm"]'))
          .find((e) => e.innerText.includes("Confirm"));
        if (confirm) {
          confirm.click();
          console.log(`👋 Unfriended (${processed.size})`);
        }
      }

      await delay(800); // slight pause before next friend
    }
  }
}

// 🚀 Run it!
smartUnfriendLoop();
