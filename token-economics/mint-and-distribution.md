---
cover: ../.gitbook/assets/Desktop-wallpaper.png
coverY: 369
---

# Mint & Distribution

The DePIN token, which we've named Solaxy (**SLX**) following a successful [naming contest](https://discord.com/channels/1128564139472736296/1135547222956720188)[ ](https://discord.com/channels/1128564139472736296/1135547222956720188)on the community Discord server, can be minted by anyone **using** a token bonding curve mechanism. This mechanism relies on an algorithmic approach embedded within a smart contract, where a reserve of DAI stablecoins backs the creation of SLX tokens. This reserve underpins the tokens' intrinsic value and security.

When users engage in minting SLX tokens, they contribute DAI to the reserve at a rate determined by the curve algorithm. On the other hand, when SLX tokens are burned, effectively reducing the token supply, DAI tokens from the reserve are returned to the burner at a rate determined by the curve algorithm. Both burning and minting play essential roles in achieving a balanced supply and demand.

_Solaxy_ will feature an uncapped token supply, intrinsically linked to its demand and price via the linear pricing function where slope; m = `0.0025` and intersect; b = `0`

$$
f(x) = mx + b
$$

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

This linear bonding curve guarantees equitable incentives for both early adopters and later participants within the M3tering protocol. The curve's slope remains consistent throughout the contract's lifespan, Furthermore, this design minimizes the typical price volatility associated with exponential curves while also redistributing some of the early adoption incentives inherent in logarithmic curves.&#x20;

To calculate the amount of DAI tokens required to mint a specific amount of SLX, simply compute the area under the linear function, which equates to the area of a trapezoid where both a and b are calculated based on `f(x)`; and `h` is the amount of SLX.&#x20;

$$
A = \frac{1}{2}\left ( a+b \right ) h
$$

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Combining these equations would give us&#x20;

$$
A = \left (h^2+2hx\right)\frac{m}{2}
$$

While the pricing function itself follows a linear model, it's worth noting that the area under the curve, and consequently the required collateral, increases exponentially. This unique characteristic is clearly shown in the graph below from actually simulating our function. explore these concepts and others for yourself using [https://bondingplayground.netlify.app/](https://bondingplayground.netlify.app/)

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Token Distribution

No tokens will be set aside for airdrops nor investors nor teams. This deliberate approach ensures the organic distribution of the token to contributors within the M3tering protocol in proportion to their contributions. However, no tokenomics feels complete without a trusty pie chart, so here's something for your viewing pleasure.

{% embed url="https://media.tenor.com/4dH0kbzBD9EAAAAd/simpsons-i-ate-some-pie.gif" %}
i 8 sum pi ... and it was delicious
{% endembed %}

{% @github-files/github-code-block %}

## Beyond Minting

Dive deep into the Burn and Exit mechanism and discover how it protects the ecosystem from MEV extractions and front running attacks

{% content-ref url="burn-and-exit-fee.md" %}
[burn-and-exit-fee.md](burn-and-exit-fee.md)
{% endcontent-ref %}
