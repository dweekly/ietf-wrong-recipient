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
header is present in an email message, the user can select an option to
indicate that they are not the intended recipient.

Similar to one-click unsubscription [RFC8058](https://www.ietf.org/rfc/rfc8058.txt), the mail service can
perform this action in the background as an HTTPS POST to the provided
URL without requiring the user's further attention to the matter.

Since it's possible the user may have a separate valid account with the
sending service, it may be important that the sender be able to tie
_which_ email was sent to the wrong recipient. For this reason, the
sender may also include an opaque blob in the header to specify the
account ID referenced in the email; this is included in the POST.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
