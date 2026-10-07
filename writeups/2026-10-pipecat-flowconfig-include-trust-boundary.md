# A Path Traversal That Wasn't: Trust Boundaries in Pipecat's `!include`

**Target:** [Pipecat](https://github.com/pipecat-ai/pipecat) — Daily's open-source voice-AI framework (16k+ stars)
**Class:** Path traversal / arbitrary file read (CWE-22) via a YAML `!include` tag
**Outcome:** Classified by maintainers as *intended behavior* (an application-trust boundary), not a Pipecat vulnerability. The docstrings were clarified to document the boundary ([PR #6034](https://github.com/pipecat-ai/pipecat/pull/6034)) and I was credited for raising it.
**Date:** October 2026

---

## TL;DR
I noticed that Pipecat's flow-config loader resolves `!include` paths with no directory confinement — `../` and absolute paths escape the config's folder and read any file the process can. It *looks* like arbitrary file read. After a careful review (mine and the maintainers'), the correct conclusion was that this sits on the **application-trust boundary**: configs loaded from disk are trusted input, like code, and untrusted input already has a safe loader that rejects `!include`. The real lesson: **a dangerous primitive is only a vulnerability when untrusted input can provably reach it.**

## The finding
Pipecat flow configs are YAML and support an `!include` tag. The constructor resolved the included path like this:

```python
def _include(loader, node):
    include_path = base_dir / str(loader.construct_scalar(node))  # no '..' / absolute check
    with include_path.open(encoding="utf-8") as f:
        return yaml.load(f, loader_class)
```

`base_dir / "../../x"` escapes `base_dir`, and `base_dir / "/etc/passwd"` overrides it entirely. Because an included file's parsed content is merged into the flow config — which becomes agent prompt/message text — a successful include can surface in the conversation. In isolation, that is a textbook arbitrary-file-read primitive, and a self-contained PoC reproduced it against Pipecat's exact loader.

## Why it looked exploitable and why it isn't (the trust boundary)
The impact depends entirely on *who controls the config*. I reported it on the hypothesis that an app might load an attacker-controlled flow via `FlowConfig.from_file()` or `from_yaml(text, base_dir=...)`. The maintainers reproduced the behavior and then made the case that this is intended:

1. **Flow configs from disk are trusted input, like application code.** `from_file` and `from_yaml(base_dir=…)` are meant for flows the *developer* wrote, who may legitimately keep shared prompts outside the flow's directory — which is why includes aren't confined.
2. **Untrusted input already has a safe path.** A config from a user/request should be loaded with `from_yaml(text)` **without** `base_dir`, which rejects `!include` entirely (there's a test for it), or with `from_json`.
3. **No reachable sink.** Pipecat itself exposes no endpoint that loads flow configs from untrusted input.

I went back through the codebase to check their specific question — can untrusted input reach `!include` *without* the app opting into the trusted loaders? — and I couldn't find a path. Every `from_file`/`from_yaml(base_dir=…)` call site is developer-controlled; the examples load a fixed path. So the honest conclusion matched theirs: this is the library-vs-application trust boundary, not a Pipecat vulnerability.

## Outcome
The maintainers' docstrings had described configs as "data with no callables," which reasonably reads as *safe to accept from users*. [PR #6034](https://github.com/pipecat-ai/pipecat/pull/6034) documents the trust boundary across `FlowConfig`, `from_file`, `from_yaml`, and the include loader, and tells developers how to load untrusted configs safely. I was credited for prompting the change.

## The lesson
This is the distinction that matters in real security work:

- A **bug class** (path traversal, SSRF, reentrancy) is a *root cause*, not an *impact*.
- A primitive becomes a **vulnerability** only when untrusted input can **reach** it across a boundary the project is responsible for defending.
- Libraries draw a trust boundary at "developer-supplied code/config." Reporting a primitive without a reachable untrusted path is reporting a *root cause with no impact*.

A gracefully-accepted "not a vulnerability" that improves the docs is still a good outcome — and knowing the difference is more valuable than the finding itself.

## References
- Pipecat docs PR (fix & credit): https://github.com/pipecat-ai/pipecat/pull/6034
- CWE-22: Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal')
