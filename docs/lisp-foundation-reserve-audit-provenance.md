# Lisp Foundation Reserve Audit Provenance

## Upstream

- Repository: `borodust/alien-works`
- Default branch: `master`
- Audited upstream HEAD: `86381f8bce44a396347cbe8de70349be5b43ece1`
- Commit message: `Expose entity handling`
- Upstream commit date: 2023-05-01T23:15:19Z

## Working fork

- Repository: `vietcong73/alien-works`
- Default branch: `master`
- Audit branch: `audit/lisp-foundation-reserve`
- Fork HEAD at audit start: `86381f8bce44a396347cbe8de70349be5b43ece1`

At audit start, the working fork HEAD exactly matched the upstream HEAD above.

## License

- License: MIT
- Copyright: Copyright (c) 2020 Pavel Korolev
- Upstream `LICENSE` must remain preserved for any copied or substantially derived code.

## Audit purpose

This fork is a source specimen and audit workspace for a separate general-purpose Lisp foundation reserve.

Target reserve repository:

`vietcong73/lisp-foundation-works-reserved`

The reserve is intended for reusable Lisp-centric infrastructure across lab systems, automation, AI/agents, simulation, and autonomous/supervisory applications.

This audit does **not** assume that alien-works will become the final implementation. The final decision may be:

- streamline/adapt selected alien-works subsystems; or
- build a smaller clean reserve using only justified MIT-derived portions and public architectural patterns.

## Architectural comparison source

Naughty Dog / Uncharted Lisp-family material will be used as a **public design reference**, especially for bounded DSLs, state/event/director semantics, and code/data middle-layer discipline.

No proprietary Naughty Dog source code is to be copied.

## Initial invariants

1. Do not modify `master` for audit experiments.
2. Do not rewrite upstream history.
3. Preserve exact source provenance for reused code.
4. Separate exploratory REPL/runtime mutation from accepted source state.
5. Make streamline-vs-rebuild an evidence-based decision after source audit.
6. Keep game-specific subsystems optional unless they provide transferable infrastructure.
