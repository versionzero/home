---
layout: post
title: "Using Google as a command-line spell-checker"
date: 2013-02-16 19:08
comments: true
categories: [code,perl]
---


At the best of times, I can maintain a mediocre level of control of
the English language.  Unfortunately, this has not translates in to an
ability to spell.

For most applications, this is fine, there's a reasonable spellchecker
built in to just about everything.  It's a problem for me when I'm at
the terminal all day though.  It's true, there are plenty of command
line checkers, but it's rare that any can handle the level of
misspelling I feed them.  To date, the most capable system of parsing
my spelling is Google.  But switching to a browser and feeding in a
word at a time is a tad cumbersome, to say the least.

This was the original impetus for the code.  It's a glorified cURL
call, but it does the trick.  It also became quite clear that for a
little extra work, we could detect things like the Urban Dictionary
(Dirionary) results as part of of the page.

For simplicity, we assume the following phrase will remain stable:

```
Did you mean to search for:[ ]*([^ ]*).*
```

This can make our code much simpler and easier to follow. It also
seem, that in the long run, unless many people adopt this method of
word-recommendation search, Google's only changes will be for
aesthetic reasons, and considering their overal minimalistic approach
to data display, we should be able to actually quite easily update
this in the future, even it if is a major word order change.

Where the above line shows up if Google is *guessing* (aka "recall
that that word is actually spelt."), the following is likely when the
"optimal" match is found, according to its rule-set: 

```
Showing results for[ ]*([^ ]*)
```

Lastly, for now, if we don see anything from Google proper, then we
may be looking for a piece of slang, so we will use the result of our
trusty old friend, Urban Dictionary:

```
Urban Dictionary:[ ]*([^ ]*)
```

The link to the full code can be found bellow, but the critical code
is here:

``` perl
my @patterns = (
   "Did you mean to search for:[ ]*([^ ]*).*",
   "Showing results for[ ]*([^ ]*)",
   "Urban Dictionary:[ ]*([^ ]*)");
open (OUTPUT, "| tee $cache_filename");
while (<INPUT>) {
  chomp; s/<[^>]*>/ /g;
  
  # Check each of the patterns we defined above:
  for my $pattern (@patterns) {
    if (/$pattern/) {
      $suggestion = "$1";
      last;
    }
  }
}
print OUTPUT "$suggestion";
```

Notice we strip all HTML from the document, so it is easier to parse.
It is odd that web-scrapping can come to this, where it just makes
more sense to strip all that is *structured* to work with simpler
structure.  Is this what people call irony?

The full code can be seen
[here](https://raw.github.com/versionzero/dotfiles/master/local/bin/google-suggest).
