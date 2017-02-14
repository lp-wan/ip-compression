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
  rfc5795:
informative:  
  I-D.minaburo-lp-wan-gap-analysis:
  I-D.ietf-lpwan-overview:

--- abstract

This document describes a header compression scheme and fragmentation functionality 
for IPv6/UDP protocols. These techniques are especially tailored for LPWA networks
and could be extended to other protocol stacks.

The Static Context Header Compression (SCHC)  offers a great level of flexibility 
when  processing the fields.
Static context means that information stored in the context which describes field values do not change during
the transmission, avoiding complex resynchronization mechanisms, incompatible
with LPWA characteristics. In most of the cases, IPv6/UDP headers are reduced
to a small identifier.

This document describes the generic compression/decompression process and apply it 
to IPv6/UDP headers. Other protocols such as CoAP will be described in a
separate document.

--- middle
 
# Introduction {#Introduction}

Header compression is mandatory to bring the internet connectivity to the node
within a LPWA network {{I-D.minaburo-lp-wan-gap-analysis}}. 

Some LPWA networks properties can be exploited for an efficient header
compression:

* Topology is star oriented, therefore all the packets follows the same path.
  For the needs of this draft, the architecture can be summarized to Things or End-Systems
  (ES) exchanging information with LPWAN Application Server (LA) through a Network Gateway (NG). 

* Traffic flows are mostly deterministic, since End-Systems embed built-in
  applications. Contrary to computers or smartphones, new applications cannot
  be easily installed.

The Static Context Header Compression (SCHC) is defined for this environment.
Static context means that values in the context field do not change during
the transmission, avoiding complex resynchronization mechanisms, incompatible
with LPWA characteristics. In most of the cases, IPv6/UDP headers are reduced
to a small context identifier.

The SCHC is indedependent of the LPWAN technology.

On the other hand, Low Power Wide Area Network (LPWAN) technologies are characterized,
among others, by a very reduced data unit and/or payload size
{{I-D.ietf-lpwan-overview}}.  However, some of these technologies
do not support layer two fragmentation, therefore the only option for
these to support IPv6 (and, in particular, its MTU requirement of
1280 bytes {{RFC2460}}) is the use of a fragmentation mechanism at the
adaptation layer below IPv6. This specification defines a fragmentation 
functionality to support the IPv6 MTU requirements over LPWAN 
technologies.


# Vocabulary

* CDF: Compression Decompression Function. Function is used for both functionnalities to compress a field or to recover its original value in the decompression phase.

* Context: A set of rules used to compress/decompress headers

* ES: End System. Node connected to the LPWAN. ES may implement SCHC.

* LA: LPWAN Application. Application sending/consuming headers to/from the End System.

* LC: LPWAN Compressor/Decompressor. Process in the network to achieve compression/decompressing headers. LC uses SCHC rules to perfom compression and decompression.

* MO: Matching Operator. Operator used to compare a value contained in a field's header with a value contained in a rule.

* Rule: A set of header field values.

* Rule ID: An identifier for a rule, LC and ES share the same rule ID for a specific flow. Rule ID 
  is sent on the LPWAN.
  
* TV: Target value. Value contained in the rule that will be matched with the value of an header field.

# Static Context Header Compression

Static Context Header Compression (SCHC) avoids context synchronization,
which is the most bandwidth-consuming operation in other header compression mechanisms
such as RoHC. Based on the fact
that the nature of data flows is highly predictable in LPWA networks, a static
context may be stored on the End-System (ES). The context must be stored in both ends. It can 
also be learned by a provisionning protocol that is out of the scope of this draft.

~~~~
     End-System                                           Appl Servers
+-----------------+                                   +---------------+
| APP1  APP2 APP3 |                                   |APP1  APP2 APP3|
|                 |                                   |               |
|       UDP       |                                   |      UDP      | 
|      IPv6       |                                   |     IPv6      |   
|                 |                                   |               |  
|      LC (contxt)|                                   |               | 
+--------+--------+                                   +-------+-------+ 
         |    +--+     +--+      +-----------+                 .
         +~ ~ |RG| === |NG| ==== |LC (contxt)| ... Internet ....
              +--+     +--+      +-----+-----+
