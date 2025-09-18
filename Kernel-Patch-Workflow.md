## Linux kernel patch workflow: from change to v2

This guide shows an end-to-end workflow for contributing a change to the Linux kernel: making the change, checking and formatting the patch, identifying maintainers, sending the email, handling feedback, and sending v2 with a cover letter.

## Prerequisites and setup

- **Tools**: git, perl, email client or SMTP, and optionally `b4`.
- **Kernel scripts**: used from the kernel tree: `scripts/checkpatch.pl`, `scripts/get_maintainer.pl`.
- **Git identity**:
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```
- **Signed-off-by on commit**: use `git commit -s` (Developer Certificate of Origin).
- **Plaintext email**: do not send HTML. Use CLI senders like `git send-email` or `b4`.
- **Optional: git send-email config**:
```bash
git config --global sendemail.smtpServer smtp.example.com
git config --global sendemail.smtpEncryption tls
git config --global sendemail.smtpUser you@example.com
git config --global sendemail.from "Your Name <you@example.com>"
git config --global sendemail.to ""
git config --global sendemail.cc ""
```
- **Optional: install b4**:
```bash
pipx install b4
# or: pip install --user b4
```

## 1) Create a topic branch and implement the change

```bash
git checkout -b topic/subsystem-short-desc
# ... edit files ...
git add -p
git commit -s
```

Follow kernel style and keep commits focused and logically separated.

## 2) Build and basic checks

- **Configure and build (example x86_64)**:
```bash
make olddefconfig
make -j"$(nproc)"
```
- **Stricter compile warnings**:
```bash
make W=1 -j"$(nproc)"
```
- **Sparse (optional)**:
```bash
make C=1 -j"$(nproc)"
```

Resolve warnings that are clearly related to your change.

## 3) Write a good commit message

- **Subject**: `subsystem/path: concise summary`
- **Body**: What/Why first, then How. Mention user-visible effect, performance, correctness, and testing performed. Wrap at ~72 characters.
- **Tags** when appropriate:
  - `Fixes: <sha1> ("offending subject")`
  - `Link: <lore or bug link>`
  - `Reported-by:`, `Tested-by:`, `Reviewed-by:`, `Acked-by:`
- Always end with `Signed-off-by: Your Name <you@example.com>` (added via `-s`).

Example:
```text
subsystem/path: short imperative summary

Explain the problem, why the change is needed, and how it fixes it.
Describe impact and testing.

Fixes: 123456789abc ("old subject")
Link: https://lore.kernel.org/r/<message-id>
Signed-off-by: Your Name <you@example.com>
```

## 4) Run checkpatch on your patch

Generate a patch and run the style checker on the patch (not on the tree diff):
```bash
git format-patch -1 --stdout | ./scripts/checkpatch.pl --strict --color=always -
```

Alternatively, save and check a file:
```bash
git format-patch -1 -o out/
./scripts/checkpatch.pl --strict --color=always out/0001-*.patch
```

Address real issues; some warnings are context-dependent. If you must keep a warning, be ready to justify it in review.

## 5) Find the right maintainers and mailing lists

Use `get_maintainer.pl` on the patch file to derive To/Cc lists:
```bash
./scripts/get_maintainer.pl --nogit-fallback --no-rolestats out/0001-*.patch
```

Typical outputs include maintainers, subsystem lists (e.g., `linux-kernel@vger.kernel.org`), and relevant reviewers. Send to all relevant lists/people.

## 6) Prepare the patch or series (with optional cover letter)

- **Single patch**:
```bash
git format-patch -1 -o out/
```
- **Series with cover letter**:
```bash
git format-patch <base>..HEAD -o out/ --cover-letter
# Edit out/0000-cover-letter.patch: explain overall context and testing
```

Cover letter content (for series): problem statement, approach, high-level changes, testing, and a short diffstat summary (left by git). For v2+, include a “Changes since v1” section.

## 7) Send patches

### Option A: git send-email

Get To/Cc and send:
```bash
to=$(./scripts/get_maintainer.pl --nogit-fallback --no-rolestats --to out/0001-*.patch)
cc=$(./scripts/get_maintainer.pl --nogit-fallback --no-rolestats --cc out/0001-*.patch)
git send-email --annotate \
  --to "$to" --cc "$cc" out/000*.patch
```

Notes:
- `--annotate` lets you review email bodies before sending.
- Ensure your email is plain text.

### Option B: b4 send (recommended by many maintainers)

```bash
b4 prep -o out
b4 send -o out --annotate
```

`b4` will help gather recipients and thread properly. It can also fetch series from lore and apply with `b4 am`.

## 8) Aftermath: tracking and responding

- Your patches will appear on lore; keep the thread link for future reference.
- Respond to review comments inline (bottom-post, quote context).
- Add appropriate tags when reviewers provide them:
  - `Reviewed-by:`, `Acked-by:`, `Tested-by:` (verbatim, with correct name/email).
- If asked to make changes, prepare v2.

## 9) Sending v2 (or later) revisions

- Amend commits and re-run checks, rebuild, and tests.
- Regenerate patches with versioning and update cover letter:
```bash
git format-patch -v2 <base>..HEAD -o out/ --cover-letter
```
- In the cover letter, add a section:
```text
Changes since v1:
- Clarify commit message rationale
- Fix style warnings from checkpatch
- Address reviewer feedback on foo/bar
```
- Keep the same subject(s), `PATCH v2` will be added automatically.
- Include a `Link:` tag in the commit message to the v1 thread on lore for traceability.

Send v2 to the same To/Cc (refresh with `get_maintainer.pl` on the new patch file), preserving maintainers and lists, and adding anyone who commented.

## 10) Tips and common patterns

- **Subsystem prefix**: match existing conventions, e.g., `drm/amd/display: ...`, `iio: accel: ...`.
- **One change per patch**: split orthogonal changes.
- **Test matrix**: build with `W=1`, run relevant selftests, boot/smoke test if feasible.
- **Stable**: If backport-worthy, add `Cc: stable@vger.kernel.org` with justification in the commit message.
- **Email hygiene**: no HTML, no attachments for patches; inline text only. Keep lines < 998 chars.

## 11) Minimal command cheat sheet

```bash
# Branch, edit, commit with SoB
git checkout -b topic/foo
git commit -s -m "subsystem: area: short summary"

# Build and warn checks
make olddefconfig && make W=1 -j"$(nproc)"

# Format and check patch
git format-patch -1 -o out/
./scripts/checkpatch.pl --strict out/0001-*.patch

# Get maintainers
./scripts/get_maintainer.pl --nogit-fallback out/0001-*.patch

# Send (git send-email)
git send-email --annotate --to "..." --cc "..." out/0001-*.patch

# Series with cover-letter and v2
git format-patch -v2 <base>..HEAD -o out/ --cover-letter
```

## 12) Simple cover letter template

```text
Subject: [PATCH v2 0/N] subsystem: short series summary

Hi all,

This series <explain problem/goal>. It <high-level approach>.

Testing: <how tested>

Changes since v1:
- <bulleted list of changes>

Thanks,
Your Name
```

This process should take you from a local change to a properly threaded, reviewed patch on the mailing lists, and then to v2 if needed.


