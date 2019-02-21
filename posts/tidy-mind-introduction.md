---
title: tidyMind - An introduction
date: 2018-11-27
---

I use [Evernote][evernote] every day, as my main note-taking app and as a place to unload things from my mind. I like it
a lot and have been using it for the last 4.5 years.

But there are some features that I miss and think I could do better 🙃.

Let's start with what I use Evernote for...

Personal side there is: I have to do lists, a shopping list, notes about software projects, my server documentation,
recipes (and some notes about how to cook food in general 😋), ideas for projects and blog posts, a bunch of (check)
lists for packing, festivals, moving, skiing, things to check out on vacations, movies to watch, gift ideas, stuff I'd
like to learn, and so on.

At work I try to document most of the things in [Confluence][confluence] so it helps everybody 😇. But I have some notes
about tickets I am working on (if I think about something related to my current ticket I write it down immediately, so
it's out of my mind and I don't forget it.)
But I also keep there some early ideas which are not yet ready to be discussed in the team, a list of lunch options
around the office and a journal where I write down what I worked on every day.

Features that I use and like are the great "Switch to" shortcut, synchronization between devices, the web client
(for personal stuff, when I'm at work), categories and stacks and a lot of features from the rich text editor
(especially lists and checkboxes).

I also tried a few alternatives: [Bear][bear] looks amazing, but for me it is missing a lot of features, especially a
better organization of documents.

With [Quiver][quiver] I like, that it has more than one document type (Markdown, Code, Latex, Diagrams), but that alone
is no real improvement over Evernote for me.

[Inkdrop][inkdrop] is the most recent solution I discovered, and with [PouchDB][pouchdb] it uses the same data
synchronization library that I want to use 😃. I looks really nice, and I like how it shows the progress of all checked
checkboxes in a document. But again, it's lacking some features I'd like to have.

So, as you might have realized by now, I want to create my own notes app. I've already been working on it on-and-off
since spring this year. The working title is **tidyMind**.

Here come the features I want to have in my app, starting with those which are supposed to make tidyMind special...

### Client-side encryption

I want to have all data encoded and decoded only in the browser. This means the server will never see the raw data and
even if the database is compromised, an attacker wouldn't be able to read the data.
I will be using the [web crypto API][web-crypto] for this. The idea is to derive two keys from the user password and use
one for the authentication with the server, and one for encryption. The second key will never be sent to the server.
The only downside from this is, that there isn't a way to provide a password reset feature. If a user loses their
password, the data is gone.

The implementation of this feature might be worth another blog post 🙂

### Unlimited nested categories

All existing apps that I tested limit document organization to one level of categories and tags. Evernote added one more
level to categories with "stacks", but this doesn't go far enough for me. One example of a tree I might want to have is:
Projects → tidyMind → Features.

### Different types of editors

Similar to Quiver, I would like to have a couple of different document types.
For everyday notes and lists I prefer a simple, uncluttered WYSIWYG view. For drafting blog posts or documentation a
Markdown editor would be better. I'd also like to have a simple spreadsheet editor for numbers like recurring expenses.

Luckily for all of these there are existing open source solutions, so I only have to integrate them and not write them
from scratch.
I have a long list of other types that might be useful as well 😀

### Private notes

This is a pretty rough idea, I don't know how to integrate it into the UI yet. But I would like to have a "private
space" within the app, with notes that shouldn't be visible at all times, i.e. when someone else might be looking at
my screen or is working with my computer.

This could be useful for things like a private journal, gift ideas or notes about your employees (in a work context).

### MVP

These are the features for my minimum viable product. At this stage I want to start using the app for personal stuff and
work:

* Account features (sign up, sign in, change password)
* Reliable sync
* WYSIWYG editor
* Client-side encryption
* Create, update, delete categories and documents, including moving things around

One big thing I will live without in the beginning is a mobile view 😬. The UI between desktop and mobile is quite
different, so I will leave this to do a bit later.

### Current state of the project

* Sign Up, Sign In and Sign Out work 👍
* Change password is to do (this is a bit tricky because of the encryption)
* The synchronization works just fine but needs some testing with conflicts
* The nested category view is done, but it's not yet possible to move stuff around
* I built a fancy switch panel that's keyboard accessible, to quickly open other documents and categories 👍
* [tinyMCE][tinymce] is implemented as a first WYSIWYG editor, I might change it to [Slate][slate] though
* I planned and partially implemented the encryption. There's still some work to do, but I have a proof of concept and
know how to do it.

Perhaps I will give some updates here about the progress 🙂

[web-crypto]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API
[confluence]: https://www.atlassian.com/software/confluence
[evernote]: https://evernote.com/
[bear]: https://bear.app/
[quiver]: http://happenapps.com/#quiver
[inkdrop]: https://www.inkdrop.info/
[pouchdb]: https://pouchdb.com/
[tinymce]: https://www.tiny.cloud/
[slate]: https://github.com/ianstormtaylor/slate