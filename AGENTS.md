# Repository instructions

## Web access

Use the Codex in-app browser by default. Use another browser only when the user
explicitly requests it or the task needs an existing authenticated session or a
capability unavailable in the in-app browser.

## Image update workflow

When updating an existing image family:

1. Confirm fixed upstream versions from primary security advisories or official
   release notes.
2. Update every currently supported branch in the repository. Do not add a new
   major branch unless the user asks for it.
3. Rename each versioned context directory, update its base-image tag, and keep
   the rest of the Dockerfile unchanged unless a build requires a compatibility
   fix.
4. Update all README examples, supported-tag lists, release loops, and relevant
   security notes. Remove superseded vulnerable contexts from the repository.
5. Run `git diff --check` before building.
6. Build with `--pull`, run the image's version command, and exercise its
   built-in extension/module checks before publishing.

For PHP, build and test every supported tag in WSL2. The Windows checkout is
available at `/mnt/d/dev/rdl/docker-images`. Follow the `wsl2-ssh-runtime` skill
and connect through `codex@localhost:2222` (fall back to `wsl2:2222` as directed
by that skill). Docker is available to the `codex` user for builds.

## Publishing

Published image names use the `superrosko` Docker Hub namespace and exact
version tags matching their context directories. For example:

```text
php/8.5.13-fpm -> superrosko/php:8.5.13-fpm
```

Docker Hub credentials are configured for the WSL2 user `rosko`, not `codex`.
Build as `codex`, but run Docker Hub pushes as `rosko`, for example:

```bash
sudo -u rosko docker push superrosko/php:8.5.13-fpm
```

Never store a sudo password, Docker Hub password, access token, or the contents
of `/home/rosko/.docker/config.json` in this repository or command text. Supply
sudo credentials only through an interactive hidden prompt when required.

If the user explicitly asks to push or publish PHP images, use
`superrosko/php` and the exact newly built tags without asking which Docker Hub
namespace or WSL user to use. Tool-required approvals and authentication prompts
still apply. Avoid PowerShell interpolation bugs: do not put Linux `$variables`
inside a double-quoted PowerShell SSH command; use a single-quoted remote command
or explicit push commands.

For Git releases, commit only the task's changes and push the current branch to
`origin`. On this Windows environment, if the default HTTPS push reports an
untrusted issuer, retain TLS verification and retry with:

```powershell
git -c http.sslBackend=schannel push origin <branch>
```

After publishing, report the Git commit, remote branch, every pushed image tag,
and its registry digest. Verify that the worktree is clean.
