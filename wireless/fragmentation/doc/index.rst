:orphan:

IEEE 802.11 Fragmentation
=========================

Goals
-----

.. by default packets that are too big are fragmented to smaller sizes
   which are better for transmitting but it can be used for better
   adaption to noisy environments or large distances? does this make sense?
   maybe fragmentation should be with large packets fragmented to reasonable size

.. -large packets are fragmented, because sending too large packets is more error-prone
   -so the mac fragments by default
   -the fragmentation can be tailored to the situation
   -if there is lots of interference, large distances, noise, etc.
   -lots of nodes

Transmitting large packets is more prone to packet errors than transmitting smaller ones. Fragmenting large packets to smaller pieces can increase 802.11 performance in the presence of interference, noise, or in a network with large distances.

The example simulation for this showcase demonstrates 802.11 MAC-level fragmentation.

| INET version: ``4.0``
| Source files location: `inet/showcases/wireless/fragmentation <https://github.com/inet-framework/inet-showcases/tree/master/wireless/fragmentation>`__

About 802.11 Fragmentation
--------------------------

Fragmentation in 802.11 reduces packet errors in certain situations, e.g. in a noisy environment. A larger packet has a higher chance of getting corrupted than a smaller one. To an extent, the error correcting mechanisms can correct some erroneous bits in a received frame, but above a certain bit error rate the error correcting mechanism becomes ineffective (below that threshold it corrects all bit errors).

.. TODO: actually, it has the same chance for corrupted bits. it is just less corrupted bits and it can be corrected.

However, fragmentation increases overhead, and thus decreases throughput and channel utilization.
The smaller packets resulting from fragmentation each have PHY and MAC headers, and each packet
transmission might be followed by a contention period and ACK (depending on block ack policy and TXOP)

In INET, 802.11 fragmentation is controlled by fragmentation policies in :ned:`Ieee80211Mac`. By default, the :ned:`Dcf` and :ned:`Hcf` submodules of :ned:`Ieee80211Mac` contain a fragmentation policy submodule (at ``mac.dcf.originatorMacDataService.fragmentationPolicy`` and ``mac.hcf.originatorMacDataService.fragmentationPolicy``). The default fragmentation policy type is :ned:`BasicFragmentationPolicy`.

:ned:`BasicFragmentationPolicy` has just one parameter, :par:`fragmentationThreshold`. Frames larger than this value are fragmented. The value includes the MAC header and MAC trailer.
The MAC overhead (header + trailer) is 28 bytes with DCF and 30 bytes with HCF.
The default for this parameter is 1500 Bytes.

For example, there is a 1000-byte packet created by the UDP application. It goes into the MAC as a 1036-byte packet (8B LLC header + 20B IPv4 header + 8B UDP header + 1000B data). The fragmentation threshold is set to 400 bytes. In the DCF (non-Qos) case, the following fragments come out of the MAC:

- 400B fragment: 24B MAC header + 8B LLC header + 20B IPv4 header + 8B UDP header + 336B data + 4B MAC trailer
- 400B fragment: 24B MAC header + 372B data + 4B MAC trailer
- 320B fragment: 24B MAC header + 264B data + 4B MAC trailer

The maximum number of fragments for packet is 16.

.. note:: :ned:`Ieee80211Mac` has a maximum transfer unit parameter (:par:`mtu`), which controls IP level
   fragmentation on the interface the MAC is part of, i.e. it tells the ``Ip`` module how to fragment packets going out on that interface. In contrast, the
   fragmentation policy controls MAC level fragmentation. If the mtu is set lower than the fragmentation threshold, MAC level fragmentation doesn't take effect, as packets are already fragmented by the IP module to smaller pieces than the MAC fragmentation threshold when they arrive at the MAC.

The Model
---------

The example simulation contains two wireless nodes. One of them sends large UDP packets to the other via Wifi in a noisy environment. The simulation will be run with fragmentation turned off and with it turned on. We'll examine the effect of fragmentation on the Wifi performance. We'll also see how TXOP and block acknowledgements affect the performance.

.. -keywords: progression, how to increase throughput, baseline without fragmentation, hcf enhancements like txop, and block ack
.. turn on frag, switch to hcf, turn on block ack

The example simulation uses the following network:

.. figure:: network3.png
   :width: 90%
   :align: center

