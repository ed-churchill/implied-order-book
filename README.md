The initial aim of this project is to use SPX options pricing data from CBOE to construct a _naive_ implied order book for the underlying asset. We do this by calculating the Gamma Exposure (GEX) of market makers which allows us to predict delta-hedging flows. More detail below: 

# Gamma Exposure (GEX)
We want to know, at each index level, how much gamma are the market makers sitting on, as this will give us a picture of how they must delta hedge. To do this, we need to know how many of each particular option the market makers hold AND whether they're long or short each option. We can know the first thing from open interest but for the second thing the data doesn't tell us whether they're long or short (they keep this secret of course!).

So a naive assumption to start with is that the dealers are long calls and short puts, which on the whole index isn't an entirely terrible assumption. We could use actual trades data to construct a more accurate picture of whether they're long or short (based on where trades are being closed relative to the bid, mid, ask can tell you whether the trade is speculative or a hedge for example.). However, this comes at a big cost and isn't easy so for now we'll just assume that the dealers are long calls and short puts.

Recall that long options have positive gamma and short options have negative gamma. So our assumption of long calls and short puts mean that in this case, calls carry positive gamma and puts negative.

The first thing to do is calculate the dollar gamma contribution of each option. This gives the option's DOLLAR CHANGE IN DELTA per ONE POINT MOVE in the index. The formula is (negate for puts):

    Option Gamma * Contract Size * Open Interest * Index Price
    
However, we actually want how much dollar delta changes per 1% MOVE IN THE INDEX, not 1 point. Since a 1% move corresponds to a point change of 0.01 * Index Price, we can simply multiply our first formula by this giving:

    Option Gamma * Contract Size * Open Interest * (Index Price)^2 * 0.01

Finally, we sum gamma contributions across all of the options to give us the total Gamma Exposure (GEX) of the dealers.

* If GEX is positive, dealers are LONG GAMMA, so if the index moves up 1%, then delta also goes up so dealers need to SELL the underlying to delta hedge. If the index moves down 1%, then delta also goes down so dealers need to BUY the underlying to delta hedge.
* If GEX is negative, dealers are SHORT GAMMA, so if the index moves up 1%, then delta goes down so dealers need to BUY the underlying to delta hedge. If the index moves down 1%, then delta goes up so dealers need to SELL the underlying to delta hedge.

If we calculate GEX at various strikes, there will be some strike price where dealers go from positve GEX to negative GEX. We can compare where the index is currently trading with this strike. Then:
* If the index is trading in the positive gamma zone, we are in a **volatility stabilizing** market and dealers are supplying liquidity to the market since their hedging flows are always in the opposite direction to the way the market is moving. Why is this? Because dealers are long gamma. So:
    * If the underlying market moves up, delta moves up, so dealers must sell the underlying, which pushes the price of the underlying back down and keeps it stable.
    * If the underlying market moves down, delta moves down, so dealers must buy the underlying, which pushes the price of the underlying back up and keeps it stable.

* If the index is trading in the negative gamma zone, we are in a **volatility destabilizing** market and dealers are taking liquidity from the market since their hedging flows are always in the same direction to the way the market is moving. Why is this? Because dealers are short gamma. So:
    * If the underlying market moves up, delta moves down, so dealers must buy the underlying, which pushes the price of the underlying up even further, which is destabilising.
    * If the underlying market moves down, delta moves up, so dealers must sell the underlying, which pushes the price of the underlying down even further, which is destabilising.
