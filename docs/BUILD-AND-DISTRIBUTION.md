# byv-agent Build and Distribution Specification

## 1. Goals

- Bun is the required runtime for development and source execution.
- The released CLI should be a standalone executable that runs without a separately installed Bun or Node.js runtime.
- Release artifacts are target-specific and tested on a clean machine.
- Configuration, credentials, Skills, Plugins, MCP servers, project files, Git, language servers, Docker, and browser engines remain external runtime resources unless explicitly embedded.
- Android APK is treated as a separate native/wrapper application, not as a direct Bun compile target.

## 2. Bun toolchain

Use Bun for:
- runtime;
- package installation and lockfile;
- workspaces;
- TypeScript/JavaScript execution;
- unit/integration test runner;
- bundler;
- standalone executable compilation;
- project scripts.

Repository baseline:
- pin the Bun release version in CI and document the supported version;
- commit bun.lock (or the lockfile format selected by the pinned Bun version);
- use frozen/locked dependency installation in CI;
- use bunfig.toml only for project-wide Bun configuration;
- use package.json scripts for repeatable commands.

Conceptual developer commands:

    bun install
    bun run dev
    bun test
    bun run typecheck
    bun run build
    bun run compile

Exact script names and source entrypoint will be established when implementation starts.

## 3. Standalone executable behavior

Bun's stable standalone compiler path is:

    bun build --compile ./apps/cli/src/index.ts --outfile ./dist/byv

This creates an executable containing the application bundle and a copy of the Bun runtime. A user can run:

    ./dist/byv --help

without separately installing Bun or Node.js.

Use `bun build --compile` as the default production path. Bun's separate ahead-of-time `bun compile` mode is experimental in the official documentation and should not be the default release path until explicitly evaluated and tested.

## 4. Initial target matrix

Build and publish a separate artifact for each supported target:

| OS | Target | Artifact convention |
|---|---|---|
| Windows x64 | bun-windows-x64 | byv.exe |
| Windows ARM64 | bun-windows-arm64 | byv.exe |
| Linux x64 glibc | bun-linux-x64 | byv |
| Linux ARM64 glibc | bun-linux-arm64 | byv |
| Linux x64 musl | bun-linux-x64-musl | byv |
| Linux ARM64 musl | bun-linux-arm64-musl | byv |
| macOS x64 | bun-darwin-x64 | byv |
| macOS ARM64 | bun-darwin-arm64 | byv |

A target is published only after a clean-machine smoke test. The table is a planned target matrix, not a claim that all targets are currently implemented.

Examples:

    bun build --compile --target=bun-windows-x64 ./apps/cli/src/index.ts --outfile ./dist/windows-x64/byv.exe
    bun build --compile --target=bun-linux-x64 ./apps/cli/src/index.ts --outfile ./dist/linux-x64/byv
    bun build --compile --target=bun-linux-arm64 ./apps/cli/src/index.ts --outfile ./dist/linux-arm64/byv
    bun build --compile --target=bun-darwin-arm64 ./apps/cli/src/index.ts --outfile ./dist/macos-arm64/byv

Cross-compilation requirements vary by host and target. CI should use dedicated runners or install the documented cross-compilation toolchain. Do not assume that every target can be built from every host.

## 5. Runtime files and external resources

Do not bake mutable or secret user data into the executable.

External at runtime:
- ~/.byv-agent configuration, sessions, memory, logs, cache, artifacts, checkpoints, plugins, and marketplaces;
- repository CLAUDE.md, AGENTS.md, .claude/, .mcp.json, .byv-agent/;
- provider API keys and secret references;
- Skill, plugin, agent, hook, and MCP content;
- project code and generated files.

Static resources may be embedded deliberately using Bun's supported asset options, but must be versioned and tested. Runtime discovery paths must remain predictable for both source and compiled execution.

The compiled app must resolve its own executable directory, user data directory, current working directory, and project root as distinct concepts. Never assume the current directory is the executable's directory.

## 6. External dependencies

A standalone byv binary does not bundle arbitrary operating-system tools.

Potential optional dependencies:
- Git;
- Docker or Podman;
- language servers;
- compilers and build tools;
- Playwright browsers;
- package managers invoked inside user repositories;
- MCP server executables.

The app must:
- detect dependencies with `byv doctor`;
- display versions and availability;
- explain which optional feature needs each dependency;
- avoid installing system tools without approval;
- report missing dependencies without crashing unrelated features.

## 7. Native dependency policy

- Prefer Bun-native APIs and Web-standard APIs.
- Use pure TypeScript/JavaScript packages when practical.
- Treat Node-API/native addons as target-specific dependencies.
- Verify every native dependency in source execution and compiled binaries on every claimed target.
- Validate dynamic imports, package assets, WASM files, subprocesses, and runtime file resolution.
- Do not claim a single cross-platform artifact; build per target.

## 8. Release pipeline

For every release:
1. Select and record the pinned Bun version.
2. Install dependencies from the committed lockfile.
3. Run formatting, lint, type checks, unit, integration, security, and compatibility tests.
4. Build the CLI for each declared target.
5. Run each binary's --version and --help smoke test.
6. Test config loading, SQLite initialization, Skill discovery, MCP config parsing, provider configuration, and project discovery.
7. Test external assets and subprocess behavior.
8. Generate SHA-256 checksums and build provenance.
9. Sign binaries where platform distribution requires or supports it.
10. Package artifacts with target names and installation instructions.
11. Publish release notes with Bun version, target list, checksums, known limitations, and compatibility baseline.

Windows releases should plan for Authenticode signing. macOS releases should plan for code signing and notarization when distributing outside local development. Linux releases should include archives and optional package formats after the raw binary is verified.

## 9. Android APK strategy

### 9.1 Important platform boundary

Bun's documented standalone executable targets cover Windows, Linux, and macOS. They do not include Android APK as a direct compile target. Android uses a different application packaging/runtime model, so renaming a Linux binary to .apk is not valid.

### 9.2 Recommended first Android product

Build an Android companion client:
- Native UI: Kotlin + Jetpack Compose (recommended) or another deliberately selected Android framework.
- API client: authenticated connection to the byv-agent local daemon or remote VPS/cloud runtime.
- Transport: HTTPS for remote use and a carefully secured local connection for same-device/network use.
- Features: chat, session list, task status, approvals, logs, diff preview, model selection, and notifications.
- Authentication: pairing flow and revocable device token; do not embed server API keys in the APK.
- Android build: Android Gradle Plugin/Gradle produces the APK/AAB and signs it with Android signing configuration.
- Bun may still be used for repository scripts, web assets, and development tooling, but it does not compile the APK itself.

### 9.3 Optional future on-device execution

If fully local coding-agent execution on Android is required, treat it as a separate feasibility track. Evaluate Android-supported process/runtime options, filesystem permissions, terminal environment, ABI, background execution limits, battery restrictions, and native dependencies. Do not promise Bun as the on-device Android runtime until an officially supported and tested target exists.

## 10. Installation UX

Unix-like systems:

    chmod +x ./byv
    ./byv
    ./byv --help

Windows:

    .\byv.exe
    .\byv.exe --help

An installer may place the binary in a user-selected PATH directory. Do not require administrator rights for a per-user installation.

## 11. Official references

- Bun compiler / standalone executable: https://bun.sh/docs/compiler
- Bun single-file executables and supported targets: https://bun.sh/docs/bundler/executables
- Bun AOT compiler status: https://bun.sh/docs/compiler/aot
- Bun installation and supported host platforms: https://bun.sh/docs/installation
- Android app build overview: https://developer.android.com/build
