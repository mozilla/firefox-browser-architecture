
# Documenting our Output

## Problem

The Browser Architecture team needs to publish its findings clearly. We would like anyone in Mozilla to be able to:

* Find what we’ve learned so far
* Find what we’re working on
* Leave feedback

The solution should:

* Be low friction for contribution
* Be flexible in how the result is published
* Ensure that documents can be found easily


## Proposal

Browser Architecture team will document its output like this:

1. The discussion phase involves conversations in the browser-arch [Google Group](https://groups.google.com/a/mozilla.com/forum/#!forum/browser-arch), [IRC channel](https://www.irccloud.com/#!/ircs://irc.mozilla.org:6697/%23browser-arch), [Slack channel](https://mozilla.slack.com/messages/C5F80LV0C/), etc - when a semi-permanent record of current thinking is needed, we use GDocs, stored in [the Browser Architecture folder](https://drive.google.com/drive/u/1/folders/0BzQINYlY78CtbGVqVDVlZlNJX0k).
2. When we've come to some internal consensus (note this doesn't mean that the proposal is final) we convert our GDocs to Markdown/Git to create an 'RFC' and get feedback from management and the wider Mozilla developer community as appropriate.
  * Use [this template](0000-template.md)
3. As there are updates we keep the Markdown up to date
4. We may publish to a wider audience using a blog post or GitBook as required


## Drawbacks / Unresolved Questions

It is possible that GDocs has such a low update friction that people will resent the PR process imposed by Github. If this turns out to be true then we will need to update this process.

This process is somewhat modelled on the [Rust RFC](https://github.com/rust-lang/rfcs) process. It isn't certain that we need the numeric prefix.


## Alternatives

* Create an index to our documentation using Google Docs, and stay in that world.
* Use Medium and or RSS to publish output.


## Links to discussion

* [Mailing list discussion](https://groups.google.com/a/mozilla.com/forum/#!topic/browser-arch/FOtfYVHbgfo)