~~~~
{: #Fig-archi title='Architecture'}

{{Fig-archi}} based on {{I-D.ietf-lpwan-overview}} terminology represent the architecture for 
compression/decompression. The Thing or End-System is running applications which produce UDP/IPv6
flows. These flows are compressed by a LPWAN Compressor (LC) to reduce the headers size. Resulting
information is sent on a frame to the LPWAN Radio Network to a Radio Gateway (RG) which forwards 
the frame to a Network Gateway.
The Network Gateway sends the data to a LC for decompression which shares the same rules with the ES. The LC can be 
located on the Network Gateway or in another places if a tunnel is established between the NG and the LC.
This architecture forms a star topology. After decompression the packet can be sent on the Internet to one
or several Application Servers (LA). 

The principle is exactly the same in the other direction.

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
compression/decompression behavior for a field. In the rule, it is recommanded
to describe the header field in the same order they appear in the packet.

The main idea of the compression scheme is to send the rule number (or rule
id) to the other end instead of known field values. When a value is known by both
ends, it is not necessary to sent it on the LPWA network. 

The field description is composed of different entries:

* A Field ID or FID is a unique value to define the field. 

* A Target Value or TV is the value used to make the comparison between 
  the packet field. The Target Value can be of any type (integer, strings,..).
  It can be a single value or a more complex structure (array, list,...). It can
  be considered as a CBOR structure.

* A Matching Operator or MO is the operator used to make the comparison between 
  the field value and the Target Value. The Matching Operator may require some 
  parameters, which can be considered as a CBOR structure. MO is only used during 
  the compression phase.

* A Compression Decompression Function or CDF is used to describe the compression
  and the decompression process. The CDF may require some 
  parameters, which can be considered as a CBOR structure.

## Rule id

Rule ids are sent between both compression/decompression elements. The size 
of the rule id is not specified on this document and can vary regarding the
LPWAN technology, the number of flows,... 

Some values in the rule id space may be reserved to other goal than header 
compression, for example fragmentation. 

Rule ids are specific to an ES. Two ES may use the same rule id for different
header compression. The LC needs to combine the rule id with the ES L2 address
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
  is tested. If no eligible rule is found, then the packet is sent without compression
  using fragmentation procedure. 

* sending: The rule number is sent to the other end followed by information resulting
  from the compression of header fields. This information is sent in the rule order for the matching
  fields. The way the rule number is sent depends on the
  layer two technology and will be specified in a specific document. For example,
  it can either be included in a Layer 2 header or sent in the first byte of
  the L2 payload.

* decompression: The receiver identifies the  sender through its device-id
  (e.g. MAC address) and select the appropriate rule through the rule number. This
  rule gives the compressed header format and associate these values to header fields.
  It applies the compression decompression function to reconstruct the original
  header fields. Compute-\* CDFs must be applied after the other CDFs.


# Matching operators {#chap-MO}

This document describes basic matching operators which must be known by both LC. They are 
not typed and can be applied indifferently to integer, string or any other type. 

* equal: a field value in a packet matches with a field value in a rule if
  they are equal.

* ignore: no check is done between a field value in a packet and a field value
  in the rule. The result of the matching is always true.

* MSB(length): a field value of length T in a packet matches with a field value
  in a rule if the most significant "length" bits are equal.
  
* match-mapping: The goal of mapping-sent is to reduce the size of a field by allocating
  shorter value. The Target Value contains a list of pairs. Each pair is composed of
  a value and short ID. This operator match if a field value is equal to one of the pair's
  value.

Matching Operators may need a list of parameters to proceed to the matching. For instance MSB requires an
integer indicating the number of bits to test.

# Compression Decompression Functions (CDF) {#chap-CDF}

The Compression Decompression Functions (CDF) describes the action taken during
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
|mapping-sent        |send index   |value form index on a table|
\--------------------+-------------+---------------------------/

~~~~
{: #Fig-function title='Compression and Decompression Functions'}

{{Fig-function}} sumarizes the functions defined to compress and decompress
a field. The first column gives the function's name. The second and third
columns outlines the compression/decompression behavior.

Compression is done in the rule order and compressed values are sent in that order in the compressed
message. The receiver must be able to find the size of each compressed field
which can be given by the rule or send with the compressed header.

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
size of the field, either implicitely (the size is known by both sides) 
or explicitely in the compressed header
field by indicating the length. This function is generally used with the "ignore" MO.

The compressor sends the Target Value stored on the rule in the compressed
header message. The decompressor restores the field value with the one received 
from the LPWAN 


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

## mapping-sent

mapping-sent is used to send a smaller index associated to the field value
in the Target Value. This function is used in conjuction with the "match-mapping" MO.

The compressor looks in the TV to find the field value and send the corresponding index.
The decompressor uses this index to restore the field value.

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

If the Flow Label field identified by the rest of the rule varies during time or is not 
known by both sides:

* TV is not set, MO is set to "ignore" and CDF is set to "value-sent"

* TV contains a stable value, MO is MSB(X) and CDF is set to LSB(20-X)

## Payload Length field

If the LPWAN technology does not add padding, this field can be elided for the 
transmission on the LPWAN network. The LC recompute the original payload length
value. The TV is not set, the MO is set to "ignore" and the CDF is "compute-IPv6-length".

If the payload is small, the TV can be set to 0x0000, the MO set to "MSB (16-s)" and the
CDF to "LSB (s)". The s parameter depends of the maximum packet length.

On other cases, the length must be sent and the CDF is replaced by "value-sent".

## Next Header field

If the Next Header field identified by the rest of the rule do not vary and is known 
by both sides, the TV should contain this Next Header value, the MO should be "equal" 
and the CDF should be "not-sent.

If the Next header  field identified by the rest of the rule varies during time or is not 
known by both sides then TV is not set, MO is set to "ignore" and CDF is set to 
"value-sent"

## Hop Limit field

The End System is generally a host and does not forward packets, therefore the
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
the source and destination prefix can be unique and stored in the context. It can 
be either a link-local prefix or a global prefix. In that case, the TV for the 
source and destination prefixes contains the values, the MO is set to "equal" and
the CDF is set to "not-sent".

In case the rule allows several prefixes, the static mapping must be used. The 
different prefixes are listed in the TV associated with a short ID. The MO is set 
to "match-mapping" and the CDF is set to "mapping-sent".

Otherwise the TV contains the prefix, the MO is set to "equal" and the CDF is set to
value-sent.

### IPv6 source and destination IID

If the ES or LA IID are based on an LPWAN address, then the IID can be reconstructed 
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

Currently no extension rules  are currently defined. They can be based on the MO and 
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

If some wellknown values are used,  the TV can contain the list of this values, the
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

The second and third rules use global addresses. The way the ES learns the
prefix is not in the scope of the document. 

The third rule compresses port numbers on 4 bits. 

# Fragmentation


## Overview

If an entire payload (e.g., IPv6) datagram fits within a single L2
data unit, it is unfragmented and a fragmentation header is not
needed.  If the datagram does not fit within a single L2 data unit,
it SHALL be broken into fragments. 

This specification defines two fragment delivery reliability options, 
namely: Unreliable and Reliable. The same reliability option MUST be 
used for all fragments of a packet.

In Unreliable, the receiver SHALL NOT issue acknowledgments and the sender
SHALL NOT perform fragment transmission retries.

In Reliable, if the fragment receiver detects any missing fragments from the
transported IPv6 packet, the receiver transmits one negative acknowledgment (NACK)
which informs the sender about received and missing fragments from the IPv6 
packet. Upon receipt of a NACK, the sender selectively retransmits the missing
fragments. If all fragments carrying the IPv6 packet are successfully received,
the receiver SHALL NOT send a NACK. If the sender does not receive a NACK, 
it assumes that all fragments carrying the IPv6 packet were successfully delivered.

## Unreliable
### Fragmentation header formats for Unreliable

In Unreliable, fragments except the last one SHALL    
   contain the fragmentation header as defined in {{Fig-Unrel-NotLast}}.

~~~~
                       <-----  R  ----->   
                       +----- ... -----+
                       |    Rule ID    |
                       +----- ... -----+
~~~~
{: #Fig-Unrel-NotLast title='Fragmentation Header for Fragments except the Last One in Unreliable'}
      
   The last fragment SHALL contain a fragmentation header that conforms to 
   the format shown in {{Fig-Unrel-Last}}.
 
~~~~
                       <-----  R  ----> <---- M ----->                   
                       +----- ... -----+---- ... ----+
                       |    Rule ID    |     MIC     |
                       +----- ... -----+---- ... ----+
~~~~
{: #Fig-Unrel-Last title='Fragmentation Header for the Last Fragment in Unreliable'} 


   Rule ID:  In Unreliable, this field has a size of R bits. Rule ID SHALL      
      be set to TBD_UNREL_A in fragments, except the last one, to 
      signal that the carried payload is a fragment, and that Unreliable 
      fragment delivery MUST be used. In the 
      last fragment, Rule ID SHALL be set to TBD_UNREL_B to identify the 
      fragment as the last one, and to signal that Unreliable fragment 
      delivery MUST be used.

   MIC: This field, of size M bits, is computed by the sender over the  
      complete IPv6 packet before fragmentation by using the TBD algorithm.

### Receiver and sender behavior for Unreliable

The recipient of link fragments SHALL use (1) the sender's L2 source
   address (if present), (2) the destination's L2 address (if present), and
   (3) Rule ID to identify all the fragments that belong to a given 
   datagram. The fragment receiver SHALL use Rule ID to determine 
   whether the fragment has to be handled as per the rules of Unreliable or 
   Reliable fragment delivery.

   Upon receipt of a link fragment, the recipient starts constructing
   the original unfragmented packet.  It uses the order 
   of arrival of each fragment to determine the location of the individual 
   fragments within the original unfragmented packet.  For example, it may 
   place the data payload of the fragments within a payload datagram 
   reassembly buffer at the location determined from the order of arrival 
   and the fragment payload sizes.  Note that the size of the reassembly buffer
   cannot be determined from fragmentation headers. 

   If a fragment recipient disassociates from its L2 network, the
   recipient MUST discard all link fragments of all partially
   reassembled payload datagrams, and fragment senders MUST discard all
   not yet transmitted link fragments of all partially transmitted
   payload (e.g., IPv6) datagrams.  Similarly, when a node first
   receives a fragment of a packet, it starts a reassembly timer.
   When this time expires, if the entire packet has not been
   reassembled, the existing fragments MUST be discarded and the
   reassembly state MUST be flushed.  The reassembly timeout MUST be set
   to a maximum of TBD seconds).

   Once the recipient has received the last fragment, it checks for the 
   integrity of the reassembled IPv6 datagram, based on the MIC received. 
   If the integrity check indicates that the reassembled IPv6 datagram does 
   not match the original IPv6 datagram (prior to fragmentation), the 
   reassembled IPv6 datagram MUST be discarded.

## Reliable
### Fragmentation header formats for Reliable
In Reliable, fragments except the last one SHALL    
   contain the fragmentation header as defined in {{Fig-Rel-NotLast}}. The total size of this fragmentation header is R bits.

~~~~
                       <----------- R ----------->    
                                        <-- N  -->   
                       +----- ... -----+-- ... --+
                       |    Rule ID    |   CFN   |
                       +----- ... -----+-- ... --+
~~~~
{: #Fig-Rel-NotLast title='Fragmentation Header for Fragments except the Last One in Reliable'}

   The last fragment SHALL contain a fragmentation header that conforms to 
   the format shown in {{Fig-Rel-Last}}. The total size of this fragmentation 
   header is R+M bits.

~~~~
                       <----------- R ---------->
                                        <-- N --> <---- M ----->                   
                       +----- ... -----+-- ... --+---- ... ----+
                       |    Rule ID    |   CFN   |     MIC     |
                       +----- ... -----+-- ... --+---- ... ----+
~~~~
{: #Fig-Rel-Last title='Fragmentation Header for the Last Fragment in Reliable'}


   Rule ID: In Reliable, this field has a size of  R – N  bits in all 
      fragments, and it SHALL be set to TBD_REL to signal that 
      the carried payload is a fragment, and that Reliable fragment 
      delivery MUST be used. 

   CFN:  CFN stands for Compressed Fragment Number. The size of the CFN 
      field is N bits. This field is an unsigned integer that carries a 
      non-absolute fragment number. 
      The CFN SHALL be set sequentially starting from 0 for the first 
      fragment, and SHALL wrap from 2^N - 2 back to 0. The CFN for the 
      last fragment has all bits set to 1.

   MIC:  MIC stands for Message Integrity Check. This field has a size of M 
      bits. It is computed by 
      the sender over the complete IPv6 packet before fragmentation by 
      using the TBD algorithm.

### NACK format

The format of a NACK is shown in {{Fig-NACK-Format}}:

~~~~
                          <-----  R  ---->
                         +-+-+-+-+-+-+-+-+----- ... ---+
                         |    Rule ID    |   bitmap    |
                         +-+-+-+-+-+-+-+-+----- ... ---+
~~~~
{: #Fig-NACK-Format title='Format of a NACK'}


  Rule ID: In all NACKs, Rule ID has a size of R bits and SHALL be set to  
      TBD_NACK to signal that the message is a NACK.

  bitmap:  the bitmap field of a NACK has a size equal to 
   Ceiling(Number_of_Fragments/8) octets, where 
   Number_of_Fragments denotes the number of fragments that carry the IPv6 
   packet. The bitmap is a sequence of bits, where the n-th bit signals 
   whether the n-th fragment transmitted has been correctly received (n-th 
   bit set to 1) or not (n-th bit set to 0). 
   
   {{Fig-Bitmap}} shows an example of a NACK, where the bitmap indicates that 
   the second and the ninth fragments have not been correctly received. 
   In this example, the IPv6 packet is carried by eleven fragments in total,
   therefore the bitmap in this example has a size of two bytes.

~~~~
                                                        1
                 <-----  R  ----> 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
                  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                  |    Rule ID    |1|0|1|1|1|1|1|1|0|1|1|X|X|X|X|X|
                  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: #Fig-Bitmap title='Example of the Bitmap in a NACK'}

### Sender behavior in Reliable

In Reliable, a sender MUST store a copy of the fragments 
that will be sent to carry an IPv6 packet. The memory resources to store 
those fragments SHALL be released as per the rules described below. 

After transmission of all fragments that carry an IPv6 packet, the 
sender waits for NACK_WAIT seconds for a possible incoming NACK. To this 
end, after the last fragment has been transmitted, the sender 
initializes a timer to NACK_WAIT seconds. If a NACK is not received 
during the NACK_WAIT interval, the sender MUST assume successful delivery
of all fragments and release the memory resources used to store the fragments. 

If a NACK is received before expiration of the NACK wait timer, the fragment 
sender processes the bitmap included in the NACK in order to determine which 
fragments need to be resent. Since the bitmap size is a multiple of eight bits, 
the fragment sender only takes into consideration the first F bitmap bits, where 
F denotes the number of fragments sent, and it MUST ignore the remaining bits in
the bitmap. 

Once the fragments that need to be resent have been identified, the sender 
renumbers these fragments, so that their CFN fields define a new sequence of
fragment numbers, which SHALL be set sequentially starting from 0 for the first
fragment, and SHALL wrap from 2^N - 2 back to 0. For example, if three fragments
have to be resent, and with N=3, their CFNs will be set to 0, 1, and 7, respectively,
regardless of their original CFN values. After fragment renumbering, the fragments 
are resent. 

After the transmission of the last retransmitted fragment, if the number of NACKs 
received during the current IPv6 packet transmission is less than 
MAX_NACKS_PER_IPv6_PACKET, the sender initializes a timer to NACK_WAIT seconds 
to listen for a possible incoming NACK. In that case, if a NACK is not received 
during the NACK_WAIT interval, the sender MUST assume successful delivery of all
the fragments and release the memory resources used to store the fragments. 
If a NACK is received before expiration of the NACK_WAIT timer, the fragment 
sender processes the NACK by following the same approach as for the first NACK 
received, and performs a new round of fragment retransmissions by iterating the
procedure described for the first round of fragment retransmissions.  

When the number of fragment retransmission rounds completed by the fragment sender
equals MAX_NACKS_PER_IPv6_PACKET, the sender MUST NOT perform any further fragment
retransmissions for the current IPv6 packet, and it MUST release the memory 
resources used to store the fragments for the current IPv6 packet.

If a fragment sender disassociates from its L2 network, it MUST discard 
all not yet transmitted link fragments of all partially transmitted
payload (e.g., IPv6) datagrams. 

### Receiver behavior in Reliable

The recipient of link fragments SHALL use (1) the sender's L2 
Source address (if present), (2) the destination's L2 address (if 
present), and (3) Rule ID and CFN to identify all the fragments that 
belong to a given datagram. The fragment receiver SHALL use 
Rule ID to determine whether the fragment has to be handled as per the 
rules of Unreliable or Reliable fragment delivery.   

Upon receipt of a link fragment, the recipient starts constructing
the original unfragmented packet.  It uses the CFN field and the order 
of arrival of each fragment to determine the location of the individual 
fragments within the original unfragmented packet.  For example, it may 
place the data payload within a payload datagram reassembly buffer at 
the location determined from the CFN, the received fragment payload 
sizes and the order of arrival.  Note that the size of the reassembly 
buffer cannot be determined from fragmentation headers, and if non-
continguous frame sequences are received, it is not always possible to 
determine the size of the missing fragment(s).

When a node first receives a fragment of a packet, it starts a reassembly timer.
When this time expires, if the entire packet has not been
reassembled, the existing fragments MUST be discarded and the
reassembly state MUST be flushed.  The reassembly timeout MUST be set
to a maximum of TBD seconds).

Once the recipient of fragments has received the last fragment, if 
the sequence of received fragments CFNs (except the special one used for 
the last fragment) is composed of consecutive values, the fragment 
receiver checks for the integrity of the reassembled IPv6 datagram, 
based on the MIC received. 
If the sequence of received fragments CFNs or the integrity check 
indicate that the reassembled IPv6 datagram does 
not match the original IPv6 datagram (prior to fragmentation), the 
recipient creates and transmits a NACK to the fragment sender. The NACK 
includes a bitmap that indicates successful or unsuccessful receipt for 
each one of the fragments that carry the IPv6 packet. 

After transmission of the NACK, and before expiration of the reassembly 
timeout, if the fragment recipient receives further (retransmitted) fragments, 
it uses the CFN and the order of arrival of each fragment to place the 
corresponding payload in its correct location in the reassembly 
buffer. Otherwise, the fragment recipient MUST NOT resend the last 
transmitted NACK. 

When the number of received retransmitted fragments equals the number of 
missing fragments, and after placing the fragment payloads in their 
correct location in the reassembly buffer, the fragment receiver 
performs a new integrity check of the reassembled IPv6 datagram based on 
the MIC received.  If the sequence of received fragments CFNs or the 
integrity check indicate that the reassembled IPv6 datagram does 
not match the original IPv6 datagram (prior to fragmentation), then two 
situations can happen:

*  If the number of NACKs sent by the receiver has reached 
        MAX_NACKS_PER_IPv6_PACKET, all partially reassembled fragment payloads 
        MUST be discarded. 

*  If the number of NACKs sent by the receiver is less than 
        MAX_NACKS_PER_IPv6_PACKET, the fragment receiver creates and transmits a 
        NACK to the fragment sender. The NACK includes a bitmap that indicates 
        currently successful or unsuccessful receipt for each one of the 
        fragments that carry the IPv6 packet. The fragment recipient then 
        iterates the operations after transmission of a NACK described in this 
        section as long as the number of NACKs sent is less than 
        MAX_NACKS_PER_IPv6_PACKET.

If a fragment recipient disassociates from its L2 network, the
recipient MUST discard all link fragments of all partially
reassembled payload datagrams.

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
comprising some overlapping parts of the original datagram) or
announcing a datagram size in the first fragment that does not
reflect the actual amount of data carried by the fragments.
Implementers should make sure that correct operation is not affected
by such events.

# Acknowledgements

Thanks to Dominique Barthel, Arunprabhu Kandasamy, Antony Markovski, Alexander
Pelov, Juan Carlos Zuniga for useful design
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
