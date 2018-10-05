---
stand_alone: true
ipr: trust200902
docname: draft-ietf-lpwan-ipv6-static-context-hc-16
cat: std
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
  street: 1137A avenue des Champs Blancs
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
- ins: D. Barthel
  name: Dominique Barthel
  org: Orange Labs
  street:
  - 28 chemin du Vieux Chêne
  - 38243 Meylan
  country: France
  email: dominique.barthel@orange.com  
normative:
  RFC2119:
  RFC7217:
  RFC8174:
informative:  
  RFC3385:
  RFC4944:
  RFC5795:
  RFC6282:
  RFC6936:
  RFC7136:
  RFC8200:
  RFC8376:

--- abstract

This document defines the Static Context Header Compression (SCHC) framework, which provides both header compression and fragmentation functionalities. SCHC has been designed for Low Power Wide Area Networks (LPWAN).

SCHC compression is based on a common static context stored in both the LPWAN device and the network side. This document defines a header compression mechanism and its application to compress IPv6/UDP headers.

This document also specifies a fragmentation and reassembly mechanism that is used to support the IPv6 MTU requirement over the LPWAN technologies. Fragmentation is needed for IPv6 datagrams that, after SCHC compression or when such compression was not possible, still exceed the layer-2 maximum payload size.

The SCHC header compression and fragmentation mechanisms are independent of the specific LPWAN technology over which they are used. This document defines generic functionalities and offers flexibility with regard to parameter settings and mechanism choices. Technology-specific settings and choices are expected to be made in other documents.

--- middle

# Introduction {#Introduction}

This document defines the Static Context Header Compression (SCHC) framework, which provides both header compression and fragmentation functionalities. SCHC has been designed for Low Power Wide Area Networks (LPWAN).

Header compression is needed for efficient Internet connectivity to the node within an LPWAN network. Some LPWAN networks properties can be exploited to get an efficient header compression:

* The network topology is star-oriented, which means that all packets between the same source-destination pair follow the same path. For the needs of this document, the architecture can simply be described as Devices (Dev) exchanging information with LPWAN Application Servers (App) through a Network Gateway (NGW).

* Because devices embed built-in applications, the traffic flows to be compressed are known in advance. Indeed, new applications cannot be easily installed in LPWAN devices, as they would in computers or smartphones.

SCHC compression uses a context in which information about header fieds is stored. This context is static: the values of the header fields do not change over time. This avoids complex resynchronization mechanisms, that would be incompatible with LPWAN characteristics. In most cases, a small context identifier is enough to represent the full IPv6/UDP headers. The SCHC header compression mechanism is independent of the specific LPWAN technology over which it is used.

LPWAN technologies impose some strict limitations on traffic. For instance, devices are sleeping most of the time and may receive data during short periods of time after transmission to preserve battery. LPWAN technologies are also characterized
by a greatly reduced data unit and/or payload size (see {{RFC8376}}).  However, some LPWAN technologies do not provide fragmentation functionality; to support the IPv6 MTU requirement of 1280 bytes {{RFC8200}}, they require a fragmentation protocol at the adaptation layer below IPv6.
Accordingly, this document defines an fragmentation/reassembly mechanism for LPWAN technologies to supports the IPv6 MTU. Its implementation is optional. If not interested, the reader can safely skip its description.

This document defines generic functionality and offers flexibility with regard to parameter settings
and mechanism choices. Technology-specific settings and choices are expected to be made in other documents.