The network contains two :ned:`AdhocHost`'s named ``wifiHost1`` and ``wifiHost2``. It also contains an :ned:`Ipv4NetworkConfigurator`, an :ned:`IntegratedVisualizer`, and an :ned:`Ieee80211ScalarRadioMedium` module.

Configuration keys for the UDP traffic, the radio and the radio medium are defined in the ``General`` configuration in :download:`omnetpp.ini <../omnetpp.ini>`.

``wifiHost1`` sends 2000-byte UDP packets to ``wifiHost1`` every 0.5 seconds, corresponding to about 32 Mbps of application-level traffic. The hosts operate in 802.11g mode with 54 Mbps data rate, so the channel is saturated.
Here is the traffic configuration in :download:`omnetpp.ini <../omnetpp.ini>`:

.. literalinclude:: ../omnetpp.ini
   :start-at: wifiHost1.numApps
   :end-at: wifiHost2.app[0].localPort
   :language: ini

We make the environment noisy by lowering the transmission power, so the relative amount of noise compared to the transmission power increases. We also make the receivers more sensitive. Here is the configuration in :download:`omnetpp.ini <../omnetpp.ini>`:

.. literalinclude:: ../omnetpp.ini
   :start-at: energyDetection
   :end-at: power
   :language: ini

We want to demonstrate that fragmenting packets is advantageous in a noisy environment, so we set
the transmission power accordingly. If there is too much noise, the fragmentation doesn't make
any difference, as communication becomes impossible. If there is too little noise, the benefit of fragmentation might not be evident, it might even be disadvantageous.

.. ``V1.1`` The simulation will be run with four scenarios, each aiming to improve performance in the noisy channel. They are defined in the following configurations:

.. ``V1.2``

The simulation will be run with four scenarios. Each scenario aims to improve the noisy channel performance of the previous one. They are defined in the following configurations:

.. TODO: there is a progression

- ``DCFnofrag``: The MAC uses DCF, and there is no fragmentation. It is expected that only a few of the large packets will be received successfully, throughput will be low.
- ``DCFfrag``: The MAC uses DCF, and packets are fragmented. Due to the smaller fragments, packet error rate should decrease, and throughput should increase compared to the previous scenario.
- ``HCFfrag``: The MAC uses HCF, packets are fragmented and trasmitted during a TXOP. Throughput should increase even more, as the sender node doesn't have to contend for channel access before transmitting each packet.
- ``HCFfragblockack``: This is the same as the previous scenario, but block acknowledgments are enabled. Throughput should increase yet again, as the receiver node doesn't have to ACK each packet individually.

The simulations will be run for 10 seconds, and we'll examine the number of packets received by the UDP application of ``wifiHost2``, and the application level throughput (the many small packets resulting from fragmentation doesn't affect the number of packets received by the UDP application, because they are defragmented by the MAC before they arrive at the UDP app).

.. TODO: or just use amount of data

.. TODO: we'll also see in what circumstances fragmentation is effective

Let's look at the details of the four configurations. Here is ``DCFnofrag`` from :download:`omnetpp.ini <../omnetpp.ini>`:

.. literalinclude:: ../omnetpp.ini
   :start-at: DCFnofrag
   :end-at: fragmentationThreshold
   :language: ini

It just sets a fragmentation threshold higher than the packet size to avoid fragmentation.

In the next configuration, ``DCFfrag``, we turn on fragmentation. Here is the configuration in :download:`omnetpp.ini <../omnetpp.ini>`:

.. literalinclude:: ../omnetpp.ini
   :start-at: DCFfrag
   :end-at: fragmentationThreshold
   :language: ini

It sets a fragmentation threshold value smaller than the packet size. Packets will be fragmented to 250-byte pieces. There are a lot of small packets, each followed by an ACK, and a contention period.

We examine how performance is affected if the small packets are sent during a TXOP, so the transmitter MAC doesn't have to contend for the channel after transmitting each packet.
In the next configuration, ``HCFfrag``, the MAC will use HCF instead of DCF, and take advantage of video priority TXOP when transmitting packets. Here is the configuration in :download:`omnetpp.ini <../omnetpp.ini>`:

.. literalinclude:: ../omnetpp.ini
   :start-at: HCFfrag
   :end-at: msduAggregationPolicy
   :language: ini

It sets the same fragmentation threshold as the previous configuration. QoS is enabled (so the MAC uses HCF). A classifier is added to sort packets into the video priority category. Frame aggregation is turned off, because we don't want to aggregate our fragments into a larger frame.

