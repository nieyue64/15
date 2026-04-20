# Phase 3 Report: Pipeline Deconstruction (applyPostFX)

## Status: COMPLETE

## What Was Done

### 1. Introduced `Renderer.postFxPipeline` Architecture
- Added `this.postFxPipeline = []` to Renderer constructor
- Added `registerPostFxStage(name, order, fn)` method to Renderer prototype
  - Prevents duplicate registration (checks by name)
  - Stages auto-sorted by `order` value
  - Recommended orders: Weather=100, Underwater=200, Custom=300+
- The native `applyPostFX` now iterates through pipeline stages AFTER running the base effects (bloom, vignette, grain)
- Each stage is wrapped in try/catch for fault isolation

### 2. Eliminated Redundant Weather PostFX Layer
- **Problem**: `__weatherPostTintInstalled` (Layer 2) was SUPERSEDED by `__weatherPostTintOptimized` (Layer 3), but both were active, causing the weather tint to be applied TWICE in certain code paths
- **Fix**: Disabled `__weatherPostTintInstalled` by changing its guard to `false`
- Layer 3 (`__weatherPostTintOptimized`) handles everything Layer 2 did, with better caching

### 3. Scoped Canvas Optimizer (from Phase 0)
- The global `HTMLCanvasElement.prototype.getContext` hijack was already replaced in Phase 0 with `TU_CanvasOptimizer.optimize(ctx)` scoped to game canvas only

### PostFX Execution Order (After Refactor)
1. **Base** (native): Bloom, vignette, color grading (warm/cool/fog), grain
2. **Weather Optimized** (`__weatherPostTintOptimized`): Weather color tint + lightning
3. **Underwater Fog** (`__underwaterFogInstalled`): Depth fog + underwater overlay
4. **Pipeline stages** (via `postFxPipeline`): Future effects register here

### Pipeline API Usage Example
```javascript
renderer.registerPostFxStage('customEffect', 150, (renderer, time, depth01, reducedMotion) => {
  const ctx = renderer.ctx;
  // ... draw custom post-processing effect
});
```

## Files Modified
- `part3_game_single (6) (9)(9)(2)(6).html` (main game file)

## Backup
- `part4_phase3_pipeline.html` - backup after Phase 3 completion

## Completion: 100%