# Requirements Notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP 14
{{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, 
as shown here.

# LPWAN Architecture {#LPWAN-Archi}

LPWAN technologies have similar network architectures but different terminologies.
Using the terminology defined in {{RFC8376}},
we can identify different types of entities in a typical LPWAN network, see {{Fig-LPWANarchi}}:

   o  Devices (Dev) are the end-devices or hosts (e.g. sensors, actuators, etc.). There can be a very high density of devices per radio gateway.

   o  The Radio Gateway (RGW), which is the end point of the constrained link.

   o  The Network Gateway (NGW) is the interconnection node between the Radio Gateway and the Internet.  

   o  LPWAN-AAA Server, which controls the user authentication and the applications.

   o  Application Server (App)

~~~~
                                           +------+
 ()   ()   ()       |                      |LPWAN-|
  ()  () () ()     / \       +---------+   | AAA  |
() () () () () () /   \======|    ^    |===|Server|  +-----------+
 ()  ()   ()     |           | <--|--> |   +------+  |APPLICATION|
()  ()  ()  ()  / \==========|    v    |=============|   (App)   |
  ()  ()  ()   /   \         +---------+             +-----------+
 Dev        Radio Gateways         NGW

~~~~
{: #Fig-LPWANarchi title='LPWAN Architecture'}                      


# Terminology {#Term}
This section defines the terminology and acronyms used in this document.

Note that the SCHC acronym is pronounced like "sheek" in English (or "chic" in French). Therefore, this document writes "a SCHC Packet" instead of "an SCHC Packet".

* App: LPWAN Application. An application sending/receiving IPv6 packets to/from the Device.

* AppIID: Application Interface Identifier. The IID that identifies the application server interface.

* Bi: Bidirectional. Characterises a Rule Entry that applies to headers of packets travelling in either direction (Up and Dw, see this glossary).

* CDA: Compression/Decompression Action. Describes the reciprocal pair of actions that are performed at the compressor to compress a header field and at the decompressor to recover the original header field value.

* Compression Residue. The bits that need to be sent (beyond the Rule ID itself) after applying the SCHC compression over each header field.

* Context: A set of Rules used to compress/decompress headers.

* Dev: Device. A node connected to an LPWAN. A Dev SHOULD implement SCHC.

* DevIID: Device Interface Identifier. The IID that identifies the Dev interface.

* DI: Direction Indicator. This field tells which direction of packet travel (Up, Dw or Bi) a Rule applies to. This allows for assymmetric processing.

* Dw: Downlink direction for compression/decompression in both sides, from SCHC C/D in the network to SCHC C/D in the Dev.

* Field Description. A line in the Rule table.

* FID: Field Identifier. This is an index to describe the header fields in a Rule.

* FL: Field Length is the length of the packet header field. It is expressed in bits for header fields of fixed lengths or as a type (e.g. variable, token length, ...) for field lengths that are unknown at the time of Rule creation. The length of a header field is defined in the corresponding protocol specification.

* FP: Field Position is a value that is used to identify the position where each instance of a field appears in the header.  

* IID: Interface Identifier. See the IPv6 addressing architecture {{RFC7136}}

* L2: Layer two. The immediate lower layer SCHC interfaces with. It is provided by an underlying LPWAN technology. It does not necessarily correspond to the OSI model definition of Layer 2.

* L2 Word: this is the minimum subdivision of payload data that the L2 will carry. In most L2 technologies, the L2 Word is an octet.
  In bit-oriented radio technologies, the L2 Word might be a single bit.
  The L2 Word size is assumed to be constant over time for each device.

* MO: Matching Operator. An operator used to match a value contained in a header field with a value contained in a Rule.

* Padding (P). Extra bits that may be appended by SCHC to a data unit that it passes to the underlying Layer 2 for transmission.
  SCHC itself operates on bits, not bytes, and does not have any alignment prerequisite. See {{Padding}}.

* Rule: A set of header field values.

* Rule entry: A column in a Rule that describes a parameter of the header field.

* Rule ID: An identifier for a Rule. SCHC C/D on both sides share the same Rule ID for a given packet. A set of Rule IDs are used to support SCHC F/R functionality.
  
* SCHC C/D: Static Context Header Compression Compressor/Decompressor. A mechanism used on both sides, at the Dev and at the network, to achieve Compression/Decompression of headers. SCHC C/D uses Rules to perform compression and decompression.
  
* SCHC Packet: A packet (e.g. an IPv6 packet) whose header has been compressed as per the header compression mechanism defined in this document. If the header compression process is unable to actually compress the packet header, the packet with the uncompressed header is still called a SCHC Packet (in this case, a Rule ID is used to indicate that the packet header has not been compressed). See {{SCHComp}} for more details.

* TV: Target value. A value contained in a Rule that will be matched with the value of a header field.

* Up: Uplink direction for compression/decompression in both sides, from the Dev SCHC C/D to the network SCHC C/D.

Additional terminology for the optional SCHC Fragmentation / Reassembly mechanism (SCHC F/R) is found in {{FragTools}}.

# SCHC overview

SCHC can be characterized as an adaptation layer between IPv6 and the underlying LPWAN technology. SCHC comprises two sublayers (i.e. the Compression sublayer and the Fragmentation sublayer), as shown in {{Fig-IntroLayers}}.

~~~~

             +----------------+
             |      IPv6      |
          +- +----------------+            
          |  |   Compression  |  
    SCHC <   +----------------+   
          |  |  Fragmentation |
          +- +----------------+         
             |LPWAN technology|
             +----------------+

~~~~
{: #Fig-IntroLayers title='Protocol stack comprising IPv6, SCHC and an LPWAN technology'}

As per this document, when a packet (e.g. an IPv6 packet) needs to be transmitted, header compression is first applied to the packet. The resulting packet after header compression (whose header may or may not actually be smaller than that of the original packet) is called a SCHC Packet.
If the SCHC Packet neds to be fragmented by the optional SCHC Fragmentation, fragmentation is then applied to the SCHC Packet. The SCHC Packet or the SCHC Fragments are then transmitted over the LPWAN. The reciprocal operations take place at the receiver. This process is illustrated in {{Fig-Operations}}.

~~~~
A packet (e.g. an IPv6 packet) 
         |                                           ^
         v                                           |
+------------------+                      +--------------------+
| SCHC Compression |                      | SCHC Decompression |
+------------------+                      +--------------------+
         |                                           ^
         |   If no fragmentation (*)                 |
         +-------------- SCHC Packet  -------------->|
         |                                           |
         v                                           |
+--------------------+                       +-----------------+
| SCHC Fragmentation |                       | SCHC Reassembly |
+--------------------+                       +-----------------+
      |     ^                                     |     ^
      |     |                                     |     |
      |     +-------------- SCHC ACK -------------+     |
      |                                                 |
      +-------------- SCHC Fragments -------------------+

        SENDER                                    RECEIVER


*: the decision to use Fragmentation or not is left to each LPWAN 
   technology over which SCHC is applied. See LPWAN 
   technology-specific documents.

~~~~
{: #Fig-Operations title='SCHC operations at the SENDER and the RECEIVER'}

## SCHC Packet format

The SCHC Packet is composed of the Compressed Header followed by the payload from the original packet (see {{Fig-SCHCpckt}}).
The Compressed Header itself is composed of a Rule ID and a Compression Residue. The Compression Residue may be absent, see {{SCHComp}}. Both the Rule ID and the Compression Residue potentially have a variable size, and generally are not a mutiple of bytes in size.
  
~~~~

|  Rule ID +  Compression Residue |
+---------------------------------+--------------------+ 
|      Compressed Header          |      Payload       |
+---------------------------------+--------------------+

~~~~ 
{: #Fig-SCHCpckt title='SCHC Packet'}

## SCHC Fragmentation message formats

The SCHC Fragment has the
following format:

~~~~

| Rule ID + DTAG + W + FCN [+ MIC ] |   Partial  SCHC Packet  |
+-----------------------------------+-------------------------+ 
|        Fragment Header            |   Fragment  Payload     |
+-----------------------------------+-------------------------+

~~~~ 
{: #Fig-SCHCfragment title='SCHC Fragment'}

The Fragment Header size is variable and depends on the Fragmentation parameters.
The Fragment payload contains a part of the SCHC Packet Compressed Header, a part of the SCHC Packet Payload or both.
Its size depends on the L2 MTU (see {{Frag}}).


The SCHC ACK
has the following format:

~~~~

|Rule ID + DTag + W|             
+------------------+-------- ... ---------+
|    ACK Header    |    encoded Bitmap    |
+------------------+-------- ... ---------+

~~~~ 
{: #Fig-SCHCack title='SCHC ACK'}

The SCHC ACK Header and the encoded Bitmap both have variable size.

SCHC Fragmentation messages are further described in {{Frag}}.

## Functional mapping

{{Fig-archi}} below maps the functional elements of {{Fig-Operations}} onto the LPWAN architecture elements of {{Fig-LPWANarchi}}.


~~~~
     Dev                                                 App
+----------------+                                  +--------------+
| APP1 APP2 APP3 |                                  |APP1 APP2 APP3|
|                |                                  |              |
|       UDP      |                                  |     UDP      |
|      IPv6      |                                  |    IPv6      |   
|                |                                  |              |  
|SCHC C/D and F/R|                                  |              |
+--------+-------+                                  +-------+------+
         |   +--+     +----+     +-----------+              .
         +~~ |RG| === |NGW | === |   SCHC    |... Internet ..
             +--+     +----+     |F/R and C/D|
                                 +-----------+
~~~~
{: #Fig-archi title='Architecture'}


SCHC C/D and SCHC F/R are located on both sides of the LPWAN transmission, i.e. on the Dev side and on the Network side.

The operation in the Uplink direction is as follows. The Device application uses IPv6 or IPv6/UDP protocols. Before sending the packets, the Dev compresses their headers using SCHC C/D and,
if the SCHC Packet resulting from the compression needs to be fragmented by SCHC, SCHC F/R is performed (see {{Frag}}).
The resulting SCHC Fragments are sent to an LPWAN Radio Gateway (RG) which forwards them to a Network Gateway (NGW).
The NGW sends the data to a SCHC F/R for re-assembly (if needed) and then to the SCHC C/D for decompression.
After decompression, the packet can be sent over the Internet
to one or several LPWAN Application Servers (App).

The SCHC F/R and C/D on the Network side can be located in the NGW, or somewhere else as long as a tunnel is established between them and the NGW. Note that, for some LPWAN technologies, it MAY be suitable to locate the SCHC F/R
functionality nearer the NGW, in order to better deal with time constraints of such technologies.

The SCHC C/D and F/R on both sides MUST share the same set of Rules.

The SCHC C/D and F/R process is symmetrical, therefore the description of the Downlink direction is symmetrical to the one above.


# Rule ID

Rule IDs are identifiers used to select the correct context either for Compression/Decompression or
for Fragmentation/Reassembly.

The size of the Rule IDs is not specified in this document, as it is implementation-specific and can vary according to the LPWAN technology and the number of Rules, among others.

The Rule IDs are used:

* In the SCHC C/D context, to identify the Rule (i.e., the set of Field Descriptions) that is used to compress a packet header.

* At least one Rule ID MAY be allocated to tagging packets for which SCHC compression was not possible (no matching Rule was found).

* In SCHC F/R, to identify the specific modes and settings of SCHC Fragments being transmitted, and to identify the SCHC ACKs, including their modes and settings. Note that when F/R is used for both communication directions, at least two Rule ID values are therefore needed for F/R.


# Compression/Decompression {#SCHComp}

Compression with SCHC
is based on using context, i.e. a set of Rules to compress or decompress headers. SCHC avoids context synchronization, which consumes considerable bandwidth in other header compression mechanisms such as RoHC {{RFC5795}}. Since the content of packets is highly predictable in LPWAN networks, static contexts MAY be stored beforehand to omit transmitting some information over the air. The contexts MUST be stored at both ends, and they can be learned by a provisioning protocol or by out of band means, or they can be pre-provisioned. The way the contexts are provisioned is out of the scope of this document.

## SCHC C/D Rules

The main idea of the SCHC compression scheme is to transmit the Rule ID to the other end instead of sending known field values. This Rule ID identifies a Rule that provides the closest match to the original packet values. Hence, when a value is known by both ends, it is only necessary to send the corresponding Rule ID over the LPWAN network.
The manner by which Rules are generated is out of the scope of this document. The Rules MAY be changed at run-time but the mechanism is out of scope of this document.

The context contains a list of Rules (see {{Fig-ctxt}}). Each Rule itself contains a list of Field Descriptions composed of a Field Identifier (FID), a Field Length (FL), a Field Position (FP), a Direction Indicator (DI), a Target Value (TV), a Matching Operator (MO) and a Compression/Decompression Action (CDA).


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
{: #Fig-ctxt title='A Compression/Decompression Context'}


A Rule does not describe how to parse a packet header to find each field. This MUST be known from the compressor/decompressor. Rules only describe the compression/decompression behavior for each header field. In a Rule, the Field Descriptions are listed in the order in which the fields appear in the packet header.

A Rule also describes what Compression Residue is sent. The Compression Residue is assembled by concatenating the residues for each field, in the order the Field Descriptions appear in the Rule.

The Context describes the header fields and its values with the following entries:

* Field ID (FID) is a unique value to define the header field.

* Field Length (FL) represents the length of the field. It can be either a fixed value (in bits) if the length is known when the Rule is created or a type if the length is variable. The length of a header field is defined in the corresponding protocol specification. The type defines the process to compute the length, its unit (bits, bytes,...) and the value to be sent before the Compression Residue.

* Field Position (FP): most often, a field only occurs once in a packet header. Some fields may occur multiple times in a header. FP indicates which occurrence  this Field Description applies to. The default value is 1 (first occurence).

* A Direction Indicator (DI) indicates the packet direction(s) this Field Description applies to. Three values are possible:

  * UPLINK (Up): this Field Description is only applicable to packets sent by the Dev to the App,

  * DOWNLINK (Dw): this Field Description is only applicable to packets sent from the App to the Dev,

  * BIDIRECTIONAL (Bi): this Field Description is applicable to packets travelling both Up and Dw.

* Target Value (TV) is the value used to match against the packet header field. The Target Value can be of any type (integer, strings, etc.). It can be a single value or a more complex structure (array, list, etc.), such as a JSON or a CBOR structure.

* Matching Operator (MO) is the operator used to match the Field Value and the Target Value. The Matching Operator may require some parameters. MO is only used during the compression phase. The set of MOs defined in this document can be found in {{chap-MO}}.

* Compression Decompression Action (CDA) describes the compression and decompression processes to be performed after the MO is applied. Some CDAs MAY require parameter values for their operation. CDAs are used in both the compression and the decompression functions. The set of CDAs defined in this document can be found in {{chap-CDA}}.

## Rule ID for SCHC C/D {#IDComp}

Rule IDs are sent by the compression function in one side and are received for the decompression function in the other side.
In SCHC C/D, the Rule IDs are specific to a Dev. Hence, multiple Dev instances MAY use the same Rule ID to define different
header compression contexts. To identify the correct Rule ID, the SCHC C/D needs to associate the Rule ID with the Dev identifier to find the appropriate Rule to be applied.


## Packet processing

The compression/decompression process follows several steps:

* Compression Rule selection: The goal is to identify which Rule(s) will be used to compress the packet's headers. When performing decompression, on the network side the SCHC C/D needs to find the correct Rule based on the L2 address; in this way, it can use the DevIID and the Rule ID. On the Dev side, only the Rule ID is needed to identify the correct Rule since the Dev typically only holds Rules that apply to itself. The Rule will be selected by matching the Fields Descriptions to the packet header as described below. When the selection of a Rule is done, this Rule is used to compress the header. The detailed steps for compression Rule selection are the following:

  * The first step is to choose the Field Descriptions by their direction, using the Direction Indicator (DI). A Field Description that does not correspond to the appropriate DI will be ignored. If all the fields of the packet do not have a Field Description with the correct DI, the Rule is discarded and SCHC C/D proceeds to consider the next Rule.

  * When the DI has matched, then the next step is to identify the fields according to Field Position (FP). If FP does not correspond, the Rule is not used and the SCHC C/D proceeds to consider the next Rule.

  * Once the DI and the FP correspond to the header information, each packet field's value is then compared to the corresponding Target Value (TV) stored in the Rule for that specific field using the matching operator (MO).

    If all the fields in the packet's header satisfy all the matching operators (MO) of a Rule (i.e. all MO results are True), the fields of the header are then compressed according to the Compression/Decompression Actions (CDAs) and a compressed header (with possibly a Compression Residue) SHOULD be obtained. Otherwise, the next Rule is tested.

  * If no eligible Rule is found, then the header MUST be sent without compression (but may require the use of the SCHC F/R process).

* Sending: If an eligible Rule is found, the Rule ID is sent to the other end followed by the Compression Residue (which could be empty) and directly followed by the payload. The Compression Residue is the concatenation of the Compression Residues for each field according to the CDAs for that Rule. The way the Rule ID is sent depends on the specific underlying LPWAN technology. For example, it can be either included in an L2 header or sent in the first byte of the L2 payload. (see {{Fig-FormatPckt}}). This process will be specified in the LPWAN technology-specific document and is out of the scope of the present document. On LPWAN technologies that are byte-oriented, the compressed header concatenated with the original packet payload is padded to a multiple of 8 bits, if needed. See {{Padding}} for details.

* Decompression: When doing decompression, on the network side the SCHC C/D needs to find the correct Rule based on the L2 address and in this way, it can use the DevIID and the Rule ID. On the Dev side, only the Rule ID is needed to identify the correct Rule since the Dev only holds Rules that apply to itself.

  The receiver identifies the sender through its device-id or source identifier (e.g. MAC address, if it exists) and selects the Rule using the Rule ID. This Rule describes the compressed header format and associates the values to the header fields.  The receiver applies the CDA action to reconstruct the original header fields. The CDA application order can be different from the order given by the Rule. For instance, Compute-\* SHOULD be applied at the end, after all the other CDAs.

~~~~

+--- ... --+------- ... -------+------------------+
|  Rule ID |Compression Residue|  packet payload  |
+--- ... --+------- ... -------+------------------+

|----- compressed header ------|

~~~~
{: #Fig-FormatPckt title='SCHC C/D Packet Format'}


## Matching operators {#chap-MO}

Matching Operators (MOs) are functions used by both SCHC C/D endpoints involved in the header compression/decompression. They are not typed and can be applied to integer, string or any other data type. The result of the operation can either be True or False. MOs are defined as follows:

* equal: The match result is True if the field value in the packet matches the TV.

* ignore: No check is done between the field value in the packet and the TV in the Rule. The result of the matching is always true.

* MSB(x): A match is obtained if the most significant x bits of the packet header field value are equal to the TV in the Rule. The x parameter of the MSB MO indicates how many bits are involved in the comparison. If the FL is described as variable, the length must be a multiple of the unit. For example, x must be multiple of 8 if the unit of the variable length is in bytes.

* match-mapping: With match-mapping, the Target Value is a list of values. Each value of the list is identified by a short ID (or index). Compression is achieved by sending the index instead of the original header field value. This operator matches if the header field value is equal to one of the values in the target list.


## Compression Decompression Actions (CDA) {#chap-CDA}

The Compression Decompression Action (CDA) describes the actions taken during the compression of headers fields, and inversely, the action taken by the decompressor to restore the original value.

~~~~
/--------------------+-------------+----------------------------\
|  Action            | Compression | Decompression              |
|                    |             |                            |
+--------------------+-------------+----------------------------+
|not-sent            |elided       |use value stored in context |
|value-sent          |send         |build from received value   |
|mapping-sent        |send index   |value from index on a table |
|LSB                 |send LSB     |TV, received value          |
|compute-length      |elided       |compute length              |
|compute-checksum    |elided       |compute UDP checksum        |
|DevIID              |elided       |build IID from L2 Dev addr  |
|AppIID              |elided       |build IID from L2 App addr  |
\--------------------+-------------+----------------------------/

~~~~
{: #Fig-function title='Compression and Decompression Actions'}

{{Fig-function}} summarizes the basic actions that can be used to compress and decompress a field. The first column shows the action's name. The second and third columns show the reciprocal compression/decompression behavior for each action.

Compression is done in the order that the Field Descriptions appear in a Rule. The result of each Compression/Decompression Action is appended to the accumulated Compression Residue in that same order. The receiver knows the size of each compressed field, which can be given by the Rule or MAY be sent with the compressed header.

### processing variable-length fields {#var-length-field}

If the field is identified in the Field Description as being of variable size, then the size of the Compression Residue value (using the unit defined in the FL) MUST first be sent as follows:

* If the size is between 0 and 14, it is sent as a 4-bits unsigned integer.

* For values between 15 and 254, 0b1111 is transmitted and then the size is sent as an 8 bits unsigned integer.

* For larger values of the size, 0xffff is transmitted and then the next two bytes contain the size value as a 16 bits unsigned integer.

If a field is not present in the packet but exists in the Rule and its FL is specified as being variable, size 0 MUST be sent to denote its absence.

### not-sent CDA

The not-sent action is generally used when the field value is specified in a Rule and therefore known by both the Compressor and the Decompressor. This action SHOULD be used with the "equal" MO. If MO is "ignore", there is a risk to have a decompressed field value different from the original field that was compressed.

The compressor does not send any Compression Residue for a field on which not-sent compression is applied.

The decompressor restores the field value with the Target Value stored in the matched Rule identified by the received Rule ID.

### value-sent CDA

The value-sent action is generally used when the field value is not known by both the Compressor and the Decompressor. The value is sent as a residue in the compressed message header. Both Compressor and Decompressor MUST know the size of the field, either implicitly (the size is known by both sides) or by explicitly indicating the length in the Compression Residue, as defined in {{var-length-field}}. This action is generally used with the "ignore" MO.

### mapping-sent CDA

The mapping-sent action is used to send an index (the index into the Target Value list of values) instead of the original value. This action is used together with the "match-mapping" MO.

On the compressor side, the match-mapping Matching Operator searches the TV for a match with the header field value and the mapping-sent CDA appends the corresponding index to the Compression Residue to be sent.
On the decompressor side, the CDA uses the received index to restore the field value by looking up the list in the TV.

The number of bits sent is the minimal size for coding all the possible indices.

### LSB CDA

The LSB action is used together with the "MSB(x)" MO to avoid sending the most significant part of the packet field if that part is already known by the receiving end.
The number of bits sent is the original header field length minus the length specified in the MSB(x) MO.

The compressor sends the Least Significant Bits (e.g. LSB of the length field). The decompressor concatenates the x most significant bits of Target Value and the received residue.

If this action needs to be done on a variable length field, the size of the Compression Residue in bytes MUST be sent as described in {{var-length-field}}.


### DevIID, AppIID CDA

These actions are used to process respectively the Dev and the App Interface Identifiers (DevIID and AppIID) of the IPv6 addresses. AppIID CDA is less common since most current LPWAN technologies frames contain a single L2 address, which is the Dev's address.

The IID value MAY be computed from the Device ID present in the L2 header, or from some other stable identifier. The computation is specific to each LPWAN technology and MAY depend on the Device ID size.

In the downlink direction (Dw), at the compressor, the DevIID CDA may be used to generate the L2 addresses on the LPWAN, based on the packet's Destination Address.

### Compute-\*

Some fields may be elided during compression and reconstructed during decompression. This is the case for length and checksum, so:

* compute-length: computes the length assigned to this field. This CDA MAY be used to compute IPv6 length or UDP length.

* compute-checksum: computes a checksum from the information already received by the SCHC C/D. This field MAY be used to compute UDP checksum.


# Fragmentation/Reassembly {#Frag}

## Overview

In LPWAN technologies, the L2 MTU typically ranges from tens to hundreds of bytes.
Some of these technologies do not have an internal fragmentation/reassembly mechanism.

The SCHC Fragmentation/Reassembly (SCHC F/R) functionality is offered as an option for such LPWAN technologies to cope with the IPv6 MTU requirement of 1280 bytes {{RFC8200}}.
It is optional to implement. If it is not needed, its description can be safely ignored.

It has been designed under the following assumptions

* Data unit out-of-sequence delivery does not occur between the entity performing fragmentation and the entity performing reassembly

* The L2 MTU value does not change while a fragmented SCHC Packet is being transmitted.

These assumptions allow reducing the complexity and overhead of the SCHC F/R mechanism.

This specification includes several SCHC F/R modes, which allow for a range of reliability options such as optional SCHC Fragment retransmission.
More modes may be defined in the future.
The same SCHC F/R mode MUST be used for all SCHC Fragments of a fragmented SCHC Packet.
This document does not make any decision with regard to which mode(s) will be used over a specific LPWAN technology. This will be defined in other technology-specific documents.

SCHC F/R uses the knowledge of the L2 Word size (see {{Term}}) to encode some messages. Therefore, SCHC MUST know the L2 Word size.
SCHC F/R usually generates SCHC Fragments and SCHC ACKs that are multiples of L2 Words.
The padding overhead is kept to the absolute minimum (see {{Padding}}).

## SCHC F/R Tools {#FragTools}

This subsection describes the different tools that are used to enable the SCHC F/R functionality defined in this document.
These tools include the SCHC F/R messages, windows, timers and header fields.

The tools are described here in a generic manner. Their application to each SCHC F/R mode is found in {{FragModes}}.

### Messages

The messages that can be used by SCHC F/R are the following

* SCHC Fragment: A data unit that carries a subset of a SCHC Packet from the sender to the receiver.

* SCHC ACK: An acknowledgement for fragmentation. This message is used by the receiver to report to the sender on the success of reception of a set of SCHC Fragments.

* SCHC ACK REQ: A request for a SCHC ACK. It can be used when the sender expects a SCHC ACK from the receiver but doesn't receive one.

* SCHC Sender-Abort: A message by the sender telling the receiver that it has aborted the transmission of a fragmented SCHC Packet.

* SCHC Receiver-Abort: A message by the receiver to tell the sender to abort the transmission of a fragmented SCHC Packet.

### Windows, Timers, Counters, Intergrity Checking {#MiscTools}

Some SCHC F/R modes may handle SCHC Fragments as collections, called windows.
When windowing is used, information on the correct reception of the SCHC Fragments belonging to a window is grouped together and may be sent back as a SCHC ACK.
Whatever the windowing choice, a SCHC ACK may report on the reception of all the SCHC Fragments composing a fragmented SCHC Packet.


Some SCHC F/R modes can use the following timers and counters

* Inactivity Timer: this timer can be used to unlock a SCHC Fragment receiver that is not receiving a SCHC F/R message while it is expecting one.

* Retransmission Timer: this timer can be used by a SCHC Fragment sender to set a timeout on expecting a SCHC ACK.

* Attempts: this counter counts the requests for a missing SCHC ACK. MAX_ACK_REQUESTS is the threshold at which the SCHC Fragment sender stops sending requests.


The reassembled SCHC Packet is checked for integrity at the receive end.
One way of doing integrity checking is by computing a MIC at the sender side and transmitting it to the receiver for comparison with the locally computed MIC.
Some LPWAN technologies under SCHC may provide other means of ensuring that all fragments composing a SCHC Packet were correctly received and reassembled.

### Header Fields {#HeaderFields}

The SCHC F/R messages use the following fields (see the related formats in {{Fragfor}})

* Rule ID: this is used to identify

  * that a SCHC F/R message is being carried, as opposed to an unfragmented SCHC Packet,

  * which SCHC F/R mode is used

  * and among this mode

     * if windowing is used, what the window size is

     * what optional fields are present

     * and what the field sizes are.

  Therefore, the Rule ID allows SCHC F/R interleaving non-fragmented SCHC Packets and SCHC Fragments that carry other SCHC Packets, or interleaving SCHC Fragments that use different SCHC F/R modes or different parameters.


* Fragment Compressed Number (FCN). The FCN is present in the SCHC Fragments header. This field conveys information about the progress in the sequence of SCHC Fragments.
  For exemple, it can contain a truncated, efficient representation of a larger-sized fragment number.
  The exact use of the FCN field is left to each SCHC F/R mode.
  However, two values are special. They help control the SCHC F/R process:

  * The FCN value with all the bits equal to 1 (All-1) signals the very last SCHC Fragment of a SCHC Packet.
  By extension, if windowing is used, the last window of a packet is called the All-1 window.

  * If windowing is used, the FCN value with all the bits equal to 0 (All-0) signals the last SCHC Fragment of a window that is not the last one of
  the SCHC packet. By extension, such a window is called an All-0 window.

  The size of the FCN field is called N. It is defined for each F/R mode and Rule ID.
  Since All-1 is special, the maximum value of FCN (as an unsigned integer) theoretically is (2^N)-2.
  However, each SCHC F/R mode or each LPWAN technology-specific document is free to specify a maximum value, called MAX_WIND_FCN, lower than (2^N)-2.
  The rationale is that MAX_WIND_FCN finely controls the width of the Bitmap in the SCHC ACK message, which may be critical to some LPWAN technologies.
  
* Datagram Tag (DTag). The DTag field is optional.

  When there is no DTag, there can be only one fragmented SCHC Packet in transit:
  only after all its fragments have been transmitted can another fragmented SCHC Packet be sent.

  If present, DTag MUST be set to the same value for all SCHC Fragments carrying the same SCHC
  packet, and to different values for different SCHC Packets. Using this field, the sender can interleave fragments from
  different SCHC Packets, while the receiver can still tell them apart.

  If a SCHC ACK message is sent, it MUST carry the same DTag value as the SCHC Fragments for which this SCHC ACK is
  intended. 

  The presence and size of the DTag field is defined by each SCHC F/R mode for each Rule ID value.
  For each new SCHC Packet processed by the sender under the same F/R mode and same RuleID, DTag MUST be sequentially
  increased, from 0 to (2^T) – 1 wrapping back from (2^T) - 1 to 0, where T is the size of DTag in bits.


* W (window): The W field is optional. It is only present if windowing is used.

  This field carries the same value for all SCHC F/R messages pertaining to the same window, and it is
  incremented for the next window. The value of W is initialised to 0 for each new SCHC Packet fragmented under the same F/R mode and Rule ID.

  The presence of W and its size (called M, in bits) is defined for each F/R mode and Rule ID.

* Message Integrity Check (MIC).
  This field is optional. If present, it only appears in the All-1 SCHC Fragments and in the All-1 ACK REQ.
  If present, its size MUST be at least an L2 Word.
  If the MIC is present, the sender MUST compute it over the complete SCHC Packet and the potential padding bits of the last SCHC Fragment.

  Sending a MIC is one way of allowing the receiver to do integrity checking on the reassembled SCHC Packet (see {{MiscTools}}).
  The MIC supports UDP checksum elision by SCHC C/D (see {{UDPchecksum}}).

  The CRC32 polynomial 0xEDB88320 (i.e. the reverse representation
  of the polynomial used e.g. in the Ethernet standard {{RFC3385}}) is RECOMMENDED as the default algorithm for computing the
  MIC. Nevertheless, other MIC lengths or other algorithms MAY be required by the technology-specific documents.

  Note that the concatenation of the complete SCHC Packet and the potential padding bits of the last SCHC Fragment does not
  generally constitute an integer number of bytes.
  For implementers to be able to used byte-oriented CRC libraries, it is RECOMMENDED that the concatenation of the
  complete SCHC Packet and the last fragment potential padding bits be zero-extended to the next byte boundary and
  that the MIC be computed on that byte array.

* C (integrity Check): C is a 1-bit field.
  This field is used in the SCHC ACK messages to report the outcome of the reassembled SCHC Packet integrity check (see {{MiscTools}}).
  A value of 1 tells that the integrity check succeeded. A value of 0 represents a failure.

* Bitmap. The Bitmap is used together with windowing.
  It is a bit field maintained by the sender during the fragmentation
  and by the receiver during the reassembly of a fragmented SCHC Packet.
  The Bitmap may also be carried in a SCHC ACK, either in extenso or in a shortened form (see {{Bitmapopt}}).
  Each bit in the internal representation of the Bitmap corresponds to a SCHC
  Fragment of the current window, and provides feedback on whether the SCHC Fragment has been received or not. The right-most
  position on the Bitmap reports if the All-0 or All-1 fragment has been correctly received or not. Feedback on the SCHC
  Fragment with the FCN == MAX_WIND_FCN value is provided by the bit in the left-most position of the Bitmap. In the Bitmap, a bit
  set to 1 indicates that the SCHC Fragment of FCN corresponding to that bit position has been correctly received.


## SCHC F/R Message Formats {#Fragfor}

This section defines the SCHC Fragment format, the SCHC ACK format, the SCHC ACK REQ format and the SCHC Abort formats.

### SCHC Fragment format

A SCHC Fragment conforms to the general format shown in {{Fig-FragFormat}}.
It comprises a SCHC Fragment Header and a SCHC Fragment Payload.
The SCHC Fragment Payload carries a subset of the SCHC Packet.
In addition, the last SCHC Fragment of a SCHC Packet carries just enough padding bits as needed to fill up an L2 Word.
The SCHC Fragment is the data unit passed on to the L2 for transmission.

~~~~   
+-----------------+-----------------------+~~~~~~~~~~~~~~~~~~~~~
| Fragment Header |   Fragment Payload    | padding (as needed)
+-----------------+-----------------------+~~~~~~~~~~~~~~~~~~~~~
~~~~
{: #Fig-FragFormat title='SCHC Fragment general format. Presence of a padding field is optional'}

#### Fragments that are not the last one {#NotLastFrag}

All SCHC Fragments except the last one of a SCHC Packet MUST conform to the detailed format defined in {{Fig-NotLastFrag}}.
The W field is optional. Its presence is specified by each SCHC F/R mode.


~~~~

 |------ Fragment Header ------|
           |-- T --|-M-|-- N --|
 +-- ... --+- ... -+---+- ... -+--------...-------+
 | Rule ID | DTag  | W |  FCN  | Fragment Payload |
 +-- ... --+- ... -+---+- ... -+--------...-------+

~~~~
{: #Fig-NotLastFrag title='Detailed Header Format for Fragments except the Last One'}


The total size of the Fragment Header is not necessarily a multiple of the L2 Word size.
To build the Fragment Payload, SCHC F/R MUST take from the SCHC Packet a number of bits that makes the SCHC Fragment an exact multiple of L2 Words. As a consequence, no padding bit is used for these fragments.


The Fragment Payload of a SCHC Fragment with FCN == All-0 (called an All-0 SCHC Fragment) MUST be at least the size of an L2 Word.
The rationale is that, even in the presence of padding, an All-0 SCHC Fragment needs to be distinguishable from the SCHC ACK REQ message, which has the same header (see {{All0ACKREQ}}).

#### All-1 SCHC Fragment {#LastFrag}

The Fragment Header of the last SCHC Fragment of a SCHC Packet MUST conform to
the detailed format shown in {{Fig-LastFrag}}.
The W field is optional. Its presence is specified by each SCHC F/R mode.
The MIC field is optional.

~~~~

|----------- Fragment Header ---------|
          |-- T --|-M-|-- N --|
+-- ... --+- ... -+---+- ... -+- ... -+------...-----+~~~~~~~~~~~~~~~~~~~~~
| Rule ID | DTag  | W | 11..1 |  MIC  | Frag Payload | padding (as needed)
+-- ... --+- ... -+---+- ... -+- ... -+------...-----+~~~~~~~~~~~~~~~~~~~~~
                        (FCN)
~~~~
{: #Fig-LastFrag title='Detailed Header Format for the last SCHC Fragment of a SCHC Packet'}
The total size of the All-1 SCHC Fragment header is generally not a multiple of the L2 Word size.
The All-1 SCHC Fragment being the last one of the SCHC Packet, SCHC F/R cannot freely choose the payload size to align the fragment to an L2 Word.
Therefore, padding bits are generally appended to the All-1 SCHC Fragment to make it a multiple of L2 Words in size.

The MIC MUST be computed on the payload and the padding bits. The rationale is that the SCHC Reassembler needs to check the correctness of
the reassembled SCHC packet but it has no way of knowing where the payload ends.
Indeed, the latter requires decompressing the SCHC Packet, which is out of the scope of the reassembler.

An All-1 SCHC Fragment payload MUST be at least the size of an L2 Word.
The rationale is that, even in the presence of padding, the SCHC All-1 ACK REQ (see {{All1ACKREQ}}) needs to be distinguishable from the All-1 SCHC Fragment.
This may entail saving an L2 Word from the payload of the previous SCHC Fragment to make the payload of this All-1 SCHC Fragment big enough.

The values for N, T and the length of MIC are not specified in this document, and SHOULD be determined in other documents (e.g. technology-specific profile documents).

### SCHC ACK format

#### All-0 SCHC ACK {#All0ACK}

This message only appears in SCHC F/R modes that use windowing.
The format of a SCHC ACK that acknowledges a window of SCHC Fragments that is not the last one of a SCHC Packet (All-0 window) is shown in {{Fig-All0-ACK-Format}}.

~~~~

            |-- T --|-M-|
+---- ... --+- ... -+---+---- ... -----+
|  Rule ID  |  DTag | W |encoded Bitmap| (no payload)
+---- ... --+- ... -+---+---- ... -----+

~~~~
{: #Fig-All0-ACK-Format title='ACK format for All-0 windows'}

The Rule ID and DTag values in the SCHC ACK message MUST be identical to the ones used in the SCHC Fragments that are being acknowledged. This allows matching the SCHC ACK and the corresponding SCHC Fragments.

#### All-1 SCHC ACK {#All1ACK}

The format of a SCHC ACK that follows the transmission of the last SCHC Fragments of a SCHC Packet is shown in {{Fig-All1-ACK-Format}}.
The W field is optional. Its presence is specified by each SCHC F/R mode.

~~~~

            |-- T --|-M-|1|
+---- ... --+- ... -+---+-+
|  Rule ID  |  DTag | W |1| (MIC correct)
+---- ... --+- ... -+---+-+

+---- ... --+- ... -+---+-+----- ... -----+
|  Rule ID  |  DTag | W |0|encoded Bitmap |(MIC Incorrect)
+---- ... --+- ... -+---+-+----- ... -----+
                         C
~~~~
{: #Fig-All1-ACK-Format title='Format of a SCHC ACK at the end of a SCHC Packet transmission'}

The All-1 SCHC ACK header contains a C bit (see {{HeaderFields}}).
If the C bit is set to 0 (integrity check failure) and if windowing is used, the Bitmap for the All-1 window follows.

The Rule ID and DTag values in the SCHC ACK messages MUST be identical to the ones used in the SCHC Fragments that are being acknowledged. This allows matching the SCHC ACK and the corresponding SCHC Fragments.

If present, the Bitmap carries information on the reception of each fragment of the window as described in {{FragTools}}.

See {{SCHCParams}} for a discussion on the size of the Bitmaps.

In order to reduce the SCHC ACK size, the Bitmap that is actually transmitted is shortened ("encoded") as explained in {{Bitmapopt}}.

#### Bitmap Encoding {#Bitmapopt}
The SCHC ACK that is transmitted is truncated by applying the following algorithm: the longest contiguous sequence of bits that starts at an L2 Word boundary of the SCHC ACK,
where the bits of that sequence are all set to 1, are all part of the Bitmap and finish exactly at the end of the Bitmap, if one such sequence exists, MUST NOT be transmitted.
Because the SCHC Fragment sender knows the actual Bitmap size, it can reconstruct the original Bitmap from the shortened bitmap.

When shortening effectively takes place, the SCHC ACK is a multiple of L2 Words, and padding MUST NOT be appended.
When shortening does not happen, padding bits MUST be appended as needed to fill up the last L2 Word.

{{Fig-Localbitmap}} shows an example where L2 Words are actually bytes and where the original Bitmap contains 17 bits, the last 15 of which are all set to 1.

~~~~                                                  

|--  SCHC ACK Header --|--------       Bitmap    --------|
|  Rule ID  |  DTag  |W|1|0|1|1|1|1|1|1|1|1|1|1|1|1|1|1|1|
     next L2 Word boundary ->|  next L2 Word |  next L2 Word |

~~~~
{: #Fig-Localbitmap title='A non-encoded Bitmap'}

{{Fig-transmittedbitmap}} shows that the last 14 bits are not sent.

~~~~   
            |-- T --|-M-|
+---- ... --+- ... -+---+-+-+-+
|  Rule ID  |  DTag | W |1|0|1|
+---- ... --+- ... -+---+-+-+-+
      next L2 Word boundary ->|

~~~~
{: #Fig-transmittedbitmap title='Optimized Bitmap format'}

{{Fig-Bitmap-Win}} shows an example of a SCHC ACK with FCN ranging from 6 down to 0, where the Bitmap indicates that the second and the fifth SCHC Fragments have not been correctly received.

~~~~                                                  
                         6 5 4 3 2 1 0 (*)
            |-- T --|-M-|
+-----------+-------+---+-+-+-+-+-+-+-+
|  Rule ID  |  DTag | W |1|0|1|1|0|1|1|            Bitmap before tx
+-----------+-------+---+-+-+-+-+-+-+-+
  next L2 Word boundary ->|<-- L2 Word -->|
    (*)=(FCN values)

+-----------+-------+---+-+-+-+-+-+-+-+~~~+
|  Rule ID  |  DTag | W |1|0|1|1|0|1|1|Pad|        Encoded Bitmap
+-----------+-------+---+-+-+-+-+-+-+-+~~~+
  next L2 Word boundary ->|<-- L2 Word -->|


~~~~
{: #Fig-Bitmap-Win title='Example of a Bitmap before transmission, and the transmitted one, for a window that is not the last one'}

{{Fig-Bitmap-lastWin}} shows an example of a SCHC ACK with FCN ranging from 6 down to 0, where MIC check has failed but the Bitmap
indicates that there is no missing SCHC Fragment.

~~~~                                                  
|- Fragmentation Header  -|6 5 4 3 2 1 7 (*)
            |-- T --|-M-|
|  Rule ID  |  DTag | W |0|1|1|1|1|1|1|1|          Bitmap before tx
    next L2 Word boundary ->|<-- L2 Word -->|
                         C
+---- ... --+- ... -+---+-+-+
|  Rule ID  |  DTag | W |0|1|                      Encoded Bitmap
+---- ... --+- ... -+---+-+-+
    next L2 Word boundary ->|
   (*) = (FCN values indicating the order)

~~~~
{: #Fig-Bitmap-lastWin title='Example of the Bitmap in ACK-Always or ACK-on-Error for the last window'}


### SCHC ACK REQ format

#### SCHC All-0 ACK REQ {#All0ACKREQ}

The SCHC All-0 ACK REQ is only used in SCHC F/R that use a windowing mechanism.
Its purpose is for a a sender to request the retransmission of a SCHC ACK by the receiver, at the end of a window which is not the last one of a SCHC Packet.
The SCHC All-0 ACK REQ format is described in {{Fig-All0ACKREQ}}.
The W field is optional. Its presence is specified by each SCHC F/R mode.


~~~~

|------ ACK REQ Header -------|
          |-- T --|-M-|-- N --|
+-- ... --+- ... -+---+- ... -+~~~~~~~~~~~~~~~~~~~~~
| Rule ID | DTag  | W |  0..0 | padding (as needed)      (no payload)
+-- ... --+- ... -+---+- ... -+~~~~~~~~~~~~~~~~~~~~~

~~~~
{: #Fig-All0ACKREQ title='All-0 empty fragment detailed format'}

Notice that the SCHC All-0 ACK REQ has the same header as an All-0 SCHC Fragment but doesn't have a a payload.
The message length allows distinguishing between a SCHC All-0 ACK REQ and a All-0 SCHC Fragment.

The size of the SCHC All-0 ACK REQ header is generally not a multiple of the L2 Word size.
Therefore, a SCHC All-0 ACK REQ generally needs padding bits. The padding bits are always less than an L2 Word.

Since an All-0 SCHC Fragment payload MUST be at least the size of an L2 Word, a receiver can distinguish a SCHC All-0 ACK REQ from an All-0 SCHC Fragment, even in the presence of padding.

#### SCHC All-1 ACK REQ {#All1ACKREQ}

The SCHC All-1 ACK REQ is used by a fragment sender to request a retransmission of the SCHC ACK after it has transmitted the last SCHC Fragment of a SCHC Packet.
It's format is described in {{Fig-All1ACKREQ}}.
The W field is optional. Its presence is specified by each SCHC F/R mode.
The MIC field is optional.

~~~~

|---------- ACK REQ Header -----------|
          |-- T --|-M-|-- N --|
+-- ... --+- ... -+---+- ... -+- ... -+~~~~~~~~~~~~~~~~~~~
| Rule ID | DTag  | W |  1..1 |  MIC  | padding as needed (no payload)
+-- ... --+- ... -+---+- ... -+- ... -+~~~~~~~~~~~~~~~~~~~

~~~~
{: #Fig-All1ACKREQ title='All-1 for Retries format, also called All-1 empty'}

Notice that the SCHC All-1 ACK REQ has the same header as an All-1 SCHC Fragment but doesn't have a a payload.
The message length allows distinguishing between a SCHC All-1 ACK REQ and a All-1 SCHC Fragment.

The size of the SCHC All-1 ACK REQ header is generally not a multiple of the L2 Word size. Therefore, a SCHC All-1 ACK REQ generally needs padding bits. The padding bits are always less than an L2 Word.

Since an All-1 SCHC Fragment payload MUST be at least the size of an L2 Word, a receiver can distinguish a SCHC All-1 ACK REQ from an All-1 SCHC Fragment, even in the presence of padding.



### SCHC Abort formats {#Aborts}


#### SCHC Sender-Abort

When a SCHC Fragment sender needs to abort an on-going fragmented SCHC Packet transmission, it sends a SCHC Sender-Abort.
Its format is described in {{Fig-All1Abort}}.
The W field is optional. Its presence is specified by each SCHC F/R mode.

~~~~

|------ Fragment Header ------|
          |-- T --|-M-|-- N --|
+-- ... --+- ... -+---+- ... -+~~~~~~~~~~~~~~~~~~~~~
| Rule ID | DTag  | W | 11..1 | padding (as needed)
+-- ... --+- ... -+---+- ... -+~~~~~~~~~~~~~~~~~~~~~

~~~~
{: #Fig-All1Abort title='SCHC Sender-Abort format'}

Notice that the SCHC Sender-Abort has the same header as an All-1 SCHC Fragment, but that it does not include a MIC nor a payload.
The message length allows distinguishing between a SCHC Sender-Abort and a SCHC Fragment.

The size of the SCHC Sender-Abort header is generally not a multiple of the L2 Word size.
Therefore, a SCHC Sender-Abort generally needs padding bits.

Padding bits are strictly less than a L2 Word size. The MIC is at least the size of an L2 Word.
Therefore, by looking at the message length, a receiver can distinguish a SCHC Sender-Abort from an SCHC All-1 ACK REQ (see {{All1ACKREQ}}), even in the presence of padding.

The Rule ID and DTag values in the SCHC Sender-Abort message MUST be identical to the ones used in the fragments of the SCHC Packet the transmission of which is being aborted.

The SCHC Sender-Abort MUST NOT be acknowledged and MUST NOT be retransmitted.


#### SCHC Receiver-Abort

When a SCHC Fragment receiver needs to abort an on-going fragmented SCHC Packet transmission, it transmits a SCHC Receiver-Abort.
Its format is described in {{Fig-ACKabort}}.
The W field is optional. Its presence is specified by each SCHC F/R mode.

~~~~

|-- Receiver-Abort Header --|

+---- ... ----+-- ... --+---+-+-+-+-+-+-+-+-+-+-+-+
|   Rule ID   |   DTag  | W | 1..1|      1..1     |
+---- ... ----+-- ... --+---+-+-+-+-+-+-+-+-+-+-+-+
          next L2 Word boundary ->|<-- L2 Word -->|

~~~~
{: #Fig-ACKabort title='SCHC Receiver-Abort format'}

Notice that the SCHC Receiver-Abort has the same header as a SCHC ACK message.
THe bits that follow the SCHC Receiver-Abort header look like a shortened Bitmap set to 1 up to
the first L2 Word boundary, followed by an extra L2 Word full of 1's.
Such a bit pattern never occurs in a regular SCHC ACK. This is how the SCHC Receiver-Abort message is recognized.


The Rule ID and DTag values in the SCHC Receiver-Abort message MUST be identical to the ones used in the fragments of the SCHC Packet the transmission of which is being aborted.


A SCHC Receiver-Abort is aligned to L2 Words, by design. Therefore, padding MUST NOT be appended.


The SCHC Sender-Abort MUST NOT be acknowledged and MUST NOT be retransmitted.


## SCHC F/R modes {#FragModes}


### No-ACK mode {#No-ACK-subsection}

In No-ACK mode, there is no feedback communication from the fragment receiver to the fragment sender.
The sender just transmits all the SCHC Fragments blindly.

SCHC ACK or SCHC ACK REQ MUST NOT be sent.
SCHC Sender-Abort MAY be sent. SCHC Receiver-Abort MUST NOT be sent.

Windowing is not used. Therefore, the W field MUST NOT be present.
The Retransmission Timer is not used.
The Attempts counter is not used.
At the receiver, one Inactivity Timer MUST be instantiated for each pair of Rule ID and optional DTag values.
The Inactivity Timer expiration value is based on the characteristics of the underlying LPWAN technology
and MUST be defined in other documents (e.g. technology-specific profile documents).

The presence and size of the MIC and DTag fields MUST be defined by each LPWAN technology-specific document.
The size of the FCN field is RECOMMENDED to be 1 bit but MAY be defined by each LPWAN technology-specific document.

#### Sender behaviour

The sender transmits all the SCHC Fragments of a SCHC Packet, in sequential order.
It MUST use the same Rule ID and optional DTag pair for all these fragments.
The All-1 FCN value MUST be used for the last SCHC Fragment, which MUST use the format described in {{LastFrag}}.
The All-1 FCN MUST NOT be used for the preceeding SCHC Fragments, which MUST use the format described in {{NotLastFrag}}.

{{Fig-NoACKModeSnd}} shows an example of a corresponding state machine.

#### Receiver behaviour

While FCN is not All-1, the receiver assembles the payloads of the SCHC Fragments that bear the same Rule ID and optional DTag.
When an All-1 SCHC Fragment is received with the same Rule ID and optional DTag, the receiver appends the All-1 SCHC Fragment Payload and the padding bits to the
previously received SCHC Fragment payloads for this SCHC Packet. The reassembly operation concludes.

Each time a SCHC Fragment is received with the same Rule ID and optional DTag, the Inactivity timer MUST be reset.

When the Inactivity timer expires, the reassembly operation concludes.

If reassembly concludes because of Inactivity Timer expiration,
the SCHC Packet being reassembled MUST be dropped and resources associated with this Rule ID and optional DTag MUST be released.

If reassembly concludes because of receiving an All-1 SCHC Fragment and if integrity checking (either through the MIC or through some other L2 means) fails,
the reassembled SCHC Packet MUST be dropped and resources associated with this Rule ID and optional DTag MUST be released.

On reception of a SCHC Sender-Abort, the receiver MAY release all resource related to the reassembly of the SCHC Fragments with the same Rule ID and optional DTag.

{{Fig-NoACKModeRcv}} shows an example of a corresponding state machine.

### ACK-Always {#ACK-Always-subsection}

In ACK-Always mode, windowing is used.
An acknowledgement, positive or negative, is fed by the fragment receiver back to the fragment sender at the end of the transmission of each window of SCHC Fragments.

In a nutshell, the fragment sender iterates retransmitting the SCHC Fragments that are reported missing until the fragment receiver reports that all the SCHC Fragments belonging to the window have been correctly received, or until too many attempts were made.
The fragment sender only advances to the next window of SCHC Fragments when it has ascertained that all the SCHC Fragments belonging to the current window have been fully and correctly received (lock-step behaviour between the sender and the receiver, at the window granularity).

The W field MUST be present and its size MUST be 1 bit.

At the sender, one W bit and one FCN counter MUST be instantiated for each pair of Rule ID and optional DTag values.
At the receiver, one W bit, one FCN counter and one Bitmap MUST be instantiated for each pair of Rule ID and optional DTag values.
At the sender, one Attempts counter MUST be instantiated for each pair of Rule ID and optional DTag values.
At the sender, one Retransmission Timer MUST be instantiated for each pair of Rule ID and optional DTag values.
At the receiver, one Inactivity Timer MUST be instantiated for each pair of Rule ID and optional DTag values.
The expiration values of the Retransmission Timer and of the Inactivity Timer are based on the characteristics of the underlying LPWAN technology
and MUST be defined in other documents (e.g. technology-specific profile documents).

The presence and size of the MIC and DTag fields MUST be defined by each LPWAN technology-specific document.

The value of N (size of the FCN field) and the value of MAX_WIND_FCN MUST be defined by each LPWAN technology-specific document.

The value of MAX_ACK_REQUESTS MUST be defined by each LPWAN technology-specific document.

#### Sender behaviour

At the beginning of the fragmentation of a new SCHC Packet, the fragment sender MUST select a Rule ID and DTag value pair for this SCHC Packet and it MUST initialize the W bit to 0.
The SCHC Packet is fragmented into pieces that will be carried by SCHC Fragment messages, which are grouped in windows.
The SCHC Fragment that comes first in a window, in fragmentation order, MUST have an FCN value of MAX_WIND_FCN.
Within a window, each successive SCHC Fragment MUST bear an FCN field decremented by 1 (unsigned integer) compared to the fragment preceeding it in fragmentation order, except for the last SCHC Fragment of the SCHC Packet.
Except for the last window, a window MUST be composed of exactly one SCHC Fragment with each possible value of FCN between MAX_WIND_FCN and 0.
In the last window, the last SCHC Fragment in fragmentation order MUST have the FCN value of All-1 and MUST use the format described in {{LastFrag}}.
The other SCHC Fragments MUST use the format described in {{NotLastFrag}}.

The fragment sender MUST start by processing the window that is first in fragmentation order.
With this window, it MUST enter a **first state** in which it MUST transmit all the SCHC Fragments composing the window.
The SCHC Fragment that bears the FCN value of All-0 or All-1 MUST be sent last in that serie.

Then, the fragment sender MUST initialize an Attempts counter to 0 for that Rule ID and DTag value pair and it MUST enter a **second state**
where it MUST start a Retransmission Timer for that Rule ID and DTag value pair
and where it MUST expect to receive a SCHC ACK.
On Retransmission Timer expiration, if Attempts is strictly less that MAX_ACK_REQUESTS, the fragment sender MUST send a SCHC ACK REQ and MUST increment the Attempts counter.
If the current window is not the last one (All-0), the SCHC ACK REQ that is sent MUST be the SCHC All-0 ACK REQ described in {{All0ACKREQ}}.
If the current window is the last one (All-1), the SCHC ACK REQ that is sent MUST be the SCHC All-1 ACK REQ described in {{All1ACKREQ}}.
If the Retransmission Timer expires while Attempts is greater or equal to MAX_ACK_REQUESTS, the fragment sender MUST send a SCHC Sender-Abort, it MUST release all resource associated with this SCHC Packet and it MUST exit with an error condition.
On receiving a SCHC ACK, if the SCHC ACK indicates that some fragments are missing at the receiver, the sender MUST increment Attempts, it MUST stop the Retransmission Timer and MUST enter a **third state**.
If the current window is not the last one (All-0) and the SCHC ACK indicates that all fragments were received correctly, the sender MUST stop the Retransmission Timer, it MUST increment W, it MUST advance to the next fragmentation window and it MUST return to the **first state**.
If the current window is the last one (All-1) and the SCHC ACK indicates that more fragments were received than the sender actually sent, then the fragment sender MUST send a SCHC Sender-Abort, it MUST release all resource associated with this SCHC Packet and it MUST exit with an error condition.
If the current window is the last one (All-1) and the SCHC ACK indicates that all fragments were received correctly yet integrity checking does not match, the fragment sender MUST send a SCHC Sender-Abort, it MUST release all resource associated with this SCHC Packet and it MUST exit with an error condition.
If the current window is the last one (All-1) and the SCHC ACK indicates that all fragments were received correctly and integrity checking does match, the sender exits succesfully.

In the **third state**, the fragment sender MUST transmit all the SCHC Fragments that have been reported missing, then it MUST return to the **second state**.

At any time in the **second state**, the sender MUST silently discard and ignore any SCK ACK bearing a W value different from its own W value.

At any time in **any state**, on receiving a SCHC Receiver-Abort with the correct Rule ID and DTag value pair, the fragment sender MUST release all resource associated with this SCHC Packet and it MUST exit with an error condition.

{{Fig-ACKAlwaysSnd}} shows an example of a corresponding state machine.

A delay between each SCHC Fragment transmission can be added to respect local regulations or other constraints imposed by the applications.


#### Receiver behaviour

On receiving a SCHC Fragment with a Rule ID and DTag pair not being processed at that time,
the receiver MUST start a process to assemble a SCHC Packet with that Rule ID and DTag value pair.
That process MUST only examine received SCHC F/R messages with that Rule ID and DTag value pair
and MUST only transmit SCHC F/R messages with that Rule ID and DTag value pair.
It MUST intialise its current window number to be the first one, it MUST initialise its current W bit to 0
and it MUST enter a **first state** and MUST process the SCHC Fragment that was just received.

Note: the purpose in the **first state** is to receive the initial transmission of the SCHC Fragments of the current window.

On entering the **first state**, the receiver MUST start an Inactivity Timer and it MUST initialise an empty Bitmap for the current window.

In the **first state**:

  - Any received SCHC Fragment bearing a W bit different from the current W bit of the receiver process MUST be silently ignored and discarded.
  In the rest of this list of actions, only SCHC Fragments that have the correct W bit are considered.
  - On reception of each SCHC Fragment, the receiver MUST reset the Inactivity Timer and MUST update the Bitmap.
  - The receiver MUST assemble the payloads of the received SCHC Fragments, based on their FCN
  and based on the current window number.
  - On expiration of the Inactivity Timer, the receiver MUST send a SCHC Receiver-Abort,
  it MUST release all resource associated with this SCHC Packet
  and it MUST exit the receive process for that SCHC Packet.
  - On receiving an All-0 SCHC Fragment, the receiver MUST send a SCHC All-0 ACK (specified in {{All0ACK}}).
  Reminder: the SCHC All-0 ACK reports on the SCHC Fragments that have been correctly received in the current window.
    * If the Bitmap indicates that all the SCHC Fragments of the current window have been correctly received,
    the receiver MUST increment its current window number,
    it MUST flip its current W bit
    and it MUST re-enter the **first state**.
    * If the Bitmap indicates that at least one SCHC Fragment is missing in the current window,
    the receiver MUST enter a **second state**.
  - On receiving an All-1 SCHC Fragment, the receiver MUST send a SCHC All-1 ACK (specified in {{All1ACK}}).
  Reminder: the SCHC All-1 ACK reports on the integrity check of the whole reassembled SCHC Packet
  and optionally on the SCHC Fragments that were correctly received in the current window.
    * If the integrity check indicates that the full SCHC Packet has been correctly reassembled,
    the receiver MUST enter the **fourth state**.
    * If the integrity check indicates that the full SCHC Packet has not been correctly reassembled,
    the receiver MUST enter a **third state**.


Note: the purpose in the **second state** is to receive the retransmitted SCHC Fragments of the current window before advancing to the next window.

On entering the **second state**, the receiver MUST reset the Inactivity Timer.

In the **second state**:

  - On reception of each SCHC Fragment or SCHC ACK REQ, the receiver MUST reset the Inactivity Timer.
  - On reception of an All-0 SCHC ACK REQ with a W bit equal to that of the receiving process, the receiver MUST send an All-0 SCHC ACK and MUST reset the Inactivity Timer.
  - On reception of a SCHC Fragment with a W bit equal to that of the receiving process,
    * If the Bitmap indicates that the SCHC Fragment had already been received, the receiver MUST silently ignore it and discard it.
    * Otherwise it MUST update the Bitmap and assemble the payload of the SCHC Fragment received.
  - On the Bitmap becoming fully populated with 1's, the receiver MUST send an All-0 SCHC ACK and MUST reset the Inactivity Timer
  - On expiration of the Inactivity Timer, the receiver MUST send a SCHC Receiver-Abort,
  it MUST release all resource associated with this SCHC Packet
  and it MUST exit the receive process for that SCHC Packet.
  - On reception of a SCHC Fragment with a W bit different from that of the receiving process
    * If the Bitmap is fully populated with 1's,
    the receiver MUST increment its current window number,
    it MUST flip its current W bit,
    it MUST enter the **first state** and it MUST the SCHC Fragment that was just received.
    * Otherwise, the receiver MUST silently ignore and discard the SCHC Fragment just received.

Note: the purpose in the **third state** is to receive the retransmitted SCHC Fragments of the last window.

On entering the **third state**, the receiver MUST reset the Inactivity Timer.

In the **third state**

  - On reception of each SCHC Fragment or SCHC ACK REQ, the receiver MUST reset the Inactivity Timer.
  - On reception of an All-1 SCHC ACK REQ with a W bit equal to that of the receiving process, the receiver MUST send an All-1 SCHC ACK and MUST reset the Inactivity Timer.
  - On reception of a SCHC Fragment with a W bit equal to that of the receiving process,
    * If the Bitmap indicates that the SCHC Fragment had already been received, the receiver MUST silently ignore it and discard it.
    * Otherwise it MUST update the Bitmap and assemble the payload of the SCHC Fragment received. It MUST compute the integrity check.
  - On the integrity check becoming True, the receiver MUST send an All-1 SCHC ACK and MUST reset the Inactivity Timer
  - On expiration of the Inactivity Timer, the receiver MUST send a SCHC Receiver-Abort,
  it MUST release all resource associated with this SCHC Packet
  and it MUST exit the receive process for that SCHC Packet.
  - On reception of a SCHC Fragment with a W bit different from that of the receiving process, the receiver MUST silently ignore and discard the SCHC Fragment just received.

Note: the purpose in the **fourth state** is to increase the chances that the sender receives a SCHC All-1 ACK for this SCHC Packet.

On entering the **fourth state**, the receiver MUST reset the Inactivity Timer.

In the **fourth state**

  - Any received SCHC F/R message with a W bit different from the current W bit of the receiver process MUST be silently ignored and discarded. 
  - Any received SCHC F/R message different from an All-1 SCHC Fragment or an SCHC All-1 ACK REQ MUST be silently ignored and discarded.
  - On receiving an All-1 SCHC Fragment or an SCHC All-1 ACK REQ, the receiver MUST send an All-1 SCHC ACK.
  - On expiration of the Inactivity Timer, the receive process for that SCHC Packet MUST exit

At any time in **any state**, when Attempts reaches MAC_ACK_REQUESTS,
the receiver MUST send a SCHC Receiver-Abort,
it MUST release all resource associated with this SCHC Packet
and it MUST exit the receive process for that SCHC Packet.

At any time in **any state**, on receiving a SCHC Sender-Abort,
the fragment sender MUST release all resource associated with this SCHC Packet
and it MUST exit the receive process for that SCHC Packet.


{{Fig-ACKAlwaysRcv}} shows an example of a corresponding state machine.


### ACK-on-Error {#ACK-on-Error-subsection}

## Supporting multiple window sizes

For ACK-Always or ACK-on-Error, implementers MAY opt to support a single window size or multiple window sizes.  The latter, when feasible, may provide performance optimizations.  For example, a large window size SHOULD be used for packets that need to be carried by a large number of SCHC Fragments. However, when the number of SCHC Fragments required to carry a packet is low, a smaller window size, and thus a shorter Bitmap, MAY be sufficient to provide feedback on all SCHC Fragments. If multiple window sizes are supported, the Rule ID MAY be used to signal the window size in use for a specific packet transmission.

Note that the same window size MUST be used for the transmission of all SCHC Fragments that belong to the same SCHC Packet.


## Downlink SCHC Fragment transmission


For downlink transmission of a fragmented SCHC Packet in ACK-Always mode, the SCHC Fragment receiver MAY support timer-based SCHC ACK retransmission. In this mechanism, the SCHC Fragment receiver initializes and starts a timer (the Inactivity Timer is used) after the transmission of a SCHC ACK, except when the SCHC ACK is sent in response to the last SCHC Fragment of a packet (All-1 fragment). In the latter case, the SCHC Fragment receiver does not start a timer after transmission of the SCHC ACK.

If, after transmission of a SCHC ACK that is not an All-1 fragment, and before expiration of the corresponding Inactivity timer, the SCHC Fragment receiver receives a SCHC Fragment that belongs to the current window (e.g. a missing SCHC Fragment from the current window) or to the next window, the Inactivity timer for the SCHC ACK is stopped. However, if the Inactivity timer expires, the SCHC ACK is resent and the Inactivity timer is reinitialized and restarted.

The default initial value for the Inactivity timer, as well as the maximum number of retries for a specific SCHC ACK, denoted MAX_ACK_RETRIES, are not defined in this document, and need to be defined in other documents (e.g. technology-specific profiles). The initial value of the Inactivity timer is expected to be greater than that of the Retransmission timer, in order to make sure that a (buffered) SCHC Fragment to be retransmitted can find an opportunity for that transmission.

When the SCHC Fragment sender transmits the All-1 fragment, it starts its Retransmission Timer with a large timeout value (e.g. several times that of the initial Inactivity timer). If a SCHC ACK is received before expiration of this timer, the SCHC Fragment sender retransmits any lost SCHC Fragments reported by the SCHC ACK, or if the SCHC ACK confirms successful reception of all SCHC Fragments of the last window, the transmission of the fragmented SCHC Packet is considered complete. If the timer expires, and no SCHC ACK has been received since the start of the timer, the SCHC Fragment sender assumes that the All-1 fragment has been successfully received (and possibly, the last SCHC ACK has been lost: this mechanism assumes that the retransmission timer for the All-1 fragment is long enough to allow several SCHC ACK retries if the All-1 fragment has not;been received by the SCHC Fragment receiver, and it also assumes that it is unlikely that several ACKs become all lost).

# Padding management {#Padding}


SCHC C/D and SCHC F/R operate on bits, not bytes. SCHC itself does not have any alignment prerequisite.
If the L2 below SCHC constrains the payload to align to some boundary, called L2 Words (for example, bytes),
SCHC will meet that constraint and produce messages with the correct alignement.
This may entail adding extra bits (called padding bits).

When padding occurs, the number of appended bits is strictly less than the L2 Word size.

Padding happens at most once for each Packet during SCHC Compression and optional SCHC Fragmentation (see {{Fig-IntroLayers}}).
If a SCHC Packet is sent unfragmented (see {{Fig-Operations-Pad}}), it is padded as needed.
If a SCHC Packet is fragmented, only the last fragment is padded as needed.


~~~~

A packet (e.g. an IPv6 packet)
         |                                           ^ (padding bits
         v                                           |       dropped)
+------------------+                      +--------------------+
| SCHC Compression |                      | SCHC Decompression |
+------------------+                      +--------------------+
         |                                           ^
         |   If no fragmentation                     |
         +---- SCHC Packet + padding as needed ----->|
         |                                           | (MIC checked
         v                                           |  and removed)
+--------------------+                       +-----------------+
| SCHC Fragmentation |                       | SCHC Reassembly |
+--------------------+                       +-----------------+
     |       ^                                   |       ^
     |       |                                   |       |
     |       +------------- SCHC ACK ------------+       |
     |                                                   |
     +--------------- SCHC Fragments --------------------+
     +--- last SCHC Frag with MIC + padding as needed ---+

        SENDER                                    RECEIVER


~~~~
{: #Fig-Operations-Pad title='SCHC operations, including padding as needed'}


Each technology-specific document MUST specify the size of the L2 Word.
The L2 Word might actually be a single bit, in which case at most zero bits of padding will be appended to any message, i.e. no padding will take place at all.


# SCHC Compression for IPv6 and UDP headers

This section lists the different IPv6 and UDP header fields and how they can be compressed.

## IPv6 version field

This field always holds the same value. Therefore, in the Rule, TV is set to 6, MO to "equal"
and CDA to "not-sent".

## IPv6 Traffic class field

If the DiffServ field does not vary and is known by both sides, the Field Descriptor in the Rule SHOULD contain a TV with
this well-known value, an "equal" MO and a "not-sent" CDA.

Otherwise (e.g. ECN bits are to be transmitted), two possibilities can be considered depending on the variability of the value:

* One possibility is to not compress the field and send the original value. In the Rule, TV is not set to any particular value, MO is set to "ignore" and CDA is set to "value-sent".

* If some upper bits in the field are constant and known, a better option is to only send the LSBs. In the Rule, TV is set to a value with the stable known upper part, MO is set to MSB(x) and CDA to LSB(y).

## Flow label field

If the Flow Label field does not vary and is known by both sides, the Field Descriptor in the Rule SHOULD contain a TV with this well-known value, an "equal" MO and a "not-sent" CDA.

Otherwise, two possibilities can be considered:

* One possibility is to not compress the field and send the original value. In the Rule, TV is not set to any particular value, MO is set to "ignore" and CDA is set to "value-sent".

* If some upper bits in the field are constant and known, a better option is to only send the LSBs. In the Rule, TV is set to a value with the stable known upper part, MO is set to MSB(x) and CDA to LSB(y).

## Payload Length field

This field can be elided for the transmission on the LPWAN network. The SCHC C/D recomputes the original payload length value. In the Field Descriptor, TV is not set, MO is set to "ignore" and CDA is "compute-IPv6-length".

If the payload length needs to be sent and does not need to be coded in 16 bits, the TV can be set to 0x0000, the MO set to MSB(16-s) where 's' is the number of bits to code the maximum length, and CDA is set to LSB(s).

## Next Header field

If the Next Header field does not vary and is known by both sides, the Field Descriptor in the Rule SHOULD contain a TV with
this Next Header value, the MO SHOULD be "equal" and the CDA SHOULD be "not-sent".

Otherwise, TV is not set in the Field Descriptor, MO is set to "ignore" and CDA is set to "value-sent". Alternatively, a matching-list MAY also be used.

## Hop Limit field

The field behavior for this field is different for Uplink and Downlink. In Uplink, since there is no IP forwarding between the Dev and the SCHC C/D, the value is relatively constant. On the other hand, the Downlink value depends of Internet routing and MAY change more frequently. One neat way of processing this field is to use the Direction Indicator (DI) to distinguish both directions:

* in the Uplink, elide the field: the TV in the Field Descriptor is set to the known constant value, the MO is set to "equal" and the CDA is set to "not-sent".

* in the Downlink, send the value: TV is not set, MO is set to "ignore" and CDA is set to "value-sent".

## IPv6 addresses fields

As in 6LoWPAN {{RFC4944}}, IPv6 addresses are split into two 64-bit long fields; one for the prefix and one for the Interface Identifier (IID). These fields SHOULD be compressed. To allow for a single Rule being used for both directions, these values are identified by their role (DEV or APP) and not by their position in the header (source or destination).

### IPv6 source and destination prefixes

Both ends MUST be synchronized with the appropriate prefixes. For a specific flow, the source and destination prefixes can be unique and stored in the context. It can be either a link-local prefix or a global prefix. In that case, the TV for the
source and destination prefixes contain the values, the MO is set to "equal" and the CDA is set to "not-sent".

If the Rule is intended to compress packets with different prefix values, match-mapping SHOULD be used. The different prefixes are listed in the TV, the MO is set to "match-mapping" and the CDA is set to "mapping-sent". See {{Fig-fields}}

Otherwise, the TV contains the prefix, the MO is set to "equal" and the CDA is set to "value-sent".

### IPv6 source and destination IID

If the DEV or APP IID are based on an LPWAN address, then the IID can be reconstructed with information coming from the LPWAN header. In that case, the TV is not set, the MO is set to "ignore" and the CDA is set to "DevIID" or "AppIID". Note that the
LPWAN technology generally carries a single identifier corresponding to the DEV. Therefore AppIID cannot be used.

For privacy reasons or if the DEV address is changing over time, a static value that is not equal to the DEV address SHOULD be used. In that case, the TV contains the static value, the MO operator is set to "equal" and the CDF is set to "not-sent".
{{RFC7217}} provides some methods that MAY be used to derive this static identifier.

If several IIDs are possible, then the TV contains the list of possible IIDs, the MO is set to "match-mapping" and the CDA is set to "mapping-sent".

It MAY also happen that the IID variability only expresses itself on a few bytes. In that case, the TV is set to the stable part of the IID, the MO is set to "MSB" and the CDA is set to "LSB".

Finally, the IID can be sent in extenso on the LPWAN. In that case, the TV is not set, the MO is set to "ignore" and the CDA is set to "value-sent".

## IPv6 extensions

No Rule is currently defined that processes IPv6 extensions. If such extensions are needed, their compression/decompression Rules can be based on the MOs and CDAs described above.

## UDP source and destination port

To allow for a single Rule being used for both directions, the UDP port values are identified by their role (DEV or APP) and not by their position in the header (source or destination). The SCHC C/D MUST be aware of the traffic direction (Uplink, Downlink) to select the appropriate field. The following Rules apply for DEV and APP port numbers.

If both ends know the port number, it can be elided. The TV contains the port number, the MO is set to "equal" and the CDA is set to "not-sent".

If the port variation is on few bits, the TV contains the stable part of the port number, the MO is set to "MSB" and the CDA is set to "LSB".

If some well-known values are used,  the TV can contain the list of these values, the MO is set to "match-mapping" and the CDA is set to "mapping-sent".

Otherwise the port numbers are sent over the LPWAN. The TV is not set, the MO is set to "ignore" and the CDA is set to "value-sent".

## UDP length field

The UDP length can be computed from the received data. In that case, the TV is not set, the MO is set to "ignore" and the CDA is set to "compute-length".

If the payload is small, the TV can be set to 0x0000, the MO set to "MSB" and the CDA to "LSB".

In other cases, the length SHOULD be sent and the CDA is replaced by "value-sent".

## UDP Checksum field {#UDPchecksum}

The UDP checksum operation is mandatory with IPv6 [RFC8200] for most
packets but recognizes that there are exceptions to that default
behavior.

For instance, protocols that use UDP as a tunnel encapsulation may
enable zero-checksum mode for a specific port (or set of ports) for
sending and/or receiving. [RFC8200] also stipulates that any node
implementing zero-checksum mode must follow the requirements specified
in "Applicability Statement for the Use of IPv6 UDP Datagrams with
Zero Checksums" [RFC6936].

6LoWPAN Header Compression [RFC6282] also authorizes to send UDP
datagram that are deprived of the checksum protection when an upper
layer guarantees the integrity of the UDP payload and pseudo-header
all the way between the compressor that elides the UDP checksum and
the decompressor that computes again it. A specific example of this is
when a Message Integrity Check (MIC) protects the compressed message
all along that path with a strength that is identical or better to
the UDP checksum.

In a similar fashion, this specification allows a SCHC compressor to
elide the UDP checks when another layer guarantees an identical or
better integrity protection for the UDP payload and the pseudo-header.
In this case, the TV is not set, the MO is set to "ignore" and the CDA is set to "compute-checksum".

In particular, when SCHC fragmentation is used, a fragmentation MIC
of 2 bytes or more provides equal or better protection than the UDP
checksum; in that case, if the compressor is collocated with the
fragmentation point and the decompressor is collocated with the
packet reassembly point, then compressor MAY elide the UDP checksum.
Whether and when the UDP Checksum is elided is to be specified in the
technology-specific documents.

Since the compression happens before the fragmentation, implementors
should understand the risks when dealing with unprotected data below
the transport layer and take special care when manipulating that data.



In other cases, the checksum SHOULD be explicitly sent. The TV is not set, the MO is set to "ignore" and the CDA is set to "value-sent".

# IANA Considerations

This document has no request to IANA.

# Security considerations

## Security considerations for SCHC Compression/Decompression
A malicious header compression could cause the reconstruction of a wrong packet that does not match with the original one. Such a corruption MAY be detected with end-to-end authentication and integrity mechanisms. Header Compression does not add more security problem than what is already needed in a transmission. For instance, to avoid an attack, never re-construct a packet bigger than some configured size (with 1500 bytes as generic default).

## Security considerations for SCHC Fragmentation/Reassembly
This subsection describes potential attacks to LPWAN SCHC F/R and suggests possible countermeasures.

A node can perform a buffer reservation attack by sending a first SCHC Fragment to a target.  Then, the receiver will reserve buffer space for the IPv6 packet.  Other incoming fragmented SCHC Packets will be dropped while the reassembly buffer is occupied during the reassembly timeout.  Once that timeout expires, the attacker can repeat the same procedure, and iterate, thus creating a denial of service attack.
The (low) cost to mount this attack is linear with the number of buffers at the target node.  However, the cost for an attacker can be increased if individual SCHC Fragments of multiple packets can be stored in the reassembly buffer.  To further increase the attack cost, the reassembly buffer can be split into SCHC Fragment-sized buffer slots. Once a packet is complete, it is processed normally.  If buffer overload occurs, a receiver can discard packets based on the sender behavior, which MAY help identify which SCHC Fragments have been sent by an attacker.

In another type of attack, the malicious node is required to have overhearing capabilities.  If an attacker can overhear a SCHC Fragment, it can send a spoofed duplicate (e.g. with random payload) to the destination. If the LPWAN technology does not support suitable protection (e.g. source authentication and frame counters to prevent replay attacks), a receiver cannot distinguish legitimate from spoofed SCHC Fragments.  Therefore, the original IPv6 packet will be considered corrupt and will be dropped. To protect resource-constrained nodes from this attack, it has been proposed to establish a binding among the SCHC Fragments to be transmitted by a node, by applying content-chaining to the different SCHC Fragments, based on cryptographic hash functionality.  The aim of this technique is to allow a receiver to identify illegitimate SCHC Fragments.

Further attacks MAY involve sending overlapped fragments (i.e. comprising some overlapping parts of the original IPv6 datagram). Implementers SHOULD make sure that the correct operation is not affected by such event.

In ACK-on-Error, a malicious node MAY force a SCHC Fragment sender to resend a SCHC Fragment a number of times, with the aim to increase consumption of the SCHC Fragment sender’s resources. To this end, the malicious node MAY repeatedly send a fake ACK to the SCHC Fragment sender, with a Bitmap that reports that one or more SCHC Fragments have been lost. In order to mitigate this possible attack, MAX_ACK_RETRIES MAY be set to a safe value which allows to limit the maximum damage of the attack to an acceptable extent. However, note that a high setting for MAX_ACK_RETRIES benefits SCHC Fragment reliability modes, therefore the trade-off needs to be carefully considered.

# Acknowledgements

Thanks to
Carsten Bormann,
Philippe Clavier,
Diego Dujovne,
Eduardo Ingles Sanchez,
Arunprabhu Kandasamy,
Rahul Jadhav,
Sergio Lopez Bernal,
Antony Markovski,
Alexander Pelov,
Charles Perkins,
Edgar Ramos,
Shoichi Sakane,
Pascal Thubert
and Juan Carlos Zuniga
for useful design consideration and comments.

--- back

# SCHC Compression Examples {#compressIPv6}


This section gives some scenarios of the compression mechanism for IPv6/UDP. The goal is to illustrate the behavior of SCHC.

The most common case using the mechanisms defined in this document will be a LPWAN Dev that embeds some applications running over CoAP. In this example, three flows are considered. The first flow is for the device management based
on CoAP using Link Local IPv6 addresses and UDP ports 123 and 124 for Dev and App, respectively.
The second flow will be a CoAP server for measurements done by the Device (using ports 5683) and Global IPv6 Address prefixes alpha::IID/64 to beta::1/64.
The last flow is for legacy applications using different ports numbers, the destination IPv6 address prefix is gamma::1/64.

 {{FigStack}} presents the protocol stack for this Device. IPv6 and UDP are represented with dotted lines since these protocols are compressed on the radio link.

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
Therefore, when such technologies are used, it is necessary to statically define an IID for the Link Local address for the SCHC C/D.

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
 |IPv6 DevIID     |64|1 |Bi|         | ignore | DevIID     ||      |
 |IPv6 APPprefix  |64|1 |Bi|FE80::/64| equal  | not-sent   ||      |
 |IPv6 AppIID     |64|1 |Bi|::1      | equal  | not-sent   ||      |
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
 |IPv6 DevIID     |64|1 |Bi|         | ignore | DevIID     ||      |
 |IPv6 APPprefix  |64|1 |Bi|[beta/64,| match- |mapping-sent||  [2] |
 |                |  |  |  |alpha/64,| mapping|            ||      |
 |                |  |  |  |fe80::64]|        |            ||      |
 |IPv6 AppIID     |64|1 |Bi|::1000   | equal  | not-sent   ||      |
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
 |IPv6 DevIID     |64|1 |Bi|         | ignore | DevIID     ||      |
 |IPv6 APPprefix  |64|1 |Bi|gamma/64 | equal  | not-sent   ||      |
 |IPv6 AppIID     |64|1 |Bi|::1000   | equal  | not-sent   ||      |
 +================+==+==+==+=========+========+============++======+
 |UDP DEVport     |16|1 |Bi|8720     | MSB(12)| LSB        || [4]  |
 |UDP APPport     |16|1 |Bi|8720     | MSB(12)| LSB        || [4]  |
 |UDP Length      |16|1 |Bi|         | ignore | comp-length||      |
 |UDP checksum    |16|1 |Bi|         | ignore | comp-chk   ||      |
 +================+==+==+==+=========+========+============++======+


~~~~
{: #Fig-fields title='Context Rules'}

All the fields described in the three Rules depicted on {{Fig-fields}} are present in the IPv6 and UDP headers.  The DevIID-DID value is found in the L2 header.

The second and third Rules use global addresses. The way the Dev learns the prefix is not in the scope of the document.

The third Rule compresses port numbers to 4 bits.


# Fragmentation Examples

This section provides examples for the different fragment reliability modes specified in this document.

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
{: #Fig-Example-Rel-Window-ACK-Loss title='Transmission in ACK-Always mode of an IPv6 packet carried by 11 fragments, with
MAX_WIND_FCN=6 and three lost fragments.'}

{{Fig-Example-Rel-Window-ACK-Loss-Last-A}} illustrates the transmission in ACK-Always mode of an IPv6 packet that needs 6 fragments, with MAX_WIND_FCN=6, three lost fragments and only one retry needed to recover each lost fragment.

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
{: #Fig-Example-Rel-Window-ACK-Loss-Last-A title='Transmission in ACK-Always mode of an IPv6 packet carried by 11 fragments,
with MAX_WIND_FCN=6, three lost fragments and only one retry needed for each lost fragment.'}

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
{: #Fig-Example-Rel-Window-ACK-Loss-Last-B title='Transmission in ACK-Always mode of an IPv6 packet carried by 11 fragments,
with MAX_WIND_FCN=6, three lost fragments, and the second ACK lost.'}

{{Fig-Example-Rel-Window-ACK-Loss-Last-C}} illustrates the transmission in ACK-Always mode of an IPv6 packet that needs 6 fragments, with MAX_WIND_FCN=6, with three lost fragments, and one retransmitted fragment lost again.

~~~~
           Sender                Receiver
             |-----W=0, FCN=6----->|
             |-----W=0, FCN=5----->|
             |-----W=0, FCN=4--X-->|
             |-----W=0, FCN=3--X-->|
             |-----W=0, FCN=2--X-->|
             |--W=0, FCN=7 + MIC-->|MIC checked: failed
             |<-----ACK, W=0-------|C=0 Bitmap:1100001
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
{: #Fig-Example-Rel-Window-ACK-Loss-Last-C title='Transmission in ACK-Always mode of an IPv6 packet carried by 11 fragments,
with MAX_WIND_FCN=6, with three lost fragments, and one retransmitted fragment lost again.'}


{{Fig-Example-MaxWindFCN}} illustrates the transmission in ACK-Always mode of an IPv6 packet that needs 28 fragments, with N=5, MAX_WIND_FCN=23 and two lost fragments. Note that MAX_WIND_FCN=23 may be useful when the maximum possible Bitmap size, considering the maximum lower layer technology payload size and the value of R, is 3 bytes. Note also that the FCN of the last fragment of the packet is the one with FCN=31 (i.e. FCN=(2^N)-1 for N=5, or equivalently, all FCN bits set to 1).

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
        |<------ACK, W=0-------|encoded Bitmap:1101111111111011
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
{: #Fig-Example-MaxWindFCN title='Transmission in ACK-Always mode of an IPv6 packet carried by 28 fragments, with N=5,
MAX_WIND_FCN=23 and two lost fragments.'}


# Fragmentation State Machines {#FSM}

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
  Clear lcl_bm       |   |  v set lcl_bm
       FCN=max value |  ++==+========+
                     +> |            |
+---------------------> |    SEND    |
|                       +==+===+=====+
|      FCN==0 & more frags |   | last frag
|    ~~~~~~~~~~~~~~~~~~~~~ |   | ~~~~~~~~~~~~~~~
|               set lcl_bm |   | set lcl_bm
|   send wnd + frag(all-0) |   | send wnd+frag(all-1)+MIC
|       set Retrans_Timer  |   | set Retrans_Timer
|                          |   |
|Recv_wnd == wnd &         |   |  
|lcl_bm==recv_bm &         |   |  +-----------------------+
|more frag                 |   |  | lcl_bm!=rcv-bm        |
|~~~~~~~~~~~~~~~~~~~~~~    |   |  | ~~~~~~~~~             |
|Stop Retrans_Timer        |   |  | Attempt++             v
|clear lcl_bm              v   v  |                +=====+=+
|window=next_window   +====+===+==+===+            |Resend |
+---------------------+               |            |Missing|
                 +----+     Wait      |            |Frag   |
not expected wnd |    |    Bitmap     |            +=======+
~~~~~~~~~~~~~~~~ +--->+               ++Retrans_Timer Exp  |          
    discard frag      +==+=+===+=+==+=+| ~~~~~~~~~~~~~~~~~ |
                         | |   | ^  ^  |reSend(empty)All-* |   
                         | |   | |  |  |Set Retrans_Timer  |
                         | |   | |  +--+Attempt++          |
MIC_bit==1 &             | |   | +-------------------------+
Recv_window==window &    | |   |   all missing frags sent
             no more frag| |   |   ~~~~~~~~~~~~~~~~~~~~~~
 ~~~~~~~~~~~~~~~~~~~~~~~~| |   |   Set Retrans_Timer                
       Stop Retrans_Timer| |   |    
 +=============+         | |   |
 |     END     +<--------+ |   |
 +=============+           |   | Attempt > MAX_ACK_REQUESTS
            All-1 Window & |   | ~~~~~~~~~~~~~~~~~~
             MIC_bit ==0 & |   v Send Abort
          lcl_bm==recv_bm  | +=+===========+
              ~~~~~~~~~~~~ +>|    ERROR    |
                Send Abort   +=============+


~~~~
{: #Fig-ACKAlwaysSnd title='Sender State Machine for the ACK-Always Mode'}


~~~~

 Not All- & w=expected +---+   +---+w = Not expected
 ~~~~~~~~~~~~~~~~~~~~~ |   |   |   |~~~~~~~~~~~~~~~~
 Set lcl_bm(FCN)       |   v   v   |discard
                      ++===+===+===+=+      
+---------------------+     Rcv      +--->* ABORT
|  +------------------+   Window     |
|  |                  +=====+==+=====+  
|  |       All-0 & w=expect |  ^ w =next & not-All
|  |     ~~~~~~~~~~~~~~~~~~ |  |~~~~~~~~~~~~~~~~~~~~~
|  |    set lcl_bm(FCN)     |  |expected = next window
|  |      send lcl_bm       |  |Clear lcl_bm
|  |                        |  |    
|  | w=expected & not-All   |  |
|  | ~~~~~~~~~~~~~~~~~~     |  |
|  |     set lcl_bm(FCN)+-+ |  | +--+ w=next & All-0
|  |     if lcl_bm full | | |  | |  | ~~~~~~~~~~~~~~~
|  |     send lcl_bm    | | |  | |  | expected = nxt wnd
|  |                    v | v  | |  | Clear lcl_bm
|  |w=expected& All-1 +=+=+=+==+=++ | set lcl_bm(FCN)
|  |  ~~~~~~~~~~~  +->+    Wait   +<+ send lcl_bm
|  |    discard    +--|    Next   |   
|  | All-0  +---------+  Window   +--->* ABORT  
|  | ~~~~~  +-------->+========+=++        
|  | snd lcl_bm  All-1 & w=next| |  All-1 & w=nxt
|  |                & MIC wrong| |  & MIC right      
|  |          ~~~~~~~~~~~~~~~~~| | ~~~~~~~~~~~~~~~~~~
|  |            set lcl_bm(FCN)| |set lcl_bm(FCN)
|  |                send lcl_bm| |send lcl_bm
|  |                           | +----------------------+
|  |All-1 & w=expected         |                        |
|  |& MIC wrong                v   +---+ w=expected &   |
|  |~~~~~~~~~~~~~~~~~~~~  +====+=====+ | MIC wrong      |
|  |set lcl_bm(FCN)       |          +<+ ~~~~~~~~~~~~~~ |
|  |send lcl_bm           | Wait End |   set lcl_bm(FCN)|
|  +--------------------->+          +--->* ABORT       |
|                         +===+====+=+-+ All-1&MIC wrong|
|                             |    ^   | ~~~~~~~~~~~~~~~|
|      w=expected & MIC right |    +---+   send lcl_bm  |
|      ~~~~~~~~~~~~~~~~~~~~~~ |                         |
|       set lcl_bm(FCN)       | +-+ Not All-1           |
|        send lcl_bm          | | | ~~~~~~~~~           |
|                             | | |  discard            |
|All-1&w=expected & MIC right | | |                     |
|~~~~~~~~~~~~~~~~~~~~~~~~~~~~ v | v +----+All-1         |
|set lcl_bm(FCN)            +=+=+=+=+==+ |~~~~~~~~~     |
|send lcl_bm                |          +<+Send lcl_bm   |
+-------------------------->+    END   |                |
                            +==========+<---------------+

       --->* ABORT
            ~~~~~~~
            Inactivity_Timer = expires
        When DWL
          IF Inactivity_Timer expires
             Send DWL Request
             Attempt++
                            
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
 |     ~~~~~~~~~~~~~~~~~~~~~~~|     |~~~~~~~~~~~~~~~~~
 |            set local-Bitmap|     |set local-Bitmap
 |      send wnd + frag(all-0)|     |send wnd+frag(all-1)+MIC
 |           set Retrans_Timer|     |set Retrans_Timer
 |                            |     |
 |Retrans_Timer expires &     |     |   lcl-Bitmap!=rcv-Bitmap
 |more fragments              |     |   ~~~~~~~~~~~~~~~~~~~~~~
 |~~~~~~~~~~~~~~~~~~~~        |     |   Attemp++        
 |stop Retrans_Timer          |     |  +-----------------+
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
|  |     All-0 empty +->+=+=+===+======+ clear lcl_Bitmap
|  |     ~~~~~~~~~~~ |    | |   ^
|  |     send bitmap +----+ |   |w=expct & not-All & full
|  |                        |   |~~~~~~~~~~~~~~~~~~~~~~~~
|  |                        |   |set lcl_Bitmap; w =nxt
|  |                        |   |     
|  |      All-0 & w=expect  |   |     w=next   
|  |      & no_full Bitmap  |   |    ~~~~~~~~  +========+
|  |      ~~~~~~~~~~~~~~~~~ |   |    Send abort| Error/ |
|  |      send local_Bitmap |   |  +---------->+ Abort  |
|  |                        |   |  | +-------->+========+
|  |                        v   |  | |   all-1       ^
|  |    All-0 empty    +====+===+==+=+=+ ~~~~~~~     |        
|  |  ~~~~~~~~~~~~~ +--+    Wait       | Send abort  |
|  |  send lcl_btmp +->| Missing Fragm.|             |
|  |                   +==============++             |
|  |                                  +--------------+
|  |                                   Uplink Only &
|  |                             Inactivity_Timer = expires
|  |                             ~~~~~~~~~~~~~~~~~~~~~~~~~~
|  |                              Send Abort
|  |All-1 & w=expect & MIC wrong     
|  |~~~~~~~~~~~~~~~~~~~~~~~~~~~~      +-+  All-1       
|  |set local_Bitmap(FCN)             | v  ~~~~~~~~~~  
|  |send local_Bitmap     +===========+==+ snd lcl_btmp
|  +--------------------->+   Wait End   +-+           
|                         +=====+=+====+=+ | w=expct &  
|       w=expected & MIC right  | |    ^   | MIC wrong
|       ~~~~~~~~~~~~~~~~~~~~~~  | |    +---+ ~~~~~~~~~
|  set & send local_Bitmap(FCN) | | set lcl_Bitmap(FCN)
|                               | |                    
|All-1 & w=expected & MIC right | +-->* ABORT          
|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ v                      
|set & send local_Bitmap(FCN) +=+==========+           
+---------------------------->+     END    |
                              +============+   
            --->* ABORT
                 Only Uplink
                 Inactivity_Timer = expires 
                 ~~~~~~~~~~~~~~~~~~~~~~~~~~
                 Send Abort                                                    
~~~~
{: #Fig-ACKonerrorRcv title='Receiver State Machine for the ACK-on-Error Mode'}


# SCHC Parameters {#SCHCParams}

This section lists the parameters that need to be defined in the LPWAN technology-specific documents.

* Most common uses cases, deployment scenarios

* Mapping of the SCHC architectural elements onto the LPWAN architecture

* Assessment of LPWAN integrity checking

* Various potential channel conditions for the technology and the corresponding recommended use of SCHC C/D and F/R

* Rule ID numbering scheme, fixed-sized or variable-sized Rule IDs, number of Rules, the way the Rule ID is transmitted

* Padding: size of the L2 Word (for most LPWAN technologies, this would be a byte; for some technologies, a bit)

* Decision to use SCHC fragmentation mechanism or not. If yes:

    * reliability mode(s) used, in which cases (e.g. based on link channel condition)

    * number of bits for FCN (N), for W (M) and for DTag (T)

    * support for interleaved packet transmission, to what extent

    * size of MIC and algorithm for its computation, if different from the default CRC32. Byte fill-up with zeroes or other mechanism, to be specified.

    * Retransmission Timer duration

    * Inactivity Timer duration

    * MAX_ACK_REQUEST value

* if L2 Word is wider than a bit and SCHC fragmentation is used, value of the padding bits (0 or 1). This is needed
because the padding bits of the last fragment are included in the MIC computation.

* Note on the "C-bit bump": When fragmenting in ACK-on-Error or ACK-Always mode, it is expected that the last window (called All-1 window) will not be
  fully utilised, i.e. there won't be fragments with all FCN values from MAX_WIND_FCN downto 1 and finally All-1.
  It is worth noting that this document does not mandate that other windows (called All-0 windows) are fully utilised either.
  This document purposely does not specify that All-1 windows use Bitmaps with the same number of bits as All-0 windows do.
  By default, Bitmaps for All-0 and All-1 windows are of the same size MAX_WIND_FCN + 1. But a technology-specific document
  MAY revert that decision. The rationale for reverting the decision could be the following: Note that the SCHC ACK sent as a
  response to an All-1 fragment includes a C bit that SCHC ACK for other windows don't have. Therefore, the SCHC ACK for the
  All-1 window is one bit bigger. An LPWAN technology with a severely constrained payload size might decide that this "bump" in
  the SCHC ACK for the last fragment is a bad resource usage. It could thus mandate that the All-1 window is not allowed to use
  the FCN value 1 and that the All-1 SCHC ACK Bitmap size is reduced by 1 bit. This provides room for the C bit without creating
  a bump in the SCHC ACK.


* Note on soliciting downlink transmissions: In some LPWAN technologies, as part of energy-saving techniques,
downlink transmission is only possible immediately after an uplink transmission.
In order to avoid potentially high delay in the downlink transmission of a fragmented SCHC Packet,
the SCHC Fragment receiver may want to perform an uplink transmission as soon as possible after reception of a SCHC
Fragment that is not the last one.
Such uplink transmission may be triggered by the L2 (e.g. an L2 ACK sent in response to a SCHC Fragment encapsulated
in a L2 PDU that requires an L2 ACK) or it may be triggered from an upper layer.

* the following parameters need to be addressed in documents other than this one but not forcely in
the LPWAN technology-specific documents:

    * The way the contexts are provisioned

    * The way the Rules as generated


# Note
Carles Gomez has been funded in part by the Spanish Government (Ministerio de Educacion, Cultura y Deporte) through the Jose
Castillejo grant CAS15/00336, and by the ERDF and the Spanish Government through project TEC2016-79988-P.  Part of his contribution to this work has been carried out during his stay as a visiting scholar at the Computer Laboratory of the University of Cambridge.
