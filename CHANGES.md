# Changelog

## 4D bounding boxes (3D + time)

### Changed

**`get_bbox_from_mask()` (`cropping_and_padding/bounding_boxes.py`) — optional time axis**
- The function now accepts both 3D masks `(z, x, y)` and 4D masks `(t, z, x, y)`.
- 3D input is internally promoted to 4D via `mask = mask[None]` and flagged with
  `has_time_dim = False`, so existing callers are unaffected.
- All per-axis scans were reindexed for the extra leading dimension
  (`mask[:, z]`, `mask[:, :, x]`, `mask[:, :, :, y]`), i.e. each spatial bound is now
  determined over *all* timesteps rather than a single volume.
- A new scan over the leading axis determines the temporal bounds.
- Return value:

  | Input | Return |
  |---|---|
  | 3D `(z, x, y)` | `[[minz, maxz], [minx, maxx], [miny, maxy]]` (unchanged) |
  | 4D `(t, z, x, y)` | `[[mint, maxt], [minz, maxz], [minx, maxx], [miny, maxy]]` |

### Compatibility

- Backwards compatible for 3D masks: same 3-entry bounding box as before.
- Callers that consume the result generically (e.g. `crop_to_bbox`, `bounding_box_to_slice`,
  `int_bbox`) work unchanged as long as they iterate over the list instead of unpacking
  exactly three entries. Any code doing `z, x, y = get_bbox_from_mask(...)` will break on
  4D input.