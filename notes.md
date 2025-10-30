# Developer Notes

## How it works

- **`HTMLMediaElement.prototype.play`** – intercepts and blocks all autoplay attempts.  
- **`dataset.userAllowed` + `userInteractionFlag`** – ensures playback starts only after a user gesture (click or key).  
- **`MutationObserver`** – watches the DOM for newly added `<video>` elements (e.g., on scroll) and pauses them immediately.  
- **`play` event listener** (capture phase) – final safety net that stops any `play` event that slipped through.  
- **`requestAnimationFrame`** – forces `muted = true/false` **after** Tumblr’s internal player logic runs, guaranteeing the desired mute state.  
- **`findCenterVideo()`** – returns the `<video>` element closest to the viewport center; used by `P`, `O`, and `M`.  
- **Click emulation** – `new MouseEvent('click')` dispatched on the target video is the only reliable way to trigger Tumblr’s native player.

## Why this design

| Feature | Reason |
|--------|--------|
| No UI overlays / icons | Prevents breakage when Tumblr redesigns its player. |
| Keyboard-only control (`P`, `O`, `M`) | Fast, unobtrusive, works while scrolling. |
| `requestAnimationFrame` for mute control | Executes after the browser repaints but before the next frame – perfect timing to override Tumblr’s mute handling. |
| Triple-layer autoplay block | `prototype.play` + `MutationObserver` + `play` event = 100% coverage. |

## If Tumblr breaks the script

1. **Check** that videos are still native `<video>` elements.  
2. **Verify** that `dispatchEvent(new MouseEvent('click'))` still triggers playback.  
3. **Update** `findCenterVideo()` selectors if the dashboard layout changes.  
4. **Test** on dashboard scroll, infinite scroll, and reblogs.

## Testing checklist

- [ ] Autoplay videos are paused on load.  
- [ ] `P` plays with sound, `O` plays muted.  
- [ ] `M` toggles mute at any time.  
- [ ] New videos added while scrolling are blocked.  
- [ ] Works after page navigation (Tumblr’s SPA).  

---

*Feel free to open an issue or PR if something stops working – the script is deliberately minimal to survive redesigns.*