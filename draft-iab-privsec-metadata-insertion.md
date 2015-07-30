---
title: Design considerations for Metadata Insertion
abbrev: privsec-metadata-insertion 
docname: draft-iab-privsec-metadata-insertion-latest
date: 2015-07-31
category: info

ipr: trust200902
area: General
workgroup: IAB
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
  I-D.iab-privsec-confidentiality-threat:

informative:

  RFC4301:
  RFC6962:
  RFC5750:
  RFC6698:
  RFC4306:
  RFC5246:
  RFC2015:
  I-D.ietf-dnsop-edns-client-subnet:
  STRINT:
    target: https://www.w3.org/2014/strint/draft-iab-strint-report.html
    title:  Strint Workshop Report
    author:
      name: Stephen Farrell
      ins: S Farrell
    date: 2014-04-06




--- abstract

The IAB has published {{I-D.iab-privsec-confidentiality-threat}} in
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
described in {{I-D.iab-privsec-confidentiality-threat}}.  The goal of
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

Given the threat model laid out in
{{I-D.iab-privsec-confidentiality-threat}}., how should protocol
designs handle the loss of metadata derived from cleartext?



Deployment considerations {#deployment}
=========================

One of the key considerations for protocol development is
incremental deployability. 



IANA Considerations {#IANA}
===================

This memo makes no request of IANA.


Security Considerations {#Security}
=======================

This memorandum describes a design pattern related emerging
from responses to the attacks described in {{RFC7258}}.  
Continued use of this design pattern lowers the impact
of mitigations to that attack.


Contributors {Contributors}
============

This document is derived in part from the work initially done on 
the Perpass mailing list and at the STRINT workshop. 

--- back


