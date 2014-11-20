---
title: Confidentiality in the Face of Pervasive Surveillance: A survey of mitigations
abbrev:privsec-mitigations 
docname: draft-iab-privsec-confidentiality-mitigations-latest
date: 2014-11-25
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
    email: ted.ietf@gmail.com

normative:
  RFC2119:
  RFC7258:
  I-D.iab-privsec-confidentiality-threat

informative:

  STRINT:
    target: https://www.w3.org/2014/strint/draft-iab-strint-report.html
    title:  Strint Workshop Report
    author:
      name: Stephen Farrell
      ins: S Farrell
    date: 2014-04-06




--- abstract

The IAB has published draft-iab-privsec-confidentiality-threat in
response to several revelations of pervasive attack on Internet
communications.  In this document we survey the mitigations to those
threats which are currently available or which might plausibly be
deployed.  We discuss these primarily in the context of Internet
protocol design, focusing on robustness to pervasive monitoring and
avoidance of unwanted cross-mitigation impacts.

--- middle

Introduction        {#problems}
============

To ensure that the Internet can be trusted by users, it is necessary
for the Internet technical community to address the vulnerabilities
exploited in the attacks document in {{RFC7258}} and the threats
described in [draft-iab-privsec-confidentiality-threats].  The goal of
this document is to describe more precisely the mitigations available
for those threats and to lay out the interactions among them should
they be deployed in combination.


Available Mitigations	{#responses}
=====================

Given the threat model laid out in
[draft-iab-privsec-confidentiality-threats]., how should the Internet
technical community respond to pervasive attack?The cost and risk
considerations discussed in it provide a guide to responses.  Namely,
responses to passive attack should close off avenues for attack that
are safe, scalable, and cheap, forcing the attacker to mount attacks
that expose it to higher cost and risk.


In this section, we discuss a collection of high-level approaches to
mitigating pervasive attacks.  These approaches are not meant to be
exhaustive, but rather to provide general guidance to protocol
designers in creating protocols that are resistant to pervasive
attack.

Protocol Definition
===================

Terminology          {#Terminology}
-----------
In this document, the key words "MUST", "MUST NOT", "REQUIRED",
"SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" are to be interpreted as described in BCP 14, RFC 2119
{{RFC2119}} and indicate requirement levels for compliant STuPiD
implementations.


Discovering External IP Address and Port
----------------------------------------

A client may discover its external IP address and the port
required for port prediction by performing a HTTP GET
request to a STuPiD server. The STuPiD server MUST reply
with the remote address and remote port in the following
format:

host ":" port

where 'host' and 'port' are defined as in {{RFC3986}}.


Storing Data
------------

Data chunks are stored using the POST request of HTTP. The
STuPiD server MUST support one URI parameter which is passed
as query-string:

'chid':  A unique ID identifying the data chunk to be stored.
The ID SHOULD be chosen from the characters of the base64url
set {{RFC4648}}.

The payload of the POST request MUST be the data to be
stored. The 'Content-Type' SHOULD be
'application/octet-stream', although a STuPiD server
implementation SHOULD simply ignore the 'Content-Type' as a
client implementation may be restricted and may not able to
specify a specific 'Content-Type'.  (E.g., in certain cases,
the peer may be limited to sending the data as
multipart-form-encoded — still, the data is stored as a
byte stream.)

STuPiD servers may reject data chunks that are larger than
some predefined limit.
This maximum size in bytes of each data chunk is RECOMMENDED
to be 65536 or more.

As HTTP already provides data transparency,
the data chunk SHOULD NOT be encoded using Base64 or any
other data transparency mechanism; in any case, the STuPiD
server will not attempt to decode the chunk.

The sender MUST wait for the HTTP response before
going on to notify the receiver.


Notification
------------

The sender notifies the receiver of the data chunk by passing
via an out-of-band channel (which is not part of the STuPiD
protocol):

The full URL from which the data chunk can be retrieved,
i.e. the same URL that was used to store the data chunk,
including the chunk ID parameter.

The exact notification mechanism over the out-of-band channel
and the definition of a session is dependent on the
out-of-band channel.  See {{xmpp}} for one
example of such an out-of-band channel.


Retrieving Data
---------------

The notified peer retrieves the data chunk using a GET request
with the URL supplied by the sender. The STuPiD server MUST
set the 'Content-Type' of the returned body to
'application/octet-stream'.


Implementation Notes
====================

A STuPiD server implementation SHOULD delete stored data some
time after it was stored. It is RECOMMENDED not to delete the
data before five minutes have elapsed after it was stored.
Different client protocols will have different reactions to
data that have been deleted prematurely and cannot be
retrieved by the notified peer; this may be as trivial as
packet loss or it may cause a reliable byte-stream to fail
({{impl}}).
(TODO: It may be useful to provide some hints in the storing
POST request.)

STuPiD clients should aggregate data in order to minimize the
number of requests to the STuPiD server per second.
The specific aggregation method chosen depends on the data
rate required (and the maximum chunk size), the latency
requirements, and the application semantics.

Clearly, it is up to the implementation to decide how the data
chunks are actually stored.  A sufficiently silly STuPiD server
implementation might for instance use a MySQL database.


Security Considerations
=======================

The security objectives of STuPiD are to be as secure as if
NAT traversal had succeeded, i.e., an on-path attacker can
overhear and fake messages, but an off-path attacker cannot.
If a higher level of security is desired, it should be
provided on top of the data relayed by STuPiD, e.g. by using
XTLS {{I-D.meyer-xmpp-e2e-encryption}}.

Much of the security of STuPiD is based on the assumption that
an off-path attacker cannot guess the chunk identifiers.  A
suitable source of randomness {{RFC4086}} should
be used to generate at least a sufficiently large part of the
chunk identifiers (e.g., the chunk identifier could be a hard
to guess prefix followed by a serial number).

To protect the STuPiD server against denial of service and
possibly some forms of theft of service, it is RECOMMENDED
that the POST side of the STuPiD server be protected by some
form of authentication such as HTTP authentication.  There is
little need to protect the GET side.

--- back


Examples  {#xmp}
========

This appendix provides some examples of the STuPiD protocol operation.

~~~~~~~~~~
   Request:

      GET /stupid.php HTTP/1.0
      User-Agent: Example/1.11.4
      Accept: */*
      Host: example.org
      Connection: Keep-Alive

   Response:

      HTTP/1.1 200 OK
      Date: Sun, 05 Jul 2009 00:30:37 GMT
      Server: Apache/2.2
      Cache-Control: no-cache, must-revalidate
      Expires: Sat, 26 Jul 1997 05:00:00 GMT
      Vary: Accept-Encoding
      Content-Length: 17
      Keep-Alive: timeout=1, max=400
      Connection: Keep-Alive
      Content-Type: application/octet-stream

      192.0.2.239:36654
~~~~~~~~~~
{: #figxmpdisco title="Discovering External IP Address and Port"}

~~~~~~~~~~
   Request:

      POST /stupid.php?chid=i781hf64-0 HTTP/1.0
      User-Agent: Example/1.11.4
      Accept: */*
      Host: example.org
      Connection: Keep-Alive
      Content-Type: application/octet-stream
      Content-Length: 11

      Hello World

   Response:

      HTTP/1.1 200 OK
      Date: Sun, 05 Jul 2009 00:20:34 GMT
      Server: Apache/2.2
      Cache-Control: no-cache, must-revalidate
      Expires: Sat, 26 Jul 1997 05:00:00 GMT
      Vary: Accept-Encoding
      Content-Length: 0
      Keep-Alive: timeout=1, max=400
      Connection: Keep-Alive
      Content-Type: application/octet-stream
~~~~~~~~~~
{: #figxmpstore title="Storing Data"}

~~~~~~~~~~
   Request:

      GET /stupid.php?chid=i781hf64-0 HTTP/1.0
      User-Agent: Example/1.11.4
      Accept: */*
      Host: example.org
      Connection: Keep-Alive

   Response:

      HTTP/1.1 200 OK
      Date: Sun, 05 Jul 2009 00:21:29 GMT
      Server: Apache/2.2
      Cache-Control: no-cache, must-revalidate
      Expires: Sat, 26 Jul 1997 05:00:00 GMT
      Vary: Accept-Encoding
      Content-Length: 11
      Keep-Alive: timeout=1, max=400
      Connection: Keep-Alive
      Content-Type: application/octet-stream

      Hello World
~~~~~~~~~~
{: #figxmpretr title="Retrieving Data"}


Sample Implementation     {#impl}
=====================

~~~~~~~~~~
<?php
header("Cache-Control: no-cache, must-revalidate");
header("Expires: Sat, 26 Jul 1997 05:00:00 GMT");
header("Content-Type: application/octet-stream");

mysql_connect(localhost, "username", "password");
mysql_select_db("stupid");

$chid = mysql_real_escape_string($_GET["chid"]);

if ($_SERVER["REQUEST_METHOD"] == "GET") {
   if (empty($chid)) {
      echo $_SERVER["REMOTE_ADDR"] . ":" . $_SERVER["REMOTE_PORT"];
   } elseif ($result = mysql_query("SELECT `data` FROM `Data` " .
                         "WHERE `chid` = '$chid'")) {
      if ($row = mysql_fetch_array($result, MYSQL_ASSOC)) {
         echo base64_decode($row["data"]);
      } else {
         header("HTTP/1.0 404 Not Found");
      }
      mysql_free_result($result);
   } else {
      header("HTTP/1.0 404 Not Found");
   }
} elseif ($_SERVER["REQUEST_METHOD"] == "POST") {
   if (empty($chid)) {
      header("HTTP/1.0 404 Not Found");
   } else {
      mysql_query("DELETE FROM `Data` " .
                  "WHERE `timestamp` < DATE_SUB(NOW(), INTERVAL 5 MINUTE)");
      $data = base64_encode(file_get_contents("php://input"));
      if (!mysql_query("INSERT INTO `Data` (`chid`, `data`) " .
                       "VALUES ('$chid', '$data')")) {
         header("HTTP/1.0 403 Bad Request");
      }
   }
} else {
   header("HTTP/1.0 405 Method Not Allowed");
   header("Allow: GET, HEAD, POST");
}
mysql_close();
?>
~~~~~~~~~~
{: #figimpl title="STuPiD Sample Implementation"}


Using XMPP as Out-Of-Band Channel  {#xmpp}
=================================

XMPP {{I-D.ietf-xmpp-3920bis}} is a good choice for
an out-of-band channel.

The notification protocol is closely modeled after XMPP's
In-Band Bytestreams (IBB, see
http://xmpp.org/extensions/xep-0047.html). Just replace the
namespace and insert the STuPiD Retrieval URI instead of the
actual Base64 encoded data, see {{figxmpnots}}.
(Note that the current proposal redundantly sends a sid and a
seq as well as the chid composed of these two; it may be
possible to optimize this, possibly sending the constant prefix
of the URI once at bytestream creation time.)

Notifications MUST be processed in the order they are
received. If an out-of-sequence notification is received for a
particular session (determined by checking the 'seq' attribute),
then this indicates that a notification has been lost. The
recipient MUST NOT process such an out-of-sequence notification,
nor any that follow it within the same session; instead, the
recipient MUST consider the session invalid.  (Adapted from
http://xmpp.org/extensions/xep-0047.html#send)

Of course, other methods can be used for setup and teardown, such as Jingle
(see http://xmpp.org/extensions/xep-0261.html).

~~~~~~~~~~
      <iq from='romeo@montague.net/orchard'
          id='jn3h8g65'
          to='juliet@capulet.com/balcony'
          type='set'>
        <open xmlns='urn:xmpp:tmp:stupid'
              block-size='65536'
              sid='i781hf64'
              stanza='iq'/>
      </iq>
~~~~~~~~~~
{: #figxmpcri title="Creating a Bytestream: Initiator requests session"}


~~~~~~~~~~
      <iq from='juliet@capulet.com/balcony'
          id='jn3h8g65'
          to='romeo@montague.net/orchard'
          type='result'/>
~~~~~~~~~~
{: #figxmpcrr title="Creating a Bytestream: Responder accepts session"}



~~~~~~~~~~
      <iq from='romeo@montague.net/orchard'
          id='kr91n475'
          to='juliet@capulet.com/balcony'
          type='set'>
        <data xmlns='urn:xmpp:tmp:stupid'
              seq='0'
              sid='i781hf64'
              url='http://example.org/stupid.php?chid=i781hf64-0'/>
      </iq>
~~~~~~~~~~
{: #figxmpnots title="Sending Notifications: Notification in an IQ stanza"}

~~~~~~~~~~
      <iq from='juliet@capulet.com/balcony'
          id='kr91n475'
          to='romeo@montague.net/orchard'
          type='result'/>
~~~~~~~~~~
{: #figxmpnota title="Sending Notifications: Acknowledging notification using IQ"}

~~~~~~~~~~
      <iq from='romeo@montague.net/orchard'
          id='us71g45j'
          to='juliet@capulet.com/balcony'
          type='set'>
        <close xmlns='urn:xmpp:tmp:stupid'
               sid='i781hf64'/>
      </iq>
~~~~~~~~~~
{: #figxmpclor title="Closing the Bytestream: Request"}

~~~~~~~~~~
      <iq from='juliet@capulet.com/balcony'
          id='us71g45j'
          to='romeo@montague.net/orchard'
          type='result'/>
~~~~~~~~~~
{: #figxmpclos title="Closing the Bytestream: Success response"}

