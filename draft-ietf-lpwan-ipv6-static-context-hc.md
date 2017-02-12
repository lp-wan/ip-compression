---
stand_alone: true
ipr: trust200902
docname: draft-ietf-lpwan-ipv6-static-context-hc-latest
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
title: LPWAN Static Context Header Compression (SCHC) for IPv6 and UDP
abbrev: LPWAN SCHC
wg: lpwan Working Group
author:
- ins: A. Minaburo
  name: Ana Minaburo
  org: Acklio
  street: 2bis rue de la Chataigneraie
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: ana@ackl.io
- ins: L. Toutain
  name: Laurent Toutain
  org: IMT-Atlantique
  street:
  - 2 rue de la Chataigneraie
  - CS 17607
  city: 35576 Cesson-Sevigne Cedex
  country: France
  email: Laurent.Toutain@imt-atlantique.fr
normative:
  rfc0768: 
  rfc4944: 
  rfc6282: 
  rfc2460: 
  rfc5795:
  I-D.minaburo-lp-wan-gap-analysis:
  I-D.ietf-lpwan-overview:

--- abstract

This document describes a header compression scheme for IPv6, IPv6/UDP based
on static contexts. This technique is especially tailored for LPWA networks
and could be extended to other protocol stacks.

The Static Context Header Compression (SCHC)  offers a great level of flexibility 
in the processing of fields.
Static context means that values in the context field do not change during
the transmission, avoiding complex resynchronization mechanisms, incompatible
with LPWA characteristics. In most of the cases, IPv6/UDP headers are reduced
to a small identifier.

This document describes the compression/decompression process and apply it 
to IPv6/UDP headers compression, Other protocols such as CoAP will be described in a
separate document.

--- middle

# Introduction {#Introduction}

Headers compression is mandatory to bring the internet protocols to the node
within a LPWA network {{I-D.minaburo-lp-wan-gap-analysis}}. 

Some LPWA networks properties can be exploited for an efficient header
compression:

* Topology is star oriented, therefore all the packets follows the same path.
  For the needs of this draft, the architecture can be summarized to End-Systems
  (ES) exchanging information with LPWAN Application Server (LA). The exchange
  goes through a LPWA Compressors (LC). Both LC maintain a static context for compression.
  Static context means that context information is not learned during the exchange.

* Traffic flows are mostly deterministic, since End-Systems embed built-in
  applications. Contrary to computers or smartphones, new applications cannot
  be easily installed.

The Static Context Header Compression (SCHC) combines the advantages of RoHC {{RFC5795}}
context, which offers a great level of flexibility in the processing of fields,
and 6LoWPAN {{RFC4944}} behavior to elide fields that are known from the other side.
Static context means that values in the context field do not change during
the transmission, avoiding complex resynchronization mechanisms, incompatible
with LPWA characteristics. In most of the cases, IPv6/UDP headers are reduced
to a small context identifier.

The SCHC is indedependant of the LPWAN technology.

# Vocabulary

* CDF: Compression Decompression Function. Function used both to compress a field or to recover its original value in the decompression phase.

* Context: A set of rules used to compress/decompress headers

* ES: End System. Node connected to the LPWAN. ES may implement SCHC.

* LA: LPWAN Application. Application sending/consuming headers to/from the End System.

* LC: LPWAN Compressor. Process in the network compression/decompressing headers. LC uses SCHC rules to perfom compression decompression.

* MO: Matching Operator. Operator used to compare a value contained in a field's header with a value contained in a rule.

* Rule: A set of header field values.

* Rule ID: An identifier for a rule, LC and ES share the same rule ID for a specific flow. Rule ID 
  is sent on the LPWAN.
  
* TV: Target value. Value contained in the rule that will be matched with the value of an header field.

# Static Context Header Compression

