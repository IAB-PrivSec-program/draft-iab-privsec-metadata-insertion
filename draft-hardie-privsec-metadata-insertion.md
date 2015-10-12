---
title: Design considerations for Metadata Insertion
abbrev: privsec-metadata-insertion 
docname: draft-hardie-privsec-metadata-insertion-latest
date: 2015-1--13
category: info

ipr: trust200902
area: General
workgroup: 
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: T. Hardie
    name: Ted Hardie
    role: editor
    email: ted.ietf@gmail.com

normative:
  RFC2119:
  RFC4949:
  RFC6973:
  RFC7258:
  RFC7624:

informative:

  RFC4301:
  RFC6962:
  RFC5750:
  RFC6698:
  RFC4306:
  RFC5246:
  RFC5389:
  RFC2015:
  rFC7239:
  I-D.ietf-dnsop-edns-client-subnet:
  STRINT:
    target: https://www.w3.org/2014/strint/draft-iab-strint-report.html
    title:  Strint Workshop Report
    author:
      name: Stephen Farrell
      ins: S Farrell
    date: 2014-04-06




--- abstract

The IAB has published {{RFC7624}} in
response to several revelations of pervasive attack on Internet
communications.  In this document we consider the implications of
protocol designs which associate metadata with encrypted flows.  
In particular, we assert that designs which do so by explicit 
actions of the end system are preferable to  designs in which 
middleboxes insert them.  

--- middle

