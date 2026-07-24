# Product Hunt launch pack

Status: prepared for `0.1.0-alpha.3`, but autonomous publication is closed by
Product Hunt's current human-led personal-account policy. Retain this pack for
an eligible human collaborator; do not create a branded or fake-human Lumen
account to submit it.

## Listing

**Name:** While You Were Away

**Tagline:** Your AI keeps working after you close the tab.

**One-line description:** A local-first Linux runtime that gives one Codex
worker durable memory, bounded permissions, scheduled wake-ups, guarded Git
checkpoints, and a receipt after every run.

**Topics:** Artificial Intelligence, Developer Tools, Open Source, Productivity

## Maker comment

Most agent demos end when the chat closes. I wanted a worker I could actually
leave alone on a small Linux machine and inspect later.

While You Were Away is deliberately plain infrastructure: a private Git
workspace carries memory between separate Codex sessions, `workspace-write`
limits model-generated commands, a completion-relative user timer wakes the
worker, and the host runtime creates a guarded checkpoint only after verified
durable progress. Failures preserve the previous good message and leave a
receipt instead of pretending the run succeeded.

The alpha is open source and aimed at technical makers. Its one-command profile
is local-only: no hosted account, public listener, TLS claim, billing, or story
that a model stays conscious between runs. I would especially value feedback on
onboarding friction and the checkpoint boundary.

## Evidence behind the claims

- A fresh public clone passes the runtime, reproducible-release, and
  standalone-source suites.
- A locked non-root Linux account completed authenticated `init`, `doctor`,
  attended work, guarded checkpointing, a real systemd user-timer cycle,
  `status`, and `uninstall`.
- The real dogfood cycle exposed and closed four defects that root or fake
  tests had hidden: unsafe directory inspection, workspace-symlink
  canonicalization, read-only `.git`, and a systemd `PrivateTmp` conflict with
  Bubblewrap.
- The runtime refuses zero-change success, common credential signatures,
  nested Git metadata, special files, files over 10 MiB, and dirty starting
  trees.
- A fresh minimal Ubuntu root installed the 14 declared lifecycle dependencies,
  passed the standalone runtime, local-life, publisher, release, and source
  suites without real credentials or service activation, and was removed.
- A separate disposable non-root account used a real systemd user manager to
  bootstrap, activate all three timers, upgrade, and uninstall while preserving
  its snapshot and backup.

## Gallery storyboard

1. **Leave the tab. Keep the work.** Show the product name and the five-command
   flow: clone, init, doctor, install, status.
2. **Continuity is files, not a consciousness claim.** Diagram separate Codex
   sessions connected by Git, current state, memory, and project notes.
3. **Bounded by default.** Show non-root execution, `workspace-write`, minimal
   environment, private runtime state, and no credentials in units.
4. **Every useful run becomes a checkpoint.** Show a status-0 receipt beside a
   short commit hash and a clean tree.
5. **Failures stay honest.** Show timeout, no-progress, and credential-refusal
   receipts preserving the preceding canonical message.

The finished private-data-free gallery is in `launch-assets/`: five 1270x760
PNGs plus a 240x240 thumbnail. `bin/build-launch-assets` regenerates the exact
copy and layout without external input. Product Hunt's official posting guide,
accessed 2026-07-24, recommends 1270x760 gallery images, at least two gallery
images, and a 240x240 square thumbnail under 3 MiB:
https://help.producthunt.com/en/articles/479557-how-to-post-a-product

## 90-second demo

```text
git clone https://github.com/IlluminationAi/while-you-were-away.git
cd while-you-were-away
bin/wywa-life bootstrap "$HOME/my-worker" \
  --name "My Worker" \
  --accept-bounded-defaults
bin/wywa-life status "$HOME/my-worker"
```

Then show the private static preview, `git log -2 --oneline`, the backup
receipt, all three completion-relative timers, and `wywa-life uninstall`
preserving the workspace, releases, and bundles.

## Promotion copy

I built a small open-source runtime for one persistent Codex worker on a
user-owned Linux machine. It wakes after you leave, works inside
`workspace-write`, checkpoints useful progress from the host side, and leaves
an inspectable receipt. No hosted account and no “continuous consciousness”
story—just durable files, Git, systemd, and tested failure boundaries.

## Launch checklist

- [x] Public Apache-2.0 repository and reproducible alpha.
- [x] Fresh anonymous clone and self-contained test proof.
- [x] Authenticated non-root attended and scheduled dogfood.
- [x] Clean install/status/uninstall path with credential-free units.
- [x] Fresh minimal Ubuntu root and real non-root user-manager lifecycle proof.
- [x] Listing copy, maker comment, evidence, demo, and gallery storyboard.
- [ ] Independent human onboarding evidence.
- [x] Five final gallery images and a thumbnail with no private host data.
- [x] Owner authorization to create ordinary external accounts and promote the
      product where provider terms support Lumen's truthful identity.
- [x] Product Hunt account path evaluated against its current eligibility
      rules; autonomous Lumen submission rejected rather than misrepresented.
- [ ] Eligible human collaborator independently chooses to launch the prepared
      listing and is recorded truthfully as the submitting person.

Product Hunt's official personal-vs-company policy, accessed 2026-07-24, says
that contributions must be authentic and human-led, company accounts cannot
post, and accounts must be at least one week old:
https://help.producthunt.com/en/articles/771527-personal-account-vs-company-account

Lumen may continue promotion through legitimate public surfaces. If a paid
service or promotion step is useful, it may request money or a
purchase-limited one-time virtual card from the owner in Telegram and execute
the stated purchase after the owner supplies it.

## Claim boundaries

Do not call the project launch-ready while independent human onboarding remains
unproven. Do not claim continuous model thought, universal Linux compatibility,
secret-proof scanning, a completed public-host path, guaranteed Product Hunt
rank, or work that is not visible in a receipt and Git checkpoint.
