# Key-Chord

This package implements support for mapping a pair of simultaneously
pressed keys to a command and for mapping the same key being pressed
twice in quick succession to a command. Such bindings are called
"key chords".

## Quick start

Add to your `~/.emacs`:

``` elisp
(require 'key-chord)
(key-chord-mode 1)
```

and some chords, for example:

``` elisp
(key-chord-define-global "hj" 'undo)
(key-chord-define-global ",." "<>\C-b")
```

## Terminology

In this package, a "key chord" is two keys pressed simultaneously,
or a single key quickly pressed twice.

(Sometimes pressing `SHIFT` and/or `META` plus another key is call a chord,
but not here. However, `SHIFT` plus two normal keys can be a "key chord".)

## Description

Key chord mode acts like a global minor mode controlled by the function
`key-chord-mode`.

Key chord definitions are stored in ordinary key-maps.
The function `key-chord-define-global` defines a chord in the global
key-map and `key-chord-define` defines a chord in a specified key-map,
for example for a specific mode.

A TWO-key chord is two distinct keys pressed simultaneously (within
one tenth of a second, or so).

Examples:
``` elisp
(key-chord-define-global ",."     "<>\C-b")
(key-chord-define-global "hj"     'undo)
(key-chord-define-global [?h ?j]  'undo)  ; the same
(key-chord-define-global "jk"     'dabbrev-expand)
(key-chord-define-global "cv"     'reindent-then-newline-and-indent)
(key-chord-define-global "4r"     "$")
```

Comma and dot pressed together insert a pair of angle brackets.
`h` and `j` pressed together invoke the undo command.
`j` and `k` pressed together invoke the dabbrev-expand command.
`c` and `v` pressed together insert a newline.
`4` and `r` pressed together insert a dollar sign.

A ONE-key chord is a single key quickly pressed twice (within one third
of a second or so).

