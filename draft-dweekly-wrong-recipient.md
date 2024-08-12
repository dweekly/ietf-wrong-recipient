---
title: "Adding a Wrong Recipient URL for Handling Misdirected Emails"
abbrev: "Wrong Recipient"
category: info

docname: draft-ietf-mailmaint-wrong-recipient-latest
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
  latest: https://dweekly.github.io/ietf-wrong-recipient/draft-dweekly-wrong-recipient.html

author:
 -
    fullname: David Weekly
    email: david@weekly.org
    city: Redwood City
    region: CA
    country: US

normative:
  RFC3986:
  RFC5322:

informative:
  RFC8058:
  RFC6376:
  RFC2369:

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

This document specifies a structured mechanism for the reporting of such
misdirected email via either HTTPS POST or email inbox, directly mirroring
the List-Unsubscribe and List-Unsubscribe-Post mechanisms of [RFC2369] and
[RFC8058] respectively.

# Description

If the Wrong-Recipient header field is present in an email message,
and the message is determined to likely not be spam, the user can select
an option to indicate that they are not the intended recipient, and the
sender is notified.

Similar to one-click unsubscription [RFC8058], the mail service can
perform this action in the background as an HTTPS POST to a provided
URL without requiring the user's further attention to the matter. A
mailto: URI may also be included for non-HTTP MUAs, akin to List-Unsubscribe
from [RFC2369].

For example of when this might be used, suppose a cable TV provider enrolls
a new subscriber over the phone. The agent enrolling the customer may
accidentally enter the wrong email address for their customer. A month
later, the provider sends a bill to this email address, which may be valid
but belongs to the wrong person. The mechanism described here gives the
inadvertent recipient a mechanism to notify the provider that they need to
update their records.

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
SHOULD include a Wrong-Recipient header field with an HTTPS URI to which the
recipient's mail client can POST and/or a mailto: URI to which an email
should be sent. If this header field is included, the mail sender MUST
ensure these endpoints are valid for a period of at least one year after
sending.

The sender MUST encode a mapping to the underlying account identifier
in the URI in order to allow the service to know which of their accounts
has an incorrect email.

The URI SHOULD include an opaque identifier or another hard-to-forge
component in addition to, or instead of, the plaintext recipient email
address and user ID in order to prevent a malicious party from exercising
the endpoint on a victim's behalf. Possible examples include using a
signature parameter to the URI or UUID with a sender-local database
lookup to retrieve the email and user ID referenced.

## Mail Recipients

When a mail client receives an email that includes a Wrong-Recipient
header field, an option SHOULD be exposed in the user interface that allows
a recipient to indicate that the mail was intended for another user, if
and only if the email is reasonably assured to not be spam.

If the user selects this option, the mail client MUST perform an
HTTPS POST to the first https URI in the Wrong-Recipient header field,
or send an empty message to the first referenced mailto: address.

To minimize XSRF attacks, the POST request MUST NOT include cookies,
HTTP authorization, or any other context information. The "wrong
recipient" reporting operation is logically unrelated to any previous
web activity, and context information could inappropriately link the
report to previous activity.

The POST body MUST include only "Wrong-Recipient=true".

If the response is a HTTP 500 type error indicating server issue, the
client MAY retry. If the HTTP response to the POST is a 200, the client
MUST NOT retry. No feedback to the user as to the success or failure
of this operation is proposed or required.

## Mail Senders After Wrong Sender Notification

When a misdelivery has been indicated by a POST to the HTTPS URI or
email to the given mailto: URI, the sender MUST make a reasonable effort
to cease emails to the indicated email address for that user account.

The POST endpoint MUST NOT issue an HTTP redirect and SHOULD return a
``200 OK`` status; the content body will be ignored.

Any GET request to the same URI MUST NOT be treated as an indication
of a wrong recipient notification, since anti-spam software may attempt
a GET request to URIs mentioned in mail headers without receiving user
consent. Senders MAY return an error ``405 Method Not Allowed`` in
response to a GET request to the URI. The sender MAY elect
to present a page at this URI responsive to a GET request that
presents the user with a form that allows them to submit the POST.

