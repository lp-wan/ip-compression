---
stand_alone: true
ipr: trust200902
docname: draft-ietf-lpwan-ipv6-static-context-hc-01
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
title: LPWAN Static Context Header Compression (SCHC) and fragmentation for IPv6 and UDP
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
- ins: C. Gomez
  name: Carles Gomez
  org: Universitat Politècnica de Catalunya
  street: 
  - C/Esteve Terradas, 7 
  - 08860 Castelldefels
  country: Spain
  email: carlesgo@entel.upc.edu
normative:
  rfc0768: 
  rfc4944: 
  rfc2460: 
informative:  
  I-D.minaburo-lp-wan-gap-analysis:
  I-D.ietf-lpwan-overview:

--- abstract

This document describes a header compression scheme and fragmentation functionality 
for IPv6/UDP protocols. These techniques are especially tailored for LPWAN (Low Power Wide Area Network) networks
and could be extended to other protocol stacks.

The Static Context Header Compression (SCHC)  offers a great level of flexibility 
when  processing the header fields.
Static context means that information stored in the context which, describes field values, does not change during
the packet transmission, avoiding complex resynchronization mechanisms, incompatible
with LPWAN characteristics. In most of the cases, IPv6/UDP headers are reduced
to a small identifier.

This document describes the generic compression/decompression process and applies it 
to IPv6/UDP headers. Similar mechanisms for other protocols such as CoAP will be described in a
separate document. Moreover, this document specifies fragmentation and reassembly mechanims for SCHC compressed packets exceeding the L2 pdu size and for the case where the SCHC compression is not possible then the IPv6/UDP packet is sent. 

--- middle
 
# Introduction {#Introduction}

Header compression is mandatory to efficiently bring Internet connectivity to the node
within a LPWAN network {{I-D.minaburo-lp-wan-gap-analysis}}. 

Some LPWAN networks properties can be exploited for an efficient header
compression:

* Topology is star oriented, therefore all the packets follow the same path.
  For the needs of this draft, the architecture can be summarized to Things or End-Systems
  (ES) exchanging information with LPWAN Application Server (LA) through a Network Gateway (NG). 

* Traffic flows are mostly known in advanced, since End-Systems embed built-in
  applications. Contrary to computers or smartphones, new applications cannot
  be easily installed.

The Static Context Header Compression (SCHC) is defined for this environment.
SCHC uses a context where header information is kept in order, this context is 
static the values on the header fields do not change during time, avoiding 
complex resynchronization mechanisms, incompatible
with LPWAN characteristics. In most of the cases, IPv6/UDP headers are reduced
to a small context identifier.

The SCHC header compression is indedependent of the specific LPWAN technology over which it will be used.

On the other hand, LPWAN technologies are characterized,
among others, by a very reduced data unit and/or payload size
{{I-D.ietf-lpwan-overview}}.  However, some of these technologies
do not support layer two fragmentation, therefore the only option for
these to support IPv6 when header compression is not possible (and, in particular, its MTU requirement of
1280 bytes {{RFC2460}}) is the use of fragmentation mechanism at the
adaptation layer below IPv6. This specification defines fragmentation 
functionality to support the IPv6 MTU requirements over LPWAN 
technologies.


# Vocabulary
This section defines the terminology and aconyms used in this document.

* CDF: Compression/Decompression Function. A function that is used for both functionnalities to compress a header field or to recover its original value in the decompression phase.

* Context: A set of rules used to compress/decompress headers

* ES: End System. Node connected to the LPWAN. An ES may implement SCHC.

* LA: LPWAN Application. An application sending/consuming IPv6 packets to/from the End System.

* LC: LPWAN Compressor/Decompressor. A process in the network to achieve compression/decompressing headers. LC uses SCHC rules to perform compression and decompression.

* MO: Matching Operator. An operator used to compare a value contained in a header field with a value contained in a rule.

* Rule: A set of header field values.

* Rule ID: An identifier for a rule, LC and ES share the same rule ID for a specific flow. Rule ID 
  is sent on the LPWAN.
  
* TV: Target value. A value contained in the rule that will be matched with the value of a header field.

# Static Context Header Compression

Static Context Header Compression (SCHC) avoids context synchronization,
which is the most bandwidth-consuming operation in other header compression mechanisms
such as RoHC. Based on the fact
that the nature of data flows is highly predictable in LPWAN networks, a static
context may be stored on the End-System (ES). The context must be stored in both ends. It can 
also be learned by using a provisionning protocol that is out of the scope of this draft.

~~~~
     End-System                                        Appl Servers
+-----------------+                                 +---------------+
| APP1  APP2 APP3 |                                 |APP1  APP2 APP3|
|                 |                                 |               |
|       UDP       |                                 |      UDP      | 
|      IPv6       |                                 |     IPv6      |   
|                 |                                 |               |  
|      LC (contxt)|                                 |               | 
+--------+--------+                                 +-------+-------+ 
         |   +--+     +--+     +-----------+                .
         +~~ |RG| === |NG| === |LC (contxt)| ... Internet ...
             +--+     +--+     +-----+-----+
