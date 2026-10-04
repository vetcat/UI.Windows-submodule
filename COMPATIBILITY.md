# Unity 6 compatibility fork

The UPM package at this repository root is `com.me.ui.windows` 1.2.9.
The compatibility integration branch is `unity6000-compat`; upstream remains
[chromealex/UI.Windows-submodule](https://github.com/chromealex/UI.Windows-submodule).
These patches start from upstream commit
`60a4bf6e47c85ad57935f633a53fc3ca8b707167`.

## Changes

- Unity 6000.6 uses full `EntityId` values for pool identity, Editor hierarchy
  callbacks, object-change tracking, animation cache keys and template creation.
  Draft-component creation stores the 64-bit ID as invariant text in EditorPrefs.
- `FlowLayoutGroup` supplies the new max-size argument using
  `LayoutUtility.DefaultMaxSize`. Older API paths stay behind version guards.
- `AnimationParameters.State` is serializable so inherited animation states
  survive prefab save/import. Endless-list registries are runtime-only.
- Package metadata declares stable Localization, Addressables, Input System,
  uGUI, Burst and Collections dependencies instead of the old preview UI package
  and invalid Git scoped registry. Localization uses its assembly name.
- FMOD is not a mandatory assembly reference. Projects enabling `FMOD_SUPPORT`
  must provide FMOD and add the corresponding assembly reference explicitly.
- The URP extension assembly compiles only when the URP package is installed;
  its package version define supplies `UIWINDOWS_URP` automatically.
- The shared WindowSystem prefab has no source-demo root or registered windows.
  Its camera/light and the optional Console camera contain no mandatory URP
  components. Configure rendering and game windows in project-owned prefabs or
  variants instead of editing the package.
- `WindowSystem.Shutdown()` releases modules and global ownership synchronously
  and once. Scene owners clean their windows (including pooled instances) first,
  call `Shutdown()`, then use ordinary `Object.Destroy` for the system GameObject.
  Deferred destruction of the old system cannot clear a replacement's singleton
  or pointer callbacks. Shutdown disables the component and is terminal;
  it does not move objects between scenes or create another UI lifecycle.

All existing `.meta` GUIDs are preserved. MVP adapters, CompositionRoot and game
logic belong outside this fork.

## Develop and update

Develop compatibility changes in a task branch in a local checkout of this fork.
For local verification use Unity Package Manager's Add package from disk, or
`UnityEditor.PackageManager.Client.Add("file:/absolute/path/to/checkout")`.
An embedded `Packages/com.me.ui.windows` directory overrides a dependency; move
it outside `Packages/` before switching, and retain its GUIDs/custom changes.

After focused Editor compilation and lifecycle/serialization checks, review and
merge the patch into `unity6000-compat`. Consumers then install/update using
`https://github.com/vetcat/UI.Windows-submodule.git#<verified-commit>`.
Commit immutable revisions in consumer manifests and the generated UPM lock.
Never make persistent fixes in `Library/PackageCache`.

## Verification record

Task [QP-1](https://linear.app/qpixelstudio/issue/QP-1/naladit-obshij-upm-workflow-uiwindows-dlya-pixellords-i)
contains the candidate revisions, PRs and actual verification results for Unity
6000.6.4f1 in PixelLords and QuantumAsteroids. Older version guards are retained;
this task does not claim a fresh validation of every older Editor version.