Introduction        {#intro}
============

To ensure that the Internet can be trusted by users, it is necessary
for the Internet technical community to address the vulnerabilities
exploited in the attacks document in {{RFC7258}} and the threats
described in {{RFC7624}}.  The goal of
this document is to address a common design pattern which emerges from 
the increase in encryption:  explicit association of metadata which 
would previously have been inferred from the plaintext protocol.


# Terminology {#terminology}

This document makes extensive use of standard security and privacy
terminology; see {{RFC4949}} and {{RFC6973}}. Terms used from
{{RFC6973}} include Eavesdropper, Observer, Initiator, Intermediary,
Recipient, Attack (in a privacy context), Correlation, Fingerprint,
Traffic Analysis, and Identifiability (and related terms). In
addition, we use a few terms that are specific to the attacks
discussed in this document. Note especially that "passive" and "active" below
do not refer to the effort used to mount the attack; a "passive attack"
is any attack that accesses a flow but does not modify it, while an
"active attack" is any attack that modifies a flow.  Some passive attacks
involve active interception and modifications of devices, rather than simple
access to the medium.  The introduced terms are:

Pervasive Attack:
: An attack on Internet communications that makes
use of access at a large number of points in the network, or otherwise
provides the attacker with access to a large amount of Internet
traffic; see {{RFC7258}}.

Passive Pervasive Attack:  
: An eavesdropping attack undertaken by a pervasive attacker, in which the
packets in a traffic stream between two endpoints are intercepted, but
in which the attacker does not modify the packets in the traffic
stream between two endpoints, modify the treatment of packets in the
traffic stream (e.g. delay, routing), or add or remove packets in the
traffic stream. Passive pervasive attacks are undetectable from the
endpoints.  Equivalent to passive wiretapping as defined in {{RFC4949}};
we use an alternate term here since the methods employed are wider
than those implied by the word "wiretapping", including the active
compromise of intermediate systems.

Active Pervasive Attack:
: An attack undertaken by a pervasive attacker, which in addition to
the elements of a passive pervasive attack, also includes modification,
addition, or removal of
packets in a traffic stream, or modification of treatment of packets
in the traffic stream. Active pervasive attacks provide more
capabilities to the attacker at the risk of possible detection at the
endpoints. Equivalent to active wiretapping as defined in {{RFC4949}}.

Observation:
: Information collected directly from communications by an
eavesdropper or observer. For example, the knowledge that
&lt;alice@example.com&gt; sent a message to &lt;bob@example.com&gt;
via SMTP taken from the headers of an observed SMTP message would be
an observation.

Inference:
: Information derived from analysis of information collected
directly from communications by an eavesdropper or observer. For
example, the knowledge that a given web page was accessed by a given
IP address, by comparing the size in octets of measured network flow
records to fingerprints derived from known sizes of linked resources
on the web servers involved, would be an inference.

Collaborator:
: An entity that is a legitimate participant in a communication, and provides information about that communication to an attacker. Collaborators may either deliberately or unwittingly cooperate with the attacker, in the latter case because the attacker has subverted the collaborator through technical, social, or other means.

Key Exfiltration:
: The transmission of cryptographic keying material for an encrypted communication
from a collaborator, deliberately or unwittingly, to an attacker.

Content Exfiltration:
: The transmission of the content of a communication from a collaborator, deliberately or unwittingly, to an attacker.

Data Minimization: 
With respect to protocol design, refers to the practice of only exposing the minimum amount of data or metadata necessary for the task supported by that protocol to the other endpoint(s) and/or devices along the path.

Design patterns	{#patterns}
=====================

Given the threat model laid out in {{RFC7624}}, how should protocol designs handle the loss of metadata derived from cleartext?

One of the core mitigations for the loss of confidentiality is data minimization, which limits the amount of data disclosed to those elements absolutely required to complete the relevant protocol exchange.  When data minimization is in effect, some information which was previously available may be removed from specific protocol exchanges.  

In some cases, other actors within a protocol context will continue to have access to the information which has been withdrawn from specific protocol exchanges.  If those actors attach the information as metadata to those protocol elements, the confidentiality effect of data minimization is lost.  While it is tempting to restore previous information flows, this design pattern should be avoided, as it contributes to the overall loss of confidentiality for the Internet.  

The restoration of information is particularly tempting for systems whose primary function is not to provide confidentiality.  A proxy providing compression, for example, may wish to restore the identity of the requesting party; similarly a VPN system used to provide channel security may believe that origin IP should be restored.   Actors considering restoring metadata may believe that they understand the relevant privacy considerations or believe that, because the primary purpose of the service was not privacy-related, none exist.  Examples of this design pattern include {{RFC7239}} and {{I-D.ietf-dnsop-edns-client-subnet}}.    

This document recommends against restoration in these cases unless a positive affirmation of approval for restoration has been received from the actor whose data will be added.  In general, we recommend that the actor add such metadata themselves so that it flows end-to-end, rather than requiring the action of other parties.  Where this is not possible, opt-in methods for consent are strongly recommended; opt-out systems, especially for previously deployed systems, may provide sufficient targeting that the most vulnerable users would be reluctant to employ them.



Deployment considerations {#deployment}
=========================

There are two common tensions associated with the deployment of systems which restore metadata.  The first is the trade-off in speed of deployment for different actors.   The "Forward-for" method cited above provides an example of this.  When used with a proxy, Forwarded-for restores the original identity of the requesting party, thus allowing a responding server to tailor responses according to the original party's region, network, or other characteristics associated with the identity.  It would, of course, be possible for the origin to add this data itself, using STUN {{RFC5389}} or a similar mechanism to first determine the identity to declare.  This would require, however, full specification and adoption of this mechanism by the end systems.  It would not be available at all during this period, and would thereafter be limited to those systems which have been upgraded to include it.  The long tail of browser deployments indicates that many systems might go without upgrades for a significant period of time.  The proxy infrastructure, in contrast, is commonly under more active management and represents a much smaller number of elements; this impacts both the general deployment difficult and the number of systems which the origin server must trust.  

The second common tension is between the metadata minimization and the desire to tailor content responses.  For origin servers whose content is common across users, the loss of metadata may have limited impact on the system's functioning.  For other systems, which commonly tailor content by region or network, the loss of metadata may imply a loss of functionality.  Where the user desires this functionality, restoration can commonly be achieved by the use of other identifiers or login procedures.  Where the user does not desire this functionality, but it is a preference of the server or a third party, this is more difficult.  At the extreme, content blocking by network origin may be a regulatory requirement.  Trusting a network intermediary to provide accurate data is, of course, fragile in this case, but it may be a part of the regulatory framework.

These tensions do not change the basic recommendation, but they suggest that the parties who are introducing encryption and data minimization for existing protocols consider carefully whether the work also implies introducing mechanisms for the end-to-end provisioning of metadata when a user has actively consented to provide it.   


IANA Considerations {#IANA}
===================

This memo makes no request of IANA.


Security Considerations {#Security}
=======================

This memorandum describes a design pattern related emerging from responses to the attacks described in {{RFC7258}}. Continued use of this design pattern lowers the impact of mitigations to that attack.


Contributors {Contributors}
============

This document is derived in part from the work initially done on  the Perpass mailing list and at the STRINT workshop. It has been discussed with the IAB's Privacy and Security program, whose review is gratefully acknowledged.

--- back


