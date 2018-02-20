---
stand_alone: true
ipr: trust200902
docname: draft-ietf-lpwan-ipv6-static-context-hc-09
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
  RFC4944: 
  RFC2460: 
  RFC3385:
  RFC7136:
  RFC5795:
  RFC7217:
informative:  
  I-D.ietf-lpwan-overview:
  I-D.zuniga-lpwan-schc-over-sigfox:
  I-D.petrov-lpwan-ipv6-schc-over-lorawan:

--- abstract

This document defines the Static Context Header Compression (SCHC) framework, which provides header compression and fragmentation functionality. SCHC has been tailored for Low Power Wide Area Networks (LPWAN).

SCHC compression is based on a common static context stored in LPWAN devices and in the network. This document applies SCHC compression to IPv6/UDP headers. This document also specifies a fragmentation and reassembly mechanism that is used to support the IPv6 MTU requirement over LPWAN technologies. Fragmentation is mandatory for IPv6 datagrams that, after SCHC compression or when it has not been possible to apply such compression, still exceed the layer two maximum payload size.

Note that this document defines generic functionality. This document purposefully offers flexibility with regard to parameter settings and mechanism choices, that are expected to be made in other, technology-specific, documents.


--- middle
 
# Introduction {#Introduction}

This document defines a header compression scheme and fragmentation functionality, both specially tailored for Low Power Wide Area Networks (LPWAN).

Header compression is needed to efficiently bring Internet connectivity to the node
within a LPWAN network. Some LPWAN networks properties can be exploited to get an efficient header
compression:

* Topology is star-oriented; therefore, all the packets follow the same path.
  For the needs of this draft, the architecture can be simply described as: Devices (Dev)
  exchange information with LPWAN Application Servers (App) through Network Gateways (NGW).

* Traffic flows can be known in advance since devices embed built-in
  applications. New applications cannot be easily installed in LPWAN devices, as they would in computers or smartphones.

The Static Context Header Compression (SCHC) is defined for this environment.
SCHC uses a context, where header information is kept in the header format order. This context is
static: the values of the header fields do not change over time. This avoids
complex resynchronization mechanisms, that would be incompatible
with LPWAN characteristics. In most cases, a small context identifier is enough to represent the full IPv6/UDP headers. 
The SCHC header compression mechanism is independent of the specific LPWAN technology over which it is used.

LPWAN technologies are also characterized,
among others, by a very reduced data unit and/or payload size
{{I-D.ietf-lpwan-overview}}.  However, some of these technologies
do not provide fragmentation functionality, therefore the only option for
   them to support the IPv6 MTU requirement of 1280 bytes
 {{RFC2460}} is to use a fragmentation protocol at the
adaptation layer, below IPv6.
In response to this need, this document also defines a fragmentation/reassembly
mechanism, which supports the IPv6 MTU requirement over LPWAN
technologies. Such functionality has been designed under the assumption that data unit out-of-sequence delivery will not happen between the entity performing fragmentation and the entity performing reassembly.

Note that this document defines generic functionality and purposefully offers flexibility with regard to parameter settings and mechanism choices, that are expected to be made in other, technology-specific, documents (e.g. {{I-D.zuniga-lpwan-schc-over-sigfox}}, {{I-D.petrov-lpwan-ipv6-schc-over-lorawan}}).


# LPWAN Architecture {#LPWAN-Archi}

LPWAN technologies have similar network architectures but different
terminology. We can identify different types of entities in a
typical LPWAN network, see {{Fig-LPWANarchi}}:

   o  Devices (Dev) are the end-devices or hosts (e.g. sensors,
      actuators, etc.). There can be a very high number of devices per radio gateway.

   o  The Radio Gateway (RGW), which is the end point of the constrained link.

   o  The Network Gateway (NGW) is the interconnection node between the Radio Gateway and the Internet.  

   o  LPWAN-AAA Server, which controls the user authentication and the
      applications. 

   o  Application Server (App)

~~~~
                                           +------+
 ()   ()   ()       |                      |LPWAN-|
  ()  () () ()     / \       +---------+   | AAA  |
() () () () () ()  /   \=====|    ^    |===|Server|  +-----------+
 ()  ()   ()     |           | <--|--> |   +------+  |APPLICATION|
()  ()  ()  ()  / \==========|    v    |=============|   (App)   |
  ()  ()  ()   /   \         +---------+             +-----------+
 Dev        Radio Gateways         NGW

