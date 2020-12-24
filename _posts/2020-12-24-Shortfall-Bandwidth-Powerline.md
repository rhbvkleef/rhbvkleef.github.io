---
layout: post
title: Shortfall of Bandwidth with Powerline Ethernet Adaptor
---

# Introduction

In [episode 34 of the Self-Hosted podcast][self-hosted-34], one of the hosts
(Chris) talks about his experiences from using a
[TP-Link AV1000][av1000-datasheet] powerline adaptor in an RV, on the RV's DC
power bus, wherein he notes that whilst his adaptors are rated for gigabit
speeds, he is only able to achieve around 300mbps.
There are a multitude of factors which might impact this performance, and this
article will explore some of them in an attempt to explain why this shortcoming
exists, and how it might be mitigated.

# HomePlug AV2

The adaptor used and linked does indeed appear to advertise supporting gigabit
speeds, but in actuality, this only refers to the fact that the ethernet plug
on the adaptors support [IEEE 802.3ab][ieee802.3ab].
The actual carrier protocol used is [HomePlug AV2][av2] (linking to Wikipedia
as I cannot access their sources, which would be extremely useful to take away
the doubt hereafter stated), which appears to be a single-duplex 1300mbps
protocol.
Due to overhead, the actual speed one would see is approximately 1000mbps (in
single-duplex) for normal TCP/IP loads, or 500mbps in full duplex.

# Interference from Solar Installations

HomePlug AV2 can choose from a wide spectrum of frequencies, anywhere between
2MHz and 86 MHz (with the bands between 30MHz and 86MHz added by AV2 since
AV1).
According to [Pieter-Tjerk de Boer in his technical notes on wide-spectrum
radio observations during and near a solar eclipse
][ptdeboer-technotes-eclipse], a significant reduction in noise is observed
in the radio spectrum between 0MHz and 29Mhz.
According to another [article posted on the website of VERON on disruptions
in the frequency spectrum due to solarpanels][veron-pv-article], significant
interference is observed between 100KHz and 450MHz.
This article also writes about how a significant amount of interference can
directly be attributed to the use of a non-compliant power-optimizer.
We can clearly see that both sources observe interference in frequencies that
overlap with frequencies used by HomePlug AV2, but we cannot even use this as
an indication that this would cause interference for HomePlug devices, as
the solar installation we are looking at has significant differences to the
installations observed in these articles.
Additionally, Chris stated in a Telegram chat room that he (quite obviously)
connected his solar panels directly to a charge controller that charges a
battery, which somewhat isolates his power network to which he has connected
his powerline adaptors, and the solar panel and it's support circuitry.
This might negate the problem either significantly or entirely.

# Unconventional Circumstances

*PLACEHOLDER conversation on how an inverter might impact -stuff-*

# Conclusion

* Gigabit is single-duplex
  * Measure with full-duplex and single-duplex loads
* Solar might interfere
  * Measure with solar disconnected
* Inverter might be problematic
  * Measure on conventional power system

# Additional notes

In the podcast, Chris discusses his wish to be able to use multiple powerline
ethernet adaptors.
According to what I can read on Wikipedia, IEEE 1901 (which HomePlug AV2 is a
part of) requires certain interoperability.
Whilst I cannot guarantee it will work, and actual experience would only be
able to provide a definitive answer, this appears to be a strong indicator that
HomePlug AV2 adaptors are indeed able to work in multiple pairs.
Whether two different sets of plugs can interoperate is not talked about, and
might be a vendor-specific feature.

# Sources

* [Jupiter Broadcasting: Self-Hosted episode 34, Take Powerline Seriously][self-hosted-34]
* [TP-Link: Datasheet for the AV1000 Gigabit Powerline Starter Kit][av1000-datasheet]
* [IEEE: 802.3ab][ieee802.3ab]
* [HomePlug AV2][av2] (Linking wikipedia as I don't have access to their sources)
* [VERON: Storingen in het frequentie spectrum door zonnepanelen (PV installaties) in Amersfoort (Dutch)][veron-pv-article]
* [Pieter-Tjerk de Boer: Radio observations of the 2015 solar eclipse][ptdeboer-technotes-eclipse]

[self-hosted-34]: https://www.jupiterbroadcasting.com/143672/take-powerline-seriously-self-hosted-34/
[av1000-datasheet]: https://static.tp-link.com/2018/201804/20180428/TL-PA7010%20KIT(US)3.0.pdf
[ieee802.3ab]: https://ieeexplore.ieee.org/document/798775
[av2]: https://en.wikipedia.org/wiki/HomePlug
[veron-pv-article]: https://www.veron.nl/vereniging/commissies-en-werkgroepen/emc-emf/storingen-en-ontstoren-in-de-praktijk/storing-door-pv-installaties-amersfoort/
[ptdeboer-technotes-eclipse]: http://www.pa3fwm.nl/signals/eclipse2015/

