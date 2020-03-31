
|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 0  | ZIP Purpose and Guidelines | Draft | Meta  | Antonio Nunez <antonio@zilliqa.com> | 2020-01-23 | 2020-03-31 

## What is a ZIP?

ZIP stands for Zilliqa Improvement Proposals. A ZIP is a design document meant for the [Zilliqa](https://www.zilliqa.com) community that lays down standards for the [Zilliqa platform's core protocol](https://github.com/Zilliqa/zilliqa). The ZIP should provide a concise technical specification of the standard's purpose and the features that it should provide. The ZIP author is responsible for building consensus within the community and documenting dissenting opinions.

## ZIP Rationale

We intend ZIPs to be the primary mechanisms for proposing new features and improvements, for collecting community technical input, and for documenting the design decisions that went into the proposal. Because the ZIPs are maintained as text files in a versioned repository, their revision history is the historical record of the proposal.

For implementers, ZIPs are a convenient way to track the progress of their implementation. Ideally each implementation maintainer would list the ZIPs that they have implemented. This will give end users a convenient way to know the current status of a given implementation.

A ZIP must meet certain minimum criteria. It must be a clear and complete description of the proposed standard. The proposed implementation, if applicable, must be solid and complete. The proposal must use the C++ programming language according to the current implementation of the platform.

## ZIP Work Flow

### Shepherding a ZIP

Parties involved in the process are you, the champion or *ZIP author*, the [*ZIP editors*](#zip-editors), and the [*Zilliqa Core Developers*](https://github.com/orgs/Zilliqa/people).

Before you begin writing a formal ZIP, you should vet your idea. Ask the Zilliqa community first if an idea is original to avoid wasting time on something that will be be rejected based on prior research. It is thus recommended to use one of the groups on the [Zilliqa Discord Channel](https://discord.gg/XMRE9tt),  or [the Issues section of this repository](https://github.com/Zilliqa/ZIP/issues). 

In addition to making sure your idea is original, it will be your role as the author to make your idea clear to reviewers and interested parties, as well as inviting editors, developers and community to give feedback on the aforementioned channels. You should try and gauge whether the interest in your ZIP is commensurate with both the work involved in implementing it and how many parties will have to conform to it.  Negative community feedback will be taken into consideration and may prevent your ZIP from moving past the Draft stage.

*In short, your role as the champion is to write the ZIP using the style and format described below, shepherd the discussions in the appropriate forums, and build community consensus around the idea.* 

### ZIP Process 

Following is the process that a successful ZIP will move along:

```
[ DRAFT ] -> [ READY ] -> [ APPROVED ] -> [ IMPLEMENTED ]
```

Each status change is requested by the ZIP author and reviewed by the ZIP editors. Use a pull request to update the status. Please include a link to where people should continue discussing your ZIP. The ZIP editors will process these requests as per the conditions below.

* **DRAFT** -- Once the champion has asked the Zilliqa community whether an idea has any chance of support, they will write a draft ZIP as a pull request. Consider including an implementation if this will aid people in studying the ZIP. A draft is a preliminary version of the ZIP that is not yet ready for final submission. Once the first draft has been merged, you may submit follow-up pull requests with further changes to your draft until such point as you believe the ZIP to be mature and ready to proceed to the next status.
  * :arrow_right: Ready -- If agreeable, ZIP editor will assign the ZIP a number (generally the issue or PR number related to the ZIP) and merge your pull request. The ZIP editor will not unreasonably deny a ZIP.
  * :x: Ready -- Reasons for denying ready status include being too unfocused, too broad, duplication of effort, being technically unsound, not providing proper motivation or addressing backwards compatibility, etc.
* **READY** -- This status means that the ZIP is ready for review by a wide audience.
  * :arrow_right: Approved -- If agreeable, the ZIP editor will assign Approved and set a review end date usually two weeks.
  * :x: Approved -- A request for Approved will be denied if material changes are still expected to be made to the draft. 
* **APPROVED** -- It signals a finalized version of the ZIP that has been in the Ready state for at least 2 weeks and any technical changes that were requested have been addressed by the author.
  * :x:  Implemented which results in material changes or substantial unaddressed technical complaints will cause the ZIP to revert to Draft.
  * :arrow_right: Implemented -- a finalized version of the ZIP that the Core Devs have decided to implement and release.

* **IMPLEMENTED** -- This ZIP represents the current state-of-the-art. An Implemented ZIP should only be updated to correct errata.

## What belongs in a successful ZIP?

Each ZIP should have the following parts:

- Preamble - RFC 822 style headers containing metadata about the ZIP, including the ZIP number, a short descriptive title (limited to a maximum of 44 characters), and the author details. 
- Abstract - A short (~200 word) description of the technical issue (contract or library standard) being addressed.
- Motivation (*optional) - It should clearly explain why the proposal is important. ZIP submissions without sufficient motivation may be rejected outright.
- Specification - The technical specification should describe the new feature or improvement to the core protocol in sufficient detail.
- Rationale - The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other platforms. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.
- Backwards Compatibility - All ZIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The ZIP must explain how the author proposes to deal with these incompatibilities. ZIP submissions without a sufficient backwards compatibility treatise may be rejected outright.
- Test Cases - Test cases for an implementation are mandatory.
- Implementations - The implementations need not be completed before the ZIP is merged as draft. Once implemented, the champion may provide pull requests to link to the implementation. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of “rough consensus and running code” is still useful when it comes to resolving many discussions of specification details.
- Copyright Waiver - All ZIPs must be in the public domain. See the bottom of this ZIP for an example copyright waiver.

## ZIP Formats and Templates

ZIPs should be written in [markdown](https://en.wikipedia.org/wiki/Markdown) format.
Image files should be included in a subdirectory of the `assets` folder for that ZIP as follows: `assets/zip-N` (where **N** is to be replaced with the ZIP number). When linking to an image in the ZIP, use relative links such as `../assets/zip-1/image.png`.

## ZIP Header Preamble

Each ZIP must begin with an [RFC 822](https://www.ietf.org/rfc/rfc822.txt) style header preamble, preceded and followed by three hyphens (`---`). This header is also termed ["front matter" by Jekyll](https://jekyllrb.com/docs/front-matter/). The headers must appear in the following order. Headers marked with "*" are optional and are described below. All other headers are required.

` zip:` *ZIP number* (this is determined by the ZIP editor)

` title:` *ZIP title*

` author:` *a list of the author's or authors' name(s) and/or username(s), or name(s) and email(s). *

` * discussions-to:` *a url pointing to the official discussion thread*

` status:` *Draft | Ready | Approved| Implemented*

`* review-period-end:` *date review period ends*

` type:` *Standards Track | Meta*

` created:` *date created on*

` * updated:` *comma separated list of dates*

` * requires:` *ZIP number(s)*

` * replaces:` *ZIP number(s)*

` * superseded-by:` *ZIP number(s)*

` * resolution:` *a url pointing to the resolution of this ZIP*

Headers that permit lists must separate elements with commas.

Headers requiring dates will always do so in the format of ISO 8601 (yyyy-mm-dd).

#### `author` header

The `author` header optionally lists the names, email addresses or usernames of the authors/owners of the ZIP. Those who prefer anonymity may use a username only, or a first name and a username. The format of the author header value must be:

> Random J. User &lt;address@dom.ain&gt;

or

> Random J. User (@username)

if the email address or GitHub username is included, and

> Random J. User

if the email address is not given.

#### `resolution` header

The `resolution` header can be used to point to a URL where the pronouncement about the ZIP is made.

#### `discussions-to` header

While a ZIP is a draft, a `discussions-to` header will indicate the mailing list or URL where the ZIP is being discussed. As mentioned above, examples for places to discuss your ZIP include [Zilliqa Discord Channel](https://discord.gg/8tpGXrB), an issue in this repo or in a fork of this repo.

No `discussions-to` header is necessary if the ZIP is being discussed privately with the author.

As a single exception, `discussions-to` cannot point to GitHub pull requests.

#### `type` header

The `type` header specifies the type of ZIP: Standards Track or Meta. 

#### `created` header

The `created` header records the date that the ZIP was assigned a number. Both headers should be in yyyy-mm-dd format, e.g. 2001-08-14.

#### `updated` header

The `updated` header records the date(s) when the ZIP was updated with "substantial" changes. 

#### `requires` header

ZIPs may have a `requires` header, indicating the ZIP numbers that this ZIP depends on.

#### `superseded-by` and `replaces` headers

ZIPs may also have a `superseded-by` header indicating that a ZIP has been rendered obsolete by a later document; the value is the number of the ZIP that replaces the current document. The newer ZIP must have a `replaces` header containing the number of the ZIP that it rendered obsolete.

## Auxiliary Files

ZIPs may include auxiliary files such as diagrams. Such files must be named ZIP-XXXX-Y.ext, where “XXXX” is the ZIP number, “Y” is a serial number (starting at 1), and “ext” is replaced by the actual file extension (e.g. “png”).

## Transferring ZIP Ownership

It occasionally becomes necessary to transfer ownership of ZIPs to a new champion. In general, we'd like to retain the original author as a co-author of the transferred ZIP, but that's really up to the original author. A good reason to transfer ownership is because the original author no longer has the time or interest in updating it or following through with the ZIP process, or has fallen off the face of the 'net (i.e. is unreachable or isn't responding to email). A bad reason to transfer ownership is because you don't agree with the direction of the ZIP. We try to build consensus around a ZIP, but if that's not possible, you can always submit a competing ZIP.

If you are interested in assuming ownership of a ZIP, send a message asking to take over, addressed to both the original author and the ZIP editor. If the original author doesn't respond to email in a timely manner, the ZIP editor will make a unilateral decision (it's not like such decisions can't be reversed :)).

## ZIP Editors

The current ZIP editors are

- Amrit Kumar ([**@AmritKumar**](https://github.com/AmritKumar))

- Antonio Nunez ([**@ansnunez**](https://github.com/ansnunez))

- Clark Yang ([**@ckyang**](https://github.com/ckyang))

- Haichuan Liu ([**@kaikawaliu**](https://github.com/kaikawaliu)).

- Han Wen Chua ([**@evesnow91**](https://github.com/evesnow91))

- Jun Hao Tan ([**@bb111189**](https://github.com/bb111189))

- Kaustubh Shamshery  ([**@KaustubhShamshery**](https://github.com/KaustubhShamshery))

- Sandip Bhoir ([**@sandipbhoir**](https://github.com/sandipbhoir))

- Vaivaswatha Nagaraj ([**@vaivaswatha**](https://github.com/vaivaswatha))


## ZIP Editor Responsibilities

For each new ZIP that comes in, an editor does the following:

- Read the ZIP to check if it is ready: sound and complete. The ideas must make technical sense, even if they don't seem likely to get to final status.
- The title should accurately describe the content.
- Check the ZIP for language (spelling, grammar, sentence structure, etc.), markup (GitHub Flavored Markdown), code style

If the ZIP isn't ready, the editor will send it back to the author for revision, with specific instructions.

Once the ZIP is ready for the repository, the ZIP editor will:

- Assign a ZIP number (generally the PR number or, if preferred by the author, the Issue # if there was discussion in the Issues section of this repository about this ZIP)

- Merge the corresponding pull request

- Send a message back to the ZIP author with the next step.

Many ZIPs are written and maintained by developers with write access to the Zilliqa codebase. The ZIP editors monitor ZIP changes, and correct any structure, grammar, spelling, or markup mistakes we see.

The editors don't pass judgment on ZIPs. We merely do the administrative & editorial part.

## History

This document was derived heavily from [Ethereum's EIP-1](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1.md) which in turn was derived from [Bitcoin's BIP-0001](https://github.com/bitcoin/bips/blob/master/bip-0001.mediawiki) written by Amir Taaki which in turn was derived from [Python's PEP-0001](https://www.python.org/dev/peps/pep-0001/). In many places text was simply copied and modified. Although the PEP-0001 text was written by Barry Warsaw, Jeremy Hylton, and David Goodger, they are not responsible for its use in the Zilliqa Improvement Proposals, and should not be bothered with technical questions specific to Zilliqa or the ZIP. Please direct all comments to the ZIP editors.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