~~~~
{: #Fig-LPWANarchi title='LPWAN Architecture'}                      


# Terminology
This section defines the terminology and acronyms used in this document.

* Abort. A type of data unit used by an endpoint involved in on-going fragmented packet transmission or reception in order to signal the other endpoint that the on-going fragmented packet transmission process is being aborted. 

* ACK. Acknowledgment. A data unit used in two of the fragmentation modes defined in this specification, which is sent by a fragment receiver to report on successful or unsuccessful reception of a set of fragments.

* All-0. The fragment format for the last frame of a window that is not the last one of a packet (see Window in this glossary).

* All-1. The fragment format for the last frame of the packet.

* All-0 empty. An All-0 fragment without a payload. It is used to request the Bitmap when the Retransmission Timer expires, in a window that is not the last one of a packet.

* All-1 empty. An All-1 fragment without a payload. It is used to request the Bitmap when the Retransmission Timer expires in the last window of a packet.

* App: LPWAN Application. An application sending/receiving IPv6 packets to/from the Device.

* APP-IID: Application Interface Identifier. Second part of the IPv6 address that identifies the application server interface.

* Bi: Bidirectional, a rule entry that applies to headers of packets travelling in both directions (Up and Dw).

* Bitmap: a field of bits in an acknowledgment message that tells the sender which fragments of a window were correctly received.

* C: Checked bit. Used in an acknowledgment (ACK) header to determine if the MIC locally computed by the receiver matches (1) the received MIC or not (0).

* CDA: Compression/Decompression Action. Describes the reciprocal pair of actions that are performed at the compressor to compress a header field and at the decompressor to recover the original header field value.

* Compress Residue. The bytes that need to be sent after applying the SCHC compression over each header field 

* Context: A set of rules used to compress/decompress headers.

* Dev: Device. A node connected to the LPWAN. A Dev may implement SCHC.

* Dev-IID: Device Interface Identifier. Second part of the IPv6 address that identifies the device interface.

* DI: Direction Indicator. This field tells which direction of packet travel (Up, Dw or Bi) a rule applies to. This allows for assymmetric processing.

* DTag: Datagram Tag. This fragmentation header field is set to the same value for all fragments carrying the same IPv6 datagram.

* Dw: Dw: Downlink direction for compression/decompression, from SCHC C/D in the network to SCHC C/D in the Dev.

* FCN: Fragment Compressed Number. This fragmentation header field carries an efficient representation of a larger-sized fragment number.

* Field Description. A line in the Rule Table.

* FID: Field Identifier. This is an index to describe the header fields in a Rule.

* FL: Field Length identifies whether a field has fixed or variable size, and for the latter it indicates its length.

* FP: Field Position is a value that is used to identify the position where each instance of a field appears in the header.  

* Fragment: A data unit that carries a subset of a SCHC packet. Fragmentation is needed when the size of a SCHC packet exceeds the available payload size of the underlying L2 technology data unit.

* IID: Interface Identifier. See the IPv6 addressing architecture {{RFC7136}}

* Inactivity Timer. A timer used at the end of the fragmentation state machine to detect when there is an error and there is no possibility to continue an on-going fragmented packet transmission.

* L2: Layer two. The immediate lower layer SCHC interfaces with. It is provided by an underlying LPWAN technology. 

* MIC: Message Integrity Check.  A fragmentation header field computed over an IPv6 packet before fragmentation, used for error detection after IPv6 packet reassembly. 

* MO: Matching Operator. An operator used to match a value contained in a header field with a value contained in a Rule.

* Retransmission Timer. A timer used by the fragment sender state machine during an on-going fragmented packet transmission to detect possible link errors when waiting for a possible incoming ACK.

* Rule: A set of header field values.

* Rule entry: A row in the rule that describes a header field.

* Rule ID: An identifier for a rule, SCHC C/D, and Dev share the same Rule ID for a specific flow. A set of Rule IDs are used to support fragmentation functionality.

* SCHC C/D: Static Context Header Compression Compressor/Decompressor. A mechanism used in both sides, at the Dev and at the network to achieve Compression/Decompression of headers. SCHC C/D uses SCHC rules to perform compression and decompression.

* SCHC packet: A packet (e.g. an IPv6 packet) whose header has been compressed as per the header compression mechanism defined in this document. If the header compression process is unable to actually compress the packet header, the packet with the uncompressed header is still called a SCHC packet (in this case, a Rule ID is used to indicate that the packet header has not been compressed).

* TV: Target value. A value contained in the Rule that will be matched with the value of a header field.

* Up: Uplink direction for compression/decompression, from the Dev SCHC C/D to the network SCHC C/D.

* W: Window bit. A fragment header field used in Window mode (see section 5), which carries the same value for all fragments of a window.

* Window:  A subset of the fragments needed to carry a packet (see section 5)

# SCHC overview

SCHC can be abstracted as an adaptation layer below IPv6 and the underlying LPWAN technology. SCHC that comprises two sublayers (i.e. the Compression sublayer and the Fragmentation sublayer), as shown in {{Fig-IntroLayers}}. 

~~~~
 
                  +----------------+
                  |      IPv6      | 
                / +----------------+            
               /  |   Compression  |  
         SCHC <   +----------------+   
               \  |  Fragmentation |
                \ +----------------+         
                  |LPWAN technology|
                  +----------------+ 

~~~~
{: #Fig-IntroLayers title='Protocol stack comprising IPv6, SCHC and an LPWAN technology'} 

As per this document, when a packet (e.g. an IPv6 packet) needs to be 
transmitted, header compression is first applied to the packet. The resulting packet after header compression (whose header may actually be smaller than that of the original packet or not) is called a SCHC packet. Subsequently, and if the SCHC packet size  exceeds the layer 2 (L2) MTU, fragmentation is then applied to the SCHC packet. This process is illustrated by {{Fig-Operations}}

~~~~
 
                         A packet (e.g. an IPv6 packet)
                             |
                             V   
                    +-----------------+
                    |   Compression   |
                    +-----------------+            
                             |
                        SCHC packet              
                             |
                             V 
                   +------------------+    
                   |   Fragmentation  |  (if needed)
                   +------------------+     
                             |
                             V     
                         Fragment(s)    (if needed)


~~~~
{: #Fig-Operations title='SCHC operations from a sender point of view: header compression and fragmentation'}

# Static Context Header Compression

In order to perform header compression, this document defines a mechanism called Static Context Header Compression (SCHC), 
which is based on using context, i.e. a set of rules to compress or decompress headers. SCHC avoids context
synchronization, which is the most bandwidth-consuming operation in
other header compression mechanisms such as RoHC {{RFC5795}}. Since the nature of data flows is highly predictable in LPWAN
networks, static contexts may be stored beforehand to omit transmitting some information over the air.
The contexts must be stored at both ends, and they can either be learned by a provisioning protocol, by out of band means, 
or they can
be pre-provisioned. The way the contexts are provisioned on both ends is out of the scope of this document.

~~~~
     Dev                                                 App
+--------------+                                  +--------------+
|APP1 APP2 APP3|                                  |APP1 APP2 APP3|
|              |                                  |              |
|      UDP     |                                  |     UDP      | 
|     IPv6     |                                  |    IPv6      |   
|              |                                  |              |  
|   SCHC C/D   |                                  |              |  
|   (context)  |                                  |              | 
+-------+------+                                  +-------+------+ 
         |   +--+     +----+     +---------+              .
         +~~ |RG| === |NGW | === |SCHC C/D |... Internet ..
             +--+     +----+     |(context)| 
                                 +---------+
~~~~
{: #Fig-archi title='Architecture'}

{{Fig-archi}} represents the architecture for compression/decompression. It is based on {{I-D.ietf-lpwan-overview}}
terminology. The Device sends application flows using IPv6 or IPv6/UDP protocols. These flows are compressed by a
Static Context Header Compression Compressor/Decompressor (SCHC C/D) to reduce the headers size. (Note that if the 
resulting data unit exceeds the maximum payload size of the underlying LPWAN technology, fragmentation is performed, see 
{{Frag}}.) The resulting
data unit is sent as one or more L2 frames to a LPWAN Radio Gateway (RG) which forwards
the frame(s) to a Network Gateway (NGW).
The NGW sends the data to an SCHC C/D for decompression. The SCHC C/D can be
located in the Network Gateway (NGW) or somewhere else as long as a tunnel is established between the NGW and the SCHC C/D. 
Note that, for some LPWAN technologies, it may be suitable to locate fragmentation and reassembly functionality nearer the 
NGW, in order to better deal with time constraints of such technologies.
The SCHC C/Ds on both sides must share the same set of Rules.
After decompression, the packet can be sent over the Internet to one
or several LPWAN Application Servers (App). 

The SCHC C/D process is symmetrical, therefore the same description applies to the reverse direction.

## SCHC Rules

The main idea of the SCHC compression scheme is to transmit the Rule ID
to the other end instead of sending known field values. This Rule ID
identifies a rule that provides the closest match to the original
packet values. Hence, when a value is known by both ends, it is only
necessary to send the corresponding Rule ID over the LPWAN network.

The context contains a list of rules (cf. {{Fig-ctxt}}). Each Rule
contains itself a list of Field Descriptions composed of a field
identifier (FID), a field length (FL), a field position (FP), a
direction indicator (DI), a target value (TV), a matching operator
(MO) and a Compression/Decompression Action (CDA).


~~~~
  /-----------------------------------------------------------------\
  |                         Rule N                                  |
 /-----------------------------------------------------------------\|
 |                       Rule i                                    ||
/-----------------------------------------------------------------\||
|  (FID)            Rule 1                                        |||
|+-------+--+--+--+------------+-----------------+---------------+|||
||Field 1|FL|FP|DI|Target Value|Matching Operator|Comp/Decomp Act||||
|+-------+--+--+--+------------+-----------------+---------------+|||
||Field 2|FL|FP|DI|Target Value|Matching Operator|Comp/Decomp Act||||
|+-------+--+--+--+------------+-----------------+---------------+|||
||...    |..|..|..|   ...      | ...             | ...           ||||
|+-------+--+--+--+------------+-----------------+---------------+||/
||Field N|FL|FP|DI|Target Value|Matching Operator|Comp/Decomp Act|||
|+-------+--+--+--+------------+-----------------+---------------+|/
|                                                                 |
\-----------------------------------------------------------------/
~~~~
{: #Fig-ctxt title='Compression/Decompression Context'}


The Rule does not describe how to delineate each field in the original packet header. This
must be known from the compressor/decompressor. The rule only describes the
compression/decompression behavior for each header field. In the rule, the Field Descriptions are listed in the order in 
which the fields appear in the packet header.

The Rule also describes the compression residue which is
transmitted in the same order as the one used by the Field Description in the Rule.

The Context describes the header fields and its values with the following entries:

* Field ID (FID) is a unique value to define the header field.

* Field Length (FL) is the length of the field in bits for fixed values or a type (variable, token length) for variable values. The length of a header field is defined in the specific standard document. 

* Field Position (FP): in case several occurences of a field exist in the
  header, FP indicatess which one is targeted. The default position is 1.
  
* A direction indicator (DI) indicating the packet direction(s) this Field Description applies to. Three values are possible:

  * UPLINK (Up): this Field Description is only applicable to packets sent by the Dev to the App,

  * DOWNLINK (Dw): this Field Description is only applicable to packets sent from the App to the Dev,

  * BIDIRECTIONAL (Bi): this Field Description is applicable to packets travelling both Up and Dw.

* Target Value (TV) is the value to compare
  the packet header field to. The Target Value can be of any type (integer, strings, etc.).
  For instance, it can be a single value or a more complex structure (array, list, etc.), such as a JSON or a CBOR structure.

* Matching Operator (MO) is the operator used to compare
  the Field Value and the Target Value. The Matching Operator may require some 
  parameters. MO is only used during the compression phase. The set of MOs defined in this document can be found in {{chap-MO}}.

* Compression Decompression Action (CDA) describes the pair of reciprocal compression
  and decompression processes. The CDA may require some
  parameters. CDA is used in both the compression and the decompression phases. The set of CDAs defined in this document can be found in {{chap-CDA}}.

## Rule ID

Rule IDs are sent by the compression element and are intended for the decompression element. The size
of the Rule ID is not specified in this document. It is implementation-specific and can vary according to the
LPWAN technology and the number of flows, among others.

Some values in the Rule ID space are reserved for functionalities other than header
compression, such as packet fragmentation. (See {{Frag}}).

Rule IDs are specific to a Dev. Hence, multiple Dev instances may use the same Rule ID to define
different header compression contexts. To identify the correct Rule ID, the
SCHC C/D needs to correlate the Rule ID with the Dev identifier to
find the appropriate Rule to be applied. For Devs with different LPWAN radio interfaces, several approaches are allowed. For example, the same set of Rule IDs may be used for packet transmission over the different interfaces. However, fragmentation may benefit from using specific Rule IDs (which are tied to specific modes and settings) for a particular underlying LPWAN technology.  


## Packet processing

The compression/decompression process follows several steps:

* 1. Compression Rule selection: The goal is to identify which Rule(s) will be used
  to compress the packet's headers. When doing compression, the Rule will be selected by matching the Field Description to the packet header as described below. When the selection of a Rule is done, the Rule-ID is used to compress the header. 
The detailed steps for compression Rule selection are the following:
  * The first step is to choose the Field Description by its direction, using the direction indicator (DI). A Field Description that does not correspond to the appropriate DI will be ignored, if all the fields of the packet do not have a Field Description with the correct DI the Rule is discarded and SCHC C/D proceeds to explore the next Rule.
  * When the DI has matched, then the next step is to identify the fields according to field position (FP). If the field position does not correspond, then the Rule is not used and the SCHC proceeds to consider the next Rule.
  * Once the DI and the FP correspond to the header information, each field's value is then compared to the corresponding target value (TV) stored in the Rule for that specific field using the matching operator (MO).
  * If all the fields in the packet's header satisfy all the matching operators (MO) of a Rule (i.e. all MO results are True), the fields of the header are then compressed according to the Compression/Decompression Actions (CDAs) and a compressed header (with possibly a compressed residue) may be obtained. Otherwise, the next Rule is tested. 
  * If no eligible Rule is found, then the header must be sent without compression, in which case the fragmentation process may be required.

* 2. Sending: If an eligible Rule is found, the Rule ID is sent to the other end followed by the Compression Residue (which could be empty) and directly followed by the payload. The product of the Compression Residue is sent in the order expressed in the Rule for the matching fields. 

The way the Rule ID is sent depends on the specific LPWAN layer two technology. For example, it can be either included in a Layer 2 header or sent in the first byte of the L2 payload. (Cf. {{Fig-FormatPckt}}).
  
This process will be specified in the LPWAN technology-specific document and is out of the scope of the present document. On LPWAN technologies that are byte-oriented, the compressed header concatenated with the original packet payload is padded to a multiple of 8 bits, if needed. See {{Padding}} for details.


* 3. Decompression: 
When doing decompression, in the NGW side the SCHC C/D needs to find the correct Rule based on the L2 PDU and in this way, it can find the Dev-ID and the Rule-ID. In the Dev side, only the Rule ID is needed to identify the correct Rule since the Dev only holds rules that apply to itself. 

The receiver identifies the sender through its device-id (e.g. MAC address, if exist) and selects the appropriate Rule from the Rule ID. ThisRule describes the compressed header format and associates the values to the header fields.  The receiver applies the CDA action to reconstruct the original header fields. The CDA application order can be different from the order given by the Rule. For instance,
  Compute-\* may be applied at the end, after all the other CDAs.
  
  
~~~~

+--- ... --+------- ... -------+------------------+--...--
|  Rule ID |Compression Residue|  packet payload  |{padding}
+--- ... --+------- ... -------+------------------+-optional-

<----- compressed header ------>

~~~~
{: #Fig-FormatPckt title='LPWAN Compressed Header Packet Format'}


## Matching operators {#chap-MO}

Matching Operators (MOs) are functions used by both SCHC C/D endpoints involved in the header 
compression/decompression. They are not typed and can be indifferently applied to integer, string
or any other data type. The result of the operation can either be True or False. MOs are defined as follows: 

* equal: Results in true if a field value in a packet and a TV are equal.

* ignore: No check is done between a field value in a packet and a TV
  in the Rule. The result of the matching is always true.

* MSB(length): A match is obtained if the most significant bits
  of the header are equal to the TV in the rule. The "length" parameter of the MSB Matching Operator
  indicates how many bits are involved in the comparison.
  
* match-mapping: With match-mapping,
  the Target Value is a list of values. Each value of the list is identified by a short ID (or index). Compression is achieved by sending the index instead of the original header field value.
  This operator matches if the header field value is equal to one of the values in the target list.
  

## Compression Decompression Actions (CDA) {#chap-CDA}

The Compression Decompression Action (CDA) describes the actions taken during
the compression of headers fields, and inversely, the action taken by the decompressor to restore
the original value.

~~~~
/--------------------+-------------+----------------------------\
|  Action            | Compression | Decompression              |
|                    |             |                            |
+--------------------+-------------+----------------------------+
|not-sent            |elided       |use value stored in ctxt    |
|value-sent          |send         |build from received value   |
|match-mapping       |send index   |value from index on a table |
|LSB(length)         |send LSB     |TV OR received value        |
|compute-length      |elided       |compute length              |
|compute-checksum    |elided       |compute UDP checksum        |
|Deviid              |elided       |build IID from L2 Dev addr  |
|Appiid              |elided       |build IID from L2 App addr  |
\--------------------+-------------+----------------------------/

~~~~
{: #Fig-function title='Compression and Decompression Functions'}

{{Fig-function}} summarizes the basic functions that can be used to compress and decompress
a field. The first column lists the actions name. The second and third
columns outline the reciprocal compression/decompression behavior for each action.

Compression is done in the rule order and compressed values are sent in that order in the compressed
message. The receiver is assumed to know the size of each compressed field
which can be given by the rule or may be sent with the compressed header. 

If the field is identified as being variable, then its size must be sent first using the following coding:

* If the size is between 0 and 14 bytes, it is sent as a 4-bit integer.

* For values between 15 and 255, the first 4 bits sent are set to 1 and the size is sent using 8 bits. 

* For higher values of size, the first 12 bits are set to 1 and the next two bytes contain the size value as a 16 bits integer. 

### not-sent CDA

The not-sent function is generally used when the field value is specified in the rule and
therefore known by both the Compressor and the Decompressor. This action is generally used with the
"equal" MO. If MO is "ignore", there is a risk to have a decompressed field
value different from the compressed field.

The compressor does not send any value in the compressed header for a field on which not-sent compression is applied.

The decompressor restores the field value with the target value stored in the matched rule.

### value-sent CDA

The value-sent action is generally used when the field value is not known by both Compressor and Decompressor.
The value is sent in the compressed message header. Both Compressor and Decompressor must know the
size of the field, either implicitly (the size is known by both sides) 
or explicitly in the compression residue by indicating the length, as defined in {{chap-CDA}}. This function is generally 
used with the "ignore" MO.

### mapping-sent

The mapping-sent is used to send a smaller index (the index into
the Target Value list of values) instead of the original value. This function is used together with the "match-mapping" MO.

On the compressor side, the match-mapping Matching Operator searches the TV for a match with the header field value and the mapping-sent CDA appends the corresponding index to the Compression Residue to be sent.
On the decompressor side, the CDA uses the received index to restore the field value by looking up the list in the TV.

The number of bits sent is the minimal size for coding all the possible indices.

### LSB CDA

The LSB action is used together with the "MSB" MO to avoid sending the higher part of the packet field if that part is already known by the receiving end.
A length can be specified in the rule to indicate
how many bits have to be sent. If the length is not specified, the number of bits sent is the
original header field length minus the length specified in the MSB MO.

The compressor sends the Least Significant Bits (e.g. LSB of the length field). The
decompressor combines the value received with the Target Value.

If this action is made on a variable length field, the remaining size in bytes must be sent before.


### DEViid, APPiid CDA

These functions are used to process respectively the Dev and the App Interface Identifiers (Deviid and Appiid) of the 
IPv6 addresses. Appiid CDA is less common since current LPWAN technologies
frames contain a single address, which is the Dev's address.

The IID value MAY be computed from the Device ID present in the Layer 2 header, or from some other stable identifier. The computation is specific for each LPWAN technology and MAY depend on the Device ID size.

In the Downlink direction, these CDA may be used to determine the L2 addresses used by the LPWAN.

### Compute-\*

These classes of functions are used by the decompressor to compute the compressed field value based on received information. 
Compressed fields are elided during compression and reconstructed during decompression.

* compute-length: compute the length assigned to this field. For instance, regarding
  the field ID, this CDA may be used to compute IPv6 length or UDP length.

* compute-checksum: compute a checksum from the information already received by the SCHC C/D.
  This field may be used to compute UDP checksum.


# Fragmentation {#Frag}

## Overview

In LPWAN technologies, the L2 data unit size typically varies from tens to hundreds of bytes.  Be it after applying SCHC header compression or when SCHC header compression is not possible, if the entire IPv6 datagram fits within a single L2 data unit, the fragmentation mechanism is not used and the packet is sent. Otherwise, the datagram SHALL be broken into fragments.

LPWAN technologies impose some strict limitations on traffic. For instance, 
devices are sleeping most of the time and may receive data during a
short period of time after transmission to preserve battery. To
adapt the SCHC fragmentation to the capabilities of LPWAN
technologies, it is desirable to enable optional fragment
retransmission and to allow a gradation of fragment delivery
reliability. This document does not make any decision with regard to
which fragment delivery reliability mode(s) will be used over a
specific LPWAN technology. These details will be defined in other technology-specific documents.


   An important consideration is that LPWAN networks typically follow a
   star topology. The fragmentation functionality defined in this document has been designed under the assumption that data unit out-of-sequence delivery will not happen between the entity performing fragmentation and the entity
   performing reassembly.  This assumption allows reducing the complexity
   and overhead of the fragmentation mechanism.

## Tools

This subsection describes the different tools that are used to enable the fragmentation functionality defined in this document, such as fields in the fragmentation header frames (see the related formats in {{Fragfor}}), timers and parameters.  

*  Rule ID. The Rule ID is present in the fragment header and in the ACK header format.  The Rule ID in a fragment header is used to identify that a fragment is being carried, what fragmentation delivery reliability mode is used and what window size is used (if multiple sizes are possible). The Rule ID  in the fragmentation header also allows interleaving non-fragmented IPv6 datagrams and fragments that carry other IPv6 datagrams. The Rule ID in an ACK identifies the message as an ACK.

*  Fragment Compressed Number (FCN).  The FCN is included in all fragments.  This field can be understood as a truncated, 
efficient representation of a larger-sized fragment number, and does not carry an absolute fragment number.  There are two FCN reserved values that are used for controlling the fragmentation process, as described next. The FCN value with all the bits equal to 1 (All-1) denotes the last
fragment of a packet. The last window of a packet is called an All-1 window.  In ACK-Always or ACK-on-error, the FCN value with all the bits equal to 0 (All-0) denotes the last
fragment of a window that is not the last one of the packet. Such a window is called an All-0 window. In the No-ACK mode, All-0 is found in all fragments but the last one. The rest of the FCN values are assigned in a sequentially
decreasing order, which has the purpose to avoid possible ambiguity for the receiver that might arise under certain
conditions.
In the fragments, this field is an unsigned integer, with a size of N bits. In the No-ACK mode, it is set to 1 bit (N=1). For the other reliability modes, it is recommended to use a number of bits (N) equal to or greater than 3. Nevertheless, the appropriate value will be defined in the corresponding technology documents. The FCN MUST be set sequentially decreasing from the highest FCN in the window (which will be used for the first fragment), and MUST wrap from 0 back to the highest FCN in the window.
   For windows that are not the last one  from a fragmented packet, the FCN for the last fragment in such windows is an All-0. This indicates that the window is finished and communication proceeds according to the reliability mode in use.
   The FCN for the last fragment in the last window is an All-1.  It is also 
   important to note that, in the No-ACK mode or when N=1, the last fragment of the packet will carry a FCN equal to 1, while all previous fragments
   will carry a FCN of 0.
   
*  Datagram Tag (DTag). The DTag field, if present, is set to the same value for all fragments carrying the same IPv6 datagram, and to different values for different datagrams. Using this field, the sender can interleave fragments from different IPv6 datagrams, while the receiver can still tell them apart.
In the fragment formats, the size of the DTag field is T bits, which may be set to a value greater than or equal to 0 bits. For each new IPv6 datagram processed by the sender, DTag MUST be sequentially increased, from 0 to 2^T – 1 wrapping back from 2^T - 1 to 0.
In the ACK format, DTag carries the same value as the DTag field in the fragments for which this ACK is intended.

*  W (window): W is a 1-bit field. This field carries the same value for all fragments of a window, and it is complemented for the next window. The initial value for this field is 0.
   In the ACK format, this field also has a size of 1 bit. In all ACKs, the W bit carries the same value as the W bit carried by the fragments whose reception is being positively or negatively acknowledged by the ACK.

*  Message Integrity Check (MIC). This field, which has a size of M bits, is computed by the sender over the complete packet (i.e. a SCHC compressed or an uncompressed IPv6 packet) before fragmentation. The MIC allows the receiver to check errors in the reassembled packet, while it also enables compressing the UDP checksum by use of SCHC compression. The CRC32 as 0xEDB88320 (i.e. the reverse representation of the polynomial used e.g. in the Ethernet standard {{RFC3385}}) is recommended as the default algorithm for computing the MIC. Nevertheless, other algorithms MAY be required in other LPWAN technology-specific documents (e.g. technology-specific profiles). 

*  C (MIC checked): C is a 1-bit field. This field is used in the ACK packets to report the outcome of the MIC check, i.e. whether the reassembled packet was correctly received or not. A value of 1 represents a positive MIC check at the receiver side (i.e. the MIC computed by the receiver matches the received MIC).
 
*  Retransmission Timer. It is used by a fragment sender after the transmission of a window to detect a transmission error  of the ACK corresponding to this window. Depending on the reliability mode, it will lead to a request for an ACK retransmission (in ACK-Always mode) or it will trigger the transmission of the next window (in ACK-on-error mode). The duration of this timer is not defined in this document and must be defined in the corresponding technology documents (e.g. technology-specific profiles).
 
*  Inactivity Timer. This timer is used by a fragment receiver to take action when there is a problem in the transmission of fragments. Such a problem could be detected by the receiver not getting a single fragment during a given period of time or not getting a given number of packets in a given period of time. When this happens, an Abort message will be sent (see related text later in this section). Initially, and each time a fragment is received, the timer is reinitialized. The duration of this timer is not defined in this document and must be defined in the specific technology document (e.g. technology-specific profiles).
 
*  Attempts. This counter counts the requests for a missing ACK. When it reaches the value MAX_ACK_REQUESTS,
the sender assume there are recurrent fragment transmission errors and determines that an Abort is needed. The default value offered
MAX_ACK_REQUESTS is not stated in this document, and it is expected to be defined in other documents (e.g. technology-
specific profiles). The Attempts counter is defined per window. It is initialized each time a new window is used.

*  Bitmap. The Bitmap is a sequence of bits carried in an ACK. Each bit in the Bitmap corresponds to a
fragment of the current window, and provides feedback on whether the fragment has been received or not. The right-most 
position on the Bitmap reports if the All-0 or All-1 fragment has been received or not. Feedback on the
fragment with the highest FCN value
is provided by the bit in the left-most position of the Bitmap. In the Bitmap, a bit set to 1 indicates that the fragment of FCN corresponding to that bit position
has been correctly sent and received. The text above describes the internal representation of the Bitmap. When inserted in the ACK for transmission from the receiver to the sender, the Bitmap may be truncated for energy/bandwidth optimisation, see more details 
in {{Bitmapopt}}

*  Abort. On expiration of the Inactivity timer, on Attempts reaching MAX_ACK_REQUESTS or upon occurence of some other error, the sender or the receiver may use the Abort frames.  When the receiver needs to abort the on-going fragmented packet transmission, it sends a data unit with the Receiver-Abort format. When the sender needs to abort the transmission, it sends a data unit with the Sender-Abort format. The Sender-Abort data unit is not acknowledged.

*  Padding (P). Padding may be used to align the last byte of a fragment with a byte boundary (see {{Padding}}). The number of bits used for padding is not defined and depends on the size of the Rule ID, DTag and FCN fields, and on the L2 payload size. Some ACKs are byte-aligned and do not need padding (see {{Bitmapopt}}).

## Delivery Reliability modes

This specification defines three delivery reliability modes, namely No-ACK, ACK-Always and ACK-on-Error. ACK-Always and ACK-on-Error operate on windows of fragments. A window of fragments is a subset of the full set of fragments needed to carry an IPv6 packet. The three delivery reliability modes are overviewed next: 

*  No-ACK. 
   No-ACK is the simplest fragment delivery reliability mode. The receiver does not generate overhead in the form of acknowledgments (ACKs).  However, this mode does not enhance delivery reliability beyond that offered by the underlying LPWAN technology. In the No-ACK mode, the receiver MUST NOT issue ACKs. See further details in {{No-ACK-subsection}}.

*  ACK-Always.    
   The ACK-Always mode provides flow control.  In
   addition, this mode is able to handle long bursts of lost fragments, since
   detection of such events can be done before the end of the IPv6 packet
   transmission, as long as the window size is short enough. However,
   such benefit comes at the expense of ACK use.
   In ACK-Always, an ACK is transmitted by the fragment receiver every time a window of fragments has been received.  A window of
   fragments is a subset of the full set of fragments needed to carry an
   IPv6 packet.  The ACK informs the sender about received
   and/or missed fragments from one window of fragments.  Upon receipt
   of an ACK that informs about any lost fragments, the sender
   retransmits the lost fragments.  When an ACK is not received by the
   fragment sender after a reasonable time, the latter sends an ACK request using the All-1 empty fragment.
   The maximum number of ACK requests is MAX_ACK_REQUESTS. See further details in {{ACK-Always-subsection}}.

*  ACK-on-Error. The ACK-on-Error mode is suitable for links offering relatively low L2
   data unit loss probability.  This mode reduces the number of ACKs
   transmitted by the fragment receiver. This may be especially beneficial in asymmetric
   scenarios, e.g. where fragmented data are sent uplink and the
   underlying LPWAN technology downlink capacity or message rate is
   lower than the uplink one.  
   In ACK-on-Error, an ACK is transmitted by the fragment
   receiver after a window of fragments have been sent, only if at least
   one of the fragments in the window has been lost. The
   ACK informs the sender about received and/or missed fragments from
   the window of fragments. Upon receipt of an ACK that informs about
   any lost fragments, the sender retransmits the lost fragments. If an ACK is not transmitted back by the receiver at the end of a window, the implicit meaning conveyed is that all fragments have been correctly received. As a consequence,  if an ACK is lost, the sender assumes that all fragments covered by the ACK have been successfully delivered. The sender will then continue transmitting the next window of fragments. If the next fragments received belong to the next window, the receiver will conclude that successful reassembly of the IPv6 packet is not possible. In that case, the receiver will abort the on-going fragmented packet transmission. As an exception to the behavior described above, the receiver MUST transmit an ACK in the last window, including the MIC calculation result, even if all the fragments of the last window have been correctly received. See further details in {{ACK-on-Error-subsection}}.
   
   
   One exception to this behavior is in the last window, where the receiver MUST transmit an ACK, even if all the fragments in the last window have been correctly received.  
 
The same reliability mode MUST be used for all fragments of a
   packet.  It is up to implementers and/or representatives of the
   underlying LPWAN technology to decide which reliability mode to use
   and whether the same reliability mode applies to all IPv6 packets
   or not.  Note that the reliability mode to be used is not
   necessarily tied to the particular characteristics of the underlying
   L2 LPWAN technology (e.g. the No-ACK mode may be used
   on top of an L2 LPWAN technology with symmetric characteristics for
   uplink and downlink).    
This document does not make any decision as to which fragment
   delivery reliability mode(s) are supported by a specific LPWAN
   technology. 

Examples of the different reliability modes described are provided
   in Appendix B.


## Fragmentation Frame Formats {#Fragfor}

This section defines the fragment format, the All-0 and All-1 frame formats, the ACK frame format and the Abort frame formats.

### Fragment format 

   A fragment comprises a fragment header, a fragment payload and padding bits (if any). A fragment conforms
   to the general format shown in {{Fig-FragFormat}}. The fragment payload carries a subset of either a SCHC header
   or an IPv6 header or the original IPv6 packet data payload. 
   A fragment is the payload of the L2 protocol data unit (PDU). Padding MAY be added if necessary, therefore a padding field is optional (this is explicitly indicated in {{Fig-FragFormat}}, but not in subsequent figures, for the sake of illustration clarity.
      
~~~~   
      +-----------------+-----------------------+----------------+
      | Fragment Header |   Fragment payload    | padding (opt.) |
      +-----------------+-----------------------+----------------+
~~~~
{: #Fig-FragFormat title='Fragment general format. Presence of a padding field is optional'}

In the No-ACK mode, fragments except the last one SHALL conform to the detailed format defined in {{Fig-NotLast}}. The total size of the fragment header is R bits.
   
~~~~
             <------------ R ---------->
                         <--T--> <--N-->
             +-- ... --+- ...  -+- ... -+--------...-------+---------+
             | Rule ID |  DTag  |  FCN  | Fragment payload | padding |
             +-- ... --+- ...  -+- ... -+--------...-------+---------+

~~~~
{: #Fig-NotLast title='Fragment Detailed Format for Fragments except the Last One, No-ACK mode'}



In ACK-Always or ACK-on-Error, fragments except the last one SHALL conform the detailed format defined in {{Fig-NotLastWin}}. The total size of the fragment header is R bits.
   
~~~~
             <------------ R ---------->
                       <--T--> 1 <--N-->
            +-- ... --+- ... -+-+- ... -+--------...-------+---------+
            | Rule ID | DTag  |W|  FCN  | Fragment payload | padding |
            +-- ... --+- ... -+-+- ... -+--------...-------+---------+

~~~~
{: #Fig-NotLastWin title='Fragment Detailed Format for Fragments except the Last One, Window mode'}
   
### All-1 and All-0 formats

The All-0 format is used for sending the last fragment of a window that is not the last window of the packet.

~~~~
     <------------ R ----------->
                <- T -> 1 <- N -> 
     +-- ... --+- ... -+-+- ... -+--- ... ---+-+
     | Rule ID | DTag  |W|  0..0 |  payload  |P|  
     +-- ... --+- ... -+-+- ... -+--- ... ---+-+
     
~~~~
{: #Fig-All0 title='All-0 fragment detailed format'}

   
The All-0 empty fragment format is used by a sender to request the retransmission of an ACK by the receiver. It is only used in ACK-Always mode.

~~~~
 <------------ R ----------->
            <- T -> 1 <- N -> 
 +-- ... --+- ... -+-+- ... -+-+
 | Rule ID | DTag  |W|  0..0 |P| (no payload)  
 +-- ... --+- ... -+-+- ... -+-+
              
~~~~
{: #Fig-All0empty title='All-0 empty fragment detailed format'}


In the No-ACK mode, the last fragment of an IPv6 datagram SHALL contain a fragment header that conforms to 
   the detaield format shown in {{Fig-Last}}. The total size of this fragment header is R+M bits.
   
~~~~
<------------- R --------->
              <- T -> <-N-> <---- M ---->
+---- ... ---+- ... -+-----+---- ... ----+---...---+-+
|   Rule ID  | DTag  |  1  |     MIC     | payload |P|
+---- ... ---+- ... -+-----+---- ... ----+---...---+-+
    
~~~~
{: #Fig-Last title='All-1 Fragment Detailed Format for the Last Fragment, No-ACK mode'}


   In any of the Window modes, the last fragment of an IPv6 datagram SHALL contain a fragment header that conforms to 
   the detailed format shown in {{Fig-LastWinMode}}. The total size of the fragment
   header in this format is R+M bits.
   
~~~~
<----------- R ------------>
           <- T -> 1 <- N -> <---- M ---->
+-- ... --+- ... -+-+- ... -+---- ... ----+---...---+-+
| Rule ID | DTag  |W| 11..1 |     MIC     | payload |P|
+-- ... --+- ... -+-+- ... -+---- ... ----+---...---+-+
                      (FCN)
~~~~
{: #Fig-LastWinMode title='All-1 Fragment Detailed Format for the Last Fragment, ACK-Always or ACK-on-Error'}
       
 In either ACK-Always or ACK-on-Error, in order to request a retransmission of the ACK for the All-1 window, the fragment sender uses the format shown in {{Fig-All1retries}}. The total size of the fragment header in this format is R+M bits.

~~~~
<------------ R ----------->
           <- T -> 1 <- N -> <---- M ---->
+-- ... --+- ... -+-+- ... -+---- ... ----+-+
| Rule ID | DTag  |W|  1..1 |     MIC     |P| (no payload)  
+-- ... --+- ... -+-+- ... -+---- ... ----+-+

~~~~
{: #Fig-All1retries title='All-1 for Retries format, also called All-1 empty'}

The values for R, N, T and M are not specified in this document, and are expected to be determined in other documents (e.g. technology-specific profile documents). 
   
### ACK format

The format of an ACK that acknowledges a window that is not the last one (denoted as All-0 window) is shown in {{Fig-ACK-Format}}.

~~~~
  <--------  R  ------->
              <- T -> 1  
  +---- ... --+-... -+-+----- ... ---+-+
  |  Rule ID  | DTag |W|   Bitmap    |P| (no payload)
  +---- ... --+-... -+-+----- ... ---+-+
                
~~~~
{: #Fig-ACK-Format title='ACK format for All-0 windows'}

To acknowledge the last window of a packet (denoted as All-1 window), a C bit (i.e. MIC checked) following the W bit is set
to 1 to indicate that the MIC check computed by the receiver matches the MIC present in the All-1 fragment. If the MIC check fails, the C bit is set to 0 and the Bitmap for the All-1 window follows.

~~~~
<--------  R  ------->  
            <- T -> 1 1
+---- ... --+-... -+-+-+-+
|  Rule ID  | DTag |W|1|P| (MIC correct)
+---- ... --+-... -+-+-+-+
                
+---- ... --+-... -+-+-+------- ... -------+-+
|  Rule ID  | DTag |W|0|      Bitmap       |P|(MIC Incorrect)
+---- ... --+-... -+-+-+------- ... -------+-+
                      C
                
~~~~
{: #Fig-ACK-Format1 title='Format of an ACK for All-1 windows'}

   
### Abort formats

When a fragment sender needs to abort the transmission, it sends the Sender-Abort format {{Fig-All1Abort}}, with all FCN bits set to 1.  When a fragment receiver needs to abort the on-going fragmented packet transmission, it transmits the Receiver-Abort format {{Fig-ACKabort}}. 

~~~~
<------------- R -----------><--- 1 byte --->
+--- ... ---+- ... -+-+-...-+-+-+-+-+-+-+-+-+
|  Rule ID  | DTag  |W| FCN |       FF      | (no MIC & no payload)  
+--- ... ---+- ... -+-+-...-+-+-+-+-+-+-+-+-+
   
~~~~
{: #Fig-All1Abort title='Sender-Abort format. All FCN fields in this format are set to 1'}


~~~~

 <----- byte boundary ------><--- 1 byte --->

 +---- ... --+-... -+-+-+-+-+-+-+-+-+-+-+-+-+
 |  Rule ID  | DTag |W| 1..1|       FF      |  
 +---- ... --+-... -+-+-+-+-+-+-+-+-+-+-+-+-+
 
~~~~
{: #Fig-ACKabort title='Receiver-Abort format'}


## Baseline mechanism

If after applying SCHC header compression (or when SCHC header compression is not possible) the entire datagram does not fit within the payload of a single L2 data unit, the datagram SHALL be broken into fragments and the fragments SHALL be sent to the fragment receiver.
The fragment receiver needs to identify all the fragments that belong to a given IPv6 datagram. To this end, the receiver SHALL use: 

 * The sender's L2 source address (if present), 
 
 * The destination's L2 address (if present),
 
 * Rule ID,
 
 * DTag (if present).
 
Then, the fragment receiver may determine the fragment delivery reliability mode that is used for this fragment based on the Rule ID in that fragment.

Upon receipt of a link fragment, the receiver starts constructing the original unfragmented packet.  It uses the FCN and the order of arrival of each fragment to determine the location of the individual fragments within the original unfragmented packet. A fragment payload may carry bytes from a SCHC compressed IPv6 header, an uncompressed IPv6 header or an IPv6 datagram data payload. For example, the receiver may place the fragment payload within a payload datagram reassembly buffer at the location determined from: the FCN, the arrival order of the fragments, and the fragment payload sizes. In Window mode, the fragment receiver also uses the W bit in the received fragments. Note that the size of the original, unfragmented packet cannot be determined from fragmentation headers.

Fragmentation functionality uses the FCN value, which has a length of N bits. The All-1 and All-0 FCN values are used to control the fragmentation transmission. The FCN MUST be assigned sequentially in a decreasing order. The first FCN of a window is RECOMMENDED to be 2^N-2, i.e. the highest possible FCN value depending on the FCN number of bits, but excluding the All-1 value. In all modes, the last fragment of a packet must contain a MIC which is used to check if there are errors or missing fragments, and must use the corresponding All-1 fragment format.  Also note that a fragment with an All-0 format is considered the last fragment of the current window.

If the recipient receives the last fragment of a datagram (All-1), it checks for the integrity of the reassembled datagram, based on the MIC received. In No-ACK, if the integrity check indicates that the reassembled datagram does not match the original datagram (prior to fragmentation), the reassembled datagram MUST be discarded. In Window mode, a MIC check is also performed by the fragment receiver after reception of each subsequent fragment retransmitted after the first MIC check. 


### No-ACK {#No-ACK-subsection}
In the No-ACK mode, there is no feedback communication from the fragment receiver. The sender will send all the fragments of a packet without any possibility of knowing if errors or losses have occurred. As, in this mode, there is no need to identify specific fragments, a one-bit FCN is used. Consequently, the FCN All-0 value is used in all fragments except the last one, which carries an All-1 FCN and the MIC.
The receiver will wait for fragments and will set the Inactivity timer. The receiver will use the MIC contained in the last fragment to check for errors.
When the Inactivity Timer expires or if the MIC check indicates that the reassembled packet does not match the original one, the receiver will release all resources allocated to reassembling this packet. The initial value of the Inactivity Timer will be determined based on the characteristics of the underlying LPWAN technology and will be defined in other documents (e.g. technology-specific profile documents).
    

### ACK-Always and ACK-on-Error 
In ACK-Always or ACK-on-Error, a jumping window protocol uses two windows alternatively, identified as 0 and 1.  A fragment with all FCN bits set to 0 (i.e. an All-0 fragment) indicates that the window is over (i.e. the fragment is the last one of the window) and allows to switch from one window to the next one.  The All-1 FCN in a fragment indicates that it is the last fragment of the packet being transmitted and therefore there will not be another window for this packet.

This section is divided in two parts, which define ACK-Always and ACK-on-Error modes, respectively.

#### ACK-Always {#ACK-Always-subsection}
In ACK-Always, the sender sends fragments by using the two-jumping-windows procedure. A delay between each fragment can be added to respect local regulations or other constraints imposed by the applications.  Each time a fragment is sent, the FCN is decreased by one.  When the FCN reaches value 0 and there are more fragments to be sent after the one at hand, the sender sends the fragment at hand using the All-0 fragment format. It starts the Retransmission Timer and waits for an ACK. By contrast, if the FCN has reached 0 and the fragment at hand is the last fragment of the packet, it is sent using the All-1 fragment format, which includes a MIC. The sender sets the Retransmission Timer and waits for the ACK.

The Retransmission Timer is dimensioned based on the LPWAN technology in use. On expiry of the Retransmission Timer, the sender sends an All-0 (resp. All-1) empty fragment to again request for the ACK for the window that ended with the All-0 (resp. All-1) fragment just sent. The window number is not changed.

When the sender receives an ACK, it checks the W bit carried by the ACK.  Any ACK carrying an unexpected W bit value is discarded.  If the W bit value of the received ACK is correct, the sender analyzes the rest of the ACK message.  If all the fragments sent for this window have been well received, and if at least one more fragment needs to be sent, the sender advances its sending window to the next window value and sends the next fragments.  If no more fragments have to be sent, then the fragmented packet transmission is finished.

However, if one or more fragments have not been received as per the ACK (i.e. the corresponding bits are not set in the Bitmap) then the sender resends the missing fragments.  When all missing fragments have been  retransmitted, the sender starts the Retransmission Timer (even if an All-0 or an All-1 has not been sent as part of this retransmission batch) and waits for an ACK. Upon receipt of the ACK, if one or more fragments have not yet been received, the counter Attempts is increased and the sender resends the missing fragments again. When Attempts reaches MAX_ACK_REQUESTS, the sender aborts the on-going fragmented packet transmission by sending an Abort message and releases any resources for transmission of the packet.  The sender also aborts an on-going fragmented packet transmission when a failed MIC check is reported by the receiver.

On the other hand, at the beginning, the receiver side expects to receive window 0.  Any fragment received but not belonging to the current window is discarded.  All fragments belonging to the correct window are accepted, and the actual fragment number managed by the receiver is computed based on the FCN value.  The receiver prepares the Bitmap to report the correctly received and the missing fragments for the current window. After each fragment is received the receiver initializes the Inactivity timer, if the Inactivity Timer expires the transmission is aborted. 

When an All-0 fragment is received, it indicates that all the fragments have been sent in the current window.  Since the 
sender is not obliged to always send a full window, some fragment number not set in the receiver memory may not correspond 
to losses.  The receiver sends the corresponding ACK, the Inactivity Timer is set and the transmission of the next window 
by the sender can start.

If an All-0 fragment has been received and all fragments of the current window have also been received, the receiver then 
expects a new Window and waits for the next fragment.  Upon receipt of a fragment, if the window value has not changed, the 
received fragments are part of a retransmission.  A receiver that has already received a fragment should discard it, 
otherwise, it updates the Bitmap.  If all the bits of the Bitmap are set to one, the receiver may send an ACK without 
waiting for an All-0 fragment and the Inactivity Timer is initialized.

On the other hand, if the window value of the next received fragment is set to the next expected window value, this means 
that the sender has received a correct Bitmap reporting that all fragments have been received.  The receiver then updates 
the value of the next expected window.

When an All-1 fragment is received, it indicates that the last fragment of the packet has been sent.  Since the last window 
is not always full, the MIC will be used to detect if all fragments of the packet have been received.  A correct MIC 
indicates the end of the transmission but the receiver must stay alive for an Inactivity Timer period to answer to any 
empty All-1 fragments the sender may send if ACKs sent by the receiver are lost. If the MIC is incorrect, some fragments 
have been lost.  The receiver sends the ACK regardless of successful fragmented packet reception or not, the Inactitivity 
Timer is set.  In case of an incorrect MIC, the receiver waits for fragments belonging to the same window. After 
MAX_ACK_REQUESTS, the receiver will abort the on-going fragmented packet transmission by transmitting a data unit with the Receiver-Abort format.  The receiver also aborts upon Inactivity Timer expiration.


#### ACK-on-Error {#ACK-on-Error-subsection}
The ACK-on-Error sender is similar to ACK-Always, the main difference being that in ACK-on-Error the ACK is not sent at the 
end of each window but only when at least one fragment of the current window has been lost (with the exception of the last 
window, see later text in this paragraph).  In Ack-on-Error,  the Retransmission Timer expiration will be considered as a positive 
acknowledgment. The Retransmission Timer is set when sending an All-0 or an All-1 fragment. When the All-1 fragment has 
been sent, then the on-going fragmented packet transmission fragmentation is finished and the sender waits for the last 
ACK. At the receiver side, when the All-1 fragment is received and the MIC check indicates successful packet reception, an ACK 
is sent nonetheless, to confirm the end of a correct transmission.  If the Retransmission Timer expires while waiting for the ACK for the last window, an All-1 empty request for
the last ACK MUST be sent by the sender to complete the fragmented packet transmission.

If the sender receives an ACK, it checks the window value.  ACKs with an unexpected window number are discarded.  If the 
window number on the received Bitmap is correct, the sender verifies if the receiver has received all fragments of the 
current window.  When at least one fragment has been lost, the counter Attempts is increased by one and the sender resends 
the missing fragments again.  When Attempts reaches MAX_ACK_REQUESTS, the sender sends an Abort message and releases all 
resources for the on-going fragmented packet transmission.  When the retransmission of missing fragments is finished, the 
sender starts listening for an ACK (even if an All-0 or an All-1 has not been sent during the retransmission) and 
initializes and starts the Retransmission Timer.  After sending an All-1 fragment, the sender listens for an ACK, 
initializes Attempts, and initializes and starts the Retransmission Timer.  If the Retransmission Timer expires, Attempts 
is increased by one and an empty All-1 fragment is sent to request the ACK for the last window. If Attempts reaches 
MAX_ACK_REQUESTS, the sender aborts the on-going fragmented packet transmission by transmitting the Sender-Abort data unit.

Unlike the sender, the receiver for ACK-on-Error has a larger amount of differences compared with ACK-Always.  First, an 
ACK is not sent unless there is a lost fragment or an unexpected behavior (with the exception of the last window, where an 
ACK is always sent regardless of fragment losses or not).  The receiver starts by expecting fragments from window 0 and 
maintains the information regarding which fragments it receives.  After receiving a fragment, the Inactivity Timer is set. 
If no further fragment is received and the Inactivity Timer expires, the fragment receiver aborts the on-going fragmented packet transmission by transmitting the Receiver-Abort data unit.

Any fragment not belonging to the current window is discarded.  The actual fragment number is computed based on the FCN 
value.  When an All-0 fragment is received and all fragments have been received, the receiver updates the expected window 
value.
 
If an All-0 fragment is received, even if another fragment is missing, all fragments from the current window have been 
sent.  Since the sender is not obligated to send a full window, a fragment number not used may not necessarily correspond 
to losses.  As the receiver does not know if the missing fragments are lost or not, it sends an ACK and reinitialises the 
Inactivity Timer.

On the other hand, after receiving an All-0 fragment, the receiver expects a new window and waits for the next fragment.  
If the window value of the next fragment has not changed, the received fragment is a retransmission.  A receiver that has 
already received a fragment should discard it.  If all fragments of a window (that is not the last one) have been received, 
the receiver does not send an ACK.  While the receiver waits for the next window and if the window value is set to the next 
value, and if an All-1 fragment with the next value window arrived the receiver aborts the on-going fragmented packet 
transmission, and it drops the fragments of the aborted packet transmission.

If the receiver receives an All-1 fragment, this means that the transmission should be finished.  If the MIC is incorrect 
some fragments have been lost.  Regardless of fragment losses, the receiver sends an ACK and initializes the Inactivity 
Timer.

Reception of an All-1 fragment indicates the last fragment of the packet has been sent.  Since the last window is not 
always full, the MIC will be used to detect if all fragments of the window have been received.  A correct MIC check 
indicates the end of the fragmented packet transmission. An ACK is sent by the fragment receiver. In case of an incorrect 
MIC, the receiver waits for fragments belonging to the same window or the expiration of the Inactivity Timer. The latter 
will lead the receiver to abort the on-going fragmented packet transmission.

### Bitmap Encoding {#Bitmapopt}

The Bitmap is transmitted by a receiver as part of the ACK format.  An ACK message may include padding at the end to align its number of transmitted bits to a multiple of 8 bits.  

Note that the ACK sent in response to an All-1 fragment includes the C bit. Therefore, the window size and thus the Bitmap size need to be determined taking into account the available space in the layer two frame payload, where there will be 1 bit less for an ACK sent in response to an All-1 fragment than in other ACKs. 

~~~~                                                  
                      <----       Bitmap bits      ---->   
| Rule ID | DTag |W|C|0|1|1|1|1|1|1|1|1|1|1|1|1|1|1|1|1|   
|--- byte boundary ----| 1 byte  next  |  1 byte next  |   
      
~~~~
{: #Fig-Localbitmap title='A non-optimized Bitmap'}

In order to reduce the resulting frame size, the Bitmap is shortened by applying the following algorithm: all the right-most contiguous bytes in the Bitmap that have all their bits set to 1 MUST NOT be transmitted.  Because the fragment sender knows the actual Bitmap size, it can reconstruct the original Bitmap even with the trailing 0xFF bytes optimized away.  In the example shown in {{Fig-transmittedbitmap}}, the last 2 bytes of the Bitmap shown in {{Fig-Localbitmap}} comprise bits that are all set to 1, therefore they are not sent.

In the last window, when the C bit value is 1, it means that the received MIC matches the one computed by the receiver, and thus the Bitmap is not sent.  Otherwise, the Bitmap is sent after the C bit. Note that the presence of a C bit may force to reduce the number of fragments in a window to allow the Bitmap to fit in a frame. That is, the maximum number of fragments of the last window is one unit smaller than that of the previous windows. 


~~~~   
     <-------   R  ------->  
                 <- T -> 1 
     +---- ... --+-... -+-+-+-+
     |  Rule ID  | DTag |W|1|0|
     +---- ... --+-... -+-+-+-+
     |---- byte boundary -----|    
     
~~~~
{: #Fig-transmittedbitmap title='Optimized Bitmap format'}

{{Fig-Bitmap-Win}} shows an example of an ACK with FCN ranging from 6 down to 0, where the Bitmap
indicates that the second and the fifth fragments have not been correctly received. 

~~~~                                                  
<------   R  ------>6 5 4 3 2 1   0 (*) 
          <- T -> 1   
+---------+------+-+-+-+-+-+-+-+-----+-------+
| Rule ID | DTag |W|1|0|1|1|0|1|all-0|padding|  Bitmap (not optimized)
+---------+------+-+-+-+-+-+-+-+-----+-------+
|<-- byte boundary --->|<----- 1 byte -----> | 
    (*)=(FCN values) 
    
+---- ... --+-... -+-+-+-+-+-+-+-+-+-+
|  Rule ID  | DTag |W|1|0|1|1|0|1|1|P|  transmitted Bitmap
+---- ... --+-... -+-+-+-+-+-+-+-+-+-+
|<-- byte boundary --->|<-- 1 byte-->| 
    
~~~~
{: #Fig-Bitmap-Win title='Example of a Bitmap before transmission, and the transmitted one, in any window except the last one'}

{{Fig-Bitmap-lastWin}} shows an example of an ACK (for N=3), where the Bitmap indicates that the MIC check has failed but there are no missing fragments. 

~~~~                                                  
 <-------   R  ------->  6 5 4 3 2 1 7 (*) 
             <- T -> 1 1
 |  Rule ID  | DTag |W|0|1|1|1|1|1|1|1|padding|  Bitmap (before tx)
 |---- byte boundary ----|  1 byte next |  1 byte next  |
                       C
 +---- ... --+-... -+-+-+-+
 |  Rule ID  | DTag |W|0|1| transmitted Bitmap
 +---- ... --+-... -+-+-+-+
 |---- byte boundary -----| 
   (*) = (FCN values indicating the order)
   
~~~~
{: #Fig-Bitmap-lastWin title='Example of the Bitmap in ACK-always or ACK-on-error for the last window, for N=3)'}


## Supporting multiple window sizes

For ACK-Always or ACK-on-Error, implementers may opt to support a single window size or multiple window sizes.  The latter, when feasible, may provide performance optimizations.  For example, a large window size may be used for packets that need to be carried by a large number of fragments.  However, when the number of fragments required to carry a packet is low, a smaller window size, and thus a shorter Bitmap, may be sufficient to provide feedback on all fragments.  If multiple window sizes are supported, the Rule ID may be used to signal the window size in use for a specific packet transmission.

Note that the same window size MUST be used for the transmission of all fragments that belong to the same packet.


## Downlink fragment transmission

In some LPWAN technologies, as part of energy-saving techniques, downlink transmission is only possible immediately after an uplink transmission. In order to avoid potentially high delay in the downlink transmission of a fragmented datagram, the fragment receiver MAY perform an uplink transmission as soon as possible after reception of a fragment that is not the last one. Such uplink transmission may be triggered by the L2 (e.g. an L2 ACK sent in response to a fragment encapsulated in a L2 frame that requires an L2 ACK) or it may be triggered from an upper layer.

For downlink transmission of a fragmented packet in ACK-Always
   mode, the fragment receiver MAY support timer-based ACK
   retransmission. In this mechanism, the fragment receiver initializes and
   starts a timer (the Inactivity Timer is used) after the transmission of an
   ACK, except when the ACK is sent in response to the last fragment of a
   packet (All-1 fragment). In the latter case, the fragment receiver does 
   not start a timer after transmission of the ACK.

   If, after transmission of an ACK that is not an All-1 fragment, and 
   before expiration of the corresponding Inactivity timer, the fragment receiver receives a fragment that belongs to
   the current window (e.g. a missing fragment from the current window) or 
   to the next window, the Inactivity timer for the ACK is stopped. However, 
   if the Inactivity timer expires, the ACK is resent and the Inactivity timer
   is reinitialized and restarted.

   The default initial value for the Inactivity timer, as well as the
   maximum number of retries for a specific ACK, denoted MAX_ACK_RETRIES,
   are not defined in this document, and need to be defined in other
   documents (e.g. technology-specific profiles). The initial value of the
   Inactivity timer is expected to be greater than that of the Retransmission
   timer, in order to make sure that a (buffered) fragment to be
   retransmitted can find an opportunity for that transmission.

   When the fragment sender transmits the All-1 fragment, it
   starts its Retransmission Timer with a
   large timeout value (e.g. several times that of the initial Inactivity timer). If an ACK
   is received before expiration of this timer, the fragment sender
   retransmits any lost fragments reported by the ACK, or if the ACK
   confirms successful reception of all fragments of
   the last window, the transmission of the fragmented packet is considered complete.
   If the timer expires, and no ACK has been received since the
   start of the timer, the fragment sender assumes that the All-1
   fragment has been successfully received (and possibly, the last ACK
   has been lost: this mechanism assumes that the retransmission timer for the
   All-1 fragment is long enough to allow several ACK retries if the All-1
   fragment has not been received by the fragment receiver, and it also
   assumes that it is unlikely that several ACKs become all lost).


# Padding management {#Padding}

The headers specified in this document, be there for compression, fragmentation or acknowledgment, are not necessarily an integer number of bytes in size.
Some LPWAN technologies have PDUs that are integer numbers of bytes.
With such a technology, the sender can append padding bits to the messages defined in this document in order to fill up the last byte of the L2 PDU, if it isn't already full.
Examples are shown in {{Fig-FormatPckt}} and {{Fig-FragFormat}}.

The receiver will tell the header, the payload and the padding apart using the following principles:

 * The size of any SCHC header is known from examining the Rule ID and the content of that header.
 
 * The payload that follows the header, if it exists, is variable in size, but is always a integer nuber of bytes.
 
 * Padding MUST not add more than 7 bits.

Therefore, the algorithm for padding elimination at the receiver is the following:

* decode the SCHC header, find its length (in bits) and set a pointer to the end of the header.

* from that pointer, extract as many blocks of 8 bits as are available in the L2 PDU. They are the payload bytes.

* the remaining bits, if any, are padding.



# SCHC Compression for IPv6 and UDP headers

This section lists the different IPv6 and UDP header fields and how they can be compressed.

## IPv6 version field

This field always holds the same value. Therefore, in the rule, TV is set to 6, MO to "equal"
and CDA to "not-sent".

## IPv6 Traffic class field

If the DiffServ field identified by the rest of the rule does not vary and is known 
by both sides, the Field Descriptor in the rule should contain a TV with this well-known value, an "equal" MO
and a "not-sent" CDA.

Otherwise, two possibilities can be considered:

* One possibility is to not compress the field and send the original value. In the rule, TV is not set to any particular value, MO is set to "ignore" and CDA is set to "value-sent".

* If some upper bits in the field are constant and known, a better option is to only send the LSBs. In the rule, TV is set to a value with the stable known upper part, MO is set to MSB(X) and CDA to LSB.

## Flow label field

If the Flow Label field identified by the rest of the rule does not vary and is known 
by both sides, the Field Descriptor in the rule should contain a TV with this well-known value, an "equal" MO
and a "not-sent" CDA.

Otherwise, two possibilities can be considered:

* One possibility is to not compress the field and send the original value. In the rule, TV is not set to any particular value, MO is set to "ignore" and CDA is set to "value-sent".

* If some upper bits in the field are constant and known, a better option is to only send the LSBs. In the rule, TV is set to a value with the stable known upper part, MO is set to MSB(X) and CDA to LSB.

## Payload Length field

If the LPWAN technology does not add padding, this field can be elided for the 
transmission on the LPWAN network. The SCHC C/D recomputes the original payload length
value. In the Field Descriptor, TV is not set, MO is set to "ignore" and CDA is "compute-IPv6-length".

If the payload length needs to be sent and is known to need less than 16 bits for its encoding, the TV can be set to 0x0000,
the MO set to "MSB (16-s)" and the
CDA to "LSB". The 's' parameter depends on the expected maximum packet length.

Otherwise, the payload length field must be sent and the CDA is replaced by "value-sent".

## Next Header field

If the Next Header field identified by the rest of the rule does not vary and is known 
by both sides, the Field Descriptor in the rule should contain a TV with this Next Header value, the MO should be "equal"
and the CDA should be "not-sent".

Otherwise, TV is not set in the Field Descriptor, MO is set to "ignore" and CDA is set to
"value-sent". Alternatively, a matching-list may also be used.

## Hop Limit field

Note that the value pattern for this field is different for Uplink and Downlink. In Uplink, since there is
no IP forwarding between the Dev and the SCHC C/D, the value is relatively constant. On the
other hand, the Downlink value depends of Internet routing and may change more frequently.
One neat way of processing this field is to use the Direction Indicator (DI) to distinguish between directions:

* in the Uplink, elide the field: the TV in the Field Descriptor is set to the known constant value, the MO
is set to "equal" and the CDA is set to "not-sent".

* in the Downlink, send the value: TV is not set, MO is set to "ignore" and
CDA is set to "value-sent".

## IPv6 addresses fields

As in 6LoWPAN {{RFC4944}}, IPv6 addresses are split into two 64-bit long fields;
one for the prefix and one for the Interface Identifier (IID). These fields should
be compressed. To allow for a single rule being used for both directions, these values are identified by their role
(DEV or APP) and not by their position in the frame (source or destination). The SCHC C/D
must be aware of the traffic direction (Uplink, Downlink) to select the appropriate
field.

### IPv6 source and destination prefixes

Both ends must be synchronized with the appropriate prefixes. For a specific flow, 
the source and destination prefixes can be unique and stored in the context. It can 
be either a link-local prefix or a global prefix. In that case, the TV for the 
source and destination prefixes contain the values, the MO is set to "equal" and
the CDA is set to "not-sent".

If the rule is intended to compress packets with different prefix values, match-mapping should be used. The
different prefixes are listed in the TV associated with a short ID. The MO is set 
to "match-mapping" and the CDA is set to "mapping-sent".

Otherwise, the TV contains the prefix, the MO is set to "equal" and the CDA is set to
"value-sent".

### IPv6 source and destination IID

If the DEV or APP IID are based on an LPWAN address, then the IID can be reconstructed 
with information coming from the LPWAN header. In that case, the TV is not set, the MO 
is set to "ignore" and the CDA is set to "DEViid" or "APPiid". Note that the 
LPWAN technology generally carries a single identifier, which corresponds
to the DEV. The SCHC C/D may also not be aware of these values. 

For privacy reasons or if the DEV address is changing over time, a static value that is not equal to the DEV address SHOULD be used.  In that case, the TV contains the static value, the MO operator is set to "equal" and the CDF is set to "not-sent". RFC 7217 provides some methods that MAY be used to derive this static identifier. 

If several IIDs are possible, then the TV contains the list of possible IIDs, the MO is 
set to "match-mapping" and the CDA is set to "mapping-sent". 

It may also happen that the IID variability only expresses itself on a few bytes. In that case, the TV is
set to the stable part of the IID, the MO is set to "MSB" and the CDA is set to "LSB".

Finally, the IID can be sent in extenso on the LPWAN. In that case, the TV is not set, the MO is set
to "ignore" and the CDA is set to "value-sent".

## IPv6 extensions

No rule is currently defined that processes IPv6 extensions.
If such extensions are needed, their compression/decompression rules can be based on the MOs and
CDAs described above.

## UDP source and destination port

To allow for a single rule being used for both directions, the UDP port values are identified by their role
(DEV or APP) and not by their position in the frame (source or destination). The SCHC C/D
must be aware of the traffic direction (Uplink, Downlink) to select the appropriate
field. The following rules apply for DEV and APP port numbers.

If both ends know the port number, it can be elided. The TV contains the port number,
the MO is set to "equal" and the CDA is set to "not-sent".

If the port variation is on few bits, the TV contains the stable part of the port number,
the MO is set to "MSB" and the CDA is set to "LSB".

If some well-known values are used,  the TV can contain the list of these values, the
MO is set to "match-mapping" and the CDA is set to "mapping-sent".

Otherwise the port numbers are sent over the LPWAN. The TV is not set, the MO is
set to "ignore" and the CDA is set to "value-sent".

## UDP length field

If the LPWAN technology does not introduce padding, the UDP length can be computed
from the received data. In that case, the TV is not set, the MO is set to "ignore" and
the CDA is set to "compute-UDP-length".

If the payload is small, the TV can be set to 0x0000, the MO set to "MSB" and the
CDA to "LSB". 

In other cases, the length must be sent and the CDA is replaced by "value-sent".

## UDP Checksum field

IPv6 mandates a checksum in the protocol above IP. Nevertheless, if a more efficient
mechanism such as L2 CRC or MIC is carried by or over the L2 (such as in the
LPWAN fragmentation process (see {{Frag}})), the UDP checksum transmission can be avoided.
In that case, the TV is not set, the MO is set to "ignore" and the CDA is set to
"compute-UDP-checksum".

In other cases, the checksum must be explicitly sent. The TV is not set, the MO is set to
"ignore" and the CDF is set to "value-sent".

# Security considerations

## Security considerations for header compression
A malicious header compression could cause the reconstruction of a 
wrong packet that does not match with the original one. Such a corruption
may be detected with end-to-end authentication and integrity mechanisms. 
Denial of Service may be produced but its arise other security problems 
that may be solved with or without header compression.

## Security considerations for fragmentation
This subsection describes potential attacks to LPWAN fragmentation 
and suggests possible countermeasures.

A node can perform a buffer reservation attack by sending a first
fragment to a target.  Then, the receiver will reserve buffer space
for the IPv6 packet.  Other incoming fragmented packets will be
dropped while the reassembly buffer is occupied during the reassembly
timeout.  Once that timeout expires, the attacker can repeat the same
procedure, and iterate, thus creating a denial of service attack.
The (low) cost to mount this attack is linear with the number of
buffers at the target node.  However, the cost for an attacker can be
increased if individual fragments of multiple packets can be stored
in the reassembly buffer.  To further increase the attack cost, the
reassembly buffer can be splitted into fragment-sized buffer slots.
Once a packet is complete, it is processed normally.  If buffer
overload occurs, a receiver can discard packets based on the sender
behavior, which may help identify which fragments have been sent by
an attacker.

In another type of attack, the malicious node is required to have
overhearing capabilities.  If an attacker can overhear a fragment, it
can send a spoofed duplicate (e.g. with random payload) to the
destination. If the LPWAN technology does not support suitable protection 
(e.g. source authentication and frame counters to prevent replay attacks), 
a receiver cannot distinguish legitimate from spoofed fragments.  Therefore, 
the original IPv6 packet will be considered corrupt and will be dropped.
To protect resource-constrained nodes from this attack, it has been proposed
to establish a binding among the fragments to be transmitted by a node, 
by applying content-chaining to the different fragments, based on cryptographic
hash functionality.  The aim of this technique is to allow a receiver to
identify illegitimate fragments.

Further attacks may involve sending overlapped fragments (i.e.
comprising some overlapping parts of the original IPv6 datagram).
Implementers should make sure that the correct operation is not affected
by such event.

In Window mode – ACK on error, a malicious node may force a fragment sender to resend a fragment a number of times, with the aim to increase consumption of the fragment sender’s resources. To this end, the malicious node may repeatedly send a fake ACK to the fragment sender, with a Bitmap that reports that one or more fragments have been lost. In order to mitigate this possible attack, MAX_FRAG_RETRIES may be set to a safe value which allows to limit the maximum damage of the attack to an acceptable extent. However, note that a high setting for MAX_FRAG_RETRIES benefits fragment delivery reliability, therefore the trade-off needs to be carefully considered.

# Acknowledgements

Thanks to Dominique Barthel, Carsten Bormann, Philippe Clavier, Eduardo Ingles Sanchez, Arunprabhu Kandasamy, 
Sergio Lopez Bernal, Antony Markovski, Alexander Pelov, Pascal Thubert, Juan Carlos Zuniga, Diego Dujovne, Edgar Ramos, and Shoichi Sakane for useful design consideration and comments. 

--- back

# SCHC Compression Examples {#compressIPv6}


This section gives some scenarios of the compression mechanism for IPv6/UDP.
The goal is to illustrate the behavior of SCHC.

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
 Management   Data
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
Therefore, when such technologies are used, it is necessary to statically define an IID for the Link
Local address for the SCHC C/D.

~~~~

Rule 0
 +----------------+--+--+--+---------+--------+------------++------+
 | Field          |FL|FP|DI| Value   | Match  | Comp Decomp|| Sent |
 |                |  |  |  |         | Opera. | Action     ||[bits]|
 +----------------+--+--+--+---------+---------------------++------+
 |IPv6 version    |4 |1 |Bi|6        | equal  | not-sent   ||      |
 |IPv6 DiffServ   |8 |1 |Bi|0        | equal  | not-sent   ||      |
 |IPv6 Flow Label |20|1 |Bi|0        | equal  | not-sent   ||      |
 |IPv6 Length     |16|1 |Bi|         | ignore | comp-length||      |
 |IPv6 Next Header|8 |1 |Bi|17       | equal  | not-sent   ||      |
 |IPv6 Hop Limit  |8 |1 |Bi|255      | ignore | not-sent   ||      |
 |IPv6 DEVprefix  |64|1 |Bi|FE80::/64| equal  | not-sent   ||      |
 |IPv6 DEViid     |64|1 |Bi|         | ignore | DEViid     ||      |
 |IPv6 APPprefix  |64|1 |Bi|FE80::/64| equal  | not-sent   ||      |
 |IPv6 APPiid     |64|1 |Bi|::1      | equal  | not-sent   ||      |
 +================+==+==+==+=========+========+============++======+
 |UDP DEVport     |16|1 |Bi|123      | equal  | not-sent   ||      |
 |UDP APPport     |16|1 |Bi|124      | equal  | not-sent   ||      |
 |UDP Length      |16|1 |Bi|         | ignore | comp-length||      |
 |UDP checksum    |16|1 |Bi|         | ignore | comp-chk   ||      |
 +================+==+==+==+=========+========+============++======+

 Rule 1
 +----------------+--+--+--+---------+--------+------------++------+
 | Field          |FL|FP|DI| Value   | Match  | Action     || Sent |
 |                |  |  |  |         | Opera. | Action     ||[bits]|
 +----------------+--+--+--+---------+--------+------------++------+
 |IPv6 version    |4 |1 |Bi|6        | equal  | not-sent   ||      |
 |IPv6 DiffServ   |8 |1 |Bi|0        | equal  | not-sent   ||      |
 |IPv6 Flow Label |20|1 |Bi|0        | equal  | not-sent   ||      |
 |IPv6 Length     |16|1 |Bi|         | ignore | comp-length||      |
 |IPv6 Next Header|8 |1 |Bi|17       | equal  | not-sent   ||      |
 |IPv6 Hop Limit  |8 |1 |Bi|255      | ignore | not-sent   ||      |
 |IPv6 DEVprefix  |64|1 |Bi|[alpha/64, match- |mapping-sent||  [1] |
 |                |  |  |  |fe80::/64] mapping|            ||      |
 |IPv6 DEViid     |64|1 |Bi|         | ignore | DEViid     ||      |
 |IPv6 APPprefix  |64|1 |Bi|[beta/64,| match- |mapping-sent||  [2] |
 |                |  |  |  |alpha/64,| mapping|            ||      |
 |                |  |  |  |fe80::64]|        |            ||      |
 |IPv6 APPiid     |64|1 |Bi|::1000   | equal  | not-sent   ||      |
 +================+==+==+==+=========+========+============++======+
 |UDP DEVport     |16|1 |Bi|5683     | equal  | not-sent   ||      |
 |UDP APPport     |16|1 |Bi|5683     | equal  | not-sent   ||      |
 |UDP Length      |16|1 |Bi|         | ignore | comp-length||      |
 |UDP checksum    |16|1 |Bi|         | ignore | comp-chk   ||      |
 +================+==+==+==+=========+========+============++======+
 
 Rule 2
 +----------------+--+--+--+---------+--------+------------++------+
 | Field          |FL|FP|DI| Value   | Match  | Action     || Sent |
 |                |  |  |  |         | Opera. | Action     ||[bits]|
 +----------------+--+--+--+---------+--------+------------++------+
 |IPv6 version    |4 |1 |Bi|6        | equal  | not-sent   ||      |
 |IPv6 DiffServ   |8 |1 |Bi|0        | equal  | not-sent   ||      |
 |IPv6 Flow Label |20|1 |Bi|0        | equal  | not-sent   ||      |
 |IPv6 Length     |16|1 |Bi|         | ignore | comp-length||      |
 |IPv6 Next Header|8 |1 |Bi|17       | equal  | not-sent   ||      |
 |IPv6 Hop Limit  |8 |1 |Up|255      | ignore | not-sent   ||      |
 |IPv6 Hop Limit  |8 |1 |Dw|         | ignore | value-sent ||  [8] |
 |IPv6 DEVprefix  |64|1 |Bi|alpha/64 | equal  | not-sent   ||      |
 |IPv6 DEViid     |64|1 |Bi|         | ignore | DEViid     ||      |
 |IPv6 APPprefix  |64|1 |Bi|gamma/64 | equal  | not-sent   ||      |
 |IPv6 APPiid     |64|1 |Bi|::1000   | equal  | not-sent   ||      |
 +================+==+==+==+=========+========+============++======+
 |UDP DEVport     |16|1 |Bi|8720     | MSB(12)| LSB(4)     || [4]  |
 |UDP APPport     |16|1 |Bi|8720     | MSB(12)| LSB(4)     || [4]  |
 |UDP Length      |16|1 |Bi|         | ignore | comp-length||      |
 |UDP checksum    |16|1 |Bi|         | ignore | comp-chk   ||      |
 +================+==+==+==+=========+========+============++======+


~~~~
{: #Fig-fields title='Context rules'}

All the fields described in the three rules depicted on {{Fig-fields}} are present
in the IPv6 and UDP headers.  The DEViid-DID value is found in the L2
header.

The second and third rules use global addresses. The way the Dev learns the
prefix is not in the scope of the document. 

The third rule compresses port numbers to 4 bits. 


# Fragmentation Examples 

This section provides examples for the different fragment delivery reliability modes specified in this document.

{{Fig-Example-Unreliable}} illustrates the transmission in No-ACK mode of an IPv6 packet that needs 11 fragments. FCN is 1 bit wide.

~~~~
        Sender               Receiver
          |-------FCN=0-------->|
          |-------FCN=0-------->|
          |-------FCN=0-------->|
          |-------FCN=0-------->|
          |-------FCN=0-------->|
          |-------FCN=0-------->|
          |-------FCN=0-------->|
          |-------FCN=0-------->|
          |-------FCN=0-------->|
          |-------FCN=0-------->|
          |-----FCN=1 + MIC --->|MIC checked: success =>
         
~~~~
{: #Fig-Example-Unreliable title='Transmission in No-ACK mode of an IPv6 packet carried by 11 fragments'}

In the following examples, N (i.e. the size if the FCN field) is 3 bits. Therefore, the All-1 FCN value is 7.

{{Fig-Example-Win-NoLoss-NACK}} illustrates the transmission in ACK-on-Error of an IPv6 packet that needs 11 fragments, with MAX_WIND_FCN=6 and no fragment loss.

~~~~
        Sender               Receiver
          |-----W=0, FCN=6----->|
          |-----W=0, FCN=5----->|
          |-----W=0, FCN=4----->|
          |-----W=0, FCN=3----->|
          |-----W=0, FCN=2----->|
          |-----W=0, FCN=1----->|
          |-----W=0, FCN=0----->|
      (no ACK)
          |-----W=1, FCN=6----->|
          |-----W=1, FCN=5----->|
          |-----W=1, FCN=4----->|
          |--W=1, FCN=7 + MIC-->|MIC checked: success =>
          |<---- ACK, W=1 ------|

~~~~
{: #Fig-Example-Win-NoLoss-NACK title='Transmission in ACK-on-Error mode of an IPv6 packet carried by 11 fragments, with MAX_WIND_FCN=6 and no loss.'}

{{Fig-Example-Rel-Window-NACK-Loss}} illustrates the transmission in ACK-on-Error mode of an IPv6 packet that needs 11 fragments, with MAX_WIND_FCN=6 and three lost fragments.

~~~~
         Sender             Receiver
          |-----W=0, FCN=6----->|
          |-----W=0, FCN=5----->|
          |-----W=0, FCN=4--X-->|
          |-----W=0, FCN=3----->|
          |-----W=0, FCN=2--X-->|             7
          |-----W=0, FCN=1----->|             /
          |-----W=0, FCN=0----->|       6543210
          |<-----ACK, W=0-------|Bitmap:1101011
          |-----W=0, FCN=4----->|
          |-----W=0, FCN=2----->|   
      (no ACK)     
          |-----W=1, FCN=6----->|
          |-----W=1, FCN=5----->|
          |-----W=1, FCN=4--X-->|
          |- W=1, FCN=7 + MIC ->|MIC checked: failed
          |<-----ACK, W=1-------|C=0 Bitmap:1100001
          |-----W=1, FCN=4----->|MIC checked: success =>
          |<---- ACK, W=1 ------|C=1, no Bitmap

~~~~
{: #Fig-Example-Rel-Window-NACK-Loss title='Transmission in ACK-on-Error mode of an IPv6 packet carried by 11 fragments, with MAX_WIND_FCN=6 and three lost fragments.'}

{{Fig-Example-Rel-Window-ACK-NoLoss}} illustrates the transmission in ACK-Always mode of an IPv6 packet that needs 11 fragments, with MAX_WIND_FCN=6 and no loss. 

~~~~
        Sender               Receiver
          |-----W=0, FCN=6----->|
          |-----W=0, FCN=5----->|
          |-----W=0, FCN=4----->|
          |-----W=0, FCN=3----->|
          |-----W=0, FCN=2----->|
          |-----W=0, FCN=1----->|
          |-----W=0, FCN=0----->|
          |<-----ACK, W=0-------| Bitmap:1111111
          |-----W=1, FCN=6----->|
          |-----W=1, FCN=5----->|   
          |-----W=1, FCN=4----->|
          |--W=1, FCN=7 + MIC-->|MIC checked: success =>
          |<-----ACK, W=1-------| C=1 no Bitmap
        (End)    

~~~~
{: #Fig-Example-Rel-Window-ACK-NoLoss title='Transmission in ACK-Always mode of an IPv6 packet carried by 11 fragments, with MAX_WIND_FCN=6 and no lost fragment.'}

{{Fig-Example-Rel-Window-ACK-Loss}} illustrates the transmission in ACK-Always mode of an IPv6 packet that needs 11 fragments, with MAX_WIND_FCN=6 and three lost fragments.

~~~~
        Sender               Receiver
          |-----W=1, FCN=6----->|
          |-----W=1, FCN=5----->|
          |-----W=1, FCN=4--X-->|
          |-----W=1, FCN=3----->|
          |-----W=1, FCN=2--X-->|             7
          |-----W=1, FCN=1----->|             /
          |-----W=1, FCN=0----->|       6543210
          |<-----ACK, W=1-------|Bitmap:1101011
          |-----W=1, FCN=4----->|
          |-----W=1, FCN=2----->|
          |<-----ACK, W=1-------|Bitmap:
          |-----W=0, FCN=6----->|
          |-----W=0, FCN=5----->|   
          |-----W=0, FCN=4--X-->|
          |--W=0, FCN=7 + MIC-->|MIC checked: failed
          |<-----ACK, W=0-------| C= 0 Bitmap:11000001
          |-----W=0, FCN=4----->|MIC checked: success =>
          |<-----ACK, W=0-------| C= 1 no Bitmap
        (End)    

~~~~
{: #Fig-Example-Rel-Window-ACK-Loss title='Transmission in ACK-Always mode of an IPv6 packet carried by 11 fragments, with MAX_WIND_FCN=6 and three lost fragments.'}

{{Fig-Example-Rel-Window-ACK-Loss-Last-A}} illustrates the transmission in ACK-Always mode of an IPv6 packet that needs 6 fragments, with MAX_WIND_FCN=6, three lost fragments and only one retry needed to recover each lost fragment. Note that, since a single window is needed for transmission of the IPv6 packet in this case, the example illustrates behavior when losses happen in the last window.

~~~~
          Sender                Receiver
             |-----W=0, FCN=6----->|
             |-----W=0, FCN=5----->|
             |-----W=0, FCN=4--X-->|
             |-----W=0, FCN=3--X-->|
             |-----W=0, FCN=2--X-->|
             |--W=0, FCN=7 + MIC-->|MIC checked: failed
             |<-----ACK, W=0-------|C= 0 Bitmap:1100001
             |-----W=0, FCN=4----->|MIC checked: failed
             |-----W=0, FCN=3----->|MIC checked: failed
             |-----W=0, FCN=2----->|MIC checked: success
             |<-----ACK, W=0-------|C=1 no Bitmap
           (End) 
~~~~
{: #Fig-Example-Rel-Window-ACK-Loss-Last-A title='Transmission in ACK-Always mode of an IPv6 packet carried by 11 fragments, fwith MAX_WIND_FCN=6, three lost framents and only one retry needed for each lost fragment.'}

{{Fig-Example-Rel-Window-ACK-Loss-Last-B}} illustrates the transmission in ACK-Always mode of an IPv6 packet that needs 6 fragments, with MAX_WIND_FCN=6, three lost fragments, and the second ACK lost. 

~~~~
          Sender                Receiver
             |-----W=0, FCN=6----->|
             |-----W=0, FCN=5----->|
             |-----W=0, FCN=4--X-->|
             |-----W=0, FCN=3--X-->|
             |-----W=0, FCN=2--X-->|
             |--W=0, FCN=7 + MIC-->|MIC checked: failed
             |<-----ACK, W=0-------|C=0  Bitmap:1100001
             |-----W=0, FCN=4----->|MIC checked: failed
             |-----W=0, FCN=3----->|MIC checked: failed
             |-----W=0, FCN=2----->|MIC checked: success
             |  X---ACK, W=0-------|C= 1 no Bitmap
    timeout  |                     |
             |--W=0, FCN=7 + MIC-->|
             |<-----ACK, W=0-------|C= 1 no Bitmap  

           (End) 
~~~~
{: #Fig-Example-Rel-Window-ACK-Loss-Last-B title='Transmission in ACK-Always mode of an IPv6 packet carried by 11 fragments, with MAX_WIND_FCN=6, three lost fragments, and the second ACK lost.'}

{{Fig-Example-Rel-Window-ACK-Loss-Last-C}} illustrates the transmission in ACK-Always mode of an IPv6 packet that needs 6 fragments, with MAX_WIND_FCN=6, with three lost fragments, and one retransmitted fragment lost again. 

~~~~
           Sender                Receiver
             |-----W=0, FCN=6----->|
             |-----W=0, FCN=5----->|
             |-----W=0, FCN=4--X-->|
             |-----W=0, FCN=3--X-->|
             |-----W=0, FCN=2--X-->|
             |--W=0, FCN=7 + MIC-->|MIC checked: failed
             |<-----ACK, W=0-------|C=0   Bitmap:1100001
             |-----W=0, FCN=4----->|MIC checked: failed
             |-----W=0, FCN=3----->|MIC checked: failed
             |-----W=0, FCN=2--X-->|
      timeout|                     |
             |--W=0, FCN=7 + MIC-->|All-0 empty
             |<-----ACK, W=0-------|C=0 Bitmap: 1111101
             |-----W=0, FCN=2----->|MIC checked: success
             |<-----ACK, W=0-------|C=1 no Bitmap
           (End) 
~~~~
{: #Fig-Example-Rel-Window-ACK-Loss-Last-C title='Transmission in ACK-Always mode of an IPv6 packet carried by 11 fragments, with MAX_WIND_FCN=6, with three lost fragments, and one retransmitted fragment lost again.'}


{{Fig-Example-MaxWindFCN}} illustrates the transmission in ACK-Always mode of an IPv6 packet that needs 28 fragments, with N=5, MAX_WIND_FCN=23 and two lost fragments. Note that MAX_WIND_FCN=23 may be useful when the maximum possible Bitmap size, considering the maximum lower layer technology payload size and the value of R, is 3 bytes. Note also that the FCN of the last fragment of the packet is the one with FCN=31 (i.e. FCN=2^N-1 for N=5, or equivalently, all FCN bits set to 1).

~~~~
           Sender               Receiver
             |-----W=0, FCN=23----->|
             |-----W=0, FCN=22----->|
             |-----W=0, FCN=21--X-->|
             |-----W=0, FCN=20----->|
             |-----W=0, FCN=19----->|
             |-----W=0, FCN=18----->|
             |-----W=0, FCN=17----->|
             |-----W=0, FCN=16----->|
             |-----W=0, FCN=15----->|
             |-----W=0, FCN=14----->|
             |-----W=0, FCN=13----->|
             |-----W=0, FCN=12----->|
             |-----W=0, FCN=11----->|
             |-----W=0, FCN=10--X-->|
             |-----W=0, FCN=9 ----->|
             |-----W=0, FCN=8 ----->|
             |-----W=0, FCN=7 ----->|
             |-----W=0, FCN=6 ----->|
             |-----W=0, FCN=5 ----->|
             |-----W=0, FCN=4 ----->|
             |-----W=0, FCN=3 ----->|
             |-----W=0, FCN=2 ----->|
             |-----W=0, FCN=1 ----->|
             |-----W=0, FCN=0 ----->|
             |                      |lcl-Bitmap:110111111111101111111111
             |<------ACK, W=0-------| Bitmap:1101111111111011
             |-----W=0, FCN=21----->|
             |-----W=0, FCN=10----->|
             |<------ACK, W=0-------|no Bitmap
             |-----W=1, FCN=23----->|
             |-----W=1, FCN=22----->|
             |-----W=1, FCN=21----->|
             |--W=1, FCN=31 + MIC-->|MIC checked: sucess =>
             |<------ACK, W=1-------|no Bitmap
           (End)
~~~~
{: #Fig-Example-MaxWindFCN title='Transmission in ACK-Always mode of an IPv6 packet carried by 28 fragments, with N=5, MAX_WIND_FCN=23 and two lost fragments.'}


# Fragmentation State Machines

The fragmentation state machines of the sender and the receiver, one for each of the different reliability modes, are described in the following figures:


~~~~
             +===========+
+------------+  Init     |                                      
|  FCN=0     +===========+                                      
|  No Window                                       
|  No Bitmap                                                      
|                   +-------+           
|          +========+==+    | More Fragments                 
|          |           | <--+ ~~~~~~~~~~~~~~~~~~~~                          
+--------> |   Send    |      send Fragment (FCN=0)                            
           +===+=======+                                                                      
               |  last fragment 
               |  ~~~~~~~~~~~~                               
               |  FCN = 1                               
               v  send fragment+MIC 
           +============+                                             
           |    END     |                                             
           +============+                       
~~~~
{: #Fig-NoACKModeSnd title='Sender State Machine for the No-ACK Mode'}



~~~~
                      +------+ Not All-1
           +==========+=+    | ~~~~~~~~~~~~~~~~~~~
           |            + <--+ set Inactivity Timer
           |  RCV Frag  +-------+
           +=+===+======+       |All-1 &
   All-1 &   |   |              |MIC correct
 MIC wrong   |   |Inactivity    |
             |   |Timer Exp.    |
             v   |              |
  +==========++  |              v
  |   Error   |<-+     +========+==+
  +===========+        |    END    |
                       +===========+ 
                                           
~~~~
{: #Fig-NoACKModeRcv title='Receiver State Machine for the No-ACK Mode'}



~~~~
              +=======+  
              | INIT  |       FCN!=0 & more frags
              |       |       ~~~~~~~~~~~~~~~~~~~~~~
              +======++  +--+ send Window + frag(FCN)
                 W=0 |   |  | FCN-
  Clear local Bitmap |   |  v set local Bitmap
       FCN=max value |  ++==+========+
                     +> |            |
+---------------------> |    SEND    |
|                       +==+=====+===+ 
|      FCN==0 & more frags |     | last frag
|    ~~~~~~~~~~~~~~~~~~~~~ |     | ~~~~~~~~~~~~~~~
|         set local-Bitmap |     | set local-Bitmap 
|   send wnd + frag(all-0) |     | send wnd+frag(all-1)+MIC 
|       set Retrans_Timer  |     | set Retrans_Timer 
|                          |     | 
|Recv_wnd == wnd &         |     |  
|Lcl_Bitmap==recv_Bitmap&  |     |  +------------------------+
|more frag                 |     |  |local-Bitmap!=rcv-Bitmap|
|~~~~~~~~~~~~~~~~~~~~~~    |     |  | ~~~~~~~~~              |
|Stop Retrans_Timer        |     |  | Attemp++               v
|clear local_Bitmap        v     v  |                 +======++
|window=next_window   +====+=====+==+==+              |Resend |
+---------------------+                |              |Missing|
                 +----+     Wait       |              |Frag   |
not expected wnd |    |    Bitmap      |              +======++
~~~~~~~~~~~~~~~~ +--->+                +-+Retrans_Timer Exp  |          
    discard frag      +==+=+===+=+===+=+ |~~~~~~~~~~~~~~~~~  |
                         | |   | ^   ^   |reSend(empty)All-* |   
                         | |   | |   |   |Set Retrans_Timer  |
MIC_bit==1 &             | |   | |   +---+Attemp++           |
Recv_window==window &    | |   | +---------------------------+   
Lcl_Bitmap==recv_Bitmap &| |   |   all missing frag sent
             no more frag| |   |   ~~~~~~~~~~~~~~~~~~~~~~ 
 ~~~~~~~~~~~~~~~~~~~~~~~~| |   |   Set Retrans_Timer                 
       Stop Retrans_Timer| |   |    
 +=============+         | |   |
 |     END     +<--------+ |   | Attemp > MAX_ACK_REQUESTS
 +=============+           |   | ~~~~~~~~~~~~~~~~~~
              All-1 Window |   v Send Abort
              ~~~~~~~~~~~~ | +=+===========+
             MIC_bit ==0 & +>|    ERROR    |
    Lcl_Bitmap==recv_Bitmap  +=============+                                        
~~~~
{: #Fig-ACKAlwaysSnd title='Sender State Machine for the ACK-Always Mode'}



~~~~
 Not All- & w=expected +---+   +---+w = Not expected
 ~~~~~~~~~~~~~~~~~~~~~ |   |   |   |~~~~~~~~~~~~~~~~
 Set local_Bitmap(FCN) |   v   v   |discard
                      ++===+===+===+=+      
+---------------------+     Rcv      +--->* ABORT 
|  +------------------+   Window     |
|  |                  +=====+==+=====+  
|  |       All-0 & w=expect |  | w =next & not-All
|  |     ~~~~~~~~~~~~~~~~~~ |  |~~~~~~~~~~~~~~~~~~~~~
|  |     set lcl_Bitmap(FCN)|  |expected = next window
|  |      send local_Bitmap |  |Clear local_Bitmap
|  |                        |  |    
|  | w=expct & not-All      |  |    
|  | ~~~~~~~~~~~~~~~~~~     |  | 
|  | set lcl_Bitmap(FCN)+-+ |  | +--+ w=next & All-0
|  | if lcl_Bitmap full | | |  | |  | ~~~~~~~~~~~~~~~
|  | send lcl_Bitmap    | | |  | |  | expct = nxt wnd
|  |                    v | v  v v  |
|  |  w=expct & All-1 +=+=+=+==+=++ | Clear lcl_Bitmap    
|  |  ~~~~~~~~~~~  +->+    Wait   +<+ set lcl_Bitmap(FCN)         
|  |    discard    +--|    Next   |   send lcl_Bitmap
|  | All-0  +---------+  Window   +--->* ABORT  
|  | ~~~~~  +-------->+========+=++        
|  | snd lcl_bm  All-1 & w=next| |  All-1 & w=nxt
|  |                & MIC wrong| |  & MIC right      
|  |          ~~~~~~~~~~~~~~~~~| | ~~~~~~~~~~~~~~~~~~ 
|  |      set local_Bitmap(FCN)| |set lcl_Bitmap(FCN)       
|  |          send local_Bitmap| |send local_Bitmap 
|  |                           | +----------------------+
|  |All-1 & w=expct            |                        |
|  |& MIC wrong                v   +---+ w=expctd &     |
|  |~~~~~~~~~~~~~~~~~~~~  +====+=====+ | MIC wrong      |
|  |set local_Bitmap(FCN) |          +<+ ~~~~~~~~~~~~~~ |
|  |send local_Bitmap     | Wait End | set lcl_btmp(FCN)|
|  +--------------------->+          +--->* ABORT       |
|                         +===+====+=+-+ All-1&MIC wrong|
|                             |    ^   | ~~~~~~~~~~~~~~~|
|                             |    +---+ send lcl_btmp  |                
|       w=expected & MIC right|         |               |
|       ~~~~~~~~~~~~~~~~~~~~~~| +-+ Not All-1           |
|        set local_Bitmap(FCN)| | | ~~~~~~~~~           |
|            send local_Bitmap| | |  discard            |
|                             | | |                     | 
|All-1 & w=expctd & MIC right | | |   +-+ All-1         |
|~~~~~~~~~~~~~~~~~~~~~~~~~~~~ v | v | v ~~~~~~~~~       |
|set local_Bitmap(FCN)      +=+=+=+=+=++Send lcl_btmp   |
|send local_Bitmap          |          |                |
+-------------------------->+    END   +<---------------+
                            ++==+======+  
       --->* ABORT
            ~~~~~~~
            Inactivity_Timer = expires
        When DWN_Link
          IF Inactivity_Timer expires
             Send DWL Request
             Attemp++
                            
~~~~
{: #Fig-ACKAlwaysRcv title='Receiver State Machine for the ACK-Always Mode'}



~~~~
                   +=======+  
                   |       |  
                   | INIT  |  
                   |       |        FCN!=0 & more frags
                   +======++  +--+  ~~~~~~~~~~~~~~~~~~~~~~
                      W=0 |   |  |  send Window + frag(FCN)
       ~~~~~~~~~~~~~~~~~~ |   |  |  FCN-
       Clear local Bitmap |   |  v  set local Bitmap
            FCN=max value |  ++=============+
                          +> |              |
                             |     SEND     |
 +-------------------------> |              |
 |                           ++=====+=======+
 |         FCN==0 & more frags|     |last frag
 |     ~~~~~~~~~~~~~~~~~~~~~~~|     |~~~~~~~~~~~~~~~~~~~~~~~~
 |            set local-Bitmap|     |set local-Bitmap
 |      send wnd + frag(all-0)|     |send wnd+frag(all-1)+MIC
 |           set Retrans_Timer|     |set Retrans_Timer
 |                            |     |
 |Retrans_Timer expires &     |     | local-Bitmap!=rcv-Bitmap 
 |more fragments              |     |  +-----------------+
 |~~~~~~~~~~~~~~~~~~~~        |     |  | ~~~~~~~~~~~~~   |
 |stop Retrans_Timer          |     |  | Attemp++        |
 |clear local-Bitmap          v     v  |                 v
 |window = next window  +=====+=====+==+==+         +====+====+
 +----------------------+                 +         | Resend  |
 +--------------------->+    Wait Bitmap  |         | Missing |
 |                  +-- +                 |         | Frag    |
 | not expected wnd |   ++=+===+===+===+==+         +======+==+
 | ~~~~~~~~~~~~~~~~ |    ^ |   |   |   ^                   |
 |    discard frag  +----+ |   |   |   +-------------------+
 |                         |   |   |     all missing frag sent
 |Retrans_Timer expires &  |   |   |     ~~~~~~~~~~~~~~~~~~~~~ 
 |       No more Frag      |   |   |     Set Retrans_Timer
 | ~~~~~~~~~~~~~~~~~~~~~~~ |   |   |   
 |  Stop Retrans_Timer     |   |   |  
 |  Send ALL-1-empty       |   |   | 
 +-------------------------+   |   | 
                               |   |
      Local_Bitmap==Recv_Bitmap|   |
      ~~~~~~~~~~~~~~~~~~~~~~~~~|   |Attemp > MAX_ACK_REQUESTS
 +=========+Stop Retrans_Timer |   |~~~~~~~~~~~~~~~~~~~~~~~
 |   END   +<------------------+   v  Send Abort
 +=========+                     +=+=========+
                                 |   ERROR   |
                                 +===========+                                      
~~~~
{: #Fig-ACKonerrorSnd title='Sender State Machine for the ACK-on-Error Mode'}



~~~~
   Not All- & w=expected +---+   +---+w = Not expected
   ~~~~~~~~~~~~~~~~~~~~~ |   |   |   |~~~~~~~~~~~~~~~~ 
   Set local_Bitmap(FCN) |   v   v   |discard
                        ++===+===+===+=+
+-----------------------+              +--+ All-0 & full 
|            ABORT *<---+  Rcv Window  |  | ~~~~~~~~~~~~ 
|  +--------------------+              +<-+ w =next
|  |                    +===+===+======+ clear lcl_Bitmap
|  |                        |   ^
|  |        All-0 & w=expect|   |w=expct & not-All & full
|  |        & no_full Bitmap|   |~~~~~~~~~~~~~~~~~~~~~~~~
|  |       ~~~~~~~~~~~~~~~~~|   |clear lcl_Bitmap; w =nxt
|  |       send local_Bitmap|   |
|  |                        |   |              +========+
|  |                        |   |  +---------->+        | 
|  |                        |   |  |w=next     | Error/ |
|  |                        |   |  |~~~~~~~~   | Abort  |
|  |                        |   |  |Send abort ++=======+
|  |                        v   |  |             ^ w=expct
|  |            All-0     +=+===+==+======+      | & all-1  
|  |     ~~~~~~~~~~~~~<---+    Wait       +------+ ~~~~~~~
|  |     send lcl_btmp    | Next Window   |     Send abort
|  |                      +=======+===+==++    
|  |  All-1 & w=next & MIC wrong  |   |  +---->* ABORT  
|  |  ~~~~~~~~~~~~~~~~~~~~~~~~~~  |   +----------------+
|  |       set local_Bitmap(FCN)  |      All-1 & w=next| 
|  |       send local_Bitmap      |         & MIC right|
|  |                              |  ~~~~~~~~~~~~~~~~~~|
|  |                              | set lcl_Bitmap(FCN)|                                    
|  |All-1 & w=expect & MIC wrong  |                    |
|  |~~~~~~~~~~~~~~~~~~~~~~~~~~~~  |   +-+  All-1            |
|  |set local_Bitmap(FCN)         v   | v  ~~~~~~~~~~  |
|  |send local_Bitmap     +=======+==+===+ snd lcl_btmp|
|  +--------------------->+   Wait End   +-+           |
|                         +=====+=+===+=+ | w=expct &  |
|       w=expected & MIC right  | |    ^   | MIC wrong |
|       ~~~~~~~~~~~~~~~~~~~~~~  | |    +---+ ~~~~~~~~~ |
|        set local_Bitmap(FCN)  | | set lcl_Bitmap(FCN)|
|                               | |                    |
|All-1 & w=expected & MIC right | +-->* ABORT          |
|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ v                      |
|set local_Bitmap(FCN)        +=+==========+           |
+---------------------------->+     END    +<----------+
                              +============+   
            --->* Only Uplink
                 ABORT
                 ~~~~~~~~
                 Inactivity_Timer = expires                                                      
~~~~
{: #Fig-ACKonerrorRcv title='Receiver State Machine for the ACK-on-Error Mode'}




# Allocation of Rule IDs for fragmentation

A set of Rule IDs has to be allocated to support different aspects of fragmentation functionality as per this document. The actual allocation of IDs is to be defined in other documents. The set MAY include:

   *  one ID or several IDs to identify a fragment as well as its reliability mode and its window size, if multiple of these are supported.
   
   *  one ID to identify the ACK message.
   
   *  one ID to identify the Abort message as per Section 9.8.


# Note
Carles Gomez has been funded in part by the Spanish Government
(Ministerio de Educacion, Cultura y Deporte) through the Jose
Castillejo grant CAS15/00336, and by the ERDF and the Spanish Government 
through project TEC2016-79988-P.  Part of his contribution to this work
has been carried out during his stay as a visiting scholar at the
Computer Laboratory of the University of Cambridge.
