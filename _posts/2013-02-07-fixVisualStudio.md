---
layout: post
title: "Tricks to make Visual Studio a real text editor"
date: 2013-02-07
categories: keyboard efficiency howto rant
---

# Tricks to make Visual Studio a real text editor

*Dedicated to the keyboard addicts out there*

## Prelude

I sit and write this at home in jEdit, my text absolute text editor of choice. If you use Notepad++, so be it. I want you to understand though that up until one week ago, I do *EVERYTHING* in jEdit. I live in Markdown. I've been learning Ruby and writing a dumb app in it. Email drafts, presentations using S9, howtos / technical documents, this post, HTML, XML, XSLT, regex, bash scripts, it just goes on and on. There is an article somewhere that speaks to learning one editor well, as then it's just a matter of learning a new language, not an IDE.

I know it's that I've used jEdit for so long that I just know the shortcuts. But some of them just make so much sense, and I have yet to program (or even find the ability to) Visual Studio to act the same way. I'm one beer away from pulling an all-nighter to get jEdit tweaked to handle c# the way I want it to; I'm sure it wouldn't take anywhere near that long, but I also know I'd get hooked, look up, and bam, it's 5am.

So I'm midway on ramping back up on Visual Studio. Why I didn't start by importing my old settings, I don't know. Intellisense in VS has become just absolutely terrible. Throw ReSharper on top and you might as well forget about TDD, let alone your keyboard.

## Things I haven't found...

*...and haven't wanted to waste hours searching for, Please help!!!*

**Navigate by blocks / paragraphs**