Static Context Header Compression (SCHC) avoids context synchronization,
which is the most bandwidth-consuming operation in RoHC. Based on the fact
that the nature of data flows is highly predictable in LPWA networks, a static
context may be stored on the End-System (ES). The other end, the LPWA Compressor
(LC) can learn the context through a provisioning protocol during the identification
phase (for instance, as it learns the encryption key).

~~~~
           End-System
      +-----------------+
      | APP1  APP2 APP3 |           
      |                 |        
      |       UDP       |       
      |      IPv6       |        
      |                 |       +-----------+         
      |      LC (contxt)|       |LC (contxt)|
      +--------+--------+       +-----+-----+ 
               |                      |
               +~ ~ RG ==== NG =======+
~~~~
{: #Fig-archi title='Architecture'}

{{Fig-archi}} based on {{I-D.ietf-lpwan-overview}} terminology represent the architecture for 
compression/decompression. The Thing or End-System is running applications which produce UDP/IPv6
flows. These flows are compressed by a LPWAN Compressor (LC) to reduce the headers size. Resulting
information is send on a frame to the LPWAN Radio Network to a Radio Gateway (RG) which forward 
the frame to a Network Gateway.
The Network Gateway sends the data to a LC for decompression. They both share the same rules. The LC can be 
located on the Network Gateway or in another places if a tunnel is established between the NG and the LC.
This architecture forms a star topology.

The context contains a list of rules (cf. {{Fig-ctxt}}). Each rule contains 
itself a list of field descriptions composed of a filed id (FID), a target
value (TV), a matching operator (MO) and a Compression/Decompression Function
(CDF).


~~~~
  +------------------------------------------------------------------+
  |                      Rule N                                      |
 +-----------------------------------------------------------------+ |
 |                    Rule i                                       | |
+----------------------------------------------------------------+ | |
|                Rule 1                                          | | |
|+--------+--------------+-------------------+-----------------+ | | |
||Field 1 | Target Value | Matching Operator | Comp/Decomp Fct | | | |
|+--------+--------------+-------------------+-----------------+ | | |
||Field 2 | Target Value | Matching Operator | Comp/Decomp Fct | | | |
|+--------+--------------+-------------------+-----------------+ | | |
||...     |    ...       | ...               | ...             | | | |
|+--------+--------------+-------------------+-----------------+ | |-+
||Field N | Target Value | Matching Operator | Comp/Decomp Fct | | |
|+--------+--------------+-------------------+-----------------+ |-+
|                                                                |
+----------------------------------------------------------------+
~~~~
{: #Fig-ctxt title='Compression Decompression Context'}


The rule does not describe the original packet format which
must be known from the compressor/decompressor. The rule just describes the
compression/decompression behavior for a field.

The main idea of the compression scheme is to send the rule number (or rule
id) to the other end instead of known field values.

Matching a field with a value and header compression are related operations;
If a field matches a rule containing the value, it is not necessary to send
it on the link. Since contexts are synchronized, reading the rule's value
is enough to reconstruct the field's value at the other end. 

On some other cases, the value need to be sent on the link to inform the
other end. The field value may vary from one packet to another, therefore
the field cannot be used to select the rule id. These values are sent in 
same order as the fields in the rule which defines the structure of the
compress header.

## Rule id

Rule id are sent between both compression/decompression element. The size 
of the rule id is not specified on this document and can vary regarding the
LPWAN technology, the number of flows,... 

Some values in the rule id space may be reserved to other goal than header 
compression, for example fragmentation. 

Rule id are specific to an ES. Two ES may use the same rule id for different
header compression. The LC needs to combine the rule id with the ES L2 address
to find the appropriate rule.

## Simple Example

A simple header is composed of 3 fields (F1, F2, F3). The compressor receives
a packet containing respectively [F1:0x00, F2:0x1230, F3:0xABC0] in those
fields. The Matching Operators (as defined in {{chap-MO}}) allow to
select Rule 5 as represented in {{Fig-ex-ctxt}}; F1 value is ignored
and F2 and F3 packet field values are matched with
those stored in the rule Target Values.

~~~~
               Rule 5
          Target Value   Matching Operator   Comp/Decomp Fct
        +--------------+-------------------+-----------------+
     F1 | 0x00         | Ignore            | not-sent        |
        +--------------+-------------------+-----------------+
     F2 | 0x1230       | Equal             | not-sent        |
        +--------------+-------------------+-----------------+
     F3 | 0xABC0       | Equal             | not-sent        |
        +--------------+-------------------+-----------------+
~~~~
{: #Fig-ex-ctxt title='Matching Rule'}


The Compression/Decompression Function (as defined in {{chap-CDF}}
describes how the fields are compressed. In this example, all the
fields are elided and only the rule number has to be sent to the other
end.

The decompressor receives the rule number and reconstructs the header using
the values stored in the Target Value column.

Note that F1 value will be set to 0x00 by the decompressor, even if the original
header field was carrying a different value.

To allow a range of values for field F2 and F3, the MSB() Matching
Operator and LSB() Compression/Decompression Function can be used (as
defined in {{chap-MO}} and {{chap-CDF}}). In that case the rule will
be rewritten as defined in {{Fig-ex-ctxt2}}.

~~~~
               Rule 5
          Target Value   Matching Operator   Comp/Decomp Fct
        +--------------+-------------------+-----------------+
     F1 | 0x00         | Ignore            | not-sent        |
        +--------------+-------------------+-----------------+
     F2 | 0x1230       | MSB(12)           | LSB(4)          |
        +--------------+-------------------+-----------------+
     F3 | 0xABC0       | MSB(12)           | LSB(4)          |
        +--------------+-------------------+-----------------+
~~~~
{: #Fig-ex-ctxt2 title='Matching Rule'}


In that case, if a packet with the following header fields [F1:0x00, F2:0x1234,
F3:0xABCD] arrives to the compressor, the new rule 5 will be selected and
sent to the other end. The compressed header will be composed of the single
byte [0x4D]. The decompressor receives the compressed header and follows
the rule to reconstruct [0x00, 0x1234, 0xABCD] applying a OR operator between
the target value stored in the rule and the compressed field value sent.


## Packet processing

The compression/decompression process follows several steps:

* compression rule selection: the goal is to identify which rule(s) will be used
  to compress the headers. To each field is associated a matching operator for
  compression. Each header field's value is compared to the corresponding target
  value stored in the rule for that field using the matching operator. If all
  the fields in the packet's header satisfied the all the matching operators of
  a rule,  the packet is processed using Compression Decompression Function associated
  with the fields. Otherwise the next rule
  is tested. [EDIT]If no eligible rule is found, then the packet is dropped.[EDIT]

* sending: The rule number is sent to the other end followed by data resulting
  from the field compression. These data are sent in the rule order for the matching
  fields. The way the rule number is sent depends of the
  layer two technology and will be specified in a specific document. For exemple,
  it can either be included in a Layer 2 header or sent in the first byte of
  the L2 payload.

* decompression: The receiver identifies the  sender through its device-id
  (e.g. MAC address) and select the appropriate rule through the rule number. This
  rule gives the compressed header format and associate these values to header fields.
  It applies the compression decompression function to reconstruct the original
  header fields. Compute-\* CDFs must be applied after the other CDFs.


# Matching operators {#chap-MO}

This document describes 3 basic matching operators which must be known by both LC. They are 
not typed and can be applied indifferently to integer, string,... 

* equal: a field value in a packet matches with a field value in a rule if
  they are equal.

* ignore: no check is done between a field value in a packet and a field value
  in the rule. The result is always true.

* MSB(length): a field value of length T in a packet matches with a field value
  in a rule if the most significant "length" bits are equal.

MO may need a list of parameters to proceed to the matching. For instance MSB requires the 
number of bits to test.

# Compression Decompression Functions (CDF) {#chap-CDF}

The Compression Decompression Functions (CDF) describe the action taken during
the compression and inversely the action taken by the decompressor to restore
the original value.

~~~~
/--------------------+-------------+---------------------------\
| Function           | Compression | Decompression             |
|                    |             |                           |
+--------------------+-------------+---------------------------+
|not-sent            |elided       |use value stored in ctxt   |
|value-sent          |send         |build from received value  |
|LSB(length)         |send LSB     |ctxt value OR rcvd value   |
|compute-IPv6-length |elided       |compute IPv6 length        |
|compute-UDP-length  |elided       |compute UDP length         |
|compute-UDP-checksum|elided       |compute UDP checksum       |
|ESiid-DID           |elided       |build IID from L2 ES addr  |
|LAiid-DID           |elided       |build IID from L2 LA addr  |
|static-mapping      |send index   |value form index on a table|
\--------------------+-------------+---------------------------/

~~~~
{: #Fig-function title='Compression and Decompression Functions'}

{{Fig-function}} sumarizes the functions defined to compress and decompress
a field. The first column gives the function's name. The second and third
columns outlines the compression/decompression behavior.

Compression is done in the rule order and send in that order in the compressed
message. The receiver must be able to find the size of each compressed field
which can be given by the rule or send with the data.

## not-sent CDF

Not-sent function is generally used when the field value is specified in the rule and
therefore known by the both sides. This function is generally used with the
"equal" MO. If MO is "ignore", there is a risk to have a decompressed field
value different from the compressed field.

The compressor do not sent any value on the compressed header for that field.

The decompressor
restores the field value with the target value stored in the matched rule.

## value-sent CDF

Value-sent function is generally used when the field value is not known by both end.
The value is sent in the compressed message header. Both ends must know the
size of the field, either implicitely or explicitely in the compressed header
field. This function is generally used with the ignore MO.

The compressor sends the target value stored on the rule on the compressed
header message. , if the matching operator
is "=". Otherwise the matching operator indicates the information that will
be sent on the link.

For a LSB operator only the Least Significant Bits are
sent.

## LSB CDF

LSB function is used to send a fixed part of the packet field header to the other end.
This function is used in conjuction with the "MSB" MO

The compressor sends the "length" Least Significant Bits. The decompressor
combines with a OR operator the value received with the Target Value.


## ESiid-DID, LAiid-DID CDF

These functions are used to process respectively the End System and the LA
Device Identifier (DID).

The IID value is computed from device ID present in the Layer 2 header. The
computation depends on the technology and the device ID  size.

[[NOTE: Behavoir must be more clearly defined. Most technologies have only
a single L2 address corresponding the ESiid. LoRaWAN has a changing L2 address,
SigFox is stable, but both also have a EUI-64 which could be better to used.]]

## static-mapping

The goal of static-mapping is to reduce the size of a field by allocating
shorter value. The mapping is known by both ends and stored in a table in
both end contexts. 

[[NOTE: specify when the mapping is not found]] 

## Compute-\*

These functions compute the field value based on received information. They
are elided during the compression and reconstructed during the decompression.

* compute-ipv6-length: compute the IPv6 length field as described in {{RFC2460}}.

* compute-udp-length: compute the IPv6 length field as described in {{RFC0768}}.

* compute-udp-checksum: compute the IPv6 length field as described in {{RFC0768}}.

# Application to IPv6 and UDP headers

This section lists the different IPv6 and UDP fields and how they can be compressed.

## IPv6 version field

This field hold always the same value, therefore the TV is 6, the MO is "equal" 
and the CDF "not-sent"


## IPv6 Traffic class field

If the DiffServ field identified by the rest of the rule do not vary and is known 
by both sides, the TV should contain this wellknown value, the MO should be "equal" 
and the CDF should be "not-sent.

If the DiffServ field identified by the rest of the rule vary during time or is not 
known by both sides:

* TV is not set, MO is set to "ignore" and CDF is set to "value-sent"

* TV contains a stable value, MO is MSB(X) and CDF is set to LSB(8-X)

## Flow label field

If the Flow Label field identified by the rest of the rule do not vary and is known 
by both sides, the TV should contain this wellknown value, the MO should be "equal" 
and the CDF should be "not-sent.

If the Flow Label field identified by the rest of the rule vary during time or is not 
known by both sides:

* TV is not set, MO is set to "ignore" and CDF is set to "value-sent"

* TV contains a stable value, MO is MSB(X) and CDF is set to LSB(20-X)

## Payload Length field

If the LPWAN technology do not add padding, this field can be elided for the 
transmission on the LPWAN network. The LC recompute the original payload length
value. The TV is not set, the MO is set to "ignore" and the CDF is "compute-IPv6-length".

if the payload is small, the TV can be set to 0x0000, the MO set to "MSB" and the
CDF to "LSB". 

On other cases, the length must be sent and the CDF is replaced by "value-sent".

## Next Header field

If the Next Header field identified by the rest of the rule do not vary and is known 
by both sides, the TV should contain this Next Header value, the MO should be "equal" 
and the CDF should be "not-sent.

If the Flow Label field identified by the rest of the rule vary during time or is not 
known by both sides then TV is not set, MO is set to "ignore" and CDF is set to 
"value-sent"

## Hop Limit field

The End System is generally a host and do not forward packets, therefore the
Hop Limit value is constant. Therefore the TV is set with a default value, the MO 
is set to "equal" and the CDF is set to "not-sent".

Otherwise the value is sent on the LPWAN: TV is not set, MO is set to ignore and 
CDF is set to "value-sent".

## IPv6 addresses fields

As in 6LoWPAN {{RFC4944}} IPv6 addresses are splited into two 64 bit long fields; 
one for the prefix and one for the Interface Identifier (IID). These fields should
be compressed. To allow a single rule, these values are identified by their role 
(ES or LA) and not by their position in the frame (source or destination). The LC
must be aware of the traffic direction (upstream, downstream) to select the appropriate
field.

### IPv6 source and destination prefixes

Both ends must be synchronized with the appropriate prefixes. For a specific flow, 
the source and destination prefixe can be unique and stored in the context. It can 
be either a link-local prefix or a global prefix. In that case, the TV for the 
source and destination prefixes contains the values, the MO is set to "equal" and
the CDF is set to "not-sent".

In case the rule allows several prefixes, the static mapping must be used. The 
different prefixes are listed in the TV associated with a short ID. The MO is set 
to "match-mapping" and the CDF is set to "mapping-sent".

Otherwise the TV contains the prefix, the MO is set to "equal" and the CDF is set to
value-sent.

### IPv6 source and destination IID

If the ES or LA IID are based on a LPWAN address, then the IID can be reconstructed 
with information coming from the LPWAN header. In that case the TV is not set, the MO 
is set to "ignore" and the CDF is set to "ESiid-DID" or "LAiid-DID". Note that the 
LPWAN technology is generally carrying a single device identifier corresponding
to the ES. The LC may also not be aware of these values. 

For privacy reasons or if the ES address is changing overt the time, it maybe better to
use a static value. In that case, the TV contains the value, the MO operator is set to
"equal" and the CDF is set to "not-sent". 

If several IID are possible, then the TV contains the list of possible IID, the MO is 
set to "match-mapping" and the CDF is set to "mapping-sent". 

Otherwise the variation of the IID may be reduced to few bytes. In that case, the TV is
set to the stable part of the IID, the MO is set to MSB and the CDF is set to LSB.

Finally, the IID can be send on the LPWAN. In that case, the TV is not set, the MO is set
to "ignore" and the CDF is set to "value-sent".

## IPv6 extensions

Currently no extensions rules are defined. They can be based on the MO and 
CDF described above.

## UDP source and destination port

To allow a single rule, these values are identified by their role 
(ES or LA) and not by their position in the frame (source or destination). The LC
must be aware of the traffic direction (upstream, downstream) to select the appropriate
field. The following rules apply for ES and LA port numbers.

If both ends knows the port number, it can be elided. The TV contains the port number,
the MO is set to "equal" and the CDF is set to "not-sent".

If the port variation are on few bits, the TV contains the stable part of the port number,
the MO is set to "MSB" and the CDF is set to "LSB".

If some wellknown value are used, the the TV can contain the list of this values, the
MO is set to "match-mapping" and the CDF is set to "mapping-sent".

Otherwise the port number are sent on the LPWAN. The TV is not set, the MO is 
set to "ignore" and the CDF is set to "value-sent".

## UDP length field

If the LPWAN technology does not introduce padding, the UDP length can be computed
from the received data. In that case the TV is not set, the MO is set to "ignore" and
the CDF is set to "compute-UDP-length".

if the payload is small, the TV can be set to 0x0000, the MO set to "MSB" and the
CDF to "LSB". 

On other cases, the length must be sent and the CDF is replaced by "value-sent".

## UDP Checksum field

IPv6 mandate a checksum in protocol above IP. Nevertheless, if a more efficient
mechanism such as L2 CRC or MIC is carried by or over the L2 (such as in the 
LPWAN fragmentation process (see XXXX)), the checksum transmission can be avoided.
In that case, the TV is not set, the MO is set to "ignore" and the CDF is set to
"compute-UDP-checksum".

In other cases the checksum must be explicitly sent. The TV is not set, the MO is set to
"ignore" and the CDF is set to "value-sent".



# Examples {#compressIPv6}


This section gives some scenarios of the compression mechanism for IPv6/UDP.
The goal is to illustrate the SCHC behavior.

## IPv6/UDP compression in a star topology

The most common case will be a LPWA end-system embeds some applications running
over
CoAP. In this example, the first flow is for the device management based
on CoAP using
Link Local addresses and UDP ports 123 and 124.
The second flow will be a CoAP server for measurements done by the end-system
(using ports 5683) and Global Addresses alpha::IID/64 to beta::1/64.
The last flow is for legacy applications using different ports numbers, the
destination is gamma::1/64.

 {{FigStack}} presents the protocol stack for this end-system. IPv6 and UDP are represented
with dotted lines since these protocols are compressed on the radio link.

~~~~
 Managment    Data
+----------+---------+---------+
|   CoAP   |  CoAP   | legacy  |
+----||----+---||----+---||----+
.   UDP    .  UDP    |   UDP   |
................................
.   IPv6   .  IPv6   .  IPv6   .
+------------------------------+
|      6LPWA L2 technologies   |
+------------------------------+
      End System or LPWA GW

~~~~
{: #FigStack title='Simplified Protocol Stack for LP-WAN'}


Note that in some LPWA technologies, only End Systems have a device ID.
Therefore it is necessary to define statically an IID for the Link
Local address for the LPWA Compressor.



~~~~
  Rule 0
  +----------------+---------+--------+-------------++------+
  | Field          | Value   | Match  | Function    || Sent |
  +----------------+---------+----------------------++------+
  |IPv6 version    |6        | equal  | not-sent    ||      |
  |IPv6 DiffServ   |0        | equal  | not-sent    ||      |
  |IPv6 Flow Label |0        | equal  | not-sent    ||      |
  |IPv6 Length     |         | ignore | comp-IPv6-l ||      |
  |IPv6 Next Header|17       | equal  | not-sent    ||      |
  |IPv6 Hop Limit  |255      | ignore | not-sent    ||      |
  |IPv6 ESprefix   |FE80::/64| equal  | not-sent    ||      |
  |IPv6 ESiid      |         | ignore | ESiid-DID   ||      |
  |IPv6 LCprefix   |FE80::/64| equal  | not-sent    ||      |
  |IPv6 LAiid      |::1      | equal  | not-sent    ||      |
  +================+=========+========+=============++======+
  |UDP ESport      |123      | equal  | not-sent    ||      |
  |UDP LAport      |124      | equal  | not-sent    ||      |
  |UDP Length      |         | ignore | comp-UDP-l  ||      |
  |UDP checksum    |         | ignore | comp-UDP-c  ||      |
  +================+=========+========+=============++======+

  Rule 1
  +----------------+---------+--------+-------------++------+
  | Field          | Value   | Match  | Function    || Sent |
  +----------------+---------+--------+-------------++------+
  |IPv6 version    |6        | equal  | not-sent    ||      |
  |IPv6 DiffServ   |0        | equal  | not-sent    ||      |
  |IPv6 Flow Label |0        | equal  | not-sent    ||      |
  |IPv6 Length     |         | ignore | comp-IPv6-l ||      |
  |IPv6 Next Header|17       | equal  | not-sent    ||      |
  |IPv6 Hop Limit  |255      | ignore | not-sent    ||      |
  |IPv6 ESprefix   |alpha/64 | equal  | not-sent    ||      |
  |IPv6 ESiid      |         | ignore | ESiid-DID   ||      |
  |IPv6 LAprefix   |beta/64  | equal  | not-sent    ||      |
  |IPv6 LAiid      |::1000   | equal  | not-sent    ||      |
  +================+=========+========+=============++======+
  |UDP ESport      |5683     | equal  | not-sent    ||      |
  |UDP LAport      |5683     | equal  | not-sent    ||      |
  |UDP Length      |         | ignore | comp-UDP-l  ||      |
  |UDP checksum    |         | ignore | comp-UDP-c  ||      |
  +================+=========+========+=============++======+

  Rule 2
  +----------------+---------+--------+-------------++------+
  | Field          | Value   | Match  | Function    || Sent |
  +----------------+---------+--------+-------------++------+
  |IPv6 version    |6        | equal  | not-sent    ||      |
  |IPv6 DiffServ   |0        | equal  | not-sent    ||      |
  |IPv6 Flow Label |0        | equal  | not-sent    ||      |
  |IPv6 Length     |         | ignore | comp-IPv6-l ||      |
  |IPv6 Next Header|17       | equal  | not-sent    ||      |
  |IPv6 Hop Limit  |255      | ignore | not-sent    ||      |
  |IPv6 ESprefix   |alpha/64 | equal  | not-sent    ||      |
  |IPv6 ESiid      |         | ignore | ESiid-DID   ||      |
  |IPv6 LAprefix   |gamma/64 | equal  | not-sent    ||      |
  |IPv6 LAiid      |::1000   | equal  | not-sent    ||      |
  +================+=========+========+=============++======+
  |UDP ESport      |8720     | MSB(12)| LSB(4)      || lsb  |
  |UDP LAport      |8720     | MSB(12)| LSB(4)      || lsb  |
  |UDP Length      |         | ignore | comp-UDP-l  ||      |
  |UDP checksum    |         | ignore | comp-UDP-c  ||      |
  +================+=========+========+=============++======+


~~~~
{: #Fig-fields title='Context rules'}

All the fields described in the three rules {{Fig-fields}} are present
in the IPv6 and UDP headers.  The ESDevice-ID value is found in the L2
header.

The second and third rules use global addresses. The way the ES learn the
prefix is not in the scope of the document. One possible way is to use a
management protocol to set up in both end rules the prefix used on the LPWA
network.

The third rule compresses port numbers on 4 bits. 

# Acknowledgements

Thanks to Dominique Barthel, Arunprabhu Kandasamy, Antony Markovski, Alexander
Pelov, Juan Carlos Zuniga for useful design
consideration.


--- back
