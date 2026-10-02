## New in v1.30

## New Features

### `--output-locale` argument

Added a new `--output-locale` argument that overrides the language used for WinGet's own output for a single invocation.
It is available on every command, accepts the same values as the `output.locale` setting (`en-US`, `de-DE`, `es-ES`, `fr-FR`, `it-IT`, `ja-JP`, `ko-KR`, `pt-BR`, `ru-RU`, `zh-CN`, `zh-TW`), and takes precedence over that setting.

`--output-locale` is independent of `--locale`: WinGet messages are shown in the locale requested by `--output-locale`, while manifest strings and package selection continue to use `--locale`.

Usage: `winget show <package> --locale zh-CN --output-locale de-DE`

### Output locale override

Added a persistent `output.locale` setting to override winget interface language using a BCP47 tag.

Usage: add `"output": { "locale": "de-DE" }` to `settings.json`.

### `--ignore-unavailable` flag for `install`

Added a new `--ignore-unavailable` flag to the `install` command. When installing multiple packages, this flag allows the operation to continue with the remaining packages instead of failing entirely when one or more packages are not found in the configured sources. This brings the same behavior previously available with `import --ignore-unavailable` to direct multi-package installs.

### COM install dependencies-only option

Added the contract version 30 `InstallOptions.InstallDependenciesOnly` property for COM callers that need to install a package's dependencies without installing the requested package.

## Bug Fixes

### Portable installer alias handling

Portable installs now preserve the original executable filename instead of renaming it when an alias is needed.
For aliases requested through `--rename`, `Commands`, or `PortableCommandAlias`, WinGet creates a hardlink alias and keeps the original file as the source executable.

This change resolves alias failures in non-symlinked scenarios, including cases where WinGet adds the install directory to `PATH` instead of creating links.
Because the alias is now created as an executable hardlink in the install location, command aliases remain available and consistent even when symlink creation is skipped.

### OS version specific installers

Manifests can now include installers that differ only by `MinimumOSVersion`, for example to declare different dependencies on different OS versions.
These are no longer rejected as duplicate installer entries, and when more than one is applicable, the installer with the highest `MinimumOSVersion` is preferred regardless of its order in the manifest.
Architecture, scope, locale and installer type preferences still take priority over `MinimumOSVersion`.

For example, a package that needs .NET Framework 3.5 can depend on the `NetFx3` Windows feature on older OS versions, and on a standalone .NET Framework 3.5 package from `10.0.28000.0`:

```yaml
Installers:
  - Architecture: x64
    InstallerUrl: https://example.com/setup.exe
    InstallerSha256: <hash>
    Dependencies:
      WindowsFeatures:
        - NetFx3
  - Architecture: x64
    InstallerUrl: https://example.com/setup.exe
    InstallerSha256: <hash>
    MinimumOSVersion: 10.0.28000.0
    Dependencies:
      PackageDependencies:
        - PackageIdentifier: Microsoft.DotNet.Framework.Runtime.3
```

### Minor Bug Fixes
* Fixed an issue where `winget search --id <msstoreId>` could fail to return a Microsoft Store package unless `--exact` was also provided.
* Updated NUnit to v4
* Fixed a crash (`0x8000ffff`) when using `--disable-interactivity` with the Resume experimental feature enabled during install operations.
* Fixed relative path handling for rooted paths.