~~~~
{: #Fig-archi title='Architecture'}

{{Fig-archi}} based on {{I-D.ietf-lpwan-overview}} terminology represents the architecture for 
compression/decompression. The Thing or End-System is running applications which produce IPv6 or IPv6/UDP
flows. These flows are compressed by a LPWAN Compressor (LC) to reduce the headers size. Resulting
information is sent on a layer two (L2) frame to the LPWAN Radio Network to a Radio Gateway (RG) which forwards 
the frame to a Network Gateway.
The Network Gateway sends the data to a LC for decompression which shares the same rules with the ES. The LC can be 
located on the Network Gateway or in another places if a tunnel is established between the NG and the LC.
This architecture forms a star topology. After decompression, the packet can be sent on the Internet to one
or several LPWAN Application Servers (LA). 

The principle is exactly the same in the other direction.

The context contains a list of rules (cf. {{Fig-ctxt}}). Each rule contains 
itself a list of fields descriptions composed of a field identifier (FID), a target
value (TV), a matching operator (MO) and a Compression/Decompression Function
(CDF).


~~~~
  +-----------------------------------------------------------------+
  |                      Rule N                                     |
 +----------------------------------------------------------------+ |
 |                    Rule i                                      | |
+---------------------------------------------------------------+ | |
|                Rule 1                                         | | |
|+--------+--------------+-------------------+-----------------+| | |
||Field 1 | Target Value | Matching Operator | Comp/Decomp Fct || | |
|+--------+--------------+-------------------+-----------------+| | |
||Field 2 | Target Value | Matching Operator | Comp/Decomp Fct || | |
|+--------+--------------+-------------------+-----------------+| | |
||...     |    ...       | ...               | ...             || | |
|+--------+--------------+-------------------+-----------------+| |-+
||Field N | Target Value | Matching Operator | Comp/Decomp Fct || |
|+--------+--------------+-------------------+-----------------+|-+
|                                                               |
+---------------------------------------------------------------+
~~~~
{: #Fig-ctxt title='Compression Decompression Context'}


The rule does not describe the original packet format which
must be known from the compressor/decompressor. The rule just describes the
compression/decompression behavior for the header fields. In the rule, it is recommended
to describe the header field in the same order they appear in the packet.

The main idea of the compression scheme is to send the rule id to the other end instead 
of known field values. When a value is known by both
ends, it is not necessary to send it on the LPWAN network. 

The field description is composed of different entries:

* A Field ID (FID) is a unique value to define the field. 

* A Target Value (TV) is the value used to make the comparison with
  the packet header field. The Target Value can be of any type (integer, strings,...).
  It can be a single value or a more complex structure (array, list,...). It can
  be considered as a CBOR structure.

* A Matching Operator (MO) is the operator used to make the comparison between 
  the field value and the Target Value. The Matching Operator may require some 
  parameters, which can be considered as a CBOR structure. MO is only used during 
  the compression phase.

* A Compression Decompression Function (CDF) is used to describe the compression
  and the decompression process. The CDF may require some 
  parameters, which can be considered as a CBOR structure.

## Rule ID

Rule IDs are sent between both compression/decompression elements. The size 
of the rule ID is not specified in this document and can vary regarding the
LPWAN technology, the number of flows,... 

Some values in the rule ID space may be reserved for goals other than header 
compression, for example fragmentation. 

Rule IDs are specific to an ES. Two ESs may use the same rule ID for different
header compression. The LC needs to combine the rule ID with the ES L2 address
to find the appropriate rule.

## Packet processing

The compression/decompression process follows several steps:

* compression rule selection: the goal is to identify which rule(s) will be used
  to compress the headers.  Each field is associated to a matching operator for
  compression. Each header field's value is compared to the corresponding target
  value stored in the rule for that field using the matching operator. If all
  the fields in the packet's header satisfied  all the matching operators of
  a rule,  the packet is processed using Compression Decompression Function associated
  with the fields. Otherwise the next rule
  is tested. If no eligible rule is found, then the packet is sent without compression,
  which may require using the fragmentation procedure. 

* sending: The rule ID is sent to the other end followed by information resulting
  from the compression of header fields. This information is sent in the order expressed in the rule for the matching
  fields. The way the rule ID is sent depends on the
  layer two technology and will be specified in a specific document. For example,
  it can either be included in a Layer 2 header or sent in the first byte of
  the L2 payload.

* decompression: The receiver identifies the  sender through its device-id
  (e.g. MAC address) and selects the appropriate rule through the rule ID. This
  rule gives the compressed header format and associates these values to header fields.
  It applies the CDF function to reconstruct the original
  header fields. CDF of Compute-\* must be applied after the other CDFs.


# Matching operators {#chap-MO}

This document describes basic matching operators (MO)s which must be known by both LC, endpoints involved in the header compression/decompression. They are 
not typed and can be applied indifferently to integer, string or any other type. The MOs and their definition are provided next:

* equal: a field value in a packet matches with a field value in a rule if
  they are equal.

* ignore: no check is done between a field value in a packet and a field value
  in the rule. The result of the matching is always true.

* MSB(length): a field value of a size equal to "length" bits in a packet matches with a field value
  in a rule if the most significant "length" bits are equal.
  
* match-mapping: The goal of mapping-sent is to reduce the size of a field by allocating
  a shorter value. The Target Value contains a list of pairs. Each pair is composed of
  a value and a short ID. This operator matches if a field value is equal to one of the pairs'
  values.

Matching Operators may need a list of parameters to proceed to the matching. For instance MSB requires an
integer indicating the number of bits to test.

# Compression Decompression Functions (CDF) {#chap-CDF}

The Compression Decompression Functions (CDF) describes the action taken during
the compression of headers fields, and inversely, the action taken by the decompressor to restore
the original value.

~~~~
/--------------------+-------------+---------------------------\
| Function           | Compression | Decompression             |
|                    |             |                           |
+--------------------+-------------+---------------------------+
|not-sent            |elided       |use value stored in ctxt   |
|value-sent          |send         |build from received value  |
|LSB(length)         |send LSB     |ctxt value OR rcvd value   |
|compute-length      |elided       |compute length             |
|compute-UDP-checksum|elided       |compute UDP checksum       |
|ESiid-DID           |elided       |build IID from L2 ES addr  |
|LAiid-DID           |elided       |build IID from L2 LA addr  |
|mapping-sent        |send index   |value from index on a table|
\--------------------+-------------+---------------------------/

~~~~
{: #Fig-function title='Compression and Decompression Functions'}

{{Fig-function}} sumarizes the functions defined to compress and decompress
a field. The first column gives the function's name. The second and third
columns outlines the compression/decompression behavior.

Compression is done in the rule order and compressed values are sent in that order in the compressed
message. The receiver must be able to find the size of each compressed field
which can be given by the rule or may be sent with the compressed header.

## not-sent CDF

Not-sent function is generally used when the field value is specified in the rule and
therefore known by the both Compressor and Decompressor. This function is generally used with the
"equal" MO. If MO is "ignore", there is a risk to have a decompressed field
value different from the compressed field.

The compressor does not send any value on the compressed header for that field on which compression is applied.

The decompressor
restores the field value with the target value stored in the matched rule.

## value-sent CDF

The value-sent function is generally used when the field value is not known by both Compressor and Decompressor.
The value is sent in the compressed message header. Both Compressor and Decompressor must know the
size of the field, either implicitely (the size is known by both sides) 
or explicitely in the compressed header
field by indicating the length. This function is generally used with the "ignore" MO.

The compressor sends the Target Value stored on the rule in the compressed
header message. The decompressor restores the field value with the one received 
from the LPWAN 


## LSB CDF

LSB function is used to send a fixed part of the packet field header to the other end.
This function is used together with the "MSB" MO

The compressor sends the "length" Least Significant Bits. The decompressor
combines with an OR operator the value received with the Target Value.


## ESiid-DID, LAiid-DID CDF

These functions are used to process respectively the End System and the LA
Device Identifier (DID).

The IID value is computed from the device ID present in the Layer 2 header. The
computation depends on the technology and the device ID  size.

## mapping-sent

mapping-sent is used to send a smaller index associated to the field value
in the Target Value. This function is used together with the "match-mapping" MO.

The compressor looks in the TV to find the field value and send the corresponding index.
The decompressor uses this index to restore the field value.

## Compute-\*

These functions are used by the decompressor to compute the compressed field value based on received information. 
Compressed fields are elided during the compression and reconstructed during the decompression.

* compute-length: compute the length assigned to this field. For instance, regarding
  the field ID, this CDF may be used to compute IPv6 length or UDP length.

* compute-checksum: compute a checksum from the information already received by the LC.
  This field may be used to compute UDP checksum. 

# Application to IPv6 and UDP headers

This section lists the different IPv6 and UDP header fields and how they can be compressed.

## IPv6 version field

This field always holds the same value, therefore the TV is 6, the MO is "equal" 
and the CDF "not-sent".


## IPv6 Traffic class field

If the DiffServ field identified by the rest of the rule do not vary and is known 
by both sides, the TV should contain this wellknown value, the MO should be "equal" 
and the CDF must be "not-sent.

If the DiffServ field identified by the rest of the rule varies over time or is not 
known by both sides, then there are two possibilities depending on the variability of the value, 
the first one there is without compression and the original value is sent, or 
the sencond where the values can be computed by sending only the LSB bits:

* TV is not set, MO is set to "ignore" and CDF is set to "value-sent"

* TV contains a stable value, MO is MSB(X) and CDF is set to LSB(8-X)

## Flow label field

If the Flow Label field identified by the rest of the rule does not vary and is known 
by both sides, the TV should contain this well-known value, the MO should be "equal" 
and the CDF should be "not-sent".

If the Flow Label field identified by the rest of the rule varies during time or is not 
known by both sides, there are two possibilities dpending on the variability of the value,
the first one is without compression and then the value is sent 
and the second where only part of the value is sent and the decompressor needs to compute the original value:

* TV is not set, MO is set to "ignore" and CDF is set to "value-sent"

* TV contains a stable value, MO is MSB(X) and CDF is set to LSB(20-X)

## Payload Length field

If the LPWAN technology does not add padding, this field can be elided for the 
transmission on the LPWAN network. The LC recompute the original payload length
value. The TV is not set, the MO is set to "ignore" and the CDF is "compute-IPv6-length".

If the payload is small, the TV can be set to 0x0000, the MO set to "MSB (16-s)" and the
CDF to "LSB (s)". The 's' parameter depends on the maximum packet length.

On other cases, the payload length field must be sent and the CDF is replaced by "value-sent".

## Next Header field

If the Next Header field identified by the rest of the rule does not vary and is known 
by both sides, the TV should contain this Next Header value, the MO should be "equal" 
and the CDF should be "not-sent".

If the Next header  field identified by the rest of the rule varies during time or is not 
known by both sides, then TV is not set, MO is set to "ignore" and CDF is set to 
"value-sent".

## Hop Limit field

The End System is generally a host and does not forward packets, therefore the
Hop Limit value is constant. So the TV is set with a default value, the MO 
is set to "equal" and the CDF is set to "not-sent".

Otherwise the value is sent on the LPWAN: TV is not set, MO is set to ignore and 
CDF is set to "value-sent".

## IPv6 addresses fields

As in 6LoWPAN {{RFC4944}}, IPv6 addresses are split into two 64-bit long fields; 
one for the prefix and one for the Interface Identifier (IID). These fields should
be compressed. To allow a single rule, these values are identified by their role 
(ES or LA) and not by their position in the frame (source or destination). The LC
must be aware of the traffic direction (upstream, downstream) to select the appropriate
field.

### IPv6 source and destination prefixes

Both ends must be synchronized with the appropriate prefixes. For a specific flow, 
the source and destination prefix can be unique and stored in the context. It can 
be either a link-local prefix or a global prefix. In that case, the TV for the 
source and destination prefixes contains the values, the MO is set to "equal" and
the CDF is set to "not-sent".

In case the rule allows several prefixes, static mapping must be used. The 
different prefixes are listed in the TV associated with a short ID. The MO is set 
to "match-mapping" and the CDF is set to "mapping-sent".

Otherwise the TV contains the prefix, the MO is set to "equal" and the CDF is set to
value-sent.

### IPv6 source and destination IID

If the ES or LA IID are based on an LPWAN address, then the IID can be reconstructed 
with information coming from the LPWAN header. In that case, the TV is not set, the MO 
is set to "ignore" and the CDF is set to "ESiid-DID" or "LAiid-DID". Note that the 
LPWAN technology is generally carrying a single device identifier corresponding
to the ES. The LC may also not be aware of these values. 

For privacy reasons or if the ES address is changing over time, it maybe better to
use a static value. In that case, the TV contains the value, the MO operator is set to
"equal" and the CDF is set to "not-sent". 

If several IIDs are possible, then the TV contains the list of possible IID, the MO is 
set to "match-mapping" and the CDF is set to "mapping-sent". 

Otherwise the value variation of the IID may be reduced to few bytes. In that case, the TV is
set to the stable part of the IID, the MO is set to MSB and the CDF is set to LSB.

Finally, the IID can be sent on the LPWAN. In that case, the TV is not set, the MO is set
to "ignore" and the CDF is set to "value-sent".

## IPv6 extensions

No extension rules are currently defined. They can be based on the MOs and 
CDFs described above.

## UDP source and destination port

To allow a single rule, the UDP port values are identified by their role 
(ES or LA) and not by their position in the frame (source or destination). The LC
must be aware of the traffic direction (upstream, downstream) to select the appropriate
field. The following rules apply for ES and LA port numbers.

If both ends knows the port number, it can be elided. The TV contains the port number,
the MO is set to "equal" and the CDF is set to "not-sent".

If the port variation is on few bits, the TV contains the stable part of the port number,
the MO is set to "MSB" and the CDF is set to "LSB".

If some well-known values are used,  the TV can contain the list of this values, the
MO is set to "match-mapping" and the CDF is set to "mapping-sent".

Otherwise the port numbers are sent on the LPWAN. The TV is not set, the MO is 
set to "ignore" and the CDF is set to "value-sent".

## UDP length field

If the LPWAN technology does not introduce padding, the UDP length can be computed
from the received data. In that case the TV is not set, the MO is set to "ignore" and
the CDF is set to "compute-UDP-length".

If the payload is small, the TV can be set to 0x0000, the MO set to "MSB" and the
CDF to "LSB". 

On other cases, the length must be sent and the CDF is replaced by "value-sent".

## UDP Checksum field

IPv6 mandates a checksum in the protocol above IP. Nevertheless, if a more efficient
mechanism such as L2 CRC or MIC is carried by or over the L2 (such as in the 
LPWAN fragmentation process (see XXXX)), the UDP checksum transmission can be avoided.
In that case, the TV is not set, the MO is set to "ignore" and the CDF is set to
"compute-UDP-checksum".

In other cases the checksum must be explicitly sent. The TV is not set, the MO is set to
"ignore" and the CDF is set to "value-sent".



# Examples {#compressIPv6}


This section gives some scenarios of the compression mechanism for IPv6/UDP.
The goal is to illustrate the SCHC behavior.

## IPv6/UDP compression 

The most common case using the mechanisms defined in this document will be a 
LPWAN end-system that embeds some applications running over
CoAP. In this example, three flows are considered. The first flow is for the device management based
on CoAP using
Link Local IPv6 addresses and UDP ports 123 and 124 for ES and LA, respectively.
The second flow will be a CoAP server for measurements done by the end-system
(using ports 5683) and Global IPv6 Address prefixes alpha::IID/64 to beta::1/64.
The last flow is for legacy applications using different ports numbers, the
destination IPv6 address prefix is gamma::1/64.

 {{FigStack}} presents the protocol stack for this End-System. IPv6 and UDP are represented
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
|    SCHC Header compression   |
|      and fragmentation       |
+------------------------------+
|      6LPWA L2 technologies   |
+------------------------------+
      End System or LPWA GW

~~~~
{: #FigStack title='Simplified Protocol Stack for LP-WAN'}


Note that in some LPWAN technologies, only the End Systems have a device ID.
Therefore, when such technologie are used, it is necessary to define statically an IID for the Link
Local address for the LPWAN compressor.



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
  |UDP Length      |         | ignore | comp-length ||      |
  |UDP checksum    |         | ignore | comp-chk    ||      |
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
  |UDP Length      |         | ignore | comp-length ||      |
  |UDP checksum    |         | ignore | comp-chk    ||      |
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
  |UDP Length      |         | ignore | comp-length ||      |
  |UDP checksum    |         | ignore | comp-chk    ||      |
  +================+=========+========+=============++======+


~~~~
{: #Fig-fields title='Context rules'}

All the fields described in the three rules {{Fig-fields}} are present
in the IPv6 and UDP headers.  The ESDevice-ID value is found in the L2
header.

The second and third rules use global addresses. The way the ES learns the
prefix is not in the scope of the document. 

The third rule compresses port numbers to 4 bits. 

# Fragmentation

## Overview

Fragmentation in LPWAN is mandatory and it is used if after the SCHC header compression the
size of the packet is larger than the L2 data unit maximum payload or if the SCHC header compression
is not able to compress the packet. In LPWAN technologies the
L2 data unit size typically varies from tens to hundreds of bytes. 
If the entire IPv6 datagram fits within a single L2
data unit, the fragmentation mechanims is not used and the packet is sent unfragmented.  
If the datagram does not fit within a single L2 data unit,
it SHALL be broken into fragments. 

Moreover, LPWAN technologies impose some strict limitations on traffic; 
therefore it is desirable to enable optional fragment retransmission, while 
a single fragment loss should not lead to retransmitting the full datagram. 
To preserve energy, Things (End Systems) are sleeping most of the time 
and may receive data during a short period of time after transmission. 

This specification enables two main fragment delivery reliability options, 
namely: Unreliable and Reliable. The same reliability option MUST be 
used for all fragments of a packet. Note that the fragment delivery reliability
option to be used is not necessarily tied to the particular characteristics of the 
underlying L2 LPWAN technology (e.g. Unreliable may be used on top of an L2 
LPWAN technology with symmetric characteristics for uplink and downlink).

In Unreliable, the receiver MUST NOT issue acknowledgments and the sender
MUST NOT perform fragment transmission retries.

In Reliable, there exist two possible suboptions, namely: packet mode and window mode.
In packet mode, the receiver may transmit one acknowledgment (ACK) after all fragments
carrying an IPv6 packet have been transmitted. The ACK informs the sender about received
and missing fragments from the IPv6 packet. In window mode, an ACK may be transmitted by 
the fragment receiver after a window of fragments have been sent. A window of fragments is
a subset of the fragments needed to carry an IPv6 packet. In this mode, the ACK informs the
sender about received and missing fragments from the window of fragments. In either mode, 
upon receipt of an ACK that informs about any lost fragments, the sender may retransmit the
lost fragments. The maximum number of ACK and retransmission rounds is TBD. In Reliable, the
same reliability suboption MUST be used for all fragments of a packet.

Some LPWAN deployments may benefit from conditioning the creation and transmission of an ACK
to the detection of at least one fragment loss (per-packet or per-window), thus leading to
negative ACK (NACK)-oriented behavior, while not having such condition may be preferred for other scenarios.

This document does not make any decision as to whether Unreliable or Reliable are used, or 
whether in Reliable a fragment receiver generates ACKs per packet or per window, or whether
the transmission of such ACKs is conditioned to the detection of fragment losses or not. 
A complete specification of the receiver and sender behaviors that correspond to each 
acknowledgment policy is also out of scope. Nevertheless, this document does provide 
examples of the different reliability options described.


## Fragment format

   A fragment comprises a fragmentation header and a fragment payload, and conforms
   to the format shown in {{Fig-FragFormat}}. The fragment payload carries a subset of either the
   IPv6 packet after header compression or an IPv6 packet which could not be compressed. 
   A fragment is the payload in the L2 protocol data unit (PDU).
      
~~~~   
      +---------------+-----------------------+
      | Fragm. Header |   Fragment payload    |
      +---------------+-----------------------+
~~~~
{: #Fig-FragFormat title='Fragment format.'}

## Fragmentation header formats

Fragments except the last one SHALL    
   contain the fragmentation header as defined in {{Fig-NotLast}}. The total size of this fragmentation header is R bits.

~~~~
                       <----------- R ----------->    
                                        <-- N  -->   
                       +----- ... -----+-- ... --+
                       |    Rule ID    |   CFN   |
                       +----- ... -----+-- ... --+
~~~~
{: #Fig-NotLast title='Fragmentation Header for Fragments except the Last One'}

   The last fragment SHALL contain a fragmentation header that conforms to 
   the format shown in {{Fig-Last}}. The total size of this fragmentation 
   header is R+M bits.

~~~~
                       <----------- R ---------->
                                        <-- N --> <---- M ----->                   
                       +----- ... -----+-- ... --+---- ... ----+
                       |    Rule ID    |  11..1  |     MIC     |
                       +----- ... -----+-- ... --+---- ... ----+
~~~~
{: #Fig-Last title='Fragmentation Header for the Last Fragment'}


   Rule ID: this field has a size of  R – N  bits in all 
      fragments. Rule ID may be used to signal whether Unreliable or Reliable are in use, 
      and within the latter, whether window mode or packet mode are used.

   CFN: CFN stands for Compressed Fragment Number. The size of the CFN field is N bits. 
      In Unreliable, N=1. For Reliable, N equal to or greater than 3 is recommended. This field
      is an unsigned integer that carries a non-absolute fragment number. The CFN MUST be set 
      sequentially decreasing from 2^N - 2 for the first fragment, and MUST wrap from 0 back to
      2^N - 2 (e.g. for N=3, the first fragment has CFN=6, subsequent CFNs are set sequentially
      and in decreasing order, and CFN will wrap from 0 back to 6). The CFN for the last fragment 
      has all bits set to 1. Note that, by this definition, the CFN value of 2^N - 1 is only used 
      to identify a fragment as the last fragment carrying a subset of the IPv6 packet being transported,
      and thus the CFN does not strictly correspond to the N least significant bits of the actual 
      absolute fragment number. It is also important to note that, for N=1, the last fragment 
      of the packet will carry a CFN equal to 1, while all previous fragments will carry a CFN of 0. 

   MIC:  MIC stands for Message Integrity Check. This field has a size of M 
      bits. It is computed by the sender over the complete IPv6 packet before fragmentation by 
      using the TBD algorithm. The MIC allows to check for errors in the reassembled IPv6 packet, 
      while it also enables compressing the UDP checksum by use of SCHC.
            
## ACK format

The format of an ACK is shown in {{Fig-ACK-Format}}:

~~~~
                          <-----  R  ---->
                         +-+-+-+-+-+-+-+-+----- ... ---+
                         |    Rule ID    |   bitmap    |
                         +-+-+-+-+-+-+-+-+----- ... ---+
~~~~
{: #Fig-ACK-Format title='Format of an ACK'}


  Rule ID: In all ACKs, Rule ID has a size of R bits and SHALL be set to
   TBD_ACK to signal that the message is an ACK.

  bitmap:  size of the bitmap field of an ACK can be equal to 0 or  
   Ceiling(Number_of_Fragments/8) octets, where Number_of_Fragments denotes
   the number of fragments of a window (in window mode) or the number of fragments
   that carry the IPv6 packet (in packet mode). The bitmap is a sequence of bits,
   where the n-th bit signals whether the n-th fragment transmitted has been 
   correctly received (n-th bit set to 1) or not (n-th bit set to 0). Remaining bits
   with bit order greater than the number of fragments sent (as determined by 
   the receiver) are set to 0, except for the last bit in the bitmap, which is set to 1 
   if the last fragment (carrying the MIC) has been correctly received, and 0 otherwise.
   Absence of the bitmap in an ACK confirms correct reception of all fragments to be 
   acknowledged by means of the ACK.  
   
   {{Fig-Bitmap}} shows an example of an ACK in packet mode, where the bitmap 
   indicates that the second and the ninth fragments have not been correctly 
   received. In this example, the IPv6 packet is carried by eleven fragments 
   in total, therefore the bitmap in has a size of two bytes. 

~~~~
                                                       1
                  <-----  R  ----> 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
                  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                  |    Rule ID    |1|0|1|1|1|1|1|1|0|1|1|0|0|0|0|1|
                  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: #Fig-Bitmap title='Example of the Bitmap in an ACK'}

{{Fig-Bitmap-Win}} shows an example of an ACK in window mode (N=3), where the bitmap
indicates that the second and the fifth fragments have not been correctly received. 

~~~~                                                  
                  <-----  R  ----> 0 1 2 3 4 5 6 7 
                  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                  |    Rule ID    |1|0|1|1|0|1|1|1|                   
                  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: #Fig-Bitmap-Win title='Example of the bitmap in an ACK (in window mode, for N=3)'}

{{Fig-NoBitmap}} illustrates an ACK without bitmap.

~~~~
                  <-----  R  ----> 
                  +-+-+-+-+-+-+-+-+
                  |    Rule ID    | 
                  +-+-+-+-+-+-+-+-+
~~~~
{: #Fig-NoBitmap title='Example of an ACK without bitmap'}

## Baseline mechanism

The receiver of link fragments SHALL use (1) the sender’s L2 source address (if present),
(2) the destination’s L2 address (if present), and (3) Rule ID to identify all the fragments
that belong to a given datagram. The fragment receiver SHALL determine the fragment delivery
reliability option in use for the fragment based on the Rule ID field in that fragment.

Upon receipt of a link fragment, the receiver starts constructing the original
unfragmented packet. It uses the CFN and the order of arrival of each fragment to
determine the location of the individual fragments within the original unfragmented packet.
For example, it may place the data payload of the fragments within a payload datagram 
reassembly buffer at the location determined from the CFN and order of arrival of the 
fragments, and the fragment payload sizes. Note that the size of the original, unfragmented
IPv6 packet cannot be determined from fragmentation headers.

In Reliable, when a fragment with all CFN bits set to 0 is received, the recipient MAY transmit
an ACK for the last window of fragments sent. Note that the first fragment of the window is 
the one sent with CFN=2^N-2. In window mode, the fragment with CFN=0 is considered the
last fragment of its window, except for the last fragment (with all CFN bits set to 1). The last
fragment of a packet is also the last fragment of the last window.

Once the recipient has received the last fragment, it checks for the integrity of the 
reassembled IPv6 datagram, based on the MIC received. In Unreliable, if the integrity 
check indicates that the reassembled IPv6 datagram does not match the original IPv6 datagram
(prior to fragmentation), the reassembled IPv6 datagram MUST be discarded. In Reliable, upon
receipt of the last fragment (i.e. with all CFN bits set to 1), the recipient MAY transmit an ACK for the last window of fragments sent (window mode) or for the whole set of fragments sent that carry a complete IPv6 packet 
(packet mode). In Reliable, the sender retransmits any lost fragments reported in the ACK. A maximum
of TBD iterations of ACK and fragment retransmission rounds are allowed per-window or per-IPv6-packet
in window mode or in packet mode, respectively. A complete specification of the mechanisms needed to
enable the above described fragment delivery reliability options is out of the scope of this document.

If a fragment recipient disassociates from its L2 network, the recipient MUST discard 
all link fragments of all partially reassembled payload datagrams, and fragment senders 
MUST discard all not yet transmitted link fragments of all partially transmitted payload
(e.g., IPv6) datagrams. Similarly, when a node first receives a fragment of a packet, it starts
a reassembly timer. When this time expires, if the entire packet has not been reassembled, 
the existing fragments MUST be discarded and the reassembly state MUST be flushed. The reassembly
timeout MUST be set to a maximum of TBD seconds).

## Examples
This section provides examples of different fragment delivery reliability options possible 
on the basis of this specification.

{{Fig-Example-Unreliable}} illustrates the transmission of an IPv6 packet that needs 11 fragments
in Unreliable.

~~~~       
        Sender               Receiver
          |-------CFN=0-------->|
          |-------CFN=0-------->|
          |-------CFN=0-------->|
          |-------CFN=0-------->|
          |-------CFN=0-------->|
          |-------CFN=0-------->|
          |-------CFN=0-------->|
          |-------CFN=0-------->|
          |-------CFN=0-------->|
          |-------CFN=0-------->|
          |-------CFN=1-------->|MIC checked =>
~~~~
{: #Fig-Example-Unreliable title='Transmission of an IPv6 packet carried by 11 fragments in Unreliable'}


{{Fig-Example-Rel-NoLoss}} illustrates the transmission of an IPv6 packet that needs 11 fragments
in Reliable, for N=3, NACK-oriented packet mode, without losses.

~~~~       
        Sender               Receiver
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4-------->|
          |-------CFN=3-------->|
          |-------CFN=2-------->|
          |-------CFN=1-------->|
          |-------CFN=0-------->|
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4-------->|
          |-------CFN=7-------->|MIC checked =>
       (no NACK)  
~~~~
{: #Fig-Example-Rel-NoLoss title='Transmission of an IPv6 packet carried by 11 fragments in Reliable,
for N=3, NACK-oriented packet mode; no losses.'}

{{Fig-Example-Rel-Loss}} illustrates the transmission of an IPv6 packet that needs 11 fragments
in Reliable, for N=3, NACK-oriented packet mode, with three losses. 

~~~~       
        Sender               Receiver
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4---X---->|
          |-------CFN=3-------->|
          |-------CFN=2---X---->|
          |-------CFN=1-------->|
          |-------CFN=0-------->|
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4---X---->|
          |-------CFN=7-------->|MIC checked =>
          |<-------NACK---------|Bitmap:1101011110100000
          |-------CFN=4-------->|
          |-------CFN=2-------->|
          |-------CFN=4-------->|MIC checked => 
       (no NACK)   
~~~~
{: #Fig-Example-Rel-Loss title='Transmission of an IPv6 packet carried by 11 fragments in Reliable, for N=3, NACK-oriented packet mode; three losses.'}

{{Fig-Example-Win-NoLoss-NACK}} illustrates the transmission of an IPv6 packet that needs 11 fragments in Reliable, window mode, for N=3, without losses. Receiver feedback is NACK-oriented. Note: in window mode, an additional bit will be needed to number windows.

~~~~
        Sender               Receiver
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4-------->|
          |-------CFN=3-------->|
          |-------CFN=2-------->|
          |-------CFN=1-------->|
          |-------CFN=0-------->|
      (no NACK)
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4-------->|
          |-------CFN=7-------->|MIC checked =>
      (no NACK)
~~~~
{: #Fig-Example-Win-NoLoss-NACK title='Transmission of an IPv6 packet carried by 11 fragments in Reliable, for N=3, NACK-oriented window mode; without losses.'}

{{Fig-Example-Rel-Window-NACK-Loss}} illustrates the transmission of an IPv6 packet that needs 11 fragments
in Reliable, window mode, for N=3, with three losses. Receiver feedback is NACK-oriented. Note: in window mode,
an additional bit will be needed to number windows.

~~~~       
        Sender               Receiver
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4---X---->|
          |-------CFN=3-------->|
          |-------CFN=2---X---->|
          |-------CFN=1-------->|
          |-------CFN=0-------->|
          |<-------NACK---------|Bitmap:11010110
          |-------CFN=4-------->|
          |-------CFN=2-------->|   
      (no NACK)     
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4---X---->|
          |-------CFN=7-------->|MIC checked =>
          |<-------NACK---------|Bitmap:11010000
          |-------CFN=4-------->|MIC checked =>
      (no NACK)    
~~~~
{: #Fig-Example-Rel-Window-NACK-Loss title='Transmission of an IPv6 packet carried by 11 fragments in Reliable,  for N=3, NACK-oriented window mode; three losses.'}

{{Fig-Example-Rel-Packet-ACK-NoLoss}} illustrates the transmission of an IPv6 packet that needs 11 fragments
in Reliable, packet mode, for N=3, without losses. Receiver feedback is positive-ACK-oriented.

~~~~       
        Sender               Receiver
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4-------->|
          |-------CFN=3-------->|
          |-------CFN=2-------->|
          |-------CFN=1-------->|
          |-------CFN=0-------->|
          |-------CFN=6-------->|
          |-------CFN=5-------->|   
          |-------CFN=4-------->|
          |-------CFN=7-------->|MIC checked =>
          |<-------ACK----------|no bitmap
        (End)    
      
~~~~
{: #Fig-Example-Rel-Packet-ACK-NoLoss title='Transmission of an IPv6 packet carried by 11 fragments in Reliable, for N=3, packet mode, positive-ACK-oriented; no losses.'}

{{Fig-Example-Rel-Packet-ACK-Loss}} illustrates the transmission of an IPv6 packet that needs 11 fragments
in Reliable, packet mode, for N=3, with three losses. Receiver feedback is positive-ACK-oriented.

~~~~       
        Sender               Receiver
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4---X---->|
          |-------CFN=3-------->|
          |-------CFN=2---X---->|
          |-------CFN=1-------->|
          |-------CFN=0-------->|
          |-------CFN=6-------->|
          |-------CFN=5-------->|   
          |-------CFN=4---X---->|
          |-------CFN=7-------->|MIC checked =>
          |<-------ACK----------|bitmap:1101011110100000
          |-------CFN=4-------->|
          |-------CFN=2-------->|
          |-------CFN=4-------->|MIC checked =>
          |<-------ACK----------|no bitmap
        (End)
      
~~~~
{: #Fig-Example-Rel-Packet-ACK-Loss title='Transmission of an IPv6 packet carried by 11 fragments in Reliable, for N=3, packet mode, positive-ACK-oriented; with three losses.'}

### Reliable, window mode, ACK-oriented

{{Fig-Example-Rel-Window-ACK-NoLoss}} illustrates the transmission of an IPv6 packet that needs 11 fragments
in Reliable, window mode, for N=3, without losses. Receiver feedback is positive-ACK-oriented. Note: in window mode,
an additional bit will be needed to number windows.

~~~~       
        Sender               Receiver
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4-------->|
          |-------CFN=3-------->|
          |-------CFN=2-------->|
          |-------CFN=1-------->|
          |-------CFN=0-------->|
          |<-------ACK----------|no bitmap
          |-------CFN=6-------->|
          |-------CFN=5-------->|   
          |-------CFN=4-------->|
          |-------CFN=7-------->|MIC checked =>
          |<-------ACK----------|no bitmap
        (End)    
      
~~~~
{: #Fig-Example-Rel-Window-ACK-NoLoss title='Transmission of an IPv6 packet carried by 11 fragments in Reliable, for N=3, window mode, positive-ACK-oriented; no losses.'}

{{Fig-Example-Rel-Window-ACK-Loss}} illustrates the transmission of an IPv6 packet that needs 11 fragments
in Reliable, window mode, for N=3, with three losses. Receiver feedback is positive-ACK-oriented. Note: in window mode,
an additional bit will be needed to number windows.

~~~~       
        Sender               Receiver
          |-------CFN=6-------->|
          |-------CFN=5-------->|
          |-------CFN=4---X---->|
          |-------CFN=3-------->|
          |-------CFN=2---X---->|
          |-------CFN=1-------->|
          |-------CFN=0-------->|
          |<-------ACK----------|bitmap:11010110
          |-------CFN=4-------->|
          |-------CFN=2-------->|
          |<-------ACK----------|no bitmap
          |-------CFN=6-------->|
          |-------CFN=5-------->|   
          |-------CFN=4---X---->|
          |-------CFN=7-------->|MIC checked =>
          |<-------ACK----------|bitmap:11010000
          |-------CFN=4-------->|MIC checked =>
          |<-------ACK----------|no bitmap
        (End)    
      
~~~~
{: #Fig-Example-Rel-Window-ACK-Loss title='Transmission of an IPv6 packet carried by 11 fragments in Reliable, for N=3, window mode, positive-ACK-oriented; with three losses.'}

# Security considerations

## Security considerations for header compression
TBD

## Security considerations for fragmentation
This subsection describes potential attacks to LPWAN fragmentation 
and proposes countermeasures, based on existing analysis of attacks
to 6LoWPAN fragmentation {HHWH}.

A node can perform a buffer reservation attack by sending a first
fragment to a target.  Then, the receiver will reserve buffer space
for the whole packet on the basis of the datagram size announced in
that first fragment.  Other incoming fragmented packets will be
dropped while the reassembly buffer is occupied during the reassembly
timeout.  Once that timeout expires, the attacker can repeat the same
procedure, and iterate, thus creating a denial of service attack.
The (low) cost to mount this attack is linear with the number of
buffers at the target node.  However, the cost for an attacker can be
increased if individual fragments of multiple packets can be stored
in the reassembly buffer.  To further increase the attack cost, the
reassembly buffer can be split into fragment-sized buffer slots.
Once a packet is complete, it is processed normally.  If buffer
overload occurs, a receiver can discard packets based on the sender
behavior, which may help identify which fragments have been sent by
an attacker.

In another type of attack, the malicious node is required to have
overhearing capabilities.  If an attacker can overhear a fragment, it
can send a spoofed duplicate (e.g. with random payload) to the
destination.  A receiver cannot distinguish legitimate from spoofed
fragments.  Therefore, the original IPv6 packet will be considered
corrupt and will be dropped.  To protect resource-constrained nodes
from this attack, it has been proposed to establish a binding among
the fragments to be transmitted by a node, by applying content-
chaining to the different fragments, based on cryptographic hash
functionality.  The aim of this technique is to allow a receiver to
identify illegitimate fragments.

Further attacks may involve sending overlapped fragments (i.e.
comprising some overlapping parts of the original IPv6 datagram).
Implementers should make sure that correct operation is not affected
by such event.

# Acknowledgements

Thanks to Dominique Barthel, Carsten Bormann, Arunprabhu Kandasamy, Antony Markovski, Alexander
Pelov, Pascal Thubert, Juan Carlos Zuniga for useful design
consideration.

In the fragmentation section, the authors have reused parts of text
available in section 5.3 of RFC 4944, and would like to thank the
authors of RFC 4944.

Carles Gomez has been funded in part by the Spanish Government
(Ministerio de Educacion, Cultura y Deporte) through the Jose
Castillejo grant CAS15/00336, and by the ERDF and the Spanish Government 
through project TEC2016-79988-P.  Part of his contribution to this work
has been carried out during his stay as a visiting scholar at the
Computer Laboratory of the University of Cambridge.


--- back
