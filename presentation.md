# 2to3
## Converting a Large Web App

<div style="text-align:center">
PyWeb-IL #77 &mdash; Sept. 3<sup>rd</sup>, 2018
</div>

<div style="text-align:center">
Tal Einat
</div>

Notes:
- Done as a consultant
- Late 2017

---

## Background: Overview

- Large webapp
    - 100% Python (Django) backend
    - ~600k Python LoC
- Python 2.7 (latest at the time was 3.6)
- Used only by few tens of internal employees, all in the US

Notes:

- Developed by an Israeli start-up purchased by client company

VVV

## Background: Project

- No real time or work-hours limit, just get it done
- Team of 5 engineers continuing to develop in parallel
- Extensive QA testing to be done at the end before rollout
- Minor regressions can be caught and fixed after rollout 
- Previous successful system-wide upgrade: Django 1.6 -> 1.11

VVV

## Background: Technical

- Poor test coverage
    - Mostly integration tests at various levels
- 79 dependencies in requirements.txt
    - ... plus a few forked and vendored deps
- Lots of funky str/unicode handling, including many untested utilities

---

## Python 3

This new major version includes various non-backwards-compatible changes.

- Notably, a major reworking of the string types
- 2.7, the final 2.x version, is a stepping stone towards 3.x
    - it includes many backported 3.x features
- Community was slow to adopt after the 3.0 release
- Some questioned whether 3.x would ever catch on
- Up to 3.3, 3.x development was focused on easing the transition from 2.x

Notes:

- More on string types later

VVV

## Python 3 Takes Over

By late 2017 Python 3 was well-established:

- Most major libraries supported Python 3
- Adoption was high, perhaps finally overcoming 2.x
- The latest version (3.6) had many great new features not found in 2.7.

We didn't yet know, but in early 2018 Python 2's end-of-life at 2020
was announced.

Notes:

New in 3.x not available in 2.7:

- asnycio and `async`/`await` (and `yield from`)
- f-strings
- matrix multiplication operator: `@`
- unpacking generalizations
    - `{**a, **b}`
    - `a, b, *rest = iterable`
- underscores in numeric literals
- pickle protocol 4

---

## Python 2 to Python 3: six?

Many libraries support 2.x and 3.x, usually using the "six" library,
such as Django.

This is less practical for web-apps though:

- Apps can usually specify a version of Python
- six makes code uglier in some cases ("readability tax")
- No automated tools convert from 2.x to six

Notes:

- 6 = 2 × 3; get it?
---

## 2to3

The Python devs spent great effort creating "2to3", a code analysis and
manipulation tool for automatically converting Python 2 code to Python 3.

Unfortunately, it could not address the many edge-cases and minor
inconsistencies found in practice.

2to3 was not well received.  Due to the great popularity of Python 2 at the
time of Python 3's release, most projects aimed to support both versions.
2to3 did not meet this need.

Notes:

It was originially suggested to maintain a 2.x codebase and use 2to3 to
release 3.x versions.  However, since 2to3 failed to cover many edge-cases
automatically, this was not practical.

---

## My Approach: 2to3!

- 2to3 was exactly what I needed.
- I reviewed other, similar tools and found that they added nothing relevant.
- Despite being somewhat neglected, it still worked very well and covered the
    great majority of required changes.
- 2to3 is **extensible!**  By writing custom "fixers" or changing existing ones,
    I could use it to address additional cases not supported out of the box.

---

<!-- .element: class="auto-fragment" -->

## Final Remarks

-   TODO!

---

# Questions?
