# Contributing

Thanks for considering a contribution. This is a small, dependency-free collection of GLSL shaders,
so the process is lightweight — but a few things about how this repo is structured matter for getting
a change right.

## Before you start

- **Keep it single-pass.** Everything here targets programs that can't do multipass shader chains
  (ScummVM, DOSBox). A change that only works as a multipass chain doesn't fit this repo.
- **Match the existing aesthetic goal.** These shaders aim for a moderate, general-purpose CRT
  "softening" — not a faithful simulation of one specific monitor, and not full CRT-authenticity
  effects (heavy curvature, bloom, a strongly visible mask). If a change pushes noticeably toward
  either extreme, say so in your pull request so it can be discussed.
- **There's no build system or test suite.** Shader source is consumed directly by the target
  frontend, so there's nothing to compile or run locally beyond the shader itself.

## Making a change

1. If you're fixing something in the core CRT algorithm (the cubic resample, scanline beam profile,
   or the `mask_weights()` phosphor mask function), note that the same logic is duplicated verbatim
   across every `crt-hyllian*.glsl` file — there's no shared include in this shader pipeline. Apply
   your fix to **all** affected files, not just the one you opened.
2. If you're adding a new fixed-resolution variant (following the `_NNNp` pattern), add a matching
   root-level `.glslp` preset alongside it, and add it to the table in `README.md`.
3. Keep the original MIT permission-notice header intact at the top of any file you modify — see
   `LICENSE` for why.

## Testing your change

There's no headless way to validate a GLSL shader in this repo. Before opening a pull request, load
the `.glslp` (or the bare `.glsl`, depending on the frontend) in an actual target program — ScummVM,
DOSBox Staging, DOSBox-X, or RetroArch — and confirm it compiles and looks right. Mention in your
pull request which frontend(s) you tested against.

## Opening a pull request

Describe what changed and why, and which preset(s)/frontend(s) it affects. Small, focused pull
requests are easier to review than ones that bundle unrelated tweaks.
