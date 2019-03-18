# Redesing of the Perl 6 Documentation System

* **Student**: Antonio Gámiz Delgado [antoniogamiz10@gmail.com]
* **Mentor**: ---

---

### Description

Redesing the tools and modules used to generate documentation in Perl6. Currently pod6
files are processed by various programs and modules (`htmlify.p6`, `Pod::To::HTML`, 
`Pod::To::BigPage`, ...), that has repeated functionality, low level of testing, tight
coupling between presentation rendering and source data, etc. So, the main idea is to 
renew these tools from scratch, allowing that future changes to the build process and 
doc system can be made easily without provoking side effects.

### Deriverables

* New `Pod::To::Abstract` module
* New `Pod::To::HTML` module
* Cache implementation with `Poth::To::Cached`
* ...

### Timeline

* May 6-27: In the Community Bonding period I will learn more about Perl6 ecosystem 
(testing methods, standards and codebase structure), get to know the community itself
and discuss with my mentor.
* May 27 - June 17: 
  - `Pod::To::Abstract` class (1 week)
  - `Pod::To::HTML` module (1 week)
  - Testing and documenting (1 week)

### Implementation

First of all we need a stable system to converts pod6 files into HTML or other markup 
languages (Markdown, for instance). In addition, the rendering method chosen should not 
be tightly coupled with the pod6 files processing.

To do that, I will create a `Pod::To::Abstract` module containing a main class where the
commong logic between all the `Pod::Block` derived elements will be handled. So the class
should implement these methods:

  ~~~
  multi sub handle ( Pod::Block::Declarator $node, ... ) { ... }
  multi sub handle ( Pod::Block::Table $node, ... ) { ... }
  multi sub handle ( Pod::Block::Named $node, ... ) { ... }
  multi sub handle ( Pod::Block::Para $node, ... ) { ... }
  multi sub handle ( Pod::Heading $node, ... ) { ... }
  multi sub handle ( Pod::Item $node, ... ) { ... }
  multi sub handle ( Pod::FormattingCode $node, ... ) { ... }
  multi sub handle ( Pod::Block::Comment $node, ... ) { ... }
  multi sub handle ( Pod::Block::Code $node, ... ) { ... }
  ~~~

In order to make it language independent, these methods will render each item separately 
using `Template::Mustache` (a cross-language templating format). So custom and default 
templates need to be made and passed to this module (the default will be HTML).

Escape/unscape characters logic is also repeated, so two auxiliar functions will be added:

~~~
  sub escape-markup ( $_ ) { ... }
  sub unescape-markup ( $_ ) { ... }
~~~
