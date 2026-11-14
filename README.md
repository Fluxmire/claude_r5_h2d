# Image to Three.js miner (claude_r5 H2d)

Entry point: `main.py` -> `async generate(image, seed, witness) -> str | None` (the competition witness runtime).
The active plan is `ACTIVE_PLAN` in `miner404/versions.py`.

## Base and changes

Base: [SomPeer/copperbellows@d481cfc](https://github.com/SomPeer/copperbellows/tree/d481cfc14057e25d5618a76098ecc54343639f30)
(its code is identical to the round-4 winner TiltGilt/VertexLoom@2385ec1; only the plan differs).

Plan relative to the round-4 winner's:

- same 13 gated rewrite steps, fallbacks, `judge_pick` gates, `replan`, `judged_tilt`, `outline_fit_last`, and
  SEARCH/REPLACE patch edits on rewrite steps 8-13;
- 6 Sol-high drafts as 2 plain + 2 proportions + 2 inventory (parts, counts, colours listed before building);
- no draft merge, no tone pass;
- `routed=True`: one Sol-low call (request 9995) labels the photo; upright furniture/structures skip both the photo
  tilt and the judged tilt, tall photos skip the photo tilt. A refused or failed label call leaves the base behaviour.
- finishing after the judged tilt, each step kept only if the renderer accepts it:
  - `texfix`: data textures left at three.js's NearestFilter defaults get trilinear sampling with mipmaps;
  - judged stretch: the module's shorter bounding-box axes are enlarged (never past 0.999 of the cube; round bodies keep
    a round section; modules naming a spout are left alone). x1.25 is used unless x1.0, x1.15 or x1.4 beats it in the
    selector (requests 9980/9981, both orders) by `STRETCH_MARGIN`; a refused call keeps x1.25.

Code changes: `miner404/route.py` (new), `miner404/tilt.py` (`photo_is_tall`), `miner404/pipeline.py` (router call,
both tilt gates, HTTP 504 `render_timeout` counted as a render timeout so a too-heavy module is sent back to be made
lighter, and the finishing steps), `miner404/versions.py` (plan). Everything else is the base verbatim.

## Local results vs the round-4 winner (VertexLoom)

Local S1 replica judge (z-ai/glm-5.3-flash, upstream stage-1 prompts). Margin = mean over 5 judge seeds of
(wins - losses) / prompts; generation on a renderer built from the subnet Pod's renderer-runtime stage (30 s preview limit).

| Set | H2d margin |
|---|---|
| 16 dev prompts (round 4) | +16.2% |
| 16 random holdout prompts (round 4) | +25.0% |
| All 32 | +20.6% |

Budget per prompt in those runs: at most 56 calls and 4.36M credits (limits 60 / 5M).
