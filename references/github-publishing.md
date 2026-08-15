# GitHub Publishing

DSH does not define a special GitHub Release schema. Use a normal public package repository, SemVer tags, and copy-ready DSH install commands.

## Repository Minimum

Include:

- plugin source or prebuilt entry;
- `package.json` with `dsh.bundle`;
- `cordis.patch.yml`;
- README with purpose, permissions/config, install, verify, remove, and pinned install commands;
- license;
- build files only when the package actually builds.

Add these GitHub topics:

- `dsh-plugin` (official discovery topic);
- `deepseek-harness`;
- `cordis` when applicable.

Do not publish `.env`, API keys, local absolute paths, caches, profile state, or unrelated generated files.

## Create and Push

Inspect before committing:

```sh
git status --short
git diff --check
git diff
git log --oneline -10
```

For a new repository:

```sh
git init -b main
git add <intended-files>
git commit -m "feat: add DeepSeek Harness plugin"
gh repo create OWNER/REPO --public --source . --remote origin --push \
  --description "DESCRIPTION"
gh repo edit OWNER/REPO \
  --add-topic dsh-plugin \
  --add-topic deepseek-harness \
  --add-topic cordis
```

If a remote already exists, inspect it and its tracking branch before pushing. Never force-push unless the user explicitly requests it.

## README Install Commands

Latest source:

```sh
dsh plugin --profile web add github:OWNER/REPO
```

Pinned source:

```sh
dsh plugin --profile web add github:OWNER/REPO#COMMIT_SHA
```

Removal:

```sh
dsh plugin --profile web remove PACKAGE_NAME
```

If the Git package uses `prepare`, add a visible note that the user must authorize the exact dependency key in the profile's `pnpm-workspace.yaml` `allowBuilds` before retrying installation.

## SemVer Release

Use `vMAJOR.MINOR.PATCH`. Start at `v0.1.0` for an experimental plugin unless the repository already has a version history.

Before tagging, ensure `package.json` and documented version agree, the intended commit is pushed, and the tested install command is in README.

```sh
git tag -a v0.1.0 -m "v0.1.0"
git push origin v0.1.0
gh release create v0.1.0 \
  --title "v0.1.0" \
  --notes-file RELEASE_NOTES.md
```

Use this release-note format:

````md
## Highlights

- What capability was added.

## Install

```sh
dsh plugin --profile web add github:OWNER/REPO#COMMIT_SHA
```

## Verify

```sh
dsh --profile web --dump-config
```

## Compatibility

- DeepSeek Harness version or commit tested.
- Node.js/package-manager requirements, if any.

## Security

- Permissions, network/filesystem access, and install-script behavior.
````

Creating a GitHub Release is useful for discoverability and changelogs but is not required by DSH installation.

## `tjsdyy/dshplugin` Marketplace

Reference: <https://github.com/tjsdyy/dshplugin#development>

That repository is a marketplace/catalog, not a plugin source template. Its current submission flow asks for a plugin submission issue containing:

```md
Repository URL: https://github.com/OWNER/REPO
Category: CATEGORY
English description: ONE_SENTENCE_ENGLISH_DESCRIPTION
Chinese description: 一句话中文描述
Tested install command: dsh plugin --profile web add github:OWNER/REPO
```

The catalog sync expects a known category, bilingual descriptions, a non-empty install command, and no duplicate owner/name. Verify the current README before submission because categories and validation may change.

Prepare the issue body, then publish only when the user asks:

```sh
gh issue create --repo tjsdyy/dshplugin \
  --title "Plugin submission: PACKAGE_NAME" \
  --body-file marketplace-submission.md
```

Do not claim marketplace acceptance; report only the issue URL and its current state.
