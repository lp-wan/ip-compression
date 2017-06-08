---
stand_alone: true
ipr: trust200902
docname: draft-ietf-lpwan-ipv6-static-context-hc-03
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
  toc: 'yes'
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
  rfc4944: 
  rfc2460: 
  rfc7136:
  rfc5795:
informative:  
  I-D.minaburo-lp-wan-gap-analysis:
  I-D.ietf-lpwan-overview:

--- abstract

This document describes a header compression scheme and fragmentation functionality 
for very low bandwidth networks. These techniques are especially tailored for LPWAN (Low Power Wide Area Network) networks.

The Static Context Header Compression (SCHC) offers a great level of flexibility 
when  processing the header fields and must be used for this kind of networks.
Static context means that information stored in the context which describes field values, does not change during
packet transmission, avoiding complex resynchronization mechanisms, incompatible
with LPWAN characteristics. In most of the cases, IPv6/UDP headers are reduced
to a small identifier called Rule ID. But sometimes the SCHC header compression will not be enough to send the packet in one L2 PDU, so this document also describes a Fragmentation protocol that must be used when needed. 

This document describes the generic compression/decompression mechanism and applies it 
to IPv6/UDP headers. Similar mechanisms for other protocols such as CoAP will be described in 
separate documents. Moreover, this document specifies fragmentation and reassembly mechanims for SCHC compressed packets exceeding the L2 PDU size and for the case where the SCHC compression is not possible then the IPv6/UDP packet is sent using the fragmentation protocol. 

--- middle
 
# Introduction {#Introduction}

Header compression is mandatory to efficiently bring Internet connectivity to the node
within a LPWAN network. Some LPWAN networks properties can be exploited for an efficient header
compression:

* Topology is star-oriented, therefore all the packets follow the same path.
  For the needs of this draft, the architecture can be summarized to Devices (Dev)
  exchanging information with LPWAN Application Server (App) through a Network Gateway (NGW). 

* Traffic flows are mostly known in advance, since devices embed built-in
  applications. Contrary to computers or smartphones, new applications cannot
  be easily installed.

The Static Context Header Compression (SCHC) is defined for this environment.
SCHC uses a context where header information is kept in the header format order. This context is 
static (the values on the header fields do not change over time) avoiding 
complex resynchronization mechanisms, incompatible
with LPWAN characteristics. In most of the cases, IPv6/UDP headers are reduced
to a small context identifier. 

The SCHC header compression mechanism is independent of the specific LPWAN technology over which it will be used.

Moreover, LPWAN technologies are also characterized,
among others, by a very reduced data unit and/or payload size
{{I-D.ietf-lpwan-overview}}.  However, some of these technologies
do not support layer two fragmentation, therefore the only option for
   these to support the IPv6 MTU requirement of 1280 bytes
 {{RFC2460}} is the use of a fragmentation protocol at the
adaptation layer below IPv6. 
This draft defines also a fragmentation 
functionality to support the IPv6 MTU requirements over LPWAN 
technologies. Such functionality has been designed under the assumption that data unit reordering will not happen between the entity performing fragmentation and the entity performing reassembly.

# LPWAN Architecture

LPWAN technologies have similar architectures but different terminology. We can identify different
types of entities in a typical LPWAN network:

   o  Devices (Dev) are the end-devices or hosts (e.g. sensors,
      actuators, etc.). There can be a high density of devices per radio gateway.

   o  The Radio Gateway (RG), which is the end point of the constrained link.

   o  The Network Gateway (NGW) is the interconnection node between the Radio Gateway and the Internet.  

   o  LPWAN-AAA Server, which controls the user authentication, the
      applications. We use the term LPWAN-AAA server because we are not assuming 
      that this entity speaks RADIUS or Diameter as many/most AAA servers do, but equally we don't want to
      rule that out, as the functionality will be similar.

   o  Application Server (App)

                                                 +------+
 ()    ()   ()         |                         |LPWAN-|
   ()  () () ()       / \         +---------+    | AAA  |
() () () () () ()    /   \========|    /\   |====|Server|  +-----------+
 ()  ()   ()        |             | <--|--> |    +------+  |APPLICATION|
()  ()  ()  ()     / \============|    v    |==============|    (App)  |
  ()  ()  ()      /   \           +---------+              +-----------+
 Dev         Radio Gateways           NGW

                       Figure 9: LPWAN Architecture


# Terminology
This section defines the terminology and acronyms used in this document.

* CDA: Compression/Decompression Action. An action that is perfomed for both functionnalities to compress a header field or to recover its original value in the decompression phase.

* Context: A set of rules used to compress/decompress headers

* Dev: Device. Node connected to the LPWAN. A Dev may implement SCHC.

* App: LPWAN Application. An application sending/receiving IPv6 packets to/from the Device.

* SCHC C/D: LPWAN Compressor/Decompressor. A process in the network to achieve compression/decompressing headers. SCHC C/D uses SCHC rules to perform compression and decompression.

* MO: Matching Operator. An operator used to match a value contained in a header field with a value contained in a Rule.

* Rule: A set of header field values.

* Rule ID: An identifier for a rule, SCHC C/D and Dev share the same Rule ID for a specific flow. 
  
* TV: Target value. A value contained in the Rule that will be matched with the value of a header field.

* FID: Field Indentifier is an index to describe the header fields in the Rule

* FP: Field Position is a list of possible correct values that a field may use

* DI: Direction Indicator is a differentiator for matching in order to be able to have different values for both sides.

* IID: Interface Identifier. See the IPv6 addressing architecture {{RFC7136}}

* Dev-IID: Device Interface Identifier. Second part of the IPv6 address to identify the device interface

* APP-IID: Application Interface Identifier. Second part of the IPv6 address to identify the application interface

* Dw: Down Link direction for compression, from SCHC C/D to Dev

* Up: Up Link direction for compression, from Dev to SCHC C/D

* Bi: Bidirectional, it can be used in both senses

# Static Context Header Compression

Static Context Header Compression (SCHC) avoids context synchronization,
which is the most bandwidth-consuming operation in other header compression mechanisms
such as RoHC {{RFC5795}}. Based on the fact
that the nature of data flows is highly predictable in LPWAN networks, a static
context may be stored on the Device (Dev). The context must be stored in both ends, and it can 
either be learned by a provisioning protocol or by out of band means or it can be pre-provosioned, etc. 
The way the context is learned on both sides is out of the scope of this document.


~~~~
     Dev                                                 App
+---------------+                                  +---------------+
| APP1 APP2 APP3|                                  |APP1  APP2 APP3|
|               |                                  |               |
|       UDP     |                                  |      UDP      | 
|      IPv6     |                                  |     IPv6      |   
|               |                                  |               |  
|    SCHC C/D   |                                  |               |  
|    (context)  |                                  |               | 
+--------+------+                                  +-------+-------+ 
         |   +--+     +----+     +---------+               .
         +~~ |RG| === |NGW | === |SCHC C/D |... Internet ...
             +--+     +----+     |(context)| 
                                 +---------+