Packets are transmitted without contention during the TXOP, but all of them are ACKed individually.
In the final configuration, ``HCFfragblockack``, we extend the previous configuration and turn on block acks, so a group of packets can be ACKed.

When block acks are enabled, the sender node sends a block ack request after the transmission of a certain number of packets. The receiver node then sends a block ack, which contains information about which packets were received correctly, and which weren't. The incorrectly received ones can be retransmitted by the sender.

Here is the configuration:

.. literalinclude:: ../omnetpp.ini
   :start-at: HCFfragblockack
   :end-at: isBlockAckSupported
   :language: ini

Results
-------

Here are the results from the simulations:

.. figure:: numberofpackets5.png
   :width: 100%
   :align: center

.. figure:: throughput.png
   :width: 100%
   :align: center

As expected, the performance is low when packets are not fragmented. The performance improves considerably (with a factor of 20) when fragmentation is turned on. In the DCF fragmentation case, the time is dominated by contention periods. The use of TXOP remedies this, and there is a two-fold increase in performance. The use of block ACKs brings another two-fold performance increase.

.. TODO: more detailed analysis ? how to further improve? turn up the tx power and turn off frag

.. the parameter study result charts

.. TODO: it should be about 13.5 Mbps if there wasnt noise

.. TODO: it actually shows the domain of usefulness for the fragmentation

Further Analysis
~~~~~~~~~~~~~~~~

We ran some parameter studies to examine the domain of effectiveness for fragmentation.
We based the parameter studies on the above simulations. We examined the average application level throughput.
(The noise level, transmission power, and other settings are the same as in the above configurations, unless otherwise noted.)

.. TODO: the noise and other settings stay the same

First, the packet size was the iteration variable (going from 100B to 2500B), with a constant fragmentation threshold of 250 bytes:

.. figure:: onlypacketsize.png
   :width: 100%
   :align: center

Performance decreases with packet size in the ``DCFnofrag`` case, as larger packets have more chance for becoming corrupted. In the cases where fragmentation is enabled, the throughput follows a similar curve. At small packet sizes, the 802.11 overhead is significant. As the packet size increases above 1000B, throughput doesn't change substantially. The difference in magnitude between the three curves of ``DCFfrag``, ``HCFfrag``, and ``HCFfragblockack`` is due to the improvements of each configuration: the use of TXOP increases throughput, and the use of TXOP + block acks increases it further.

Secondly, the fragmentation threshold was iterated, using a 2000-byte packet size:

.. figure:: threshold.png
   :width: 100%
   :align: center

In the ``DCFnofrag`` case, the fragmentation threshold is set to 2250B to avoid fragmentation. The throughput value for the ``DCFnofrag`` case is on the right of the chart, and it's the lowest value.
The other three curves are similar in this case as well, with the difference in magnitude attributed to the use of TXOP and TXOP + block ack. The smaller fragments have more chance of being successfully received.

.. TODO: some more?

Next, the packet size was iterated, but on a wider range (from 100B to 6000B). The MAC can fragment packets to a maximum of 16 fragments. Due to the larger packets, the fragmentation threshold was set to around 1/16th of the packet size (taking MAC headers into account), so that the MAC always fragments packets to 16 pieces.

.. figure:: packetsize.png
   :width: 100%
   :align: center

``DCFnofrag`` has an advantage when packets are small, since fragmenting small packets to 16 fragments entails a lot of overhead. Otherwise, the three curves where fragmentation is enabled are similar, and the difference in magnitude is attributed to the use of TXOP and TXOP + block ack.

Then, the transmission power was iterated to examine performance at different noise levels:

.. figure:: snir.png
   :width: 100%
   :align: center

It is apparent that in this scenario, the domain in which fragmentation is actually useful is a very small range. Above a certain SNIR threshold (where relative noise becomes low enough), the fragmentation decreases performance, and below another threshold it doesn't make any difference. The range of SNIR difference when fragmentation is advantageous is about 50%. TODO: does that make sense?

.. note:: The parameter study configurations are defined in :download:`parameterstudy.ini <../parameterstudy.ini>`. The charts are available in ``ParameterStudy.anf``.

Sources: :download:`omnetpp.ini <../omnetpp.ini>`, :download:`FragmentationShowcase.ned <../FragmentationShowcase.ned>`

Discussion
----------

Use `this <https://github.com/inet-framework/inet-showcases/issues/TODO>`__ page in the GitHub issue tracker for commenting on this showcase.
