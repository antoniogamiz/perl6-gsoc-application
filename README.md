# Redesign of the Perl 6 Documentation System

- **Student**: Antonio Gámiz Delgado [antoniogamiz10@gmail.com]
- **Mentor**: ---

---

### Description

Currently, pod files are processed by various scripts and modules (`htmlify.p6`,
`Pod::To::HTML`, `Pod::To::BigPage`,...), that has repeated functionality, low
level of testing, tight coupling between presentation rendering and source data
and compiles the files several times. So, the main idea is to renew some of these
tools from scratch and integrate new modules (like `Pod::Cached`) allowing that
future changes to the build process and doc system can be made easily without
provoking side effects and reducing doc build time.

### Deriverables

- `mini-docs` repository
- Health link tool (described below)
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

All these steps are headed to reach `A new documentation system`, that some people
in the community have already started. Currently, the doc repository contains several
modules that could be independent and, in general, there is a big lack of test
coverage so I will spend a considerable part of the time to reduce this fact.

#### Minidoc repository

The first thing we need to do is a mini doc repository (issue [#2529](https://github.com/perl6/doc/issues/2529)),
mocking the current doc, called `mini-docs`, containing a subset of the actual documentation.
The purpose of this repository is to let make tests faster, using a low number of pod files
rather than the entire `doc` repository. The subset chosen will have to be self-contained so
that it can be rebuilt it with the actual tooling without errors.

#### Link Scraper

We need a link-scraper to gather all existing links in docs.perl6.org in order to know how
many links are failing and why. This scraper will be used each time an important change is
made to the main doc repo to assure that the number of broken links is lower, or at least,
constant between changes and to track several errors.

Link problems have been recurring for a long time, issues like [#561](https://github.com/perl6/doc/issues/561)
(with top priority), \#[1825](https://github.com/perl6/doc/issues/1825) (404 errors) or [#585](https://github.com/perl6/doc/issues/2529)
(doubled links). This tool will use the [checklink](https://metacpan.org/pod/distribution/W3C-LinkChecker/bin/checklink.pod)
tool to check the links health. Moreover, in order to get an informative output, I will need to
create a mini tool to generate reports about the failing links (such as classify them by error
code, link form, etc.).

In addition, we will need to save the current existing links in order to make sure that links
are not lost by future changes to the documentation system.

We can publish this tool as a health checker specialized in Perl6 Documentation page ( maybe
`Perl6::LinkHealth` would be a good name).

#### doc/lib/\* Spinning-off and Cache System

In the [lib folder](https://github.com/perl6/doc/tree/master/lib) there are several modules
defined that can be taken apart to independent modules in the Perl6 Ecosystem. As [#1937](https://github.com/perl6/doc/issues/2529)
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

I am currently studying Computer Science and Mathematics at the University of Granada, Spain.
I have been programming in several languages before (C++, Java, Javascript, Python, etc.) but I
had never used Perl6 so I am totally new to it!

These days I have been using Learning Perl 6 book by Brian D Foy in order to learn the basics
about the language and I have taken part in a couple of squashatons. I am even writing my
first Perl 6 module! (a simple one to be honest [+info](https://github.com/antoniogamiz/Math-ConvergenceMethods)).
