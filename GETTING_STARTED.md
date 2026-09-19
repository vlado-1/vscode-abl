# Getting Started

This repository contains the source code for the **OpenEdge ABL** Visual Studio Code extension. The extension provides ABL language features, project configuration, compilation support, debugging integration, and tools for OpenEdge projects.

## Prerequisites

You need:

- Visual Studio Code 1.97 or newer.
- Node.js and pnpm.
- An OpenEdge installation with a developer license. The project is tested with OpenEdge 12.2, 12.8, and 13.0, although other versions may work.
- Java for the ABL language server and debugger. The extension can use a Java executable configured in VS Code, or a bundled JRE when one is included in a packaged build.

The extension does not include an OpenEdge runtime. Configure at least one runtime in VS Code settings:

```jsonc
"abl.configuration.defaultRuntime": "12.8",
"abl.configuration.runtimes": [
  {
    "name": "12.8",
    "path": "C:\\Progress\\OpenEdge-12.8"
  }
]
```

The runtime path must point to the OpenEdge installation directory, not its `bin` directory.

## Installing Dependencies

This workspace uses pnpm intentionally. Install dependencies with:

```powershell
pnpm install
```

The dependency `jsonc-parser` is required by [src/shared/openEdgeConfigFile.ts](src/shared/openEdgeConfigFile.ts) to read commented JSON project files.

Useful scripts from [package.json](package.json):

```powershell
pnpm run webpack       # Development bundle
pnpm run webpack-dev   # Development bundle in watch mode
pnpm run lint          # ESLint
pnpm run package       # Production bundle with hidden source maps
```

The generated extension bundle is written to `dist/extension.js`.

## Repository Layout

- [src/](src): TypeScript extension implementation.
- [src/extension.ts](src/extension.ts): VS Code extension entry point and activation logic.
- [src/shared/](src/shared): OpenEdge project configuration, file information, and ABL execution helpers.
- [src/tools/](src/tools): compile, syntax-check, listing, XREF, debug-listing, and preprocess integrations.
- [src/debugAdapter/](src/debugAdapter): ABL debugger integration.
- [resources/](resources): language configuration, schemas, helper ABL procedures, and packaging resources.
- [run_env/sample-oe-projects/](run_env/sample-oe-projects): small sample OpenEdge projects for local testing.
- [.vscode/launch.json](.vscode/launch.json): Extension Development Host launch configurations.
- [webpack.config.js](webpack.config.js): webpack configuration for the extension bundle.
- [Jenkinsfile](Jenkinsfile): CI dependency preparation, packaging, and publishing workflow.

## Running the Extension Locally

1. Open the repository root in VS Code.
2. Run `pnpm install`.
3. Configure an OpenEdge runtime as described above.
4. Open the Run and Debug view.
5. Select `Test Workspace` or `Sample project 1`.
6. Press F5.

VS Code runs the extension in a separate Extension Development Host window. The launch configuration runs the `build` task first, which invokes the `webpack` script.

The sample workspace is [run_env/sample-oe-projects/sample.code-workspace](run_env/sample-oe-projects/sample.code-workspace). It opens three sample projects. Each project has an `openedge-project.json` file at its root.

## OpenEdge Project Configuration

An OpenEdge project is normally identified by an `openedge-project.json` file in its root directory. The smallest useful configuration declares an OpenEdge version and at least one source directory:

```jsonc
{
  "name": "project1",
  "version": "1.0",
  "oeversion": "12.8",
  "charset": "utf-8",
  "graphicalMode": true,
  "buildPath": [
    { "type": "source", "path": "src" }
  ],
  "buildDirectory": "build",
  "dbConnections": [],
  "numThreads": 1
}
```

Important configuration concepts:

- `oeversion` must match a runtime configured in VS Code.
- `buildPath` identifies source directories that the extension monitors and compiles.
- `buildDirectory` is where generated build data is stored.
- `graphicalMode` selects `prowin` versus `_progres` behavior.
- `dbConnections` describes databases, schema files, and aliases used by the project.
- `profiles` can override settings such as the runtime, graphical mode, database connections, or build directory.

The extension starts its language client when activated, reads global runtime settings, discovers project configuration, and launches the Java-based ABL language server. OpenEdge sessions are then used for compilation, analysis, and execution.

## Generated and CI-Provided Files

Some files are intentionally ignored by Git and are normally created by CI or packaging steps:

- `dist/`: webpack output.
- `resources/abl-lsda.jar`: ABL language server and debug adapter.
- `resources/abl.tmLanguage.json`: TextMate grammar copied from the `abl-tmlanguage` package.
- `resources/grammar-version.txt`: generated grammar version information.
- `jre/`: optional bundled Java runtime used by packaged extensions.
- `*.vsix`: packaged extension files.

[Jenkinsfile](Jenkinsfile) downloads the language-server JAR and Java runtimes, copies the grammar, generates the grammar version, and packages platform-specific VSIX files. A source checkout can therefore build the JavaScript bundle successfully while still lacking the assets required for a fully functional Extension Development Host session.

## Troubleshooting

Check these first:

- Confirm that `openedge-project.json` exists at the project root and is valid JSONC.
- Confirm that `oeversion` matches a configured runtime name.
- Confirm that at least one `.p`, `.w`, or `.cls` file is open in a configured project.
- Open the Output view and inspect the `ABL Language Server` channel.
- Check the project's `.builder` directory for compiler and language-server logs.
- Make sure source directories are declared with `buildPath`; `propath` entries alone are not monitored for changes.
- Check that Java is available, or set `abl.langServerJavaExecutable` to an explicit Java executable.
- Remember that untrusted workspaces are not supported by the extension.

## Packaging

The production bundle is created with:

```powershell
pnpm run package
```

The VS Code packaging tool can then create a VSIX package. CI adds the language-server JAR, grammar, and platform-specific JRE before publishing packages to the Visual Studio Marketplace and Open VSX.

## Recommended Extensions

The repository recommends:

- Prettier
- ESLint
- SonarLint

These recommendations are declared in [.vscode/extensions.json](.vscode/extensions.json).

## Contribution Notes

Keep changes focused on the extension source and update the relevant configuration or documentation when behavior changes. Run the following before opening a pull request:

```powershell
pnpm run lint
pnpm run webpack
```

The extension is MIT-licensed. OpenEdge and ABL are trademarks of Progress Software Corporation, and the language-server implementation has separate Riverside Software copyright terms described in [README.md](README.md).
