---
title: "Decentraland Analytics"
last_modified_at: 2022-03-16
classes: wide
excerpt: "LAND price dynamics"
header:
  teaser: "/assets/portfolio/decentraland/sales_parcels.png"
---
<img src="/assets/portfolio/decentraland/sales_parcels.png" width="700"/>

Since its birth in 2015, Decentraland has been one of the most popular virtual reality (VR) platforms paired with NFTs. Within the platform, users can create, experience, and monetize their VR content and applications. Specifically, the finite, traversable, 3D virtual space is called _LAND_, an NFT asset maintained on the Ethereum blockchain, transactions of which remain traceable and immutable.

The 16m x 16m smallest unit of LAND is named a _parcel_, identified by cartesian coordinates (x,y). On each parcel, an owner can build a _scene_ by using design tools like the [Builder](https://builder.decentraland.org/){: target="_blank"} or the Decentraland SDK, creating and offering a parcel-specific virtual experience to other users. Initially, the values of both x and y spanned from -150 to 0 to 150, constituting 90,601 (i.e., 301x301) different parcels. Then the upper-right corner of LAND has been expanded by the AETHERIAN project, and, at the time of writing, there exist 92,598 parcels in total.

Using the [OpenSea API](https://docs.opensea.io/reference/api-overview){: target="_blank"}, I retrieved all the historical events (e.g., auctions, sales, transfers) of 92,598 parcels. Between 2018 and 2020, 244,695 transactions are retrieved, 6,107 transactions (i.e., 6.6%) of which were the actual sales. I first focus on those sales to study the determinants of virtual real estate prices.

For the analysis, I use the log of selling prices in USD (i.e., $$\log{(Selling \text{ } price)}$$) as a focal dependent variable. The figure below presents the decreasing trend of the monthly average of $$\log{(Selling \text{ } price)}$$.

<img src="/assets/portfolio/decentraland/sales_yearmonth.png" width="700"/>



                              PanelOLS Estimation Summary                           
    ================================================================================
    Dep. Variable:              LpriceUSD   R-squared:                        0.2280
    Estimator:                   PanelOLS   R-squared (Between):              0.2850
    No. Observations:                6107   R-squared (Within):               0.1666
    Date:                Wed, Mar 16 2022   R-squared (Overall):              0.2280
    Time:                        18:52:22   Log-likelihood                   -6485.1
    Cov. Estimator:             Clustered                                           
                                            F-statistic:                      56.062
    Entities:                        3284   P-value                           0.0000
    Avg Obs:                       1.8596   Distribution:                 F(32,6074)
    Min Obs:                       1.0000                                           
    Max Obs:                       23.000   F-statistic (robust):             43.163
                                            P-value                           0.0000
    Time periods:                      27   Distribution:                 F(32,6074)
    Avg Obs:                       226.19                                           
    Min Obs:                       91.000                                           
    Max Obs:                       602.00                                           
                                                                                    
                                 Parameter Estimates                              
    ==============================================================================
                Parameter  Std. Err.     T-stat    P-value    Lower CI    Upper CI
    ------------------------------------------------------------------------------
    const          6.1293     1.2678     4.8345     0.0000      3.6439      8.6148
    road          -1.0688     0.0887    -12.047     0.0000     -1.2427     -0.8949
    distRoad      -1.8280     0.1964    -9.3066     0.0000     -2.2130     -1.4429
    distPlaza      0.5261     0.0973     5.4060     0.0000      0.3354      0.7169
    distOrigin    -1.0467     0.0930    -11.252     0.0000     -1.2291     -0.8643
    payMANA        5.2092     1.2680     4.1082     0.0000      2.7235      7.6950
    payETH         4.1433     1.2743     3.2514     0.0012      1.6452      6.6415
    YM_2018-11    -0.2370     0.0409    -5.7994     0.0000     -0.3171     -0.1569
    YM_2018-12    -0.1219     0.0440    -2.7702     0.0056     -0.2081     -0.0356
    YM_2019-01    -0.1471     0.0451    -3.2627     0.0011     -0.2355     -0.0587
    YM_2019-02    -0.1812     0.0958    -1.8903     0.0588     -0.3691      0.0067
    YM_2019-03     0.0926     0.0415     2.2332     0.0256      0.0113      0.1739
    YM_2019-04    -0.1149     0.0475    -2.4175     0.0157     -0.2080     -0.0217
    YM_2019-05    -0.2146     0.0589    -3.6458     0.0003     -0.3299     -0.0992
    YM_2019-06    -0.3097     0.0525    -5.8935     0.0000     -0.4127     -0.2067
    YM_2019-07    -0.1098     0.0513    -2.1397     0.0324     -0.2104     -0.0092
    YM_2019-08    -0.0469     0.0730    -0.6428     0.5204     -0.1900      0.0962
    YM_2019-09    -0.1390     0.0529    -2.6291     0.0086     -0.2426     -0.0354
    YM_2019-10     0.0316     0.0502     0.6294     0.5291     -0.0669      0.1301
    YM_2019-11    -0.0546     0.0693    -0.7881     0.4306     -0.1903      0.0812
    YM_2019-12    -0.1225     0.0542    -2.2599     0.0239     -0.2287     -0.0162
    YM_2020-01    -0.0245     0.0908    -0.2699     0.7872     -0.2025      0.1535
    YM_2020-02     0.1003     0.0545     1.8401     0.0658     -0.0065      0.2071
    YM_2020-03     0.0188     0.0494     0.3816     0.7028     -0.0779      0.1156
    YM_2020-04    -0.0741     0.0708    -1.0469     0.2952     -0.2129      0.0647
    YM_2020-05    -0.1102     0.0514    -2.1428     0.0322     -0.2110     -0.0094
    YM_2020-06    -0.0676     0.0657    -1.0288     0.3036     -0.1964      0.0612
    YM_2020-07     0.0317     0.0560     0.5667     0.5710     -0.0781      0.1416
    YM_2020-08    -0.3091     0.0975    -3.1698     0.0015     -0.5003     -0.1179
    YM_2020-09    -0.4392     0.0512    -8.5798     0.0000     -0.5395     -0.3388
    YM_2020-10    -0.3550     0.1094    -3.2442     0.0012     -0.5695     -0.1405
    YM_2020-11    -0.2674     0.1254    -2.1315     0.0331     -0.5133     -0.0215
    YM_2020-12    -0.5890     0.0451    -13.046     0.0000     -0.6775     -0.5005
    ==============================================================================
