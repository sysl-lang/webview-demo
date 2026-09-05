# webview-demo

A window, some HTML, and two functions the page can call — the shortest thing that proves
[`sh.sysl.webview`](https://github.com/sysl-lang/webview) works end to end.

```
brew install sysl-lang/tap/webview
sysl run main.sysl
```

A window opens with a counter and two buttons. **count** calls a sysl closure that increments a
variable and answers the new value; **quit** calls one that stops the loop. Closing the window, or
pressing quit, ends the program and prints how many clicks it saw.

## Why this repository exists

**The package's own test suite cannot open a window, and never will be able to in CI.**
`webview_create` calls `[NSApplication run]` and blocks until the application-did-finish-launching
notification, which never arrives in a session with no window server attached — `launchctl
managername` answers `Background` rather than `Aqua` there. A test that opened a window would **hang
rather than fail**, which is the worst way for a test to be wrong, so the suite deliberately stops at
the boundary: struct layout, error codes, and that the trampolines have addresses C can call.

Everything past `create` is proven here, by a person running it in a graphical session. That is:
the window, the HTML, `bind` delivering a JavaScript call to a sysl closure, and the closure's answer
resolving the page's promise.

**Run it from a terminal you logged into graphically** — Terminal.app or iTerm on macOS, a desktop
session on Linux. Over plain SSH, or from a background agent, it will hang at `create` for exactly
the reason above.

## What it exercises, line by line

| | |
|---|---|
| `webview(debug = true)` | opens the window, with WKWebView's inspector available |
| `title`, `size` | before `run`, as everything must be |
| `bind("count", …)` | a sysl **closure** capturing `clicks`, reachable from JavaScript |
| `bind("quit", …)` | a closure capturing the webview itself, calling `terminate` from inside a callback |
| `html` | a whole page from a string, with no server anywhere |
| `run` | blocks until the window closes |

The answer a bound function gives is **JSON** — a bare number here, which is why this needs no JSON
library at all. Anything richer would import `sh.sysl.json`.

## Licence

ISC.