~~~~
{: #Fig-archi title='Architecture'}

{{Fig-archi}} based on {{I-D.ietf-lpwan-overview}} terminology represents the architecture for 
compression/decompression. The Device is sending applications flows using IPv6 or IPv6/UDP protocols. These flows are compressed by an Static Context Header Compression Compressor/Decompressor (SCHC C/D) to reduce headers size. Resulting
information is sent on a layer two (L2) frame to the LPWAN Radio Network to a Radio Gateway (RG) which forwards 
the frame to a Network Gateway (NGW).
The NGW sends the data to a SCHC C/D for decompression which shares the same rules with the Dev. The SCHC C/D can be 
located on the Network Gateway (NGW) or in another place as long as a tunnel is established between the NGW and the SCHC C/D. SCHC C/D in both sides must share the same set of Rules.
After decompression, the packet can be sent on the Internet to one
or several LPWAN Application Servers (App). 

The SCHC C/D process is bidirectional, so the same principles can be applied in the other direction.

## SCHC Rules

The main idea of the SCHC compression scheme is to send the Rule id to the other end that match as much as possible the original packet values instead 
of sending known field values. When a value is known by both
ends, it is not necessary sent through the LPWAN network. 

The context contains a list of rules (cf. {{Fig-ctxt}}). Each Rule contains 
itself a list of fields descriptions composed of a field identifier (FID), a field position (FP), a direction indicator (DI), 
a target value (TV), a matching operator (MO) and a Compression/Decompression Action
(CDA).


~~~~
  /--------------------------------------------------------------\
  |                      Rule N                                  |
 /--------------------------------------------------------------\|
 |                    Rule i                                    ||
/--------------------------------------------------------------\||
|  (FID)         Rule 1                                        |||
|+-------+--+--+------------+-----------------+---------------+|||
||Field 1|FP|DI|Target Value|Matching Operator|Comp/Decomp Act||||
|+-------+--+--+------------+-----------------+---------------+|||
||Field 2|FP|DI|Target Value|Matching Operator|Comp/Decomp Act||||
|+-------+--+--+------------+-----------------+---------------+|||
||...    |..|..|   ...      | ...             | ...           ||||
|+-------+--+--+------------+-----------------+---------------+||/
||Field N|FP|DI|Target Value|Matching Operator|Comp/Decomp Act|||
|+-------+--+--+------------+-----------------+---------------+|/
|                                                              |
\--------------------------------------------------------------/
~~~~
{: #Fig-ctxt title='Compression/Decompression Context'}


The Rule does not describe the original packet format which
must be known from the compressor/decompressor. The rule just describes the
compression/decompression behavior for the header fields. In the rule, the description of the header field must be performed in the format packet order.

The Rule describes also the compressed header fields which are transmitted regarding their position
in the rule which is used for data serialization on the compressor side and data deserialization on the decompressor side.

The Context describes the header fields and its values with the following entries:

* A Field ID (FID) is a unique value to define the header field. In the context the name of the header field is not used, only the identifier.

* A Field Position (FP) indicating if several instances of the field exist in the 
  headers which one is targeted. The default position is 1
  
* A direction indicator (DI) indicating the packet direction. Three values are possible:

  * UP LINK (Up) when the field or the value is only present in packets sent by the Dev to the App,

  * DOWN LINK (Dw) when the field or the value is only present in packet sent from the App to the Dev and 

  * BIDIRECTIONAL (Bi) when the field or the value is present either upstream or downstream. 

* A Target Value (TV) is the value used to make the comparison with
  the packet header field. The Target Value can be of any type (integer, strings,...).
  For instance, it can be a single value or a more complex structure (array, list,...). It can
  be considered as a CBOR structure.

* A Matching Operator (MO) is the operator used to make the comparison between 
  the Field Value and the Target Value. The Matching Operator may require some 
  parameters. MO is only used during the compression phase.

* A Compression Decompression Action (CDA) is used to describe the compression
  and the decompression process. The CDA may require some 
  parameters, CDA are used in both compression and decompression phases. 

## Rule ID

Rule IDs are sent between both compression/decompression elements. The size 
of the Rule ID is not specified in this document, it is implementation-specific and can vary regarding the
LPWAN technology, the number of flows, among others. 

Some values in the Rule ID space may be reserved for goals other than header 
compression as fragmentation. (See {{Frag}}). 

Rule IDs are specific to a Dev. Two Devs may use the same Rule ID for different
header compression. To identify the correct Rule ID, the SCHC C/D needs to combine the Rule ID with the Dev L2 identifier
to find the appropriate Rule.

## Packet processing

The compression/decompression process follows several steps:

* compression Rule selection: The goal is to identify which Rule(s) will be used
  to compress the packet's headers. When doing compression from Dw to Up the SCHC C/D needs to find the 
  correct Rule to use by identifying its Dev-ID and the Rule-ID. In the Up situation only the Rule-ID is used. 
  The next step is to choose the fields by their direction, using the 
  direction indicator (DI), so the fields that does not correspond to the DI will be excluded. 
  Next, then fields are identified according to their field identifier (FID) and field position (FP). 
  If the field position does not correspond then the Rule is not use and the SCHC take next Rule.
  Once the DI and the FP correspond to the header information, each field's value is then compared to the corresponding 
  target value (TV) stored in the Rule for that specific field using the matching operator (MO).
  If all the fields in the packet's header satisfy all the matching operators (MOs) of a Rule (i.e. all results are True),
  the fields of the header are then processed according to the Compression/Decompession Actions (CDAs) 
  and a compressed header is obtained. Otherwise the next rule is tested. 
  If no eligible rule is found, then the header must be sent without compression, in which case the fragmentation process 
  must be required.

* sending: The Rule ID is sent to the other end followed by information resulting
  from the compression of header fields. This information is sent in the order expressed in the Rule for the matching
  fields. The way the Rule ID is sent depends on the specific LPWAN
  layer two technology and will be specified in a specific document, and is out of the scope of this document. 
  For example, it can be either included in a Layer 2 header or sent in the first byte of
  the L2 payload. (cf. {{Fig-FormatPckt}}). 

* decompression: In both directions, The receiver identifies the sender through its device-id
  (e.g. MAC address) and selects the appropriate Rule through the Rule ID. This
  Rule gives the compressed header format and associates these values to header fields.
  It applies the CDA action to reconstruct the original
  header fields. The CDA application order can be different of the order given by the Rule. For instance
  Compute-\* may be applied at end, after the other CDAs.

~~~~
 
+--- ... ---+-------------- ... --------------+
|  Rule ID  |Compressed Hdr Fields information|
+--- ... ---+-------------- ... --------------+

~~~~
{: #Fig-FormatPckt title='LPWAN Compressed Format Packet'}


# Matching operators {#chap-MO}

Matching Operators (MOs) are functions used by both SCHC C/D endpoints involved in the header 
compression/decompression. They are not typed and can be applied indifferently to integer, string 
or any other data type. The result of the operation can either be True or False. MOs are defined as follows: 

* equal: A field value in a packet matches with a TV in a Rule if they are equal.

* ignore: No check is done between a field value in a packet and a TV
  in the Rule. The result of the matching is always true.

* MSB(length): A matching is obtained if the most significant bits of the length field value bits of the header are equal to the TV in the rule.
  
* match-mapping: The goal of mapping-sent is to reduce the size of a field by allocating
  a shorter value. The Target Value contains a list of values. Each value is idenfied by a short ID (or index). 
  This operator matches if a field value is equal to one of those target values. The MSB Matching Operator needs a parameter,
  indicating the number of bits, to proceed to the matching.

# Compression Decompression Actions (CDA) {#chap-CDA}

The Compression Decompression Actions (CDA) describes the action taken during
the compression of headers fields, and inversely, the action taken by the decompressor to restore
the original value.

~~~~
/--------------------+-------------+----------------------------\
|  Action            | Compression | Decompression              |
|                    |             |                            |
+--------------------+-------------+----------------------------+
|not-sent            |elided       |use value stored in ctxt    |
|value-sent          |send         |build from received value   |
|mapping-sent        |send index   |value from index on a table |
|LSB(length)         |send LSB     |TV OR received value        |
|compute-length      |elided       |compute length              |
|compute-checksum    |elided       |compute UDP checksum        |
|Deviid-DID          |elided       |build IID from L2 Dev addr  |
|Appiid-DID          |elided       |build IID from L2 App addr  |
\--------------------+-------------+----------------------------/

~~~~
{: #Fig-function title='Compression and Decompression Functions'}

{{Fig-function}} sumarizes the basics functions defined to compress and decompress
a field. The first column gives the action's name. The second and third
columns outlines the compression/decompression behavior.

Compression is done in the rule order and compressed values are sent in that order in the compressed
message. The receiver must be able to find the size of each compressed field
which can be given by the rule or may be sent with the compressed header. 

If a field is identified as variable, then its size is sent before the value as indicated in each CDAs

## not-sent CDA

Not-sent function is generally used when the field value is specified in the rule and
therefore known by the both Compressor and Decompressor. This action is generally used with the
"equal" MO. If MO is "ignore", there is a risk to have a decompressed field
value different from the compressed field.

The compressor does not send any value on the compressed header for the field on which compression is applied.

The decompressor
restores the field value with the target value stored in the matched rule.

## value-sent CDA

The value-sent action is generally used when the field value is not known by both Compressor and Decompressor.
The value is sent in the compressed message header. Both Compressor and Decompressor must know the
size of the field, either implicitly (the size is known by both sides) 
or explicitly in the compressed header
field by indicating the length. This function is generally used with the "ignore" MO.

The compressor sends the Target Value stored on the rule in the compressed
header message. The decompressor restores the field value with the one received 
from the LPWAN 

## mapping-sent

mapping-sent is used to send a smaller index associated to the list of values
in the Target Value. This function is used together with the "match-mapping" MO.

The compressor looks in the TV to find the field value and send the corresponding index.
The decompressor uses this index to restore the field value.

The number of bits sent is the minimal size to code all the possible indexes.

## LSB CDA

LSB action is used to avoid sendind the fixed part of the packet field header to the other end.
This action is used together with the "MSB" MO. A length can be specified in the rule to indicate
how many bits have to be sent. If not length is specified, the number of bits sent are the
field length minus the bits length specified in the MSB MO.

The compressor sends the "length" Least Significant Bits. The decompressor
combines with an OR operator the value received with the Target Value.


## DEViid, APPiid CDA

These functions are used to process respectively the Dev and the App Interface Identifiers (Deviid and Appiid) of the 
IPv6 addresses. Appiid CDA is less common, since current LPWAN technologies
frames contain a single address.

The IID value can be computed from the Device ID present in the Layer 2 header. The
computation is specific for each LPWAN technology and depends on the Device ID size.

In the downstream direction, these CDA are used to determine the L2 addresses used by the LPWAN.

## Compute-\*

Thes classes of functions are used by the decompressor to compute the compressed field value based on received information. 
Compressed fields are elided during compression and reconstructed during decompression.

* compute-length: compute the length assigned to this field. For instance, regarding
  the field ID, this CDA may be used to compute IPv6 length or UDP length.

* compute-checksum: compute a checksum from the information already received by the SCHC C/D.
  This field may be used to compute UDP checksum.

# Application to IPv6 and UDP headers

This section lists the different IPv6 and UDP header fields and how they can be compressed.

## IPv6 version field

This field always holds the same value, therefore the TV is 6, the MO is "equal" 
and the "CDA "not-sent"".

## IPv6 Traffic class field

If the DiffServ field identified by the rest of the rule do not vary and is known 
by both sides, the TV should contain this well-known value, the MO should be "equal" 
and the CDA must be "not-sent.

If the DiffServ field identified by the rest of the rule varies over time or is not 
known by both sides, then there are two possibilities depending on the variability of the value, 
the first one is to do not compressed the field and sends the original value, or 
the second where the values can be computed by sending only the LSB bits:

* TV is not set to any value, MO is set to "ignore" and CDA is set to "value-sent"

* TV contains a stable value, MO is MSB(X) and CDA is set to LSB

## Flow label field

If the Flow Label field identified by the rest of the rule does not vary and is known 
by both sides, the TV should contain this well-known value, the MO should be "equal" 
and the CDA should be "not-sent".

If the Flow Label field identified by the rest of the rule varies during time or is not 
known by both sides, there are two possibilities depending on the variability of the value,
the first one is without compression and then the value is sent 
and the second where only part of the value is sent and the decompressor needs to compute the original value:

* TV is not set, MO is set to "ignore" and CDA is set to "value-sent"

* TV contains a stable value, MO is MSB(X) and CDA is set to LSB

## Payload Length field

If the LPWAN technology does not add padding, this field can be elided for the 
transmission on the LPWAN network. The SCHC C/D recomputes the original payload length
value. The TV is not set, the MO is set to "ignore" and the CDA is "compute-IPv6-length".

If the payload is small, the TV can be set to 0x0000, the MO set to "MSB (16-s)" and the
CDA to "LSB (s)". The 's' parameter depends on the maximum packet length.

On other cases, the payload length field must be sent and the CDA is replaced by "value-sent".

## Next Header field

If the Next Header field identified by the rest of the rule does not vary and is known 
by both sides, the TV should contain this Next Header value, the MO should be "equal" 
and the CDA should be "not-sent".

If the Next header field identified by the rest of the rule varies during time or is not 
known by both sides, then TV is not set, MO is set to "ignore" and CDA is set to 
"value-sent". A matching-list may also be used.

## Hop Limit field

The End System is generally a device and does not forward packets, therefore the
Hop Limit value is constant. So the TV is set with a default value, the MO 
is set to "equal" and the CDA is set to "not-sent".

Otherwise the value is sent on the LPWAN: TV is not set, MO is set to ignore and 
CDA is set to "value-sent".

Note that the field behavior differs in upstream and downstream. In upstream, since there is 
no IP forwarding between the Dev and the SCHC C/D, the value is relatively constant. On the
other hand, the downstream value depends of Internet routing and may change more frequently.
One solution could be to use the Direction Indicator (DI) to distinguish both directions to
elide the field in the upstream direction and send the value in the downstream direction.

## IPv6 addresses fields

As in 6LoWPAN {{RFC4944}}, IPv6 addresses are split into two 64-bit long fields; 
one for the prefix and one for the Interface Identifier (IID). These fields should
be compressed. To allow a single rule, these values are identified by their role 
(DEV or APP) and not by their position in the frame (source or destination). The SCHC C/D
must be aware of the traffic direction (upstream, downstream) to select the appropriate
field.

### IPv6 source and destination prefixes

Both ends must be synchronized with the appropriate prefixes. For a specific flow, 
the source and destination prefix can be unique and stored in the context. It can 
be either a link-local prefix or a global prefix. In that case, the TV for the 
source and destination prefixes contains the values, the MO is set to "equal" and
the CDA is set to "not-sent".

In case the rule allows several prefixes, mapping-list must be used. The 
different prefixes are listed in the TV associated with a short ID. The MO is set 
to "match-mapping" and the CDA is set to "mapping-sent".

Otherwise the TV contains the prefix, the MO is set to "equal" and the CDA is set to
value-sent.

### IPv6 source and destination IID

If the DEV or APP IID are based on an LPWAN address, then the IID can be reconstructed 
with information coming from the LPWAN header. In that case, the TV is not set, the MO 
is set to "ignore" and the CDA is set to "DEViid" or "APPiid". Note that the 
LPWAN technology is generally carrying a single device identifier corresponding
to the DEV. The SCHC C/D may also not be aware of these values. 

If the DEV address has a static value that is not derivated from the EUI-64, 
then TV contains the value, the MO operator is set to
"equal" and the CDA is set to "not-sent". 

If several IIDs are possible, then the TV contains the list of possible IIDs, the MO is 
set to "match-mapping" and the CDA is set to "mapping-sent". 

Otherwise the value variation of the IID may be reduced to few bytes. In that case, the TV is
set to the stable part of the IID, the MO is set to MSB and the CDA is set to LSB.

Finally, the IID can be sent on the LPWAN. In that case, the TV is not set, the MO is set
to "ignore" and the CDA is set to "value-sent".

## IPv6 extensions

No extension rules are currently defined. They can be based on the MOs and 
CDAs described above.

## UDP source and destination port

To allow a single rule, the UDP port values are identified by their role 
(DEV or APP) and not by their position in the frame (source or destination). The SCHC C/D
must be aware of the traffic direction (upstream, downstream) to select the appropriate
field. The following rules apply for DEV and APP port numbers.

If both ends know the port number, it can be elided. The TV contains the port number,
the MO is set to "equal" and the CDA is set to "not-sent".

If the port variation is on few bits, the TV contains the stable part of the port number,
the MO is set to "MSB" and the CDA is set to "LSB".

If some well-known values are used,  the TV can contain the list of this values, the
MO is set to "match-mapping" and the CDA is set to "mapping-sent".

Otherwise the port numbers are sent on the LPWAN. The TV is not set, the MO is 
set to "ignore" and the CDA is set to "value-sent".

## UDP length field

If the LPWAN technology does not introduce padding, the UDP length can be computed
from the received data. In that case the TV is not set, the MO is set to "ignore" and
the CDA is set to "compute-UDP-length".

If the payload is small, the TV can be set to 0x0000, the MO set to "MSB" and the
CDA to "LSB". 

On other cases, the length must be sent and the CDA is replaced by "value-sent".

## UDP Checksum field

IPv6 mandates a checksum in the protocol above IP. Nevertheless, if a more efficient
mechanism such as L2 CRC or MIC is carried by or over the L2 (such as in the 
LPWAN fragmentation process (see section {{Frag}})), the UDP checksum transmission can be avoided.
In that case, the TV is not set, the MO is set to "ignore" and the CDA is set to
"compute-UDP-checksum".

In other cases the checksum must be explicitly sent. The TV is not set, the MO is set to
"ignore" and the CDF is set to "value-sent".



# Examples {#compressIPv6}


This section gives some scenarios of the compression mechanism for IPv6/UDP.
The goal is to illustrate the SCHC behavior.

## IPv6/UDP compression 

The most common case using the mechanisms defined in this document will be a 
LPWAN Dev that embeds some applications running over
CoAP. In this example, three flows are considered. The first flow is for the device management based
on CoAP using
Link Local IPv6 addresses and UDP ports 123 and 124 for Dev and App, respectively.
The second flow will be a CoAP server for measurements done by the Device
(using ports 5683) and Global IPv6 Address prefixes alpha::IID/64 to beta::1/64.
The last flow is for legacy applications using different ports numbers, the
destination IPv6 address prefix is gamma::1/64.

 {{FigStack}} presents the protocol stack for this Device. IPv6 and UDP are represented
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
|      LPWAN L2 technologies   |
+------------------------------+
         DEV or NGW

~~~~
{: #FigStack title='Simplified Protocol Stack for LP-WAN'}


Note that in some LPWAN technologies, only the Devs have a device ID.
Therefore, when such technologies are used, it is necessary to define statically an IID for the Link
Local address for the SCHC C/D.



~~~~
  Rule 0
  +----------------+--+--+---------+--------+-------------++------+
  | Field          |FP|DI| Value   | Match  | Function    || Sent |
  +----------------+--+--+---------+----------------------++------+
  |IPv6 version    |1 |Bi|6        | equal  | not-sent    ||      |
  |IPv6 DiffServ   |1 |Bi|0        | equal  | not-sent    ||      |
  |IPv6 Flow Label |1 |Bi|0        | equal  | not-sent    ||      |
  |IPv6 Length     |1 |Bi|         | ignore | comp-length ||      |
  |IPv6 Next Header|1 |Bi|17       | equal  | not-sent    ||      |
  |IPv6 Hop Limit  |1 |Bi|255      | ignore | not-sent    ||      |
  |IPv6 DEVprefix  |1 |Bi|FE80::/64| equal  | not-sent    ||      |
  |IPv6 DEViid     |1 |Bi|         | ignore | DEViid-DID  ||      |
  |IPv6 APPprefix  |1 |Bi|FE80::/64| equal  | not-sent    ||      |
  |IPv6 APPiid     |1 |Bi|::1      | equal  | not-sent    ||      |
  +================+==+==+=========+========+=============++======+
  |UDP DEVport     |1 |Bi|123      | equal  | not-sent    ||      |
  |UDP APPport     |1 |Bi|124      | equal  | not-sent    ||      |
  |UDP Length      |1 |Bi|         | ignore | comp-length ||      |
  |UDP checksum    |1 |Bi|         | ignore | comp-chk    ||      |
  +================+==+==+=========+========+=============++======+

  Rule 1
  +----------------+--+--+---------+--------+-------------++------+
  | Field          |FP|DI| Value   | Match  | Function    || Sent |
  +----------------+--+--+---------+--------+-------------++------+
  |IPv6 version    |1 |Bi|6        | equal  | not-sent    ||      |
  |IPv6 DiffServ   |1 |Bi|0        | equal  | not-sent    ||      |
  |IPv6 Flow Label |1 |Bi|0        | equal  | not-sent    ||      |
  |IPv6 Length     |1 |Bi|         | ignore | comp-length ||      |
  |IPv6 Next Header|1 |Bi|17       | equal  | not-sent    ||      |
  |IPv6 Hop Limit  |1 |Bi|255      | ignore | not-sent    ||      |
  |IPv6 DEVprefix  |1 |Bi|alpha/64 | equal  | not-sent    ||      |
  |IPv6 DEViid     |1 |Bi|         | ignore | DEViid-DID  ||      |
  |IPv6 APPprefix  |1 |Bi|beta/64  | equal  | not-sent    ||      |
  |IPv6 APPiid     |1 |Bi|::1000   | equal  | not-sent    ||      |
  +================+==+==+=========+========+=============++======+
  |UDP DEVport     |1 |Bi|5683     | equal  | not-sent    ||      |
  |UDP APPport     |1 |Bi|5683     | equal  | not-sent    ||      |
  |UDP Length      |1 |Bi|         | ignore | comp-length ||      |
  |UDP checksum    |1 |Bi|         | ignore | comp-chk    ||      |
  +================+==+==+=========+========+=============++======+

  Rule 2
  +----------------+--+--+---------+--------+-------------++------+
  | Field          |FP|DI| Value   | Match  | Function    || Sent |
  +----------------+--+--+---------+--------+-------------++------+
  |IPv6 version    |1 |Bi|6        | equal  | not-sent    ||      |
  |IPv6 DiffServ   |1 |Bi|0        | equal  | not-sent    ||      |
  |IPv6 Flow Label |1 |Bi|0        | equal  | not-sent    ||      |
  |IPv6 Length     |1 |Bi|         | ignore | comp-length ||      |
  |IPv6 Next Header|1 |Bi|17       | equal  | not-sent    ||      |
  |IPv6 Hop Limit  |1 |Bi|255      | ignore | not-sent    ||      |
  |IPv6 DEVprefix  |1 |Bi|alpha/64 | equal  | not-sent    ||      |
  |IPv6 DEViid     |1 |Bi|         | ignore | DEViid-DID  ||      |
  |IPv6 APPprefix  |1 |Bi|gamma/64 | equal  | not-sent    ||      |
  |IPv6 APPiid     |1 |Bi|::1000   | equal  | not-sent    ||      |
  +================+==+==+=========+========+=============++======+
  |UDP DEVport     |1 |Bi|8720     | MSB(12)| LSB(4)      || lsb  |
  |UDP APPport     |1 |Bi|8720     | MSB(12)| LSB(4)      || lsb  |
  |UDP Length      |1 |Bi|         | ignore | comp-length ||      |
  |UDP checksum    |1 |Bi|         | ignore | comp-chk    ||      |
  +================+==+==+=========+========+=============++======+


~~~~
{: #Fig-fields title='Context rules'}

All the fields described in the three rules depicted on {{Fig-fields}} are present
in the IPv6 and UDP headers.  The DEViid-DID value is found in the L2
header.

The second and third rules use global addresses. The way the Dev learns the
prefix is not in the scope of the document. 

The third rule compresses port numbers to 4 bits. 

# Fragmentation {#Frag}

## Overview

Fragmentation support in LPWAN is mandatory when the underlying LPWAN technology is not capable of fulfilling the IPv6 MTU requirement. Fragmentation is used if, after SCHC header compression, the size of the resulting IPv6 packet is larger than the L2 data unit maximum payload. Fragmentation is also used if SCHC header compression has not been able to compress an IPv6 packet that is larger than the L2 data unit maximum payload. In LPWAN technologies, the L2 data unit size typically varies from tens to hundreds of bytes. 
If the entire IPv6 datagram fits within a single L2 data unit, the fragmentation mechanism is not used and the packet is sent unfragmented.  
If the datagram does not fit within a single L2 data unit, it SHALL be broken into fragments. 

Moreover, LPWAN technologies impose some strict limitations on traffic; 
therefore it is desirable to enable optional fragment retransmission, while 
a single fragment loss should not lead to retransmitting the full IPv6 datagram. 
On the other hand, in order to preserve energy, Devices are sleeping most of the time and 
may receive data during a short period of time after transmission. In 
order to adapt to the capabilities of various LPWAN technologies,
this specification allows for a gradation of fragment delivery
reliability. This document does not make any decision with regard to which 
fragment delivery reliability option is used over a specific LPWAN 
technology.

An important consideration is that LPWAN networks typically follow the star topology, and therefore data unit reordering is not expected in such networks. This specification assumes that reordering will not happen between the entity performing fragmentation and the entity performing reassembly. This assumption allows to reduce complexity and overhead of the fragmentation mechanism.

## Reliability options: definition 

This specification defines the following three fragment delivery reliability options:

   o  No ACK

   o  Window mode - ACK “always”

   o  Window mode - ACK on error 

   The same reliability option MUST be used for all fragments of a packet. It is up to implementers and/or representatives of the underlying LPWAN technology to decide which reliability option to use and whether the same reliability option applies to all IPv6 packets or not.  Note that the reliability option to be used is not necessarily tied to the particular characteristics of the underlying L2 LPWAN technology (e.g. the No ACK reliability option may be used on top of an L2 LPWAN technology with symmetric characteristics for uplink and downlink).
   
   In the No ACK option, the receiver MUST NOT issue acknowledgments (ACK). 

   In Window mode – ACK “always”, an ACK is transmitted by the fragment
   receiver after a window of fragments have been sent.  A window of
   fragments is a subset of the full set of fragments needed to carry an
   IPv6 packet.  In this mode, the ACK informs the sender about received
   and/or missing fragments from the window of fragments.

   In Window mode – ACK on error, an ACK is transmitted by the fragment
   receiver after a window of fragments have been sent, only if at least one 
   of the fragments in the window has been lost. In this mode, the ACK informs the sender about received and/or missing fragments from the window of fragments.
  
   In Window mode, upon receipt of an ACK that informs about any lost fragments, the sender retransmits the lost fragments. The maximum number of ACKs to be sent by the receiver for a specific window, denoted MAX_ACKS_PER_WINDOW, is not stated in this document, and it is expected to be defined in other documents (e.g. technology-specific profiles).

   This document does not make any decision as to which fragment delivery 
   reliability option(s) need to be supported over a specific LPWAN 
   technology.  
   
   Examples of the different reliability options described are provided in Appendix A.

## Reliability options: discussion

This section discusses the properties of each fragment delivery 
   reliability option defined in the previous section. 

   No ACK is the most simple fragment delivery reliability option. With this 
   option, the receiver does not generate overhead in the form of ACKs. 
   However, this option does not enhance delivery reliability beyond that 
   offered by the underlying LPWAN technology.

   The Window mode - ACK on error option is based on the optimistic expectation that the underlying links will offer relatively low L2 data unit loss probability. This option reduces the number of ACKs transmitted by the fragment receiver compared to the Window mode - ACK “always” option. This may be especially beneficial in asymmetric scenarios, e.g. where fragmented data are sent uplink and the underlying LPWAN technology downlink capacity or message rate is lower than the uplink one. However, if an ACK is lost, the sender assumes that all fragments covered by the ACK have been successfully delivered. In contrast, the Window mode - ACK “always” option does not suffer that issue, at the expense of an ACK overhead increase. 
   
The Window mode – ACK “always” option provides flow control. In addition, it is able to handle long bursts of lost fragments, since detection of such events can be done before end of the IPv6 packet transmission, as long as the window size is short enough. However, such benefit comes at the expense of higher ACK overhead.

## Tools

This subsection describes the different tools that are used to enable the described fragmentation functionality and the different reliability options supported. Each tool has a corresponding header field format that is defined in the next subsection. The list of tools follows: 

o  Rule ID. The Rule ID is used in fragments and in ACKs. The Rule ID in a fragment is set to a value that indicates that the data unit being carried is a fragment. This also allows to interleave non-fragmented IPv6 datagrams with fragments that carry a larger IPv6 datagram. Rule ID may also be used to signal which reliability option is in use for the IPv6 packet being carried. In an ACK, the Rule ID signals that the message this Rule ID is prepended to is an ACK.  

o  Compressed Fragment Number (CFN). The CFN is included in all fragments. This field can be understood as a truncated, efficient representation of a larger-sized fragment number, and does not necessarily carry an absolute fragment number. A special CFN value signals the last fragment that carries a fragmented IPv6 packet. In Window mode, the CFN is augmented with the W bit, which has the purpose of avoiding possible ambiguity for the receiver that might arise under certain conditions

o  Datagram Tag (DTag). The DTag field, if present, is set to the same value for all fragments carrying the same IPv6 datagram, allows to interleave fragments that correspond to different IPv6 datagrams.

o  Message Integrity Check (MIC). It is computed by the sender over the complete IPv6 packet before fragmentation by using the TBD algorithm. The MIC allows the receiver to check for errors in the reassembled IPv6 packet, while it also enables compressing the UDP checksum by use of SCHC.

o  Bitmap. The bitmap is a sequence of bits included in the ACK for a given window, that provides feedback on whether each fragment of the current window has been received or not.

## Formats

This section defines the fragment format, the fragmentation header formats, and the ACK format.

### Fragment format

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

### Fragmentation header formats

In the No ACK option, fragments except the last one SHALL contain the fragmentation header as defined in {{Fig-NotLast}}. The total size of this fragmentation header is R bits. 
   
~~~~
             <------------ R ---------->
                         <--T--> <--N-->
             +-- ... --+- ...  -+- ... -+
             | Rule ID |  DTag  |  CFN  |
             +-- ... --+- ...  -+- ... -+

~~~~
{: #Fig-NotLast title='Fragmentation Header for Fragments except the Last One, No ACK option'}



In any of the Window mode options, fragments except the last one SHALL    
   contain the fragmentation header as defined in {{Fig-NotLastWin}}. The total size of this fragmentation header is R bits. 
   
~~~~
             <------------ R ---------->
                       <--T--> 1 <--N-->
            +-- ... --+- ... -+-+- ... -+
            | Rule ID | DTag  |W|  CFN  |
            +-- ... --+- ... -+-+- ... -+

~~~~
{: #Fig-NotLastWin title='Fragmentation Header for Fragments except the Last One, Window mode'}
   
   
   The last fragment of an IPv6 datagram SHALL contain a fragmentation header that conforms to 
   the format shown in {{Fig-Last}}. The total size of this fragmentation 
   header is R+M bits. 

~~~~
              <------------- R ------------>
                            <- T -> <- N -> <---- M ----->
              +---- ... ---+- ... -+- ... -+---- ... ----+
              |   Rule ID  | DTag  | 11..1 |     MIC     |
              +---- ... ---+- ... -+- ... -+---- ... ----+
~~~~
{: #Fig-Last title='Fragmentation Header for the Last Fragment'}


   * Rule ID: This field has a size of R - T - N - 1 bits in all fragments that are not the last one, when Window mode is used. In all other fragments, the Rule ID field has a size of R – T – N bits. 
   
   * DTag: The size of the DTag field is T bits, which may be set to a value greater than or equal to 0 bits. The DTag field in all fragments that carry the same IPv6 datagram MUST be set to the same value. DTag MUST be set sequentially increasing from 0 to 2^T - 1, and MUST wrap back from 2^T - 1 to 0.

   * CFN: This field is an unsigned integer, with a size of N bits, that carries the CFN of the fragment. In the No ACK option, N=1. For the rest of options, N equal to or greater than 3 is recommended. The CFN MUST be set sequentially decreasing from the highest CFN in the window (which will be used for the first fragment), and MUST wrap from 0 back to the highest CFN in the window. The highest CFN in the window MUST be a value equal to or smaller than 2^N-2. (Example 1: for N=5, the highest CFN value may be configured to be 30, then subsequent CFNs are set sequentially and in decreasing order, and CFN will wrap from 0 back to 30. Example 2: for N=5, the highest CFN value may be set to 23, then subsequent CFNs are set sequentially and in decreasing order, and the CFN will wrap from 0 back to 23). The CFN for the last fragment has all bits set to 1. Note that, by this definition, the CFN value of 2^N - 1 is only used to identify a fragment as the last fragment carrying a subset of the IPv6 packet being transported, and thus the CFN does not strictly correspond to the N least significant bits of the actual absolute fragment number. It is also important to note that, for N=1, the last fragment of the packet will carry a CFN equal to 1, while all previous fragments will carry a CFN of 0.
      
   * W: W is a 1-bit field. This field carries the same value for all fragments of a window, and it is complemented for the next window. The initial value for this field is 1.

   * MIC: This field, which has a size of M bits, carries the MIC for the IPv6 packet.
   
   The values for R, N, T and M are not specified in this document, and have to be determined in other documents (e.g. technology-specific profile documents).
   
### ACK format

The format of an ACK is shown in {{Fig-ACK-Format}}:

~~~~
                <-------  R  ------>
                             <- T ->  
                +---- ... --+-... -+----- ... ---+
                |  Rule ID  | DTag |   bitmap    |
                +---- ... --+-... -+----- ... ---+
~~~~
{: #Fig-ACK-Format title='Format of an ACK'}


  Rule ID: In all ACKs, Rule ID has a size of R - T bits.
   
  DTag: DTag has a size of T bits. DTag carries the same value as the DTag field in the fragments carrying the IPv6 datagram for which this ACK is intended.

  bitmap: This field carries the bitmap sent by the receiver to inform the sender about whether fragments in the current window have been received or not. size of the bitmap field of an ACK can be equal to 0 or Ceiling(Number_of_Fragments/8) octets, where Number_of_Fragments denotes the number of fragments of a window. The bitmap is a sequence of bits, where the n-th bit signals whether the n-th fragment transmitted in the current window has been correctly received (n-th bit set to 1) or not (n-th bit set to 0). Remaining bits with bit order greater than the number of fragments sent (as determined by the receiver) are set to 0, except for the last bit in the bitmap, which is set to 1 if the last fragment has been correctly received, and 0 otherwise. Absence of the bitmap in an ACK confirms correct reception of all fragments to be acknowledged by means of the ACK.
  
   
{{Fig-Bitmap-Win}} shows an example of an ACK (N=3), where the bitmap
indicates that the second and the fifth fragments have not been correctly received. 

~~~~                                                  
              <------  R  ------>
                          <- T -> 0 1 2 3 4 5 6 7
              +---- ... --+-... -+-+-+-+-+-+-+-+-+
              |  Rule ID  | DTag |1|0|1|1|0|1|1|1|
              +---- ... --+-... -+-+-+-+-+-+-+-+-+

~~~~
{: #Fig-Bitmap-Win title='Example of the bitmap in an ACK (in Window mode, for N=3)'}

{{Fig-NoBitmap}} illustrates an ACK without a bitmap.

~~~~
                    <------  R  ------>
                                <- T -> 
                    +---- ... --+-... -+
                    |  Rule ID  | DTag |
                    +---- ... --+-... -+

~~~~
{: #Fig-NoBitmap title='Example of an ACK without a bitmap'}

Note that, in order to exploit the available L2 payload space to the fullest, a bitmap may have a size smaller than 2^N bits. In that case, the window in use will have a size lower than 2^N-1 fragments. For example, if the maximum available space for a bitmap is 56 bits, N can be set to 6, and the window size can be set to a maximum of 56 fragments.  

## Baseline mechanism

The receiver of link fragments SHALL use (1) the sender's L2 source address (if present), (2) the destination's L2 address (if present), (3) Rule ID and (4) DTag to identify all the fragments that belong to a given IPv6 datagram. The fragment receiver may determine the fragment delivery reliability option in use for the fragment based on the Rule ID field in that fragment.

Upon receipt of a link fragment, the receiver starts constructing the original unfragmented packet. It uses the CFN and the order of arrival of each fragment to determine the location of the individual fragments within the original unfragmented packet. For example, it may place the data payload of the fragments within a payload datagram reassembly buffer at the location determined from the CFN and order of arrival of the fragments, and the fragment payload sizes. In Window mode, the fragment receiver also uses the W bit in the received fragments. Note that the size of the original, unfragmented IPv6 packet cannot be determined from fragmentation headers.

When Window mode - ACK on error is used, the fragment receiver starts a timer (denoted "ACK on Error Timer") upon reception of the first fragment for an IPv6 datagram. The initial value for this timer is not provided by this specification, and is expected to be defined in additional documents. This timer is reset every time that a new fragment carrying data from the same IPv6 datagram is received. In Window mode – ACK on error, upon timer expiration, if neither the last fragment of the IPv6 datagram nor the last fragment of the current window (i.e. with CFN=0) have been received, an ACK MUST be transmitted by the fragment receiver to indicate received and not received fragments for the current window.

Note that, in Window mode, the first fragment of the window is the one sent with CFN=2^N-2. Also note that, in Window mode, the fragment with CFN=0 is considered the last fragment of its window, except for the last fragment of the whole packet (with all CFN bits set to 1), which is also the last fragment of the last window. Upon receipt of the last fragment of a window, if Window mode – ACK “Always” is used, the fragment receiver MUST send an ACK to the fragment sender. The ACK provides feedback on the fragments received and lost that correspond to the last window.

If the recipient receives the last fragment of an IPv6 datagram, it checks for the integrity of the reassembled IPv6 datagram, based on the MIC received. In No ACK mode, if the integrity check indicates that the reassembled IPv6 datagram does not match the original IPv6 datagram (prior to fragmentation), the reassembled IPv6 datagram MUST be discarded. If Window mode - ACK “Always” is used, the recipient MUST transmit an ACK to the fragment sender. The ACK provides feedback on the fragments that correspond to the last window. If Window  mode - ACK on error is used, the recipient MUST NOT transmit an ACK to the sender if no losses have been detected for the last window. If losses have been detected, the recipient MUST then transmit an ACK to the sender to provide feedback on the last window.

When Window mode - ACK “Always” is used, the fragment sender starts a timer (denoted "ACK Always Timer") after transmitting the last fragment of a fragmented IPv6 datagram. The fragment sender also starts the ACK Always Timer after transmitting the last fragment of a window. The initial value for this timer is not provided by this specification, and is expected to be defined in additional documents. Upon expiration of the timer, if no ACK has been received for the current window, the sender retransmits the last fragment, and it reinitializes and restarts the timer.  Note that retransmitting the last fragment of a window as described serves as an ACK request. The maximum number of requests for a specific ACK, denoted MAX_ACK_REQUESTS, is not stated in this document, and it is expected to be defined in other documents (e.g. technology-specific profiles).

In all Window mode options, the fragment sender retransmits any lost fragments reported in an ACK.

If a fragment recipient disassociates from its L2 network, the recipient MUST discard all link fragments of all partially reassembled payload datagrams, and fragment senders MUST discard all not yet transmitted link fragments of all partially transmitted payload (e.g., IPv6) datagrams. Similarly, when a node first receives a fragment of a packet, it starts a reassembly timer. When this time expires, if the entire packet has not been reassembled, the existing fragments MUST be discarded and the reassembly state MUST be flushed. The value for this timer is not provided by this specification, and is expected to be defined in technology-specific profile documents.

## Supporting multiple window sizes

For Window mode operation, implementers may opt to support a single window size or multiple window sizes. The latter, when feasible, may provide performance optimizations. For example, a large window size may be used for IPv6 packets that need to be carried by a large number of fragments. However, when the number of fragments required to carry an IPv6 packet is low, a smaller window size, and thus a shorter bitmap, may be sufficient to provide feedback on all fragments. If multiple window sizes are supported, the Rule ID may be used to signal the window size in use for a specific IPv6 packet transmission.

## Aborting fragmented IPv6 datagram transmissions

For several reasons, a fragment sender or a fragment receiver may want to abort the on-going transmission of one or several fragmented IPv6 datagrams. The entity (either the fragment sender or the fragment receiver) that triggers abortion transmits to the other endpoint a format that only comprises a Rule ID (of size R bits), which signals abortion of all on-going fragmented IPv6 packet transmissions. The specific value to be used for the Rule ID of this abortion signal is not defined in this document, and is expected to be defined in future documents.

Upon transmission or reception of the abortion signal, both entities MUST release any resources allocated for the fragmented IPv6 datagram transmissions being aborted. 


## Downlink fragment transmission

In some LPWAN technologies, as part of energy-saving techniques, downlink transmission is only possible immediately after an uplink transmission. In order to avoid potentially high delay for fragmented IPv6 datagram transmission in the downlink, the fragment receiver MAY perform an uplink transmission as soon as possible after reception of a fragment that is not the last one. Such uplink transmission may be triggered by the L2 (e.g. an L2 ACK sent in response to a fragment encapsulated in a L2 frame that requires an L2 ACK) or it may be triggered from an upper layer.


# Security considerations

## Security considerations for header compression
A malicious header compression could cause the reconstruction of a 
wrong packet that does not match with the original one, such corruption 
may be detected with end-to-end authentication and integrity mechanisms. 
Denial of Service may be produced but its arise other security problems 
that may be solved with or without header compression.

## Security considerations for fragmentation
This subsection describes potential attacks to LPWAN fragmentation 
and proposes countermeasures, based on existing analysis of attacks
to 6LoWPAN fragmentation.

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

Thanks to Dominique Barthel, Carsten Bormann, Philippe Clavier, Arunprabhu Kandasamy, Antony Markovski, Alexander
Pelov, Pascal Thubert, Juan Carlos Zuniga and Diego Dujovne for useful design consideration and comments. 

--- back

# Fragmentation examples

This section provides examples of different fragment delivery reliability options possible on the basis of this specification.

{{Fig-Example-Unreliable}} illustrates the transmission of an IPv6 packet that needs 11 fragments in the No ACK option.

~~~~
        Sender               Receiver
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
{: #Fig-Example-Unreliable title='Transmission of an IPv6 packet carried by 11 fragments in the No ACK option'}

{{Fig-Example-Win-NoLoss-NACK}} illustrates the transmission of an IPv6 packet that needs 11 fragments in Window mode - ACK on error, for N=3, without losses.

~~~~
        Sender               Receiver
          |-----W=1, CFN=6----->|
          |-----W=1, CFN=5----->|
          |-----W=1, CFN=4----->|
          |-----W=1, CFN=3----->|
          |-----W=1, CFN=2----->|
          |-----W=1, CFN=1----->|
          |-----W=1, CFN=0----->|
      (no ACK)
          |-----W=0, CFN=6----->|
          |-----W=0, CFN=5----->|
          |-----W=0, CFN=4----->|
          |-----W=0, CFN=7----->|MIC checked =>
      (no ACK)

~~~~
{: #Fig-Example-Win-NoLoss-NACK title='Transmission of an IPv6 packet carried by 11 fragments in Window mode - ACK on error, for N=3, without losses.'}

{{Fig-Example-Rel-Window-NACK-Loss}} illustrates the transmission of an IPv6 packet that needs 11 fragments in Window mode - ACK on error, for N=3, with three losses.

~~~~
         Sender             Receiver
          |-----W=1, CFN=6----->|
          |-----W=1, CFN=5----->|
          |-----W=1, CFN=4--X-->|
          |-----W=1, CFN=3----->|
          |-----W=1, CFN=2--X-->|
          |-----W=1, CFN=1----->|
          |-----W=1, CFN=0----->|
          |<-------ACK----------|Bitmap:11010111
          |-----W=1, CFN=4----->|
          |-----W=1, CFN=2----->|   
      (no ACK)     
          |-----W=0, CFN=6----->|
          |-----W=0, CFN=5----->|
          |-----W=0, CFN=4--X-->|
          |-----W=0, CFN=7----->|MIC checked
          |<-------ACK----------|Bitmap:11010001
          |-----W=0, CFN=4----->|MIC checked =>
      (no ACK)    

~~~~
{: #Fig-Example-Rel-Window-NACK-Loss title='Transmission of an IPv6 packet carried by 11 fragments in Window mode - ACK on error, for N=3, three losses.'}

{{Fig-Example-Rel-Window-ACK-NoLoss}} illustrates the transmission of an IPv6 packet that needs 11 fragments in Window mode - ACK "always", for N=3, without losses. Note: in Window mode, an additional bit will be needed to number windows. 

~~~~
        Sender               Receiver
          |-----W=1, CFN=6----->|
          |-----W=1, CFN=5----->|
          |-----W=1, CFN=4----->|
          |-----W=1, CFN=3----->|
          |-----W=1, CFN=2----->|
          |-----W=1, CFN=1----->|
          |-----W=1, CFN=0----->|
          |<-------ACK----------|no bitmap
          |-----W=0, CFN=6----->|
          |-----W=0, CFN=5----->|   
          |-----W=0, CFN=4----->|
          |-----W=0, CFN=7----->|MIC checked =>
          |<-------ACK----------|no bitmap
        (End)    

~~~~
{: #Fig-Example-Rel-Window-ACK-NoLoss title='Transmission of an IPv6 packet carried by 11 fragments in Window mode - ACK "always", for N=3, no losses.'}

{{Fig-Example-Rel-Window-ACK-Loss}} illustrates the transmission of an IPv6 packet that needs 11 fragments in Window mode - ACK "always", for N=3, with three losses.

~~~~
        Sender               Receiver
          |-----W=1, CFN=6----->|
          |-----W=1, CFN=5----->|
          |-----W=1, CFN=4--X-->|
          |-----W=1, CFN=3----->|
          |-----W=1, CFN=2--X-->|
          |-----W=1, CFN=1----->|
          |-----W=1, CFN=0----->|
          |<-------ACK----------|bitmap:11010111
          |-----W=1, CFN=4----->|
          |-----W=1, CFN=2----->|
          |<-------ACK----------|no bitmap
          |-----W=0, CFN=6----->|
          |-----W=0, CFN=5----->|   
          |-----W=0, CFN=4--X-->|
          |-----W=0, CFN=7----->|MIC checked
          |<-------ACK----------|bitmap:11010001
          |-----W=0, CFN=4----->|MIC checked =>
          |<-------ACK----------|no bitmap
        (End)    

~~~~
{: #Fig-Example-Rel-Window-ACK-Loss title='Transmission of an IPv6 packet carried by 11 fragments in Window mode - ACK "Always", for N=3, with three losses.'}

# Rule IDs for fragmentation

Different Rule IDs may be used for different aspects of fragmentation functionality as per this document. A summary of such Rule IDs follows:   

   *  A fragment, and the reliability option in use for the IPv6 datagram being carried: i) No ACK, ii) Window mode - ACK on error, iii) Window mode - ACK “always”. In Window mode, a specific Rule ID may be used for each supported window size.
     
   *  An ACK message.
   
   *  An abort message: i) ABORT_TX, ii) ABORT_RX, iii) Abort all on-going transmissions.


# Note
Carles Gomez has been funded in part by the Spanish Government
(Ministerio de Educacion, Cultura y Deporte) through the Jose
Castillejo grant CAS15/00336, and by the ERDF and the Spanish Government 
through project TEC2016-79988-P.  Part of his contribution to this work
has been carried out during his stay as a visiting scholar at the
Computer Laboratory of the University of Cambridge.
