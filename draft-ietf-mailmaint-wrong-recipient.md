---
title: "Adding a Wrong Recipient URL for Handling Misdirected Emails"
abbrev: "Wrong Recipient"

docname: draft-ietf-mailmaint-wrong-recipient-latest
updates: 8058
submissiontype: IETF
cat: std
number:
date:
v: 3
area: ART
workgroup: MAILMAINT
consensus: true
keyword:
 - email
venue:
  group: MAILMAINT
  type: Working Group
  mail: mailmaint@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/mailmaint/
  github: dweekly/ietf-wrong-recipient
  latest: https://dweekly.github.io/ietf-wrong-recipient/draft-ietf-mailmaint-wrong-recipient.html

author:
 -
    fullname: David Weekly
    email: david@weekly.org
    city: Redwood City
    region: CA
    country: US
 -
    fullname: John Levine
    email: standards@standcore.com
    country: US

normative:
  RFC2369:
  RFC8058:

--- abstract

This document describes a mechanism for an email recipient to indicate to a
sender that they are not the intended recipient.

--- middle

# Introduction

Many users with common names and/or short email addresses receive
transactional emails from service providers intended for others. These
emails can't be unsubscribed (as they are transactional) but neither are
they spam. These emails commonly are from a noreply@ email address; there
is no standards-based mechanism to report a "wrong recipient" to the
sender. Doing so is in the interest of all three involved parties: the
inadvertent recipient (who does not want the email), the sender (who wants
to be able to reach their customer and who does not want the liability of
transmitting PII to a third party), and the intended recipient.

This document proposes a structured mechanism for the reporting of such
misdirected email via HTTPS POST, updating
the List-Unsubscribe-Post mechanism of [RFC8058].

# Proposal

There ought be a mechanism whereby a service can indicate
it has an endpoint to indicate a "wrong recipient" of an email. If this
header field is present in an email message, the user can select an option to
indicate that they are not the intended recipient.

Updating the one-click unsubscription [RFC8058], the mail service can
perform this action in the background as an HTTPS POST to the provided
URL without requiring the user's further attention to the matter.

Since it's possible the user may have a separate valid account with the
sending service, it may be important that the sender be able to tie
_which_ email was sent to the wrong recipient. For this reason, the
sender may also include an opaque blob in the header field to specify the
account ID referenced in the email; this is included in the POST.

Note that this kind of misdelivery shouldn't be possible if a service
has previously verified the user's email address for the account.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# High-Level Goals

Allow a recipient to stop receiving emails intended for someone else.

Allow a service to discover when they have the wrong email for a user.

# Out of Scope

This document does not propose a mechanism for automatically discovering
whether a given user is the correct recipient of an email, though it is
possible to use some of the signals in an email, such as the intended
recipient name, to infer a possible mismatch between actual and intended
recipients.

# Implementation

## Mail Senders When Sending

Mail Senders that wish to be notified when a misdelivery has occurred
SHOULD include a List-Unsubscribe: header field [RFC2369] and a
List-Unsubscribe-Post: header containing
"Wrong-Recipient=One-Click".

The sender MUST encode a mapping to the underlying account identifier
in the List-Unsubscribe: URI as described in Section 3.1 of [RFC8058].

## Mail Recipients

When a mail client receives an email that includes a Wrong-Recipient
header field, an option SHOULD be exposed in the user interface that allows
a recipient to indicate that the mail was intended for another user, if
the email is reasonably assured to not be spam.

If the user selects this option, the mail client performs an
HTTPS POST to the first https URI in the List-Unsubscribe header field
as described in section 3.2 of [RFC8058].

The POST body MUST include only "Wrong-Recipient=One-Click".

## Mail Senders After Wrong Sender Notification

When a misdelivery has been indicated by a POST to the HTTPS URI or
email to the given mailto: URI, the sender MUST make a reasonable effort
to cease emails to the indicated email address for that user account.

The sender SHOULD make a best effort to attempt to discern a correct
email address for the user account, such as by using a different known
email address for that user, postal mail, text message, phone call,
app push, or presenting a notification in the user interface of the
service. How the sender should accomplish this task is not part of
this specification.

# Additional Requirements

The email needs at least one valid authentication identifier, as
described in Section 4 of [RFC8058].

# Examples

## Signed HTTPS URI

Header fields in Email:

    List-Unsubscribe: <https://example.com/wrongrecip/uid12345/siga29c83d>
    List-Unsubscribe-Post: Wrong-Recipient=One-Click

Resulting POST request:

    POST /wrongrecip/uid12345/siga29c83 HTTP/1.1
    Host: example.com
    Content-Length: 25

    Wrong-Recipient=One-Click

# Security Considerations

The considerations are similar to those in Section 6 of [RFC8058].

A bad actor with access to the user's email could maliciously
indicate the recipient was a Wrong Recipient with any services that
used this protocol, causing mail delivery and potentially account
access difficulties for the user.

# IANA Considerations

This document makes no requests to IANA.

--- back

# Acknowledgments
{:numbered="false"}

Many thanks to John Levine for helping shepherd this document as well
as Oliver Deighton and Murray Kucherawy for their kind and actionable
feedback on the language and first draft of the proposal. Thanks to
Eliot Lear for helping guide the draft to the right hands for review.
A detailed review by Jim Fenton was much appreciated and caught a number
of key issues. Many thanks to the members of IETF ART for vigorous
discussion thereof and for feedback from the MAILMAINT working group.
