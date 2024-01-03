---
title: "Adding a Wrong Recipient URL for Handling Misdirected Emails"
abbrev: "Wrong Recipient"
category: info

docname: draft-dweekly-wrong-recipient-latest
submissiontype: independent  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "dweekly/ietf-wrong-recipient"
  latest: "https://dweekly.github.io/ietf-wrong-recipient/draft-dweekly-wrong-recipient.html"

author:
 -
    fullname: David Weekly
    organization: Capital One
    email: david@weekly.org

normative:

informative:
  RFC8058:
  RFC7489:

--- abstract

This document describes a mechanism for an email recipient to indicate that
they are not the intended recipient of an email, providing that signal back
to the originating mail server.

--- middle

# Introduction

Email recipients today have no clear option as to how to best signal to a
provider that they are not the correct recipient of an email. This is a
different issue than either an unsubscription request from a mailing list
or reporting an email as spam, since the service itself may be a valid
sender attempting to reach some user for a valid purpose, but the sender
may have inadvertently recorded the wrong email address either due to
user error or data entry error.

There is collective benefit to all parties if a service is able to detect
when an email address is incorrect for a user; the intended recipient,
the service, and the inadvertent recipient all prefer correct delivery.

Consequently, there ought be a mechanism whereby a service can indicate
it has an endpoint to indicate a "wrong recipient" of an email. If this
header field is present in an email message, the user can select an option to
indicate that they are not the intended recipient.

Similar to one-click unsubscription [RFC8058], the mail service can
perform this action in the background as an HTTPS POST to the provided
URL without requiring the user's further attention to the matter.

Since it's possible the user may have a separate valid account with the
sending service, it may be important that the sender be able to tie
_which_ email was sent to the wrong recipient. For this reason, the
sender may also include an opaque blob in the header field to specify the
account ID referenced in the email; this is included in the POST.

Note that this kind of misdelivery shouldn't be possible if a service
has previously verified the user's email address.

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
SHOULD include a Wrong-Recipient header field with an HTTPS URI to which the
recipient's mail client can POST. If this header field is included, the mail
sender MUST ensure this endpoint is valid.

The sender MUST encode a mapping to the underlying account identifier
in the URI in order to allow the service to know which of their users
has an incorrect email.

## Mail Recipients

When a mail client receives an email that includes a Wrong-Recipient
header field, an option SHOULD be exposed in the user interface that allows
a recipient to indicate that the mail was intended for another user.

If the user selects this option, the mail client MUST perform an
HTTPS POST to the URI in the Wrong-Recipient header field.

## Mail Senders After Wrong Sender Notification

When a misdelivery has been indicated by a POST to the HTTPS URI, the
sender MUST make a reasonable effort to cease emails to the indicated
email address for that user account.

Any GET request to this URI MUST be ignored, since anti-spam software
may attempt a GET request to URIs mentioned in mail headers.

The sender SHOULD make a best effort to attempt to discern a correct
email address for the user account. How the sender should accomplish
this task is not part of this specification.

# Additional Requirements

The email needs at least one valid authentication identifier.  In
this version of the specification the only supported identifier type
is DKIM [RFC7489], that provides a domain-level identifier in the
content of the "d=" tag of a validated DKIM-Signature header field.

The Wrong-Recipient header field needs to be included in the "h=" tag of a
valid DKIM-Signature header field.

# Examples

Header in Email

```
Wrong-Recipient: <https://example.com/wrong-recipient?uid=12345&email=user@example.org>
```

Resulting POST request

```
POST /wrong-recipient?uid=12345&email=user@example.org HTTP/1.1
Host: example.com
```

# Security Considerations

The Wrong-Recipient header field will contain the recipient address, but
that is already exposed in other header fields like To:.

The user ID of the recipient with the sending service may be exposed
by the Wrong-Recipient URI, which may not be desired but a sender
may use an opaque blob to perform a mapping to a user ID on their
end without leaking any information to outside parties.

A bad actor with access to the user's email could maliciously
indicate the recipient was a Wrong Recipient with any services that
used this protocol, causing mail delivery and potentially account
access difficulties for the user.


# IANA Considerations

IANA will be requested to add a new entry to the "Permanent Message Header Field
Names" registry.

Header field name: Wrong-Recipient

Protocol: mail

Status: Provisional

Author/Change controller: IETF

Specification document(s): *** This document ***

Related information: none

--- back

# Acknowledgments
{:numbered="false"}

Many thanks to John Levine, Oliver Deighton, and Murray Kucherawy for their
kind and actionable feedback on the language and proposal. Thanks to
Eliot Lear for helping guide the draft to the right hands for review.
