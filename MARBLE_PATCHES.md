# Marble Lightship patch — Issue #468

- Upstream: niantic-lightship/ardk-upm, 3.17.0-2512092207.
- Commit: `03f2bdc13fbc14e1fb16c6231ad7f413fa8ab143`.
- All 1,056 upstream files matched the official recursive Git tree blob hashes before patching.
- Cache package.json and macOS binary differed; restored both from the pinned upstream before the original patch.
- Package name/version, GUIDs, assemblies, native plugins, dependencies and license remain upstream.
- This fork is the authoritative patch source. Consumers must pin its commit SHA; do not patch Library/PackageCache.

## Local patch: issue-468-depth-recovery-v1 (2026-09-27)

Only four upstream files change:

- `Runtime/Subsystems/Common/ExternalTexture/LightshipExternalTexture.cs`: reject missing/invalid descriptors before comparing types; retain the last valid wrapper and descriptor for recovery. Valid incompatible type changes still throw.
- `Runtime/ARFoundation/Overrides/Occlusion/Extension/Features/OcclusionComponent.cs`: accept only valid, nonzero native Texture2D descriptors at the platform depth boundary; return a texture on success and null on failure.
- `Runtime/ARFoundation/Overrides/Occlusion/Extension/Features/ZBufferOcclusion.cs`: reset the material to its default depth when acquisition fails; publish `_MarbleDepthAvailable` on attach and each material update.
- `Assets/Shaders/ZBufferOcclusion.shader`: discard missing-depth fragments before sampling frame/fused depth, leaving color and depth untouched, including stabilization variants.

OcclusionMesh, CPU Fastest acquisition, requested depth modes, camera switching, Scene settings and application authentication are unchanged. No per-frame missing-descriptor warning is added.

## Verification

Marble selects this fork through a commit-pinned Git UPM dependency. Upstream Git blob hashes were checked for all 1,056 files; only the four files above differ after patching. All upstream files, including debug symbols, are retained by this fork.

Tests in `Assets/Marble/Tests/Presentations/Managers/Lightship*Tests.cs` exercise the real internal SDK methods through test-only reflection, an isolated provider, genuine native texture handles and a shader render/depth probe. There are 16 parameterized cases. They have NOT yet passed Unity Test Runner: the running Editor's main-thread API requests time out. Standalone Unity Roslyn compile checks do not establish runtime correctness. Player builds and device validation are pending.

Detailed results and limitations: [Marble verification record](https://github.com/MarbleXR/marble/blob/fix/issue-468-lightship-depth-recovery/docs/implementations/20260921_3_lightship-invalid-depth-texture/verification-1.md).

## Updating / removing the patch

1. Select and pin an upstream version; inspect all four behaviors, including missing-frame rendering. NSDK 4.1 alone still contains the None/type-check defect.
2. Run the descriptor, platform-depth and rendering tests against the unpatched candidate, plus existing ARRoom occlusion/session and AR Foundation shutdown tests.
3. Validate iOS/Android builds and devices: ON/OFF, front/back camera, resume, exit/re-entry, capture and existing anchors. Verify CPU Fastest and GPU fallback separately.
4. Only when upstream passes, point the consumer manifest at the validated official commit; resolve and check the lock and all dependent packages.
5. If upstream does not pass, retain/rebase this fork patch and update the upstream identity, patch ID and validation record. Never fix only PackageCache.
