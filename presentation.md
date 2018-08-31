# 2to3
## Converting a Large Web App to Python 3

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

## Background

<div style="text-align:center">
![Monolith](images/washington monument.jpg "Monolith") <!-- .element height="35%" width="35%" -->
</div>

VVV

## Background: Overview

- Late 2017
- Large webapp
    - Single monolithic codebase
    - 100% Python (Django) backend
    - ~600k Python LoC
- Python 2.7
- Used only by few tens of internal employees, all in the US

Notes:

- Developed by an Israeli start-up purchased by client company

VVV

## Background: Project

- Team of 5 engineers working on codebase
    - Continued working while I worked on the upgrade
- Extensive QA testing to be done before rollout
- Minor regressions after rollout are acceptable

Notes:

- This means merges! 
- Previous successful system-wide upgrade: Django 1.6 -> 1.11

VVV

## Background: Technical

- Poor test coverage
    - Mostly integration tests at various levels
    - Lots of utils, mostly untested
- 79 dependencies in requirements.txt
    - ... plus a few forked and vendored deps
- Lots of funky str/unicode handling, including many untested utilities

---

## Python 3

This new major version includes various non-backwards-compatible changes.

- Notably, a major reworking of the string types
- 2.7, the last 2.x, is a stepping stone towards 3.x
    - it includes many backported 3.x features
- Fix lots of language design mistakes

Notes:

- More on string types later
- "Hindsight is 20/20"

VVV

## Python 3: A Rocky Start

- Community was slow to adopt after the 3.0 release
- Some questioned whether 3.x would ever catch on
- Up to 3.3, 3.x development was focused on easing the transition from 2.x

VVV

### Diffusion of Innovations

<div style="text-align:center">
![Diffusion of Ideas](images/diffusion_of_ideas.svg "Diffusion of Ideas") <!-- .element height="75%" width="75%" -->
</div>

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

Notes:

- Surprise! (not)

---

## 2to3: Divide an Conquer

- Default mode: run on a whole project, applying all fixers.
- Resulted in one *huge* diff.  No good for me: 
    - I wanted to manually review the changes.
    - I wanted to apply changes incrementally.

VVV

## 2to3: Divide and Conquer (cont.)

Possible MO: Upgrade each module/package individually.

- Some very large modules (~10k LoC)
    - So still huge diffs
- Mixes many different types of changes
    - Lots of mental context switching
    
VVV

## 2to3: Divide and Conquer (cont. #2)

Alternative MO: Apply each "fixer" separately.

- Avoid excessive mental context switching
- Allows cataloguing fixer-specific issues
- Allows tweaking an re-running fixers
- Allows making codebase-wide manual tweaks
    - Lots of regexp search/replace

---

## 2to3: Incremental Application

- Fixers have inter-dependencies
- Need to be run in a certain order
- Total 52 lib2to3 fixers
    - ... of which 4 are optional

```python
# lib2to3/fixer_base.py
class BaseFix(object):
    """Optional base class for fixers."""

    run_order = 5   # Fixers will be sorted by
                    # run order before execution.
                    # Lower numbers will be run first.
```

Notes:
- The 4 optional fixers weren't relevant
- I read lib2to3 code for listing and loading fixers
- I only did this once to create an ordered list

VVV

## 2to3: Incremental Application

```python
# lib2to3/fixes/fix_isinstance.py
"""Fixer that cleans up a tuple argument to isinstance after
the tokens in it were fixed.  This is mainly used to remove
double occurrences of tokens as a leftover of the
long -> int / unicode -> str conversion.

eg.  isinstance(x, (int, long)) -> isinstance(x, (int, int))
       -> isinstance(x, int)
"""

class FixIsinstance(fixer_base.BaseFix):

    run_order = 6
```

Notes:
- The 4 optional fixers weren't relevant
- I read lib2to3 code for listing and loading fixers
- I only did this once to create an ordered list


---

## Dependencies

I went through the ~80 deps one by one:

- ~50% already supported Python 3
- ~50% just needed to be upgraded
- of the few left:
    - some dropped
    - some had drop-in replacements
    - one I forked and upgraded

End result: a single "update dependencies" commit.

---

<!-- .element: class="auto-fragment" -->

## Final Remarks

-   TODO!

---

# Questions?