In jEdit, and really, I think Kate, gEdit and any text editor that is worth considering, `<ctrl>+<up>` moves up to the next block of text. Guess what `<ctrl>+<down>` does. :-D That's a freebee. Guess what VS does by default? **IT FRIGGIN SCROLLS!!!** Please, anyone, answer me why I would want it to scroll. PageUp, PageDown work still, and there's always scroll lock if I really want it. Come to think of it, Word does this too. Do the m$ Devs not use their keyboards? Should we blame it on m$ UX? (No, because that doesn't really exist).

Oh, and guess what a real editor does when you throw `<shift>` into the mix? It selects the block. I know, I'm sorry, I didn't mean to make you gasp and lose your breath. Guess what VS does? NOTHING!!!!!!!!

**Rant about the stupid Keyboard Editor**

I'm using VS 2012. VS 2012. Did you catch that? VS 2012. Here is what the keyboard editor in jEdit looks like:

![image: jedit keyboard config](/home/damon/Dropbox/Photos/screenshot/jedit-keyboard.png)

![image: online example](http://xilize.sourceforge.net/Reference/guide/images/goShortcuts.png)

Your first comment? When did you write this, 1980? **MY POINT EXACTLY!!!** Look at how much real estate is there, and get this, commands can have alternate bindings! And, the window can be resized to fit your needs.

Now, check out your nice, modal, statically sized window that VS 2012 gives you, you mouse-addict you. Look at all that generous space it gives to actually see the shortcuts! Seriously m$, with all the good, open source keyboard editors out there, can't you figure out how to spruce yours up a bit???

![image: jedit keyboard config](/home/damon/Dropbox/Apps/Greenshot/microshaft-keyboard.png)


**Intelligent `<ctrl>+<backspace`**

I don't know what else to call this. Again, jEdit, Kate, and gEdit just work; when you press this, it deletes the last statement, breaking on whitespace and depending on the editor, sometimes the underscore.

Visual Studio? It will graciously work as you expect when you're dealing with a word with whitespace,*AS LONG AS* you are not at the start of a line. If you happen to be on the first word, it will go ahead and delete the closing brace on the previous line for you, because that's obviously what you wanted.

The fix for VS? In 2010, there was a plug-in you could add to change the behavior. In 2012, best I have been able to find is the mouseists' ReSharper.

Don't get started on ReSharper accepts keyboard shortcuts, that's not my point. Turn off intellisense but leave Resharper on... didn't think so.

## Calm down dude...

I'm back. So this post was originally not going to be a hate on m$, but Tux is sitting here just smiling at me as the rant just spews out so effortlessly here, and I recall how much I'm fumbling with a stupid IDE that really is only good for one language. Right, mellow.

Here are a few hacks I've done to make VS a bit less mousesist.

**Intellisense**

Irony on the name aside, you already know if you're a TDD'er that the balance here is key. You want something that is available if and when you want it. You don't want a crutch, because you'll find you type two chars and wait for a prompt... slow. You don't want autocomplete because you'll find that it just throws some random callout in when you wanted to create a new class.

You basically want to disable all auto-complete, and that will work fine as long as you don't install ReSharper. 

**ReSharper**

It's good for what it's good for, but gets in the way for everything else. If you are a mousesist, you'll probably love this. I hate it, but after using it, there are a few things I really like about it.

One thing I really hate about it is, well, have you seen their menu and all the options? Good luck making sense out of that. They give you a search, but do you know what to search for?

I was hoping to find a disable all, or learn one feature as you go thing, but no, it's just a huge mess all in your face, all at once. And you CANNOT turn off it's auto-complete. The bug that they supposedly fixed in 5.1 is still present in 7x:

+ You type a line; 
+ you hit escape multiple times, most likely cussing as it thinks you want an instance of ICantBelieveYouDontWantToUseThisRidiculouslyLongClassName, when really you wanted to write a failing test around ICanLiveWithoutResharper.
+ You go back to fix the ICant...., backspace, give it your parens, and next thing you know, the variable name you were going to use is GONE!

Ah, sorry, a bit snarky. Back to bending ReSharper to your will...

You can try to find the option in ReSharper, but FYI they like to move it a lot. The fastest way is to Google a shorter post than this. The second fastest way is to go to the archaic keyboard options, search for Resharper, scoll the stupidly long list, looking for Toggle Resharper or something like that.

Bind it to `<ctrl>+givemebackmykeyboard` or something shorter (and realistic, m$ only supports double-chords) like `<ctrl>+<shift>+x` and ignore whatever dumbass command ReSharper has bound to that, you're not going to use it anyway.

Then, keep ReSharper off for the majority of the time.

+ Test (red)
+ Make it pass (green)
+ Add more tests if necessary
+ Refactor, possibly enabling ReSharper for some tips.

With all that hatred for ReSharper and Visual Studio, you might ask why I'm using it, me too. I'm probably not going to buy it. It helps for finding naming inconsistencies, it helps a bit with refactoring, and helps a bit with finding dead code when that part is working correctly. The rest is just kitchen sink and YAGNI.

**Markdown support**

Everyone in the world supports it. OK I guess that's an exaggeration, but README.markdown or .md is recognized by github, so why not VS?

Go to whatever the Tools: Addons menu is (something like Extensions and Addons; lots of spaces and characters, just look for that) and search for Markdown. Realize that VS is dumb and realizes it, so it acknowledges that you would want 50x plugins, so it starts with installed plugins. Change that to online, search again. Install the Markdown plugin, now you have some syntax highlighting as well as a Markdown preview.

Or again, use a real editor. At least there's a plugin for this I guess, so I should shut up.

**NCrunch**

Not much to do here, but I'm currently writing integration tests, not unit tests. These can unfortunately be a bit slow, so I do toggle this on `<c><s>+e`and off`<c><s>+d` and keep the engine mode to impacted tests only.

Also see my post about [NCrunch](http://professional.damonoverboe.org/ncrunch-on-the-fly-testing-in-parallel) for a few suggestions on tweaking it. 

Finally, I've rebound the commands for TestExplorer.RunTest (C+r,t) and TestExplorer.DebugTest(C+r,C+t) to NCrunch.DebugCoveringUseNewProcess and NCrunch.RuncoveringTestsUseNewProcess to speed up the test run and take advantage of it running in the background.
