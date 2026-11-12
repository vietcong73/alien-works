# 04. Chess Engine & AI Bot: Gerbil Scheme Source Code Review

**Project:** [mark-watson/gerbil_scheme_book — chess-game](https://github.com/mark-watson/gerbil_scheme_book/tree/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game)  
**Reviewed commit:** [`1335cb2708926213ba62afd2998754358f655db1`](https://github.com/mark-watson/gerbil_scheme_book/commit/1335cb2708926213ba62afd2998754358f655db1), November 11, 2026  
**Scope:** Project's actual Gerbil Scheme source: `engine.ss`, `ai.ss`, `cli.ss`, and `perft.ss`, plus its Makefile and project README for context.  
**Method:** Direct static inspection of function bodies and their call paths. No `gxi`/`gxc` runtime was available in the review environment; no successful Gerbil compilation or perft execution is claimed.  
**Disposition:** `REAL_GERBIL_APPLICATION / SEARCH_CORRECTNESS_DEFECTS_IDENTIFIED / MOVE_GENERATION_TESTS_INCOMPLETE / RUNTIME_VALIDATION_PENDING`

## Implementation map

- [`engine.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/engine.ss): 64-square vector board, piece lists and king-square caches, FEN input/output, attack detection, pseudo-legal and legal moves, castling, promotions, en passant, make/unmake, and incremental Zobrist hash.
- [`ai.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/ai.ss): material and piece-square-table evaluation, endgame detection, move ordering, transposition table (exact/lower/upper bounds), depth-limited negamax/alpha-beta, capture-based quiescence, iterative deepening.
- [`cli.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/cli.ss): terminal board renderer, UCI move input, game loop, player/bot modes.
- [`perft.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/perft.ss): recursive move-count check at opening depths 1–3 and incremental-hash validation.

## Strengths

1. **Actual Gerbil implementation:** the chess engine and AI are not wrappers around Python or a third-party chess binary. Source is compact, and the core domain objects are `defstruct` records.
2. **Cached geometry and piece lists:** knight/king moves and sliding-piece rays are precomputed, while each board stores its active piece squares and both king positions.
3. **Undo-first search mechanics:** search and legal filtering apply moves to a mutable board and restore them, avoiding full 64-cell copies at each node.
4. **Incremental deterministic Zobrist hash:** captures, promotions, castling and en passant update the active hash via xor; recomputation is exposed for verification.
5. **Standard search components:** alpha-beta negamax, iterative deepening, move ordering and a depth/bound transposition table are all present in actual Gerbil functions.
6. **Testability surface:** UCI moves/FEN and a perft script make differential testing against a known chess reference feasible.

## Source-grounded findings

### CH-F01: Quiescence search permits stand-pat while in check

**Source:** [`ai.ss:263–296`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/ai.ss#L263-L296). `quiescence-search` immediately evaluates a stand-pat score and only searches captures/promotions. A side-to-move in check cannot legally stand pat; legal quiet evasions may be mandatory. If the full-width search reaches depth zero in check, the routine can return a score without resolving check. This is a **source-confirmed search correctness defect** for reachable in-check leaf nodes. The standard remedy is to generate all legal evasions when in check, with no stand-pat cutoff in that branch.

### CH-F02: Castling can manufacture a rook from an inconsistent FEN

**Source:** [`engine.ss:304–365`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/engine.ss#L304-L365), [`591–625`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/engine.ss#L591-L625), and [`696–710`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/engine.ss#L696-L710). The FEN parser accepts castling rights directly. Castling generation checks king, empty transit squares and attacked squares, but does not confirm the rook exists on its home square. `board-make-move!` then unconditionally writes a rook to the destination. Concrete FEN boundary example: `4k3/8/8/8/8/8/8/4K3 w K - 0 1`. This is an **invalid-position/FEN-validation defect**, not evidence that legal play from a valid starting position creates pieces spontaneously.

### CH-F03: Perft counts do not fail the command on mismatch

**Source:** [`perft.ss:29–46`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/perft.ss#L29-L46). Expected values `20`, `400`, `8902` are compared and a textual `PASS` or `FAIL` is printed. The mismatch branch does not raise an error or return a failing process status. Thus `make test` may appear successful to CI despite a node-count mismatch. This is a **test-harness enforcement defect**, not proof the expected opening counts are wrong.

### CH-F04: Perft's undo-hash check is tautological

**Source:** [`perft.ss:13–25`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/perft.ss#L13-L25) and [`engine.ss:759–763`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/engine.ss#L759-L763). `board-unmake-move!` assigns `board-zobrist-hash` directly from the saved snapshot; the test then merely compares that field against the same snapshot. It does **not independently recompute the hash after undo**, nor compare the restored position with a pre-move FEN. The forward-move hash check is meaningful; this particular post-undo check is not a sufficient state-restoration test.

### CH-F05: Unsupported UCI coordinates can reach numeric comparison with `#f`

**Source:** [`cli.ss:72–104`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/cli.ss#L72-L104). `parse-move` validates string length but does not check whether both coordinate lookups succeeded before comparing integer move fields with `from` and `to`. Input such as `z9z9` meets the length constraint but maps to `#f`, potentially raising a numeric-type exception instead of returning an invalid-move result. **Static invalid-input path**; no Gerbil execution was performed.

### CH-F06: Piece-square tables reward opening pawns for staying on home ranks

**Source:** [`ai.ss:27–36`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/ai.ss#L27-L36), [`120–159`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/ai.ss#L120-L159), and [`engine.ss:57–68`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/engine.ss#L57-L68). The board indexes `a1=0`, whereas the stored pawn PST gives all second-rank squares +50. As a result, the PST term alone decreases when White plays `e2e4` (+50 at e2 → +25 at e4), discouraging central pawn development rather than encouraging it. This is a **demonstrable evaluation-design weakness**, not a rules violation or standalone proof of suboptimal overall play. The `move-value` ordering routine also indexes the PST without mirroring by color.

### CH-F07: Draw-rule coverage is incomplete

**Source:** [`cli.ss:133–143`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/cli.ss#L133-L143), [`ai.ss:198–202`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/ai.ss#L198-L202), [`engine.ss:215–225`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/engine.ss#L215-L225). The CLI handles checkmate/stalemate and a 50-move clock condition, but no threefold repetition history or insufficient-material checks are evident. This is a **feature/correctness coverage limitation**, not proof of a crash.

### CH-F08: Reference perft coverage is narrow

**Source:** [`perft.ss:29–46`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/chess-game/perft.ss#L29-L46). Only the initial position at depths 1–3 is checked (20, 400, 8902). No dedicated vectors target castling, en passant, promotion, pins, complex check evasions or undo-state equivalence. The README's broad legality claims therefore exceed what this particular test harness alone qualifies.

## Verdict and qualification

**Positive:** a real, compact Gerbil game engine with meaningful move generation, cached geometry, incremental hashing and AI search. These are technically instructive implementations, and the project is substantially more than an example API call. The separation between `engine.ss`, `ai.ss`, `cli.ss` and `perft.ss` is clean and useful.

**Negative:** search correctness around checking positions, permissive FEN/castling handling, weak test enforcement, narrow perft coverage, questionable positional evaluation incentives, and limited chess draw rules. These shortcomings do not invalidate the engine's instructional value, but they prevent treating the 'complete, high-performance' README wording as an independently qualified conclusion.

**Suggested regression gate:** perft on canonical special-move positions; make/unmake FEN + recomputed hash equality; legal in-check quiescence with quiet evasions; invalid UCI coordinates; inconsistent FEN castling rights; draw-rule handling; and deterministic search comparisons to a reference engine. No user or upstream repository was modified.