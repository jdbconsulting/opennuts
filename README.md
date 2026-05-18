# opennuts

The OpenNuts platform: a precision-first geometric modeling kernel, the
IDE that drives it, and the cloud surface they share. Each piece is
self-contained and built independently; this top-level folder is just
the umbrella superproject and pins one git submodule per piece.

## Subprojects

| folder                                                | role                                                                                                       |
| ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| [`opennuts-gmk/`](opennuts-gmk/README.md)             | The C++23 Geometric Modeling Kernel (GMK), CLI, LSP & geometry servers, WASM bindings.                     |
| [`opennuts-app/`](opennuts-app/README.md)             | The OpenNuts IDE: browser-only Eclipse Theia + Monaco + WebGL viewer + Auth0 + hybrid local/cloud files.   |
| [`opennuts-server/`](opennuts-server/README.md)       | AWS Lambda / S3 backend. Auth0-gated project storage and async job orchestration.                          |
| [`opennuts-schemlang/`](opennuts-schemlang/README.md) | Upstream SchemLang ECAD language; the OpenNuts MCAD language is an extension of it.                        |

## Cloning

Each subproject is a git submodule pinned to a specific commit.
Clone with `--recurse-submodules` to fetch them all in one step:

```bash
git clone --recurse-submodules git@github.com:jdbconsulting/opennuts.git
```

If you've already cloned without that flag, populate the submodules
in place:

```bash
git submodule update --init --recursive
```

To pull the latest tip of every submodule (instead of the pinned
commit) and stage the new pin in the superproject:

```bash
git submodule update --remote --merge
git add opennuts-*
git commit -m "bump submodules"
```

## High-level architecture

```
                        +-------------------------------------------+
                        |              opennuts-app/                |
                        |  Eclipse Theia, browser-only target.      |
                        |  Monaco + WebGL2 + WASM kernel.           |
                        |                                           |
                        |  Sources of files (any/all at once):      |
                        |    1. OPFS (Theia's built-in BrowserFS)   |
                        |    2. Local disk  (File System Access API)|
                        |    3. opennuts-cloud:// (Lambda + S3)     |
                        +------+--------------------------+---------+
                               |                          |
              WASM (in-browser)|                          | HTTPS + Bearer JWT
                               v                          v
                  +-------------------------+   +-----------------------+
                  |      opennuts-gmk/      |   |    opennuts-server/   |
                  |  GMK kernel (B-rep,     |   |  Lambda + S3 + DDB +  |
                  |  NURBS, tessellation)   |   |  SQS; Auth0-gated.    |
                  +-------------------------+   +-----------------------+
```

The kernel runs entirely in the browser: the IDE serves
`opennuts-gmk`'s WASM as a static asset and drives it directly. The
cloud is purely additive — sign in to mount private project folders;
without signing in, the user works against their local disk via the
File System Access API or against the in-browser OPFS workspace.

## Quick start

End-to-end "build the kernel, deploy the server, run the IDE":

```bash
# 1. Build the kernel WASM
source ~/emsdk/emsdk_env.sh
emcmake cmake -S opennuts-gmk -B opennuts-gmk/build-wasm
cmake --build opennuts-gmk/build-wasm

# 2. (Optional) Deploy opennuts-server — only needed for the cloud-mount feature
cd opennuts-server
sam build && sam deploy --guided        # supply Auth0Domain + Auth0Audience

# 3. Build and run the IDE
cd ../opennuts-app
npm install
npm run build:browser
npm run start:browser                   # http://localhost:3000
```

See each subproject's README for details:

  - [`opennuts-gmk/README.md`](opennuts-gmk/README.md) -- kernel
    build, CLI commands, architecture notes, language grammar.
  - [`opennuts-app/README.md`](opennuts-app/README.md) -- browser-only
    Theia, Auth0 status item, local + cloud file mounts.
  - [`opennuts-server/README.md`](opennuts-server/README.md) -- SAM
    stack: Auth0 + Lambda + S3 + DynamoDB + SQS; deploy + API contract.

## License

Internal project; copyright the OpenNuts authors.
