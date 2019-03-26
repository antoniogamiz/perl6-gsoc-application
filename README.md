# Redesign of the Perl 6 Documentation System

- **Student**: Antonio Gámiz Delgado [antoniogamiz10@gmail.com]
- **Project Idea**: Based on [A redesign of the Perl6 Documentation System](https://github.com/perl6/doc/wiki/A-Redesign-of-the-Perl-6-Documentation-System)
---

### Abstract

Currently, pod6 files are processed by various scripts and modules (`htmlify.p6`,
`Pod::To::HTML`, `Pod::To::BigPage`,...), that have redundant functionality, low
level of testing, tight coupling between presentation rendering and source data.
Even pod6 files are compiled several times. 

What I intend to do in order to change that? I have three objectives:

  - Improve the stability of the system facing changes without provoking undesirable side effects.
  - Speed up the build process to be able to make changes faster.
  - Put together a lot of work in the docs done by several contributors during the last few years.

### Deriverables

- `mini-docs` repository:
  - Find correct doc subset (conditions described in Minidoc repository section ).
  - Make a new repository in the Perl6 org, add the files and document it.
  - If everything is correct, close issue [#2529](https://github.com/perl6/doc/issues/2529)
- Link Health tool:
  - Functionality to scrape recursively all links in docs.perl6.org.
  - Functionality to store and classify all links in folders by status code.
  - Functionality to compare links between different executions and throw appropiated warnings.
  - Document all utilities made.
  - Publish in Perl6 Ecosystem.
- doc/lib/\* Spinning-off and Cache System:
  - New independent `Perl6::Documentable` and ``Perl6::Type`` module documented and tested using `mini-doc`.
  - `Perl6::Documentable::Registry`:
    - in-memory-cache support.
    - Dependency tree.
    - Documentation plus tests.
- `Pod::To::HTML` and GitHub:
  - Error fixing in `Pod::To::HTML`.
  - Use templates instead of hardcoded html code.
  - Functionality to read pod6 files using `Perl6::Parser`.

### Timeline

- May 6-27: In the Community Bonding period I will learn more about Perl6 ecosystem
  (testing methods, standards and code base structure), get to know the community itself
  and maybe I will try to make some progress because the first two weeks of June I have university tests. I will compensate this time in the rest of the coding period.
- May 27 - June 9 (2 weeks): Minidoc repository plus `Perl6::LinkHealth` development.
- June 9 - June 30 (4 weeks): Spinning off lib modules plus `Perl6::Documentable::Registry` in-memory-cache support.
- June 30 - July 14 (1 weeks): `Pod::Cached` test suite plus its integration in the main repo.
- July 14 - August 4 (3 weeks): `Pod::To::HTML` fixing plus GitHub rendering pod files (hopefully).
- August 4 - August 19 (2 weeks): accommodate delays, complete documentation, revise tests and clean up.

**Note:** you will be able to follow at all times the progress of the project in this repository as I'll be writing and keeping a journal with everything I do. It will be updated every two or three days.

### Implementation

All these steps have as a target [A new documentation system](https://github.com/perl6/doc/wiki/A-Redesign-of-the-Perl-6-Documentation-System), that some people in the community have already started. Currently, the doc repository contains [several modules](https://github.com/perl6/doc/tree/master/lib) that could be independent. In addition, there is a huge lack of test coverage so I will try to change that, develop new tools to improve the documentation process and upgrade
existing ones to make them faster.

#### Mini-doc repository

Currently, site-generation tools are tested by generating the entire site; hence, it takes a great amount of time
to complete and if some test shows an error, you need to furtherly wait so as to check whether or not it has been fixed.
Thus, a `mini-doc` repository will be made (as discussed in [#2529](https://github.com/perl6/doc/issues/2529)).

This repository will contain a self-contained subset of the current [doc folder](https://github.com/perl6/doc/tree/master/doc). The `mini-doc` repo needs to fulfil some conditions.
It has to be:

- Big enough to cover most of the use cases.
- Small enough to be lightweight: because this repo is expected to be downloaded from the site-generating tools
  to run the tests.
- Self-contained: this means that a doc site can be generated from these files. For instance, `Mu`, `Cool`
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
When we find the correct  subset and we check that a doc site can be generated without any problems, every tool or file 
(but \*.pod6) will be deleted.

#### Link Scraper

Link problems have been recurrent for a long time, issues like [#561](https://github.com/perl6/doc/issues/561)
(with top priority), \#[1825](https://github.com/perl6/doc/issues/1825) (404 errors) or [#585](https://github.com/perl6/doc/issues/2529) (doubled links). As resulf of that, we need a link-scraper to gather all existing links in docs.perl6.org in order to know how many links are failing and why. This scraper will be used each time an important change is made to the main doc repo to make sure that the number of broken links is lower, or at least,
constant between changes and to track several errors.

I will use the [checklink](https://metacpan.org/pod/distribution/W3C-LinkChecker/bin/checklink.pod)
tool  and [Cro::HTTP](https://github.com/croservices/cro-http) to check the links health. The process will 
start with the [doc main page](https://docs.perl6.org/) and will look for new links recursively.

The output of this tool will be stored in a directory called `links`, which will have the following structure: 

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

Each info.csv file (csv format has been chosen but support for json could be considered) will contain all links 
that have thrown the error code of its directory name. Each line of these files will be like:

~~~
link, status_code, response_message, site_where_the_link_was_found
~~~

[status_code, response_message, site_where_the_link_was_found] are stored to have some debug information about 
problematic links.

In addition, with the idea of keeping track of all existing links, an extra folder will be handled by this tool: 
`all`. This directory will contain csv files whose name follows the format: `time_day_month`. Each of these files
include all the links found in one execution and will be compared each time a new execution finishes throwing a warning
if the number of links is lower (or greater) than before to check if some links have been lost.

We can publish this tool as a health checker specialized in the [Perl6 Docs](https://docs.perl6.org/) ( maybe
`Perl6::LinkHealth` would be a good name).

#### doc/lib/\* Spinning-off and Cache System

Right now, there are several modules defined in the [lib folder](https://github.com/perl6/doc/tree/master/lib) that can be taken apart to independent modules in the Perl6 Ecosystem. As [#1937](https://github.com/perl6/doc/issues/2529)
and [#2573](https://github.com/perl6/doc/issues/2529) issues say, `Perl6::Documentable`, `Perl6::Documentable::Registry`
and `Perl6::Type` need a test suite covering most of the use cases (currently there scarcely are
any). Moreover, documentation about these modules almost does not exist, so new people that need to
change or fix something about them (like me) have to guess what to do. Hence, a detailed
documentation will be made for them.

Moreover, the current cache-system relies on precomp pod6 files, which are read and then used by the tools (in `htmlify.p6` using Registry: [line](https://github.com/perl6/doc/blob/a2254ac37c25f0f710224c1aadd1e8a1693cc193/htmlify.p6#L197)). So the thing is, why to handle precomp files, which need to be read each time they are used, instead of handle everything in memory?

So, this is the plan: 

 - Take `Perl6::Documentable` and `Perl6::Type` apart from the main repo to independent modules, document them, use `mini-doc` repository to test them faster and lastly integrate them again.
 - Take `Perl6::Documentable::Registry` apart and add in-memory-cache support, using a dependency tree to invalidate all files affected by a change and only recompile that files instead of the whole set (maybe using [Pod::Load](https://github.com/JJ/p6-pod-load)).
 - Add tests:
   - First set of tests: this will cover that each function behaves as expected.
   - Second set of tests: cheking that the dependency tree invalidates and recompiles the correct pod6 files.

If all of these steps have been made correctly, the integration of the new modules with the main repo should only be a matter of installing them and change some paths.

#### `Pod::To::HTML` and GitHub

As you can see on this issue [#55](https://github.com/perl6/Pod-To-HTML/issues/55), Pod6 was
close to get rendered to HTML in GitHub, but due to the problem with pod files being compiled
has not made this possible. So, a [new parser](https://github.com/drforr/perl6-Perl6-Parser-Pure)
for Perl6 has been released, developed by Jeff Goff. This parser could be used to process
pod files without executing anything in them. Hence, using this new parser, we could get GitHub
to render pod files!

This would mean that we will have to use `Pod::To::HTML` to render pod files, but currently, this module has some
[problems](https://github.com/perl6/Pod-To-HTML). Then, first we need to solve them and maybe start using templates ([Template::Mustache](https://github.com/softmoth/p6-Template-Mustache)) in the rendering process, as Richard Hainsworth has done in [Pod::Render](https://github.com/finanalyst/pod-render).

Finally, this is the plan for this part:
  - Fix the most important errors in `Pod::To::HTML`.
  - Start using Mustache templates.
  - Create a test suite covering each render function (to avoid current html errors present in several pages in the docs).
  - Add necessary functionality to `Pod::To::HTML` to read pod6 files using `Perl6::Parser`.

#### Closing

Eventually, in my opinion it will be necessary to update the state of the doc system and tooling in order to plan what to do next. Hence, I will write a post in the wiki section of the doc repo, explaining the work done so far.

### About me

I am currently studying a double degree in Computer Science and Mathematics at the University of 
Granada, Spain. I have been programming in several languages before (C++, Java, Javascript, Python,
etc.) but I am not so much experienced in Perl6.

These days I have been using Learning Perl 6 book by Brian D Foy so as to learn the basics
about the language and I have taken part in a couple of squashatons (you can see some of my contributions [here](https://github.com/perl6/doc/commits?author=antoniogamiz)), written a simple [math module](https://github.com/antoniogamiz/Math-ConvergenceMethods) and I'm part of the [Perl6 Weekly Challenge](https://p6weekly.wordpress.com/)!.
