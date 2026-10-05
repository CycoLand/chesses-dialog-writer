# Chesses Dialog Writer

A browser-based tool for writing and revising campaign character dialog for
the [Chesses](https://github.com/CycoLand/chesses-on-android) game.

**Open the tool:** https://cycoland.github.io/chesses-dialog-writer/

This repo contains only the static tool itself (`index.html`). It has no
game code. It reads and writes character dialog directly in the
`chess-variants-app/dialog-data/` folder of the
[`chesses-on-android`](https://github.com/CycoLand/chesses-on-android) repo,
over the GitHub REST API, using a personal access token you provide.

Hosted here (a separate public repo) rather than in the game repo itself
because the game repo is private, and GitHub Pages requires a public repo
on the Free plan.

The canonical source for `index.html` is
[`chess-variants-app/tools/dialog-writer/dialog-writer.html`](https://github.com/CycoLand/chesses-on-android/blob/main/chess-variants-app/tools/dialog-writer/dialog-writer.html)
in the game repo — if you need to change the tool itself, edit it there
and copy the result here (or ask whoever maintains the pipeline to do it).

## How to use it

1. Open the link above.
2. Get a GitHub personal access token with `repo` scope (there's a link in
   the sidebar that pre-fills the right scope for you) and someone needs
   to have given you write access to `CycoLand/chesses-on-android` first.
3. Enter your name and the token, hit Connect.
4. Pick an unclaimed character (white dot), hit Claim, write, Save as often
   as you like, then Mark Rewritten when done.

See the full writing guidelines and workflow docs in the game repo's
`chess-variants-app/docs/13-dialog-guidelines.md` and
`chess-variants-app/tools/dialog-writer/README.md`.
