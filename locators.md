## Definitions

* **Canonical**
Canonical in the [mathematical sense](https://en.wiktionary.org/wiki/canonical):
distinguished among entities of its kind,
so that it can be picked out in a way
that does not depend on any arbitrary choices.

> These arbitrary choices are, in our case, in 2 dimensions:
(1) packed/unpacked, and
(2) difference in protocol.
See current WD.

* **Locator**
URL that points to either an entire PWP or a resource within a PWP.

* **PWP Locator**
A Locator to the PWP as a whole. This PWP _may_ be in a state.

* **Relative locator**
A locator to a resource within a PWP, without specifying the location of the PWP.

* **Absolute locator**
A combination of a PWP Locator and a relative locator.

* **Canonical locator**
a PWP Locator, that is the same for a PWP, resilient against changing states.

> The various states, that may have their distinct locators/URI-s,
contain exactly the same abstract content.

* **Reading system**
A system that is able to handle the PWPs data and metadata,
and can map canonical locators to the actual resources.
This could be the future browser, a current browser with a plugin,
a desktop application, an app, ...

## Proposal

The canonical locator of a resource of a PWP is constructed
by assuming that the PWP is published unpacked.

So, if you have a file `mona lisa.jpg`,
and that file is in subfolder `/img/`,
and a book is published under `http://book.org/books/1/pwp`,
then the canonical locator is `http://book.org/books/1/pwp/img/mona%20lisa.jpg`.
When someone downloads the book to `file://book.packed`,
and creates a locator to `mona lisa.jpg`,
the locator to that file will remain `http://book.org/books/1/pwp/img/mona%20lisa.jpg`,
but the actual location will be the file `img/mona%20lisa.jpg`
inside the package `file://book.packed`.

### Q1: So, how does a reading system go from the canonical locator to the actual locator?

Every PWP has metadata [1].
One of these fields is the canonical URL, i.e., `http://book.org/books/1/pwp` [2].
When locating a resource within a local PWP using a canonical locator,
you can use the canonical URL from the PWP’s metadata
to map from the canonical locator to the local locator.
E.g., fetching `img/mona%20lisa.jpg` within `file://book.packed`.

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
and you can keep the original canonical URL as extra metadata [3].

>This could result in a trail of breadcrumbs,
when you make changes on already changed PWPs

Smart reading systems might use these breadcrumbs to allow for indirection.
E.g., when someone adds an annotation to `http://book.org/books/1/pwp/img/mona%20lisa.jpg`,
the annotation-aware reading system sees in the metadata
that this PWP with canonical URL `http://school.org/course/10/pwp`
is derived from `http://book.org/books/1/pwp`,
so it can do a best effort to try and locate `img/mona%20lisa.jpg`
within `http://school.org/course/10/pwp`,
and show the annotation at the correct position.

### Q4: So, what if the PWP isn’t published unpacked? What do I get when I `GET http://book.org/books/1/pwp/img/mona%20lisa.jpg`?

In one way or another [4], you get the canonical URL from that GET request,
and fetch the PWP that is hosted there
(e.g., a packed PWP hosted on `http://book.org/books/1/pwp`).
Your reading system can then again map the canonical locator to the relative locator,
and fetch the resource within the package.

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
______

[1]: Where this metadata resides is to be discussed.
Inside the manifest makes sense, but including it in the html pages
might also be an option.

[2]: E.g., &lt;link rel="http://pwp.org/canonical" href="http://book.org/books/1/pwp"/&gt;

[3]: E.g., &lt;link rel="http://pwp.org/breadcrumb" href="http://book.org/books/1/pwp"/&gt;

[4]: E.g., using HTTP Link Headers, TBD.

## Caveats

Things to remember, discussed on the calls

* The content of a resource must not be forced to change when the state change.
* What happens with updating PWPs (e.g., books with stock exchange information),
  given there are two types:
    * PWPs whose actual content (e.g., html code) change when they get updated.
    * PWPs that have widgets that fetch the latest information when the PWP is opened.
* Packed PWPs could have different packaging formats, e.g., `tar`, `zip`, etc.
