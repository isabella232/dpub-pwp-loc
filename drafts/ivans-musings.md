These are just my personal notes for now, i.e., more for further discussion. They have been triggered by the [writeup of Ben](https://github.com/w3c/dpub-pwp-loc/blob/gh-pages/locators.md) and the [subsequent discussion on the telco](https://www.w3.org/2016/02/10-dpub-loc-minutes.html). I will rely, whenever necessary, on the terminology described in the current writeup.


## The problem

Let us clarify what the problem we try to solve... (The URI-s below are slightly different than the ones in the [writeup](https://github.com/w3c/dpub-pwp-loc/blob/gh-pages/locators.md) to emphasize the differences.)

Let us suppose we have:

* A (conceptual) PWP (denoted by **P**) that includes the resource for an image of the Mona Lisa.
* **P** has a set of metadata (or “manifest”) associated to it. That set will be referred to by **M**. (The format and syntax is to be defined later.)
* **P** is published on the server **S**, and it is published in *at least one* of the different states (“at least one” insofar as there is no requirement that the publisher publishes all these states, it may choose to publishing only one):
   * In an "unpacked" state, i.e., as a hierarchy of files on the server. The "top level" of this unpacked state is available through the URL `http://book.org/books/1/`: this is the (absolute) locator of **P** *in this state*. Let us refer to this as **Lu**.
     * In this setup, the image of the Mona Lisa has the URL `http://book.org/books/1/img/mona_lisa.jpg`. This is the absolute locator of the image *in this state*.  
   * In a "packed" state, namely as part of a, say, zip file (but with a PWP-specific media type an extension). This is available through the URL `http://book.org/packed-books/1/package.pwp`: this is the (absolute) locator of **P** *in this state*. Let us refer to this as **Lp**.
     * (For the sake of simplicity we can assume that the zip file reflects, internally, the same file structure as the unpacked state. If not, some of the translations in the **M** may become more complicated.)
   * There may be other states, essentially similarly packed states but using a different archiving technology (e.g., tar.gz). Their treatments are similar; we will not consider them in what follows because it does not change the various considerations.
* The PWP is assigned a separate, “canonical”  URL `http://book.org/published-books/1`. This is an *(absolute) locator*, as opposed to an abstract identifier, i.e., it can be dereferenced via an HTTP invocation directly and should return some information. Note that, as the example shows, the exact URL string may be structurally *different* from both **Lu** and **Lp** (i.e., it is not necessarily a substring of one or the other). This URL is the *Canonical Locator of* **P**; let us refer to this thereinafter as **L**.
	* Giving a priority to the unpacked state in terms of (file) structure the *Canonical Locator* of the Mona Lisa image is `http://book.org/published-books/1/mona_lisa.jpg`   


The goal of the PWP is that (quoting from the PWP draft): 

>  “In this vision, the current format- and workflow-level separation between offline/portable and online (Web) document publishing is diminished to zero. These are merely two dynamic manifestations of the same publication: content authored with online use as the primary mode can easily be saved by the user for offline reading in portable document form.”

The translation, in practical terms, is that, whenever possible, the absolute locator, i.e., **L**  should be used when referring to **P** in, e.g., annotations. This is also true URL-s derived from **L** like `http://book.org/published-books/1/img/mona_lisa.jpg`. 

## Reminders

Some things we may already have an agreement on:

* **M** contains a reference to **L**, and **M** is also available in all the different states of **P**.
* It is to be expected that a client will need to include/use a PWP Processor of some sort. 
  * E.g., it is a specific reading app, containing both the PWP Processor and the Reading system; it can be an extension/script of some sort in a browser whereby the script implements the PWP Processor and the Reading System tries to push all rendering on the browser's core engine, etc.
* It is to be expected that some sort of server configuration _may_ be necessary, to be performed by the publishers. This may mean making some files (e.g., **M**) at disposal on the server, it _may_ mean setting some server configurations using, e.g., `.httaccess` files, etc.
* We should aim for a model, whereby the Reading System operates exclusively in terms of Canonical Locators and all mapping to and from that conceptual view of the world to specific, non-canonical locators should be done by the PWP Processor.
    * Although our discussions should be technology dependent, it is clear that this aspect of a PWP Processor, more exactly the implementation strategy thereof, resembles that of Service Workers. Ie, if Service Workers are not available, then the implementation of a PWP Processor may require to implement some of the Service Worker aspects.
    
## The questions (and attempted answers)

Two fundamental (and related) questions that we have answer:

* **Q1**: What is the answer of **S** to a `HTTP Get http://book.org/published-books/1` request?
* **Q2**: How does a PWP Processor accesses the Mona Lisa image when `http://book.org/published-books/1/img/mona_lisa.jpg` is requested by the Reading System?

### Possible answer to **Q2**

Let us make the following assumptions:

1. The PWP Process has access to the information in **M**.
2. As a consequence, **M** contains the list of available states that are available on **S**. In other words, it “knows” **Lp** and **Lu**, together with their media types. 
3. In some way or other, the PWP Processor also has a *preference* on the priorities of the different states. 
   * Whether this preference is based on some local consideration, like network speed, PWP size, etc., or is set by the user, is a different problem that will require separate considerations.)
   * Obviously, if the PWP Processor already has the cached publication, than that will prevail and there may be no HTTP request in the first place. The question really refers to the situation of a first access. 

If these assumptions are correct, then the answer to **Q2** becomes easy:

1. Based on the information on **L** (which is available in **M**) the processor can derive Relative Locator for the image, i.e., `img/mona_lisa.jpg`
2. Based on the preference and the values of **Lp** and **Lu** it can access the image:
   * If the preferred state is the packed one, then **Lp** accessed, unpacked, and the image is localized within the unpacked content (and that usually means using the relative URL as some sort of a file system path within that unpacked content)
   * If the preferred state is the unpacked one, then the locator of the image is constructed by using the value of **Lu** and the Relative Locator, yielding `http://book.org/books/1/img/mona_lisa.jpg`.
   
### Possible answer to **Q1**
   
For the previous mechanism to work, the following statement to **Q1** seems to be the essential feature:

* The answer to `HTTP Get http://book.org/published-books/1` *must* make **M** available to the PWP Processor  

There is no one, and only one, way to achieve that; a PWP Processor must be prepared to some alternatives. The final list should be part of a more precise specification for a PWP, but some of the conceivable alternatives for a complete answer to **Q1** are:

* the **M** itself (e.g., a JSON file, and RDFa+HTML file, etc., whatever is specified for the exact format and media type of **M** at some point)
* a package in some predefined PWP format that _must_ include **M**
* an HTML or an SVG file, representing, e.g., the cover page of the publication, with **M** referred to in the [`Link`](https://tools.ietf.org/html/rfc5988) header of the HTTP Response
	* in the case of HTML, the `<link>` element may also be used with, probably, a higher priority than the `Link` header 
		
Various scenarios can be set up on the server side. For example, when `http://book.org/published-books/1` is requested, 

* **S** returns **M**. Ie, the publisher publishes the metadata for all its publications at one place, referring to the various, available, states on its site.
* **S** returns the index file in HTML **Lu** with the relevant `<link>` element
  * this can be achieved, for example, if a symbolic link (in UNIX terms) has been set up on **S** pointing from `published-books/1` to `books/1`
* If the various states are published in the same directory, i.e., for example, if **Lu** is set to `http://ex.org/books/xxx/`, **Lp** is set to `http://ex.org/books/xxx.pwp` and **L** is announced as `http://ex.org/books/xxx` (note the missing trailing `/`) then **S** may have its own logic deciding which state to return with the corresponding **M**. The logic may be based on:
  * the local preferences of the publisher hardwired in the behavior of **S**
  * if the PWP Processor accompanies its request with the [`Accept`](https://tools.ietf.org/html/rfc7231#section-5.3.2) header, indicating the states that are acceptable by the server and with what priority, **S** may be set up to honor that part of the request (i.e., implementing content negotiations)
  
There may be other scenarios on the server side; it is out of scope for PWP to define those, though.
  




 


