# Patched Bun Binary (Temporary)

This is a patched build of Bun v1.3.11 that exposes `reactFastRefresh` on `Bun.Transpiler` — needed for AbsoluteJS's unbundled ESM HMR.

**PR**: https://github.com/oven-sh/bun/pull/28312

## Current Status

- **Now**: Use this binary
- **When PR CI is approved**: Switch to `bunx bun.pr 28312`
- **When PR is merged**: Update Bun to the release that includes it

## Setup (Linux x64 only)

```bash
# Download the binary
mkdir -p ~/.local/bin
curl -L https://github.com/alexkahndev/bun-react-refresh-patch/raw/main/bun-patched -o ~/.local/bin/bun-patched
chmod +x ~/.local/bin/bun-patched

# Create a symlink for easy switching
mkdir -p /tmp/patched-bun
ln -sf ~/.local/bin/bun-patched /tmp/patched-bun/bun
```

## Usage with AbsoluteJS

```bash
# Option 1: Source the helper script (updates PATH for current shell)
source scripts/use-patched-bun.sh
bun run dev

# Option 2: Prefix any command
./scripts/use-patched-bun.sh bun run dev

# Option 3: Manual PATH override
PATH="/tmp/patched-bun:$PATH" bun run dev
```

## What the patch does

Adds `reactFastRefresh: boolean` option to `new Bun.Transpiler()`. This enables per-file React Fast Refresh injection (~0.1ms) without going through `Bun.build()` (~2-150ms). AbsoluteJS uses this for its unbundled ESM module server, achieving 10-20ms HMR.

The change is 3 lines across 2 files in Bun's source:
- `src/bun.js/api/JSTranspiler.zig`: parse the option, pass to parser
- `src/transpiler.zig`: propagate to parser features
- `packages/bun-types/bun.d.ts`: TypeScript type definition
