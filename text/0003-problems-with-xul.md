# XUL problems

XUL is often mentioned as a source of problems for developers. This document
aims to list the different kinds of problems that exist.

## XUL ownership

XUL is largely unowned. While there are developers who know and understand the
XUL code they are few and mostly dedicated to other projects at present.

## XUL bugs

XUL contains bugs. This isn’t due to anything particularly special about XUL but
when we encounter bugs we frequently choose to not fix them, instead trying to
find workarounds in the UI instead. See [bug 354527](https://bugzilla.mozilla.org/show_bug.cgi?id=354527)
for example.

## No new features

There is no new feature implementation going on in XUL compared to HTML which
receives constant improvements to meet the demands of the web.

## Performance issues

It is thought that XUL’s performance is worse than that of HTML where we have spent
large amounts of time optimising for the web but little amount of time optimising
for XUL.

## Availability of example code and help

XUL is a proprietary technology developed by Mozilla and only used by Mozilla. When
developers want to find solutions to problems or implement new features the only
example code and documentation they have available is our code and our documentation.

## Learning curve

Related, when new developers join Mozilla it is almost certain they will not have
heard of or used XUL before and so there is a learning curve to understanding and
becoming proficient in the language.

## Recruiting challenge

Even if the learning curve is small relative to the productive output of an engineer
who has learned the technology, recruiting for a role that involves using a
proprietary language is more difficult than recruiting for a role that involves using
(and learning, if needed) a popular language. This applies to both new contributors
as well as employees.
Besides the cost of learning the language and the uncertainty about whether the work
will be enjoyable, which may dissuade some candidates, the more important factor is the
opportunity cost for career development relative to developing expertise in a popular
language: engineers with “XUL experience” don’t benefit from it when looking for their
next job (or even position within the company).

## Frameworks

Often developers want to use popular JS frameworks like jQuery and React to develop new
features. Commonly these frameworks are built for HTML and don’t work correctly in XUL
documents.

## SVG and HTML in XUL

Because XUL is missing certain features often developers want to embed HTML or SVG
elements inside of XUL. Because of differences in the box model this sometimes leads
to layout not working correctly.

## Gecko Complexity

XUL increases the complexity of Gecko generally, which imposes costs on Gecko
development and maintenance (such as the Stylo work).