The sender SHOULD make a best effort to attempt to discern a correct
email address for the user account, such as by using a different known
email address for that user, postal mail, text message, phone call,
app push, or presenting a notification in the user interface of the
service. How the sender should accomplish this task is not part of
this specification.

# Additional Requirements

The email needs at least one valid authentication identifier.  In
this version of the specification the only supported identifier type
is DKIM [RFC6376], that provides a domain-level identifier in the
content of the "d=" tag of a validated DKIM-Signature header field.

The Wrong-Recipient header field needs to be included in the "h=" tag of a
valid DKIM-Signature header field.

# Header Syntax

The following ABNF imports fields and WSP from [RFC5322] and URI from [RFC3986].
An https URI, mailto URI, or one of each are permitted. Other URI protocols
MUST NOT be used.

    fields =/ wrong-recipient

    wrong-recipient = "Wrong-Recipient:" 0*1WSP "<" URI ">"
        *(0*1WSP "," 0*1WSP "<" URI ">") 0*WSP

# Examples

## Signed HTTPS URI

Header in Email:

    Wrong-Recipient: <https://example.com/wrong-recipient?
        uid=12345&email=user@example.org&sig=a29c83d>

Resulting POST request:

    POST /wrong-recipient?uid=12345&email=user@example.org&sig=a29c83d HTTP/1.1
    Host: example.com
    Content-Length: 20

    Wrong-Recipient=true

## UUID HTTPS URI

Header in Email:

    Wrong-Recipient: <https://example.com/wrong-recipient?
        uuid=c002bd9a-e015-468f-8621-9baf6fca12aa>

Resulting POST request:

    POST /wrong-recipient?uuid=c002bd9a-e015-468f-8621-9baf6fca12aa HTTP/1.1
    Host: example.com
    Content-Length: 20

    Wrong-Recipient=true

## Combined mailto: and HTTPS URIs

Header in Email:

    Wrong-Recipient:
        <https://example.com/wrong-recipient?
            uuid=c002bd9a-e015-468f-8621-9baf6fca12aa>,
        <mailto:wrong-recipient.c002bd9a-e015-468f-8621-9baf6fca12aa
            @example.org>


# Security Considerations

The Wrong-Recipient header field may contain the recipient address, but
that is already exposed in other header fields like To.

The user ID of the recipient with the sending service may be exposed
by the Wrong-Recipient URI, which may not be desired but a sender
can instead use an opaque blob to perform a mapping to a user ID on their
end without leaking any information to outside parties, such as the UUID
examples given above.

A bad actor with access to the user's email could maliciously
indicate the recipient was a Wrong Recipient with any services that
used this protocol, causing mail delivery and potentially account
access difficulties for the user.

The Wrong-Recipient POST provides a strong hint to the mailer that
the address to which the message was sent was valid, and could in
principle be used as a way to test whether an email address is valid.
It also may expose the recipient's location and ISP via IP address.
However, unlike passive methods like embedding tracking pixels, the
mechanism proposed here takes an active user action. Nonetheless,
MUAs ought only expose this Wrong Recipient option if relatively
confident that the email is not spam.

A sender with a guessable URI structure and no use of either signed
parameters or a UUID would open themselves up to a malicious party
POST'ing email credentials for victims, potentially causing difficulty.
Senders should be strongly encouraged to use a signature or opaque
blob as suggested. No algorithm for creating such a signature or
opaque blob is included in this standard since only the sender needs
to validate the correctness of the hard-to-forge URL.

# IANA Considerations

IANA has registered a new entry to the "Provisional Message Header Field
Names" registry, to be made permanent if this proposal becomes a standard.

    Header field name: Wrong-Recipient
    Protocol: mail
    Status: Provisional
    Author/Change controller: IETF
    Specification document(s): *** This document ***
    Related information: none

--- back

# Acknowledgments
{:numbered="false"}

Many thanks to John Levine for helping shepherd this document as well
as Oliver Deighton and Murray Kucherawy for their kind and actionable
feedback on the language and first draft of the proposal. Thanks to
Eliot Lear for helping guide the draft to the right hands for review.
A detailed review by Jim Fenton was much appreciated and caught a number
of key issues. Many thanks to the members of IETF ART for vigorous
discussion thereof and for feedback from the new MAILMAINT working group
chaired by Ken Murchison.
