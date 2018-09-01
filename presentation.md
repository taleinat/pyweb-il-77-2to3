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

- Fix lots of language design mistakes
- Notably, a major reworking of the string types
- 2.7 includes many backported 3.x features
- 2.x → 2.7 → 3.x
    - ~~2.8~~ will never be

Notes:

- "Hindsight is 20/20"
- 2.7 is intended as a stepping stone from 2.x to 3.x
- More on string types later

VVV

## Python 3: A Rocky Start

- 3.0 release → Community slow to adopt
- Some questioned whether 3.x would ever catch on
- Up to 3.3, 3.x development was focused on easing the transition from 2.x

VVV

### Diffusion of Innovations

<div style="text-align:center">
![Diffusion of Ideas](images/diffusion_of_ideas.svg "Diffusion of Ideas") <!-- .element height="75%" width="75%" -->
</div>

Notes:

- 3.0 release is at the left edge of the graph
- by late 2017, somewhere in the middle

VVV

## Python 3 Takes Over

By late 2017 Python 3 was well-established:

- Most major libraries supported Python 3
- Adoption was high
    - Perhaps finally overcoming 2.x
- Versions 3.4-3.6 introduced many great new features not found in 2.7.
    - Practical incentive to use 3.x

We didn't yet know, but in early 2018 Python 2's end-of-life at 2020
was announced.

Notes:

New in 3.x not available in 2.7 (incl. backport libs):

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

Less practical for web-apps:

- Apps can usually specify a version of Python
- Incurs a "tax" on coding, style and readability
    - esp. for dict-heavy code
- No automated tools convert from 2.x to six

Notes:

- 6 = 2 × 3; get it?
- Highly recommended for Python libs

---

## 2to3

- Refactoring tool created by the Python devs
- Intention: keep 2.x code, use 2to3 for 3.x releases
- 2to3 couldn't handle the "very long tail" of real edge cases
- Not well received, hardly used, fell into obscurity

Notes:

The Python devs (incl. Guide) spent great effort creating "2to3", a refactoring
tool for automatically converting Python 2 code to Python 3.

The intention was that devs would initially continue to maintain a 2.x codebase,
and use 2to3 to release 3.x versions.  Unfortunately, 2to3 could not address
the many edge-cases and minor inconsistencies found in practice, making the
intended use impractical.

2to3 was not well received and fell into obscurity.

---

## My Approach: 2to3!

- 2to3 was exactly what I needed!
- Covered the great majority of required changes
- Worked very well
- 2to3 is **extensible!**

Notes:

- Surprise! (not)
- Obscure but not neglected
- I reviewed other, similar tools and found that they added nothing relevant.
- By writing custom "fixers" or changing existing ones, I could use 2to3
    to address additional cases not supported out of the box.

---

## 2to3: Divide an Conquer

- Default mode: run on a whole project, applying all fixers.
- Resulted in one *huge* diff.  No good for me: 
    - I wanted to manually review the changes.
    - I wanted to apply changes incrementally.

Notes:

- I wanted the result nice & readable, not just correct.

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
- Non-default way to use 2to3; not documented.

---

## 2to3: Incremental Application

- Fixers have inter-dependencies
- Total 52 fixers (4 are optional)
- Need to be run in a certain order

```python
# lib2to3/fixer_base.py
class BaseFix(object):
    run_order = 5   # Fixers will be sorted by
                    # run order before execution.
                    # Lower numbers will be run first.

    order = "post"  # Does the fixer prefer pre-
                    # or post-order traversal
```

Notes:
- The 4 optional fixers weren't relevant
- pre/post order turned out to be irrelevant
- I read lib2to3 code for listing and loading fixers
- I only did this once to create a properly sorted list

VVV

## 2to3: Fixer Example

```python
# lib2to3/fixes/fix_isinstance.py
"""Fixer that cleans up a tuple argument to isinstance after
the tokens in it were fixed.  This is mainly used to remove
double occurrences of tokens as a leftover of the
long -> int / unicode -> str conversion.

eg.  isinstance(x, (int, long)) → isinstance(x, (int, int))
       -> isinstance(x, int)
"""

class FixIsinstance(fixer_base.BaseFix):

    run_order = 6
```

Notes:
- This example shows why order matters
- Default `run_order` is 5
- The 4 optional fixers weren't relevant
- I read lib2to3 code for listing and loading fixers
- I only did this once to create an ordered list

VVV

## Manual Tweak Example: list()

- Many fixers for things that now return iterators, e.g. map(), dict.keys()
- These are wrapped with `list(...)`
- ... except in special cases:
    - `for X in map(...):`
    - `sorted(map(...))`
- Special cases sometimes missed
- `filter(foo, map(...))` missed
- Result: Lots of manual tweaking

Notes:

- In hindsight, manual tweaking here was possibly a mistake

---

## Manual Fix: Integer Division

- The division operator (`/`) now does true division on ints.
- No 2to3 fixer for this!
    - impossible to automate all cases
    - but why not simple cases?
    - `3 / 2`

Notes:

- This is a glaring omission IMO
- A few more esoteric changes also have no fixers

VVV

## Manual Fix: Integer Division

I regex searched through the codebase with this:

```python
re.compile(r'''
    ^
    (?:
        [^\n#'"]                |  # ignore comments
        '(?:[^'\n\\]|\\[^\n])*' |  # single-quote string
        "(?:[^"\n\\]|\\[^\n])*"    # double-quote string
    )*
    # exactly one slash
    (?<!/) / (?!/)
    # not followed by a float
    (?!\s*\-?\s*(?:
        \d+\.        |
        \d+e[+-]?\d+ |
        float\s*\(
    ))
''', re.VERBOSE)
```

Notes:

- find in all files → tweak regexp → repeat
    - until few enough results (just over 100)
- actually fixed ~10 cases that would have been broken
- https://regex101.com/r/9ZCcH7/1

VVV

## Manual Fix: Integer Division

![Integer Division Regex](images/int_division_regex.png)

- find in all files → tweak regexp → repeat
    - until few enough results (just over 100)
- actually fixed ~10 cases that would have been broken
- https://regex101.com/r/9ZCcH7/1

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

VVV

## Dependencies
### Dropped due to inclusion in stdlib

1. enum34
2. funcsigs (required by mock)
3. futures
4. mock
5. simplejson
    - required refactoring imports: `simplejson` → `json`
6. wsgiref

Notes:

- fixing the `simplejson` imports would have been a good, simple fixer

---

<!-- .element: class="auto-fragment" -->

## Final Remarks

-   TODO!

---

# Questions?
