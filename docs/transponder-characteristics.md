# P4XT Transponder Characteristics
White Paper – May 24 2021
Wally Ritchie

## Introduction

### Purpose
This paper provides preliminary information regarding the expected characteristics of the P4XT Digital Muliplexed Transponder (DMT). The primary purpose is to support analysis associated with link budgets. It should also be useful as a primer to the architecture of the DMT, especially when used in conjunction with the P4XT Workshop Presentation available from the ORI site at https://www.youtube.com/watch?v=fCmzS6jBhHg&list=PLSfJ4B57S8Dk5bvXoJMnzkUXXheNBdIP1&index=2

Please find link budgets at https://github.com/phase4ground/documents/tree/master/Engineering/Link_Budget

### Scope
This document is preliminary and subject to change as the design progresses. Nevertheless, these characteristics are expected to be close to the final design with respect to link budgeting.

## Description

### General Architecture

The DMT is designed to provide a relay service in geosynchronous orbit for Amateur Radio Satellite service earth stations. The design is all digital including digital only uplinks and a single digital downlink. The onboard multiplexors provide the digital uplink data and pass it through the single downlink.
Although designed primarily for GEO, the DMT is also useful in HEO and other scenarios where symbol rates of around 0.4Mbaud to 8Mbaud are feasible. The DVB-S2 transmitter is useful over a wider range of symbol rates where different multiplexing schemes are more appropriate including non-relay applications.
It is important to recognize that this architecture is radically different from that of a convention bent pipe transporter, whether regenerative or not. The DMT is a receiver – multiplexor – transmitter chain with processing that determines whether data is forwarded from uplink to downlink. Except for some special modes to support transfer of digitized analog uplink waveforms, only digital transmissions are supported which may carry real-time voice/data and IP packets.

## Uplink

### Introduction

The receive subsystem has both narrowband and wideband multichannel receivers. We will focus on the narrowband receiver as this will be the one used by 99% or more or ordinary amateur radio stations. The digital uplink protocol is M17 protocol, with adaptations for space and wider bandwidth. The M17 protocol can be found here: https://m17-protocol-specification.readthedocs.io/en/latest/

## Isochronous Operation

The narrowband multipler operates isochronously at TBD frames per second. A jitter buffer is required at the receiver which introduces a fixed 20ns - 40ms delay. Each frame is TBD ms.

## Channelization

The basic narrowband channel spacing is expected to be 12.5 kHz with 160 channels. Each transmission must fit within the allocation. 

M17 standard uses 4FSK modulation running at 4800 symbols/s (9600 bits/s) with a deviation index h=0.33 for transmission in 9 kHz channel bandwidth. Channel spacing is 12.5 kHz.

The key parameters are the implementation loss of the demodulation scheme, its raw bit rate, and its occupied bandwidth.

## Error Correction

Two distinct forward error correction schemes are used for different parts of the transmission in M17. The Link Setup Frame (LSF) is convolutionally encoded using rate 1/2 coder with constraint K=5, and the subsequent frames are punctured and encoded using a Golay (24, 12) code. This is a very high-level description of the forward error correction scheme, but gives an idea of code coverage and complexity. 


## Framing

160 channels are provided occupying 2 MHz of uplink bandwidth. Note that wideband channels may occupy an additional 2MHz or more of bandwidth. (The bands are split in the front-end digital processing using FFTs). 160 channels can support 320 users in PTT mode and more than a thousand channels in NET mode. Essentially, idle stations are parked and gain the channel when the current occupant drops. This process is assisted by the protocols. Operation is also possible with 1 channel per user which supports 160 simultaneous users.

## Narrowband Multiplex (AI: Needs work) 

~~The NBM is transparent to the receivers as long as the basic parameters are met. Once the transmission is authenticated, the payload bits (e.g. 96) plus 2 label bits are passed to the NBM. The label bits allow the multiplex to signal primary or second data (e.g. voice data) and payload/control). This allows the NBM to insert common data e.g. telemetry and logs when the frame contains no data. The authenticated call sign is inserted in the control data in conformance with local administration (e.g. FCC) rules.
The NBM allocates channels as Nx800 payloads. Each 800 bits occupies 32 bits of the downlink. A 2400 bps channel requires three 32 bit words. Labels are outside the payload with 2 bits per channel packed into 32-bit words. This allows receivers to know the location and label for each channel to find the channels of interest. 160 active channels produce 15360 payload bits plus 320 label bits for 15680 data bits. Additional control bits define the channel format. ~~

Each channel has a dedicated data rate. Users must select channels of the desired bandwidth. This can change on the fly based on demand but not more than once per 25 frame superframe (1 second).

The NBM always produces N DVB-S2 BBFRAMES each 40 ms cycle. The number depends on the allocated narrowband bandwith (in bits). About 40 2400 bit channels can be accommodated per frame and they appear in the downlink as a UDP packet encapsulated with GSE. The use of UDP allows simulation of the multiplexor through the internet. 

## Downlink

### Introduction

The single downlink fully complies with the DVB-S2 standard. This is a complex standard with a myriad of configuration options. We use a subset tailored to our application. 

### Symbol Rates

The downlink operates at a fixed symbol rate, typically between 1 Mbaud and 8 Mbaud. The rate can be changed from time to time but this is not a hitless operation. The major tradeoff is symbol rate vs downlink CN. This depends primarily on transmit power and earth station antenna size.

### MODCOD

In DVB-S2 the modulation and coding can be changed frame to frame. This allows a mix of earth station performance with ¼ QPSK at the low end and 8PSK 8/9 at the upper end (for non-linear transmitters). It is desirable. The basic design parameters are how many BBFRAMES are allocated to the NBM per 40ms window. Remaining capacity is allocated to the wideband channels. 

### Frame Size

DVB-S2 allows 16200 or 64800 bits (not symbols) for BBFRAMES. For maximum flexibility we use only the short frame size. 

### Pilots

DVB-S2 allows each frame to include pilots or not. We always use pilot on to ease the burder on the receiver when skipping over BBFRAMES with MODCODs outside the earth station's capability. 

### Output Filters and Spectral Regroth
The output filter allows rollofs from .15 to .35. We normally use the largest setting. In addition, we must allow for spectrum regrowth when using non-linear output stages. This can be modelled as part of the implementation loss of the transmitter and is typically as much as 3dB. This is offset by the gain in DC efficiency.

