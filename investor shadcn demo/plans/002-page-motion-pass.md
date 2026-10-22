# 002 — Page motion pass: bridge five teleporting state changes

- **Status**: DONE (applied directly at the user's request; plan written after the fact)
- **Commit**: n/a (not a git repository). Pre-change backup: `saxo-investor.v12-pre-motion.html`
- **Severity**: MEDIUM
- **Category**: Missed opportunities / Physicality
- **Estimated scope**: 1 file (`saxo-investor.html`), about 25 lines of CSS and 12 lines of JS

All values reuse the file's existing token `--ease: cubic-bezier(0.23, 1, 0.32, 1)`. No new easing curve was added.

| # | Surface | Before | After |
| --- | --- | --- | --- |
| 1 | Corporate-actions alert, dismiss | `display:none`, page jumps 70px | Collapses via `grid-template-rows 1fr→0fr` 220ms + `margin-bottom` 220ms + `opacity` 150ms, then `display:none` on `transitionend` (320ms timeout fallback) |
| 2 | Positions group collapse / expand | Rows pop in and out | Rows fade in on re-open: `opacity 0→1`, 150ms. Collapse stays instant. The chevron already rotated. Sorting and tab changes do not animate |
| 3 | Chart ↔ table toggle | `display` swap | Outgoing fades 80ms, incoming fades 150ms |
| 4 | Allocation tab change | Donut and legend snap | Donut fades in 150ms. Legend and segment angles do not animate. Not played on first load |
| 5 | KPI bars and meters, first load only | Appear fully drawn | `scaleY` / `scaleX` 0→1, 300ms, 30ms stagger across the 10 dividend bars, class removed after 1s so it never replays |

Reduced motion: #1 keeps a 100ms opacity fade only. #2–#5 are removed.

## Boundaries (what was deliberately NOT animated)

Chart line draw-in, headline count-up, hover tooltip easing, sidebar collapse (animates layout width), Positions sort reorder, and any further additions to the ticker or the invite card.

## Verification done

Checked by script at 1600px: the transition values and end states for each item, the alert's page jump (70px, equal to the banner height plus margin), no console errors. **Not checked by eye:** how the 150ms fades feel on dense numeric rows. The preview pane freezes animations, so frame-by-frame feel is untested.
