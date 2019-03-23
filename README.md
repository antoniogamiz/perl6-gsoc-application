# Redesign of the Perl 6 Documentation System

- **Student**: Antonio Gámiz Delgado [antoniogamiz10@gmail.com]
- **Project Idea**: [A redesign of the Perl6 Documentation System](https://github.com/perl6/doc/wiki/A-Redesign-of-the-Perl-6-Documentation-System)
---

### Abstract

Currently, pod files are processed by various scripts and modules (`htmlify.p6`,
`Pod::To::HTML`, `Pod::To::BigPage`,...), that has repeated functionality, low
level of testing, tight coupling between presentation rendering and source data
and compiles the files several times. So, the main idea is to renew some of these
tools from scratch and integrate new modules (like `Pod::Cached`) allowing that
future changes to the build process and doc system can be made easily without
provoking side effects and reducing doc build time.

### Deriverables

- `mini-docs` repository:
  - Find correct doc subset (conditions described in Minidoc repository section ).
  - Make a new repository in the Perl6 org, add the files and document it.
  - If everything is correct, close issue [#2529](https://github.com/perl6/doc/issues/2529)
- Link Health tool:
  - Functionality to scrape recursively all links in docs.perl6.org.
  - Functionality to store and classify all links in folders by status code.
  - Functionality compare links between different executions and throws appropiated warnings.
  - Document all utilities made.
  - Publish in Perl6 Ecosystem.
- Tests suite for `Perl6::Documentable`, `Perl6::Type` and `Pod::Cached`
- `Pod::Cached` support for `Perl6::Documentable`
- Successful integration of `Pod::Cached` in the doc repository.
- Pod6 files rendering by GitHub.

### Timeline

- May 6-27: In the Community Bonding period I will learn more about Perl6 ecosystem
  (testing methods, standards and code base structure), get to know the community itself
  and discuss with my mentor.
- May 27 - June 9 (2 weeks): Minidoc repository plus `Perl6::LinkHealth` development.
- June 9 - June 30 (3 weeks): Spinning off lib modules plus `Perl6::Cached` support.
- June 30 - July 14 (2 weeks): `Pod::Cached` test suite plus its integration in the main repo.
- July 14 - August 4 (3 weeks): `Pod::To::HTML` fixing plus GitHub rendering pod files (hopefully).
- August 4 - August 19 (2 weeks): accommodate delays, complete documentation, revise tests and clean up.

### Implementation

All these steps have as a target [A new documentation system](https://github.com/perl6/doc/wiki/A-Redesign-of-the-Perl-6-Documentation-System), that some people in the community have already started. Currently, the doc repository contains [several modules](https://github.com/perl6/doc/tree/master/lib) that could be independent. In addition, there is a big lack of test coverage so I will try to change that, develop new tools to improve the documentation process and upgrade
existing ones to make them faster.

#### Mini-doc repository

Currently, site-generation tools are tested by generating the entire site, logically, this needs a big time
to complete and if some test shows an error, you need to wait a long time again to check if it has been fixed.
So, in order to fix this fact, a `mini-doc` repository will be made (as discussed in [#2529](https://github.com/perl6/doc/issues/2529)).

This repository will contain a self-contained subset of the current [doc folder](https://github.com/perl6/doc/tree/master/doc). The `mini-doc` repo needs to fulfil some conditions.
It has to be:

* Big enough to cover most of the use cases.
* Small enough to be lightweight: because this repo is expected to be downloaded from the site-generating tools
  to run the tests.
* Self-contained: this means that a doc site can be generated from these files. For instance, `Mu`, `Cool`
and `Any` could be chosen.

The repo structure will be something like:

~~~
doc/
  Language/
    *.pod6
  Programs/
    *.pod6
  Type/
    *.pod6
~~~

At the beginning, this repo will be an exact clone of the current [doc repository](https://github.com/perl6/doc). 
When we find the correct  subset and we check that a doc site can be generated without problems, any tool or file 
(except \*.pod6) will be deleted.

#### Link Scraper


Link problems have been recurring for a long time, issues like [#561](https://github.com/perl6/doc/issues/561)
(with top priority), \#[1825](https://github.com/perl6/doc/issues/1825) (404 errors) or [#585](https://github.com/perl6/doc/issues/2529) (doubled links).

So We need a link-scraper to gather all existing links in docs.perl6.org in order to know how
many links are failing and why. This scraper will be used each time an important change is
made to the main doc repo to assure that the number of broken links is lower, or at least,
constant between changes and to track several errors.

I will use the [checklink](https://metacpan.org/pod/distribution/W3C-LinkChecker/bin/checklink.pod)
tool  and [Cro::HTTP](https://github.com/croservices/cro-http) to check the links health. The process will 
start with the [doc main page](https://docs.perl6.org/) and will look for new links recursively.

The output of this tool will be stored in a directory called `links`, which will have this structure: 

~~~
links/
  200/
    info.csv
  404/
    info.csv
  xxx/ # whatever http error code
    info.csv
  all/
    hh_dd_mm.csv
~~~

Each info.csv file (csv format has been chosen but support for json could be made too) will contain all links 
that have thrown the error code of its directory name. Each line of these files will be like:

~~~
link, status_code, response_message, site_where_the_link_was_found
~~~

[status_code, response_message, site_where_the_link_was_found] are stored to have some debug information about 
problematic links.

In addition, to be able to keep track of all existing links, an additional folder will be handled by this tool: 
`all`. This directory will contain csv files whose name follows the format: `time_day_month`. Each of these files
include all links found in one execution and will be compared each time a new execution finishs throwing a warning
if the number of links is lower (or greater) to check if some links have been lost.

We can publish this tool as a health checker specialized in the [Perl6 Docs](https://docs.perl6.org/) ( maybe
`Perl6::LinkHealth` would be a good name).

#### doc/lib/\* Spinning-off and Cache System

There are several modules defined in the [lib folder](https://github.com/perl6/doc/tree/master/lib) that can be taken apart to independent modules in the Perl6 Ecosystem. As [#1937](https://github.com/perl6/doc/issues/2529)
and [#2573](https://github.com/perl6/doc/issues/2529) issues say, `Perl6::Documentable` and
`Perl6::Type` need a test suite which covers most of the use cases (currently there is almost
none). Moreover, documentation about these modules does not exist, so new people that need to
change or fix something about them (like me) have to guess what they do. So a detailed
documentation will be made for them.

On top of that, `Perl6::Documentable` has to be adapted to use `Pod::Cached` to reduce the
number of pod file compilations (currently they are compiled 3 times). To reduce this number
to 1, we will integrate `Pod::Cached` in the main doc repository and it will be used by the tests
`Pod::Documentable`. In addition, a wider test suite will be made to this module.

#### `Pod::To::HTML` and GitHub

As you can see in this issue [#55](https://github.com/perl6/Pod-To-HTML/issues/55), Pod6 was
close to get rendered to HTML in GitHub, but due to the problem with pod files being compiled
has not made this possible. So, a [new parser](https://github.com/drforr/perl6-Perl6-Parser-Pure)
for Perl6 has been released, developed by Jeff Goff. This parser maybe could be used to process
pod files without executing anything in them. Hence, using this new parser, we could get GitHub
to render pod files!

So this part of the project, if time allows it, will consist on trying to incorporate the new
parser to `Pod::To::HTML` and solve several HTML generation problemas in this module (for instance,
link generation).

### About me

I am currently studying a double degree in Computer Science and Mathematics at the University of 
Granada, Spain. I have been programming in several languages before (C++, Java, Javascript, Python,
etc.) but I had never used Perl6 so I am totally new to it!

These days I have been using Learning Perl 6 book by Brian D Foy in order to learn the basics
about the language and I have taken part in a couple of squashatons. I am even writing my
first Perl 6 module! (a simple one to be honest [+info](https://github.com/antoniogamiz/Math-ConvergenceMethods)).
