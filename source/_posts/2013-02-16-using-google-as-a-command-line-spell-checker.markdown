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

This was the original impetus for the following piece of code.  It's a
glorified cURL call, but it does the trick.  The link to the full code
can be found bellow, but the critical code is here:

``` perl
# Now we define a set of rules we can use to extract suggestions.
# Some of them look at Google's return values, while others use
# Wikipedia, and Urban Dictionary, etc.
my @patterns = (

# It seems that we can take advantage of the information the Google
# search engine leaks when it returns results.  The word that
# typically is most significant to a document is bolded.  This means
# that we can simply pick one or more of the bolded results.  For now
# we will pic the first link, and see how this works.

#		(Birthday - Wikipedia, the free encyclopedia

# For simplicity, we assume the following phrase will remain stable.
# This we can make our code much simpler and easier to follow.  It
# also seem, that in the long run, unless many people adopt this
# method of word-recommendation search, google's only changes would be
# for aesthetic reasons, and considering their minimalistic display
# data, we should be able to actually quite easily update this in the
# future, even it if is a major word order change:
		
		"Did you mean to search for:[ ]*([^ ]*).*",
		
# Where the above line shows up if Google is *guessing* (aka "recall
# that that word is actually spelt."), the following is likely when
# the "optimal" match is found, according to its ruleset:
		
		"Showing results for[ ]*([^ ]*)",
		
# If we don see anything from Google, then we may be looking for a
# piece of slang, so we will use our trusty old Urban Dictionary
# friend:
		
		"Urban Dictionary:[ ]*([^ ]*)");

# Ask Google for a spelling suggestion for the given word.  We write
# the results to the cache file via the 'tee' command which will echo
# them for the command line user to see.
open (OUTPUT, "| tee $cache_filename");
while (<INPUT>) {
  
  # Strip all HTML from the document, so it is easier to parse.  It is
  # odd that web-scraping can come to this, where it just makes more
  # sense to strip all that is *structured* to work with simpler
  # structure.  Is this what people call irony?
  chomp;
  s/<[^>]*>/ /g;
  
  # Check each of the patterns we defined above:
  for my $pattern (@patterns) {
    if (/$pattern/) {
      $suggestion = "$1";
      last;
    }
  }
}

# Finally, print the suggestion: 
print OUTPUT "$suggestion";
```

The full code can be seen
[here](https://raw.github.com/versionzero/dotfiles/master/local/bin/google-suggest).
