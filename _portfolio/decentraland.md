---
title: "Decentraland Analytics"
last_modified_at: 2022-03-16
classes: wide
excerpt: "LAND price dynamics"
header:
  teaser: "/assets/images/lands_181920-th.png"
---
<img src="/assets/images/lands_181920.png" alt="surface-graph" width="700"/>

Since its birth in 2015, Decentraland has been one of the most popular virtual reality (VR) platforms paired with NFTs. Within the platform, users can create, experience, and monetize their VR content and applications. Specifically, the finite, traversable, 3D virtual space is called _LAND_, an NFT asset maintained on the Ethereum blockchain, transactions of which remain traceable and immutable.

The 16m x 16m smallest unit of LAND is named a _parcel_, identified by cartesian coordinates (x,y). On each parcel, an owner can build a _scene_ by using design tools like the [Builder](https://builder.decentraland.org/){: target="_blank"} or the Decentraland SDK, creating and offering a parcel-specific virtual experience to other users. Initially, the values of both x and y spanned from -150 to 0 to 150, constituting 90,601 (i.e., 301x301) different parcels. Then the upper-right corner of LAND has been expanded by the AETHERIAN project, and, at the time of writing, there exist 92,598 parcels in total.

Using the [OpenSea API](https://docs.opensea.io/reference/api-overview){: target="_blank"}, I retrieved all the historical events (e.g., auctions, sales, transfers) of parcels. Between 2018 and 2020, 244,695 transactions of 92,598 parcels are retrieved. In particular, 6,107 transactions (i.e., 6.6%) were the actual sales. I first focus on those sales to study the determinants of virtual real estate prices.