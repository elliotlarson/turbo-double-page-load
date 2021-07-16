# Turbo Double Page Load

When you have Turbo installed and you click a link to a page on your site with a different pack, Turbo recognizes that the page header has changed and reloads the page... whoa, double page request (not as cool as a double rainbow).

This is a basic Rail app with two pages, that highlight this issue:

* `http://localhost:3000/page1`
* `http://localhost:3000/page2`

Page one requires a JS pack `application1` and page two requires another pack `application2`.

When you first load page one, the log outputs a single request (sweet):

```text
Started GET "/pages/page1" for ::1 at 2021-07-15 18:55:14 -0700
Processing by PagesController#page1 as HTML
  Rendering layout layouts/application.html.erb
  Rendering pages/page1.html.erb within layouts/application
  Rendered pages/page1.html.erb within layouts/application (Duration: 0.5ms | Allocations: 89)
  Rendered layout layouts/application.html.erb (Duration: 10.2ms | Allocations: 2583)
Completed 200 OK in 13ms (Views: 12.1ms | Allocations: 2908)
```

When you click the "page2" link on this page, the log outputs the double request resulting from Turbo seeing that the header has changed with a reference to the new pack (not sweet):

```text
Started GET "/pages/page2" for ::1 at 2021-07-15 18:55:17 -0700
Processing by PagesController#page2 as HTML
  Rendering layout layouts/application.html.erb
  Rendering pages/page2.html.erb within layouts/application
  Rendered pages/page2.html.erb within layouts/application (Duration: 0.5ms | Allocations: 89)
  Rendered layout layouts/application.html.erb (Duration: 7.6ms | Allocations: 2584)
Completed 200 OK in 12ms (Views: 11.1ms | Allocations: 3744)


Started GET "/pages/page2" for ::1 at 2021-07-15 18:55:17 -0700
Processing by PagesController#page2 as HTML
  Rendering layout layouts/application.html.erb
  Rendering pages/page2.html.erb within layouts/application
  Rendered pages/page2.html.erb within layouts/application (Duration: 0.2ms | Allocations: 89)
  Rendered layout layouts/application.html.erb (Duration: 14.9ms | Allocations: 2578)
Completed 200 OK in 16ms (Views: 15.8ms | Allocations: 2899)
```

That's right, request... and then a request.  Double tap is good for zombies, not requests.

While on page two, if you click the "page2" link, you only get a single page request (packs have not changed):

```text
Started GET "/pages/page2" for ::1 at 2021-07-15 18:55:22 -0700
Processing by PagesController#page2 as HTML
  Rendering layout layouts/application.html.erb
  Rendering pages/page2.html.erb within layouts/application
  Rendered pages/page2.html.erb within layouts/application (Duration: 0.4ms | Allocations: 89)
  Rendered layout layouts/application.html.erb (Duration: 8.2ms | Allocations: 2588)
Completed 200 OK in 10ms (Views: 9.6ms | Allocations: 2913)
```

But, if you click the "page1" link, you again get the double page request (because packs have changed):

```text
Started GET "/pages/page1" for ::1 at 2021-07-15 18:55:24 -0700
Processing by PagesController#page1 as HTML
  Rendering layout layouts/application.html.erb
  Rendering pages/page1.html.erb within layouts/application
  Rendered pages/page1.html.erb within layouts/application (Duration: 0.9ms | Allocations: 89)
  Rendered layout layouts/application.html.erb (Duration: 16.2ms | Allocations: 2695)
Completed 200 OK in 19ms (Views: 17.9ms | Allocations: 3019)


Started GET "/pages/page1" for ::1 at 2021-07-15 18:55:24 -0700
Processing by PagesController#page1 as HTML
  Rendering layout layouts/application.html.erb
  Rendering pages/page1.html.erb within layouts/application
  Rendered pages/page1.html.erb within layouts/application (Duration: 0.4ms | Allocations: 89)
  Rendered layout layouts/application.html.erb (Duration: 16.6ms | Allocations: 2651)
Completed 200 OK in 19ms (Views: 18.0ms | Allocations: 2973)
```

## Why does this matter?

If you just have a single pack for your app, this doesn't happen to you, so you get to be happy (at least about this).  But, if you have pages with unique javascript being loaded in the header, you will encounter a double page request.  If the page isn't an expensive page to load, perhaps this won't be a huge deal for you, but if the page is heavy at all, it's potentially not cool.

Also, if you happen to be using the whole `javascript_packs_with_chunks_tag` approach to break up the JS for sections of your site, you're going to run into this issue... like a friend of ours did... not us... someone else. `Frowny sad face :(`.
