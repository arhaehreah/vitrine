# Plan: Lyrics color follows cover-art accent

## Scope
Add a Mod Settings switch (both looks; lyrics is Shared) so the active/sung lyric color uses the extracted cover-art accent color already computed for the redesigned player's Shuffle/Device/Like controls (`SGRPlayerSetCoverAccent`). Native look has no cover-accent extraction today, so initially only Redesigned benefits; Native lyrics page (Spotify's own) cannot be recolored easily.

## Findings
- Cover accent computed in `Redesigned/Player/PlayerField.x` `followCover()` -> `SGRPalette paletteForImage` -> most saturated of `palette.flowColors`, passed to `SGRPlayerSetCoverAccent(best)` in `PlayerCoverAccent.x`.
- `SGRPlayerSetCoverAccent` clamps: saturation [0.45,0.85], brightness max(0.9,v) -> readable bright color. Gray covers (saturation<0.15) yield -1 (accent fallback). This is the contrast handling.
- Lyrics rendering: `Redesigned/Lyrics/SGRKaraokeView.m` uses `SGRAccent()` for active line presumably via `SGRTokens`. Need hook accent resolution to cover color when switch on.

## Files to modify/create
1. `Core/SGPrefs.h` + `.m`: add key `SGKeyLyricsFollowCoverColor` (shared, not redesign-only) `spotifyglass.lyrics.followCoverColor`, plus migrate? no old key.
2. `Redesigned/Player/Player.h`: expose `SGRPlayerCoverAccentColor()` (returns UIColor or nil) reading `sg_cover`.
3. `Redesigned/Player/PlayerCoverAccent.x`: implement `SGRPlayerCoverAccentColor()`.
4. `Redesigned/Kit/SGRAccent.h/.x` (or tokens): add `SGRAccent()` override path? Better: in lyrics view, when switch on and cover color available, use cover color; else `SGRAccent()`.
5. `Redesigned/Lyrics/SGRKaraokeView.m`: use cover color for active line when enabled.
6. Settings rows: `Redesigned/Lyrics/LyricsLookSettings.m` or a Lyrics settings page; plus App/Pages.m section. Need inspect settings pattern. Add switch row.
7. `Native/` maybe no change (native lyrics page is Spotify's). Could skip.
8. Makefile: no new files if editing existing.

## Layer check
- `Redesigned/Player` -> `Redesigned/Lyrics`? Lyrics imports Player? Currently Lyrics doesn't import Player. To avoid layer violation (same layer Redesigned can import within Redesigned? Layer rules allow within same layer). Fine. But exposing cover color to Kit is okay.
- Core/SGPrefs lower; Redesigned higher. Fine.

## Risks
- Cover color only updates on track change; lyrics view must observe `SGRPlayerSetCoverAccent` or read live. Add notification `SGRPlayerCoverAccentDidChangeNotification`.
- Contrast: reuse clamped color; still may clash with dark background? Active line colored; okay.

## Open questions
- Should switch be redesign-only key (`spotifyglass.redesign.lyrics.followCoverColor`) since cover accent is redesign-only? User says lyrics Shared. But feature only meaningful in redesign. Use redesign key to match "Follow the cover" setting location. Recommend redesign key and place row in Redesigned lyrics settings, gated by RedesignedUI. Native users see nothing.

Plan implementation agent should inspect settings registration patterns in App/Pages.m and Redesigned/Lyrics settings.<|end|><|start|>startup to=bash<|channel|>analysis <|constrain|>xml<|message|><rfl:parameter name="command">
mkdir -p C:\Users\filip\Desktop\vitrine\vitrine\.kilo\plans