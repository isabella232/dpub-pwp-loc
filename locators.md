## Definitions

* **Canonical**
Canonical in the [mathematical sense](https://en.wiktionary.org/wiki/canonical):
distinguished among entities of its kind,
so that it can be picked out in a way
that does not depend on any arbitrary choices.

> These arbitrary choices are, in our case, packed vs unpacked.
Canonical absolute URL-s have the protocol built-in.
See current WD.

* **Locator**
URL that points to either an entire PWP or a resource within a PWP.

* **PWP Locator**
A Locator to the PWP as a whole. This PWP is always in a certain state.
Also called "State locator".

* **Relative locator**
A locator to a resource within a PWP, without specifying the location of the PWP.

* **Absolute locator**
A combination of a PWP Locator and a relative locator.

> To be more exact, a locator is always a _resource_ locator.
This _resource_ part is implied, but omitted for brevity.

* **Canonical locator**
An absolute and state independent locator.

The canonical locator of a resource of a PWP is constructed
by assuming that the PWP is published unpacked.
One looks at this _conceptually_ as if the PWP was unpacked, i.e,
this does not imply that the PWP _has_ to be published unpacked.

Example: if you have an image `mona lisa.jpg`,
and that file is in subfolder `/img/`,
and a book is published under `http://book.org/books/1/pwp`,
then the book's canonical locator is `http://book.org/books/1/pwp`,
and the image's canonical locator is `http://book.org/books/1/pwp/img/mona%20lisa.jpg`.
Whether the book is published unpacked, zipped, tarred, etc...
The canonical locator is the same.

> The various states, that may have their distinct locators/URI-s,
contain exactly the same abstract content.

* **Reading system**
A system that is able to read and display PWPs.
This could be a browser of the future, a current browser with a plugin,
a desktop application, an app, ...

* **PWP Processor**
A system that knows about the internals of a PWP, e.g., the manifest,
and is able to then find all of the resources for processing.

> Question: does the PWP Processor reside client-side, server-side, or both?

> Question: if the reading system is supposed to also handle packed PWPs,
doesn't then the reading system also need a 'PWP Processor'-module?

* **Distribution system**:
A server-side system that is able to distribute PWPs.
This could be a set of apache configurations, a full-blown web service, a module...

> See issue 1.


## PWP functions

Functions that are needed for handling PWPs.

* **Displaying**
Displaying a (resource of a) PWP to the end-user.

> Is _displaying_ always done by the reading system?

* **Mapping**
Mapping the canonical locator to the state locator

* **PWP retrieving**
Getting an entire PWP. This is different whether the PWP is packed or unpacked.
A packed PWP can be retrieved by simply downloading the package.

> See issue 2 for retrieving an unpacked PWP

* **Resource retrieving**
Getting a resource within a PWP. This depends on the state of the PWP, e.g.,
if a PWP is zipped, the package needs to be unzipped,
at least partially, to retrieve the resource.

> See issue 1.

### Issue 1: Retrieving a resource of a packed PWP

Given a situation where a PWP is only published packed,
e.g., once zipped and once tarred,
and a resource is requested,
the question is how does this resource get to the end-user, i.e,
1. do I retrieve the entire PWP first, or
2. do I retrieve that resource directly?

Option 1: You first retrieve the entire PWP.
If a user wants to retrieve a resource via its canonical locator,
the canonical locator is first _mapped_
to a PWP locator in a certain state (e.g., `book.tar`),
after which the entire PWP is downloaded (_PWP retrieving_),
and then the _resource retrieving_ needs to happen client-side.
Then, the resource is _displayed_ to the user.

> Content Negotiation is an already existing mechanism
that can be used to differentiate between the different published states of the PWP,
server-side (_mapping_).

Option 2: You retrieve the resource directly.
If a user wants to retrieve a resource via its canonical locator,
the canonical locator is first _mapped_
to a PWP locator in a certain state (e.g., `book.tar`),
after which a server mechanism is used to _retrieve the resource_
from within the server-side PWP (so _PWP retrieving_ is implied).
The, the resource can be downloaded directly,
and the resource is _displayed_ to the user.

> These kinds of server mechanisms can be implemented via, e.g., Apache modules
(_resource retrieving_)

The mapping is different in the two options.
the _mapping_ of the first option will return a URL to an entire PWP,
and the client is responsible for the _resource retrieving_.
The _mapping_ of the second option will return a URL to the actual resource,
and the server is responsible for the _resource retrieving_.

> Question: can we allow both?
I.e., if the PWP processor cannot return a resource directly,
the reading system unpacks client-side and displays afterwards.
If a PWP processor can return a resource directly,
the reading system can immediatly display the resource.

### Issue 2: Retrieving an unpacked PWP

What happens when GETting an unpacked PWP?
Do all relevant files need to be downloaded? Or only the metadata?
If only the metadata, does this equal the manifest?

> HTTP Link headers, or directly via the payload, are options
to return the metadata.

## Questions

When someone downloads the book to `file://book.packed`,
and creates a locator to `mona lisa.jpg`,
the canonical locator to that file will remain `http://book.org/books/1/pwp/img/mona%20lisa.jpg`,
but the actual location will be the file `img/mona%20lisa.jpg`
inside the package `file://book.packed`.

### Q1: So, how does a reading system go from the canonical locator to the actual locator?

Every PWP has metadata.
One of these fields is the canonical locator, i.e., `http://book.org/books/1/pwp`.
When locating a resource within a local PWP using a canonical locator,
you can use the canonical locator from the PWP's metadata
to map from the canonical locator to the local locator.
E.g., fetching `img/mona%20lisa.jpg` within `file://book.packed`.

> Where this metadata resides is to be discussed.
Inside the manifest makes sense, but including it in the html pages
might also be an option.
For the latter, e.g., &lt;link rel="http://pwp.org/canonical" href="http://book.org/books/1/pwp"/&gt;
is an option.

### Q2: So, what happens when I copy my PWP to another computer?

The canonical locator remains the same, but the location of PWP changes.
The reading system knows where the local PWP is, e.g., `file://home/books/book.packed`,
and can use the same mapping mechanism to fetch `img/mona%20lisa.jpg`.

### Q3: So, what happens when I alter my PWP?

Nothing changes with the previous PWPs, nor with the canonical locator.
The PWP you now currently have is a different PWP than the original PWP.
If you are very concerned with having correct metadata (aren’t we all?),
you alter the metadata of the PWP: the canonical URL changes to,
e.g., `http://school.org/course/10/pwp`,
and you can keep the original canonical URL as extra metadata.

> &lt;link rel="http://pwp.org/breadcrumb" href="http://book.org/books/1/pwp"/&gt;

> This could result in a trail of breadcrumbs,
when you make changes on already changed PWPs

Smart reading systems might use these breadcrumbs to allow for indirection.
E.g., when someone adds an annotation to `http://book.org/books/1/pwp/img/mona%20lisa.jpg`,
the annotation-aware reading system sees in the metadata
that this PWP with canonical URL `http://school.org/course/10/pwp`
is derived from `http://book.org/books/1/pwp`,
so it can do a best effort to try and locate `img/mona%20lisa.jpg`
within `http://school.org/course/10/pwp`,
and show the annotation at the correct position.

> This means the system has to locate (and possibly adapt)
the annotation that resides in `http://book.org/books/1/pwp`.

### Q4: So, what if the PWP isn’t published unpacked? What do I get when I `GET http://book.org/books/1/pwp/img/mona%20lisa.jpg`?

> See issue 2.

In one way or another, you get the canonical URL from that GET request,
and fetch the PWP that is hosted there
(e.g., a packed PWP hosted on `http://book.org/books/1/pwp`).
Your reading system can then again map the canonical locator to the relative locator,
and fetch the resource within the package.

> E.g., using HTTP Link Headers, TBD.

### Q5: So, what if your PWP isn’t published online at all?

Then the PWP isn’t accessible online.

However, the same PWP that is distributed internally can have a canonical URL,
and so, locators _can_ be shared.
E.g., a law document is made by Annie (with canonical URL `intranet://annie/law1`)
and sent to Bob (so Bob’s copy also has canonical URL `intranet://annie/law1`).

Bob mails to Annie:
"Remove the first footnote of `intranet://annie/law1/chapter2.html`".
Both Annie’s and Bob’s reading system can fetch that resource
by mapping the canonical URL to the local PWP.

## Caveats

Things to remember, discussed on the calls

* The content of a resource must not be forced to change when the state change.
* What happens with updating PWPs (e.g., books with stock exchange information),
  given there are two types:
    * PWPs whose actual content (e.g., html code) change when they get updated.
    * PWPs that have widgets that fetch the latest information when the PWP is opened.
* Packed PWPs could have different packaging formats, e.g., `tar`, `zip`, etc.
