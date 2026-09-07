# Image encoder benchmark for jellyfin/jellyfin#17775

Cold/warm timings of `GET /Items/{id}/Images/Primary?maxWidth=663&maxHeight=995&quality=90`
for 20 movie posters, measured on the same server, same database copy and same items.

- Hardware: Intel i3-13100 (8 threads), Docker 28, Jellyfin 12.0.0 runtime image.
- Each item: one cold request against an empty image cache, then two warm requests.
- `a-skia`: Shadowghost's `libvips` branch (37c45b7), `ImageEncoder=Skia` (default) — the Skia path as shipped today.
- `a-vips`: same build, `ImageEncoder=NetVips`.
- `b-skia-fix`: PR #17775 branch (064de21) — Skia with the direct sharpening kernel.

## Files

- `timings-<config>.csv` — columns `item_id, cold_ms, warm1_ms, warm2_ms, bytes` (milliseconds, output size in bytes).
- `items.txt` — the 20 item ids in measurement order (identical for all three runs).
- `*_crop1_4x.png` — 4x crops of "3 Idiots" (the poster with the largest single-pixel delta) at the max-delta location, libvips vs. PR fix.
- `*_crop2_center_4x.png` — 4x crops of the same poster at the image centre as a neutral reference.

## Summary

| config | cold median | cold p95 | warm median |
|---|---|---|---|
| a-skia | 4551 ms | 5185 ms | 16 ms |
| b-skia-fix | 655 ms | 916 ms | 16 ms |
| a-vips | 153 ms | 284 ms | 15 ms |

PSNR over the 18 posters that are actually resized: old Skia vs. PR fix median ~47 dB (44–54),
libvips vs. either Skia variant median ~36 dB (34–47). Output dimensions identical for all 60 files.
