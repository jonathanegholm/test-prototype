# 001 — Animate the account dropdown from its trigger

- **Status**: DONE
- **Commit**: n/a (the folder is not a git repository, so there is no hash to stamp)
- **Severity**: MEDIUM
- **Category**: Physicality & origin (with Interruptibility)
- **Estimated scope**: 1 file (`saxo-investor.html`), about 12 lines of CSS, 0 lines of JS

## Problem

The "All accounts" switcher menu appears and disappears instantly because it is toggled with `display:none` / `display:block`. `display` cannot transition, so the menu has no connection to the trigger it came from.

```css
/* saxo-investor.html:49-50 — current */
.menu{position:absolute;top:100%;left:0;background:#fff;border:1px solid var(--bd);border-radius:var(--r);min-width:220px;padding:4px;z-index:5;display:none;font-size:14px;font-weight:400}
.menu.open{display:block}
```

```css
/* saxo-investor.html:165 — current (shadcn override block; adds shadow, radius, margin-top) */
.menu{border-radius:var(--r-md);padding:4px;box-shadow:0 4px 6px -1px rgb(0 0 0/.1),0 2px 4px -2px rgb(0 0 0/.1);min-width:200px;margin-top:4px}
```

The menu is opened and closed from JS by toggling the `open` class. This code is correct and must not change:

```js
// saxo-investor.html:316-321 — current, do not edit
$('acct').onclick=e=>{ ... $('acctMenu').classList.toggle('open')};
$('acct').onkeydown=e=>{ ... $('acctMenu').classList.toggle('open') ... classList.remove('open') };
document.addEventListener('click',e=>{if(!e.target.closest('#acct'))$('acctMenu').classList.remove('open')});
```

## Target

The menu grows out of its top-left corner (where the title text sits) and fades in. It uses transitions, not keyframes, so rapid open/close retargets from the current state instead of restarting. Closing is faster than opening. Only `opacity` and `transform` animate; `visibility` is switched on a delay so the closed menu is not focusable or clickable.

```css
/* target — replace the display lines, keep every other .menu declaration */
.menu{
  display:block;
  visibility:hidden;
  opacity:0;
  transform:scale(.96);
  transform-origin:top left;
  pointer-events:none;
  transition:
    opacity 100ms cubic-bezier(0.23, 1, 0.32, 1),
    transform 100ms cubic-bezier(0.23, 1, 0.32, 1),
    visibility 0s linear 100ms;
}
.menu.open{
  visibility:visible;
  opacity:1;
  transform:scale(1);
  pointer-events:auto;
  transition:
    opacity 150ms cubic-bezier(0.23, 1, 0.32, 1),
    transform 150ms cubic-bezier(0.23, 1, 0.32, 1),
    visibility 0s linear 0s;
}
@media (prefers-reduced-motion:reduce){
  .menu{transform:none}
  .menu.open{transform:none}
}
```

Values and why:
- Open 150ms, close 100ms: within the 150–250ms dropdown budget, and the close snaps because the user has already decided.
- Curve `cubic-bezier(0.23, 1, 0.32, 1)` is a strong ease-out. Never `ease-in`.
- Start at `scale(.96)`, never `scale(0)`.
- Origin `top left`: the menu is positioned at `left:0; top:100%` under the title, so that corner is the trigger edge. (There is no Base UI `--transform-origin` variable here, so it is hard-coded.)
- Reduced motion drops the scale but keeps the opacity fade.

## Repo conventions to follow

- This is a single plain-HTML file with no motion library and no easing tokens. Existing transitions are inline literal values, for example line 11: `transition:background-color .15s ease,color .15s ease,border-color .15s ease,transform .1s ease`.
- Do not introduce tokens or a library. Use the literal values above.
- The reduced-motion convention already exists at the top of the file: `@media(prefers-reduced-motion:reduce){*{transition:none!important}}`. That rule removes all transitions under reduced motion, including the opacity fade. The target block above intentionally keeps the fade, so also change that global rule in Step 3.

## Steps

1. In `saxo-investor.html` at line 49, edit the `.menu{...}` declaration: delete `display:none;` only. Leave every other property in place.
2. At line 50, replace `.menu.open{display:block}` with the `.menu` and `.menu.open` transition blocks from Target. Add the `display:block; visibility:hidden; opacity:0; transform:scale(.96); transform-origin:top left; pointer-events:none; transition:...` properties to the base `.menu` rule (line 49 or a following rule — it must not conflict with line 165, which only sets radius/padding/shadow/min-width/margin-top).
3. At line 11 area, change `@media(prefers-reduced-motion:reduce){*{transition:none!important}}` to `@media(prefers-reduced-motion:reduce){*{transition-duration:.01ms!important}.menu{transition:opacity 100ms linear!important}}`, and add the `.menu, .menu.open {transform:none}` reduced-motion rule from Target. Net effect under reduced motion: no movement, and a short opacity fade on the menu only.
4. Do not edit the JS at lines 315-321.

## Boundaries

- Do NOT touch the JS, the markup, the chevron icon, or any other component's transitions.
- Do NOT change `.menu div` item styling (lines 51-53, 166-167).
- Do NOT add dependencies, libraries or animation keyframes.
- Do NOT animate `height`, `width`, `top`, or `box-shadow`.
- If the CSS you find does not match the excerpts above (line numbers have drifted or the rules differ), STOP and report instead of improvising.

## Verification

- **Mechanical**: open the file in a browser and check the console for errors (expected: none). In DevTools, confirm `.menu` computed `visibility` is `hidden` when closed and `visible` when open.
- **Feel check**: click "All accounts" and confirm:
  - The menu grows from its top-left corner, under the title text, not from the center.
  - Opening feels quick but not abrupt; closing (click elsewhere, Escape, or choose an item) is visibly snappier than opening.
  - Spamming the title click never restarts the animation from zero; it reverses smoothly from wherever it was.
  - Tab and keyboard: with the menu closed, nothing inside it takes focus or clicks (it is `visibility:hidden` and `pointer-events:none`).
  - In DevTools Animations panel, set playback to 10% and confirm only opacity and transform change (no layout shift of the page below).
  - Enable `prefers-reduced-motion` (Rendering panel) and confirm there is no scale movement but a short fade remains.
- **Done when**: the menu fades and scales in from the top-left in about 150ms, closes in about 100ms, never flashes when toggled rapidly, and the other components behave exactly as before.
