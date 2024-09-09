---
title: "Phoenix Live View Render Children"
date: 2024-09-06T17:23:48-04:00
draft: false
toc: false
images:
tags: 
  - elixir, phoenix, liveview, testing
---

A few weeks ago, I introduced a regression into our application that should have been caught by the test suite. This bug was in a LiveView that rendered an embedded LiveView. The embeded LiveView contains a Javascript client hook on mount that updates the sockets assigns and finally the DOM based on the results of a `handle_call/3` callback. That's a difficult few sentences to follow, but the following diagram encapsulates my explanation nicely:

```
Server
+----------------------------+                         
|      Parent LiveView       |                         
|                            |     Client                    
|                            |     +------------------+
|   +---------------+        |     |                  |
|   |  Embedded LV  |        |     |   JS Mount Hook  |
|   |               |        |     |                  |
|   |               |        |     |                  |
|   |               +--------+---->|                  |
|   |               |        |     |                  |
|   |     "do-work" |<-------+-----+ handle_call/3    |
|   +---------------+        |     |                  |
|                            |     +------------------+
|                            |                         
+----------------------------+                         
```

There was an untested bug within the `handle_call/3` callback from the client. In order to assert the output of an embedded LiveView `handle_call/3` function from the parent, one needs to reach for the `live_children/1` function.

In my case, I had introduced a bug into the "do-work" function which raised an error. `live_children/1` gives me the ability to have the test suite catch this bug at an 'integration' test level. The `live_chilren/1` [docs](https://github.com/phoenixframework/phoenix_live_view/blob/v0.17.11/lib/phoenix_live_view/test/live_view_test.ex#L909) state:

> Returns the current list of LiveView children for the parent LiveView.
> Children are returned in the order they appear in the rendered HTML.

Additionally, I need to call the "do-work" callback from the parent process. This is accomplished with the `render_hook/3` [function](https://github.com/phoenixframework/phoenix_live_view/blob/v0.17.11/lib/phoenix_live_view/test/live_view_test.ex#L873) which:

> Sends a hook event to the view or an element and returns the rendered result.

The test code looks similar to:

```ex
{:ok, view, html} = live(conn)
# match on the parent `view` process
assert [embedded_lv] = live_children(view)
# match on a list of child LiveView processes

render_hook(embedded_lv, "do-work")
```

The combination of `live_children/1` with `render_hook/3` gives you the ability to call hooks from a parent LiveView process. This approach is more useful than testing a callback in isolation.