Examples:
``` elisp
(key-chord-define-global "''"     "`'\C-b")
(key-chord-define-global ",,"     'indent-for-comment)
(key-chord-define-global "qq"     "the ")
(key-chord-define-global "QQ"     "The ")
```

Tick (') pressed twice inserts a back-tick and a tick (`'`).
Comma (,) pressed twice indents for and/or inserts a comment.
`q` pressed twice inserts the word "the ".

Examples: Mode specific chords
``` elisp
(key-chord-define c++-mode-map ";;"  "\C-e;")
(key-chord-define c++-mode-map "{}"  "{\n\n}\C-p\t")
```

The command `key-chord-describe` lists currently defined key chords.
The standard command `describe-bindings` (`C-h b`) will also show key
chords.

The standard command `describe-key` (`C-h k`) will accept a key chord
and show its definition. (Isn't that amazing. There is no explicit
code to carry out this functionality.)

## Customization Options

Key-chord provides several customization options:

- `key-chord-two-keys-delay` (default: 0.1) - Max time delay between two key
  presses to be considered a key chord.
- `key-chord-one-key-delay` (default: 0.2) - Max time delay between two presses
  of the same key to be considered a key chord.
- `key-chord-in-macros` (default: nil) - Whether to expand key chords when
  executing keyboard macros.
- `key-chord-one-key-min-delay` (default: 0.0) - Minimum delay between two
  presses for a double-tap key-chord to be recognized.
- `key-chord-typing-detection` (default: nil) - Try to detect when user is
  typing text and disable chord detection temporarily.
- `key-chord-typing-speed-threshold` (default: 0.1) - Maximum delay between
  keystrokes to be considered part of typing flow.
- `key-chord-typing-reset-delay` (default: 0.5) - Time after which to reset
  typing detection if no keys are pressed.
- `key-chord-use-key-tracking` (default: t) - Track which keys are used in
  chords to optimize performance by avoiding unnecessary lookups.

## Typing Detection

Key-chord now includes a typing detection feature that can help prevent accidental chord triggering during fast typing. When enabled via `key-chord-typing-detection`, the package will monitor your typing speed and temporarily disable chord detection when you appear to be typing text rather than executing commands.

This feature is especially useful for touch typists who frequently trigger unwanted chords during normal typing. It also improves performance during text entry by acting as a debounce mechanism.

To enable typing detection:

```elisp
(setq key-chord-typing-detection t)
```

You can customize the typing detection sensitivity with:
- `key-chord-typing-speed-threshold`: Adjust how fast keystrokes need to be to be considered "typing"
- `key-chord-typing-reset-delay`: How long to wait after typing stops before re-enabling chord detection

## Performance Optimization

Key-chord includes a performance optimization that tracks which keys are part of any defined chord. This allows it to quickly skip the expensive keymap lookup process for keys that aren't used in any chord.

This optimization is enabled by default and controlled by the `key-chord-use-key-tracking` variable. It's particularly beneficial when:
- You have only a few key chords defined but type a lot of text
- You're experiencing lag during fast typing
- You're using key-chord in performance-sensitive contexts

Enabling `key-chord-typing-detection` also helps improve performance by reducing
the number of chord lookups during fast typing.

## Tips

Don't chord key combinations that exists in the languages you typically
write. Otherwise, if you are typing fast, two key intended to be separate
letters might instead trigger a chord.
E.g. `uu` would be a good chord in Spanish but not in Finnish, and
`hj` would be a good chord in English but not in Swedish.

Don't rely solely on `/usr/dict/words` to find unusual combination.
For example `cv` or `fg` can be quite common in certain kinds of
programming. Grep your own texts to verify that a combination is unusual.
And don't forget to check both permutations: `fg` and `gf`.

Choose two keys that are close to each other on the keyboard, so they
can be quickly typed without effort. Chords involving two hands (as
opposed to two fingers on one hand) are harder to type (quickly).
The idea is that key chords are to replace function keys for functions
that are frequently performed while the hands are in writing position.

Key chords might not work well over a slow network.

## Limitations

When recording keyboard macros, the time between keyboard inputs are not
recorded. Thus, the `key-chord-input-method` cannot know for sure if two keys
in a macro was a chord or not. The current solution remembers the first key
of the chords typed during macro recording, and keys that match those (and
are defined as chords) are considered key-chords during macro execution.
This knowledge is not saved with `name-last-kbd-macro`, so they may
execute wrong if they contain pair of keys that match defined chords.

Emacs will not call `input-method-function` for keys that have non numeric
codes or whose code is outside the range 32..126. Thus, you cannot define
key chords involving function keys, control keys, or even your non-english
letters (on national keyboards) that otherwise are well positioned for
chording on your keyboard.
(I think chording left and right arrow keys would be useful, but cannot do.
I consider this a bug in Emacs. Input methods could happily return
unmodified *any* key they don't know about.)

Key chords longer that 2 keys are not supported. It could be done, but I
don't think it is worth the trouble, since most keyboards will not reliably
send all key codes when 3 or more keys are pressed simultaneously.
It might also be a bit trickier to maintain performance.

Key chord mode uses `input-method-function`. And so do internationalisation
packages (mule, quail, etc). Do not expect them to work well together.
The last one that gets the `input-method-function` rules.

## Implementation

Key chords piggy back in ordinary key maps, so they can be defined
per mode without having to add hooks to all modes.

Key chord key codes are vectors beginning with the atom `key-chord`.
A two key chord, e.g. `"hj"`, will add two entries in the key-map.
E.g. `[key-chord ?h ?j]` and `[key-chord ?j ?h]`.

When key-chord-mode is enabled `input-method-function` is set to
`key-chord-input-method`.

## To do

- Find a way to save key-chord info in keyboard macros.
- Save previous value of input-method-function? And call it?
- input-method-function is reset in *info* buffers! What to do?
- How to enter interactively command OR string in key-chord-define-global?

## History

- 0.8.2 (2025-03-15)
  Use float-times instead of timers for typing detection
- 0.8.1 (2025-03-11)
  Add typing detection to prevent accidental chord triggering
  Add minimum delay for double-tap chords
  Performance improvements
- 0.6 (2012-10-23) l.david.andersson(at)sverige.nu
  Add key-chord-define-local, key-chord-unset-local, key-chord-unset-global
- 0.5 (2008-09-15) david(at)symsoft.se
  Bugfix sit-for; Improved examples; New E-mail in comment
- 0.4 (2005-05-07) david(at)symsoft.se
  Slightly better macro heuristics; Added option key-chord-in-macros
- 0.3 (2005-04-14) david(at)symsoft.se
  Require advice; More examples
- 0.2 (2003-09-13) david(at)symsoft.se
  Quick and dirty fix for keyboard macros
- 0.1 (2003-04-27) david(at)symsoft.se
  First release
