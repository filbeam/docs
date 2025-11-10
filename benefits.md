# Benefits

In this page, we will look at the benefits of using Filecoin Beam for your data delivery from Filecoin Onchain Cloud Storage.

### For Filecoin Onchain Cloud clients & App developers

#### One URL for all retrievals

Without Filecoin Beam, data stored on each Filecoin SP is retrievable from a different URL. e.g. [yablu.net](http://yablu.net) and [https://pdp.zapto.org/](https://pdp.zapto.org/). Filecoin Beam consolidates all FWSS content under a single canonical URL -[ https://filbeam.io](https://filcdn.io) - removing integration complexity and enabling a standardized access experience across SPs. This lowers the barrier to adoption for developers and improves ecosystem composability.

#### Global Retrieval Performance

Retrieval latency without Filecoin Beam depends on the physical proximity of the user to the SP. Filecoin Beam introduces a globally distributed cache layer, significantly improving retrieval speeds by reducing cross-continental round trips. This infrastructure enables consumer-grade performance for apps built on Filecoin FWSS.

#### High Reliability

If you store files with a certain SP without Filecoin Beam included, then you are relying on the uptime and reliability of that SP to power your application. With Filecoin Beam on top, you can rely on the global network of nodes to serve and cache your data, and even locate a backup if certain SPs are down.

### For Filecoin SPs

#### Security from the Internet

Filecoin Beam acts as a protective perimeter for SPs, shielding them from unpredictable internet traffic, DDoS attacks, and automated scraping - effectively functioning as a firewall for Filecoin. This reduces operational risk and infrastructure load on SPs.

#### Incentivised Retrieval

A long-standing limitation in the Filecoin ecosystem has been the absence of retrieval incentives. Filecoin Beam addresses this by introducing incentivized earn-per-byte egress - rewarding SPs for serving data to Filecoin Beam and this solving the years-old retrieval incentives problem.&#x20;

#### Fully Trustless Filecoin SPs

Since we have innovated on how Filecoin Beam pays the Filecoin SPs for their egress, we are able to bootstrap a pay-per-byte protocol at this layer of the architecture. Filecoin Beam lays the foundation for a protocol-level pay-per-byte model, allowing SPs to be compensated in a trustless, verifiable way for data retrieval. I.e. If we can pay SPs per byte, then SPs can be trustless on both the storage and retrieval side of the Filecoin offering, with Filecoin Beam acting as the trust anchor. What’s more, other CDNs and gateways can then adopt the pay-per-byte protocol, fostering competition, decentralization, and resilience at the access layer.

\
