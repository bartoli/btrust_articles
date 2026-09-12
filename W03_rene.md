# Rene Pickhardt's contributions to the Lightning Network

When starting to explore [clboss source code](https://github.com/ksedgwic/clboss) this month, i discovered that [Core Lightning](https://github.com/ElementsProject/lightning) contains components named renepay and askrene. Those implement implementation of [Pickhardt payments](https://arxiv.org/abs/2107.05322), researched by Rene Pickhard and Stefan richter. It's not often that your contributuon to one domain is big enough that reknowned projects name parts of their code after you!<br>
This article is my attempt to explain a few of the big things Rene's researche brought to the Lightning Network, and give my personal recognition for his work.

## #ZeroBaseFee
When a Lightning node route payments from other nodes (Multi Path Payments), they are able to ask a fee in exchange for moving the liquidity in their channels for that payment. The fee calculation is based on tww parameters of their outpund channel for that payment:
- base fee : amount paid for any payment that goes this channel as outbound. It will be the same amount regardless of the value of the payment that is routed
- fee rate : this parameter generates a fee proportional to the amopunt being moved.
Any node can use any value they want for those two parameters for each of their channel, depending on the fee strategy they want.

Then when someone wants to do a payment to another Lightning node they don't have a channel with, they will have to find possible / cheap routes between nodes for that payments, by using those fee parameters to estimate the cost of each path.<br>
Here, it's intuitive that calculating the cost of each channel based on 2 parameters is slightly mor eexpensive than from a single parameter, but the difference is not substantial.
Now, imagine you are trying to do a very big payment (wich would have more chances to fail), and you also want to split it into multiple small payments to increase the chances of success. Now, you also have to find, at the same time:
- how to split the amount into smaller amounts for good cost and chances of success
- AND what path to take for each of those split payments.
So, it becomes obvious that the calculation is complex, so adding some parameters to it now can complexifies the caclulation a lot more. Rene's research expressed how algorithms suddently become a lot harder to solve when there is a base fee to take into account.

You might also have figured another obvious reason : if each payment has a base fee, then splitting a big payment into multiple smaller ones to increase it's chances of success also quickly makes it uneconomical to split it too much..
LND nodes for example, used to have a non null base fee of 1 sat, so the problem was already quite widespread along the newtork.

So out of Rene's research, came the suggestion that nodes should aim to set their base fee to 0, to encourage split payments as a way to increase payment's chance of success. At that time, the #zerobasefee hashtag went live.

## Payment valves
In hydraulic systems, a valve is an apparatus able to direct, regulate or control the flow of a fluid. By using valves, you can provide optimization or protection to the flows of your system.

In the lightning network, you know a channel has a given capacity, because this is announced by the nodes when they open it. But, you cannot know how is liquidity distributed between the two peers that share that channel. So you can't know if you will be able to use them for a multi-path payment before trying.

And for a node operator, they might also want to make sure all their liquidity does not go away after a few very big routed payments, by forcing a maximum size for the payments they want to route. 

Rene argues that a feature that already exists in the Lightning Network, but is particularly unused, can act as a valve to improve the reliability of Lightning payments : **htlc_maximum_msat**

*htlc_maximum_msat* is one of the parameters node operators can configure for each of their channels. It defines what is the maximum value they accept to route through this channel when it is used as the outbound channel of a routing hop. It must not be confused with maximum htlc 'count', which is the maximum number of concurrent ongoing htlcs a channel can have.

And what it can bring is really easy to understand, even without requiring to have a complex strategy : imagine you set the maximum htlc size for a channel to some value that is always less than the current balance of the channel. Now, there is no way another node will ask you to route a payment that is not possible because you wouldn't have enough liquidity on your side of the channel!

Obviously, a difficulty of this is that, unlike channel fee parameters, you can't just choose a fee value once per channel. You will need to have some kind of service that will regularly updates the maximum htlc size

### Personal return of experience
I use [scripts](https://github.com/bartoli/lnshortcut/blob/master/lnd_chan_mgr.py) for managing channel policies of the channels of [my routing node](https://amboss.space/node/02c521e5e73e40bad13fb589635755f674d6a159fd9f7b248d286e38c3a46f8683), setting max_htlc_msat value, among other fee policy parameters.
The scripts run regularly (but not too often because you can be banned by nodes if you 'spam' too much gossip messages to propagate your new channel settings), and adapt to the current state of the channels or the rest of the network.

One of the first thing that i noticed, when i started setting max_htlc_msat, was in the logs of my LND node: The log was previously filled with 'Insufficient balance' errors because of all the nodes trying to route payments through my depleted channnels. Now, i almost never see those errors.<br>
For the remote nodes, this means they have lost one less payment attempt when my channel wouldn't have been useable. And a payment attempt is not cheap. If a multi-path payment is initiated, new commitment transactions must have been renegociated. And if that payment fails at an intermediate hop, commitment transactions must be updated again to revert the intermediate state.

Another thing that seems to occur, but that i can't prove only from my node's logs alone, is an increase of the number of routed payments. Previously, after a certain number of payment failure for insufficient balance, payment attempts on some channeld had a tendency to stop. Because nodes had classified my node as not useable after all the failure they experienced.
But when you start setting the max_htlc_amount to something lower than your channel's balance, then you know that the 'Insufficient balance' error will not occur. And if the max_htlc_msat value is too low for a payment attempt, then a node will simply not try to use you as a path. they won't store 'i have history of failed payments with that node'. So you end up higher in their own ranking of node reliability

### Regarding privacy

One of the arguments against using *max_htlc_msat* this way is privacy. One could agree that channels not disclosing their balance is actually a feature, and not a limitation. You don't want to disclose what payment is occuring. And if you publish your change of channel balances when updating *max_htlc_msat*, then it kind of is what you are doing.

Here is my answer:
- I am running a routing node. So my personnal transactions are not really visible. You may not want to do this if your node is only for personal use
- I mostly connect to other middle size routing nodes. So i am never the start nor the end of a route, always just some middle hop
- Finally, channels policy can't be updated that often, or your node would be banned for spamming the network. I update, at most, the policy of a given channel twice per day. While on most days, i route 100+ payments. So i'm far from divulging enough information to be able to differentiate individual payments going through my node

## Other contributions of Rene Pickhardt to the Lightning Network
Here, i did not even explain the whole extent of what Pickhardt Payments brings, but merely scratched the surface.
If you want to know more, i encourage you to follow the different things he published or the different talks he gave

##
References:
[Pickhardt payments](https://arxiv.org/abs/2107.05322)
[Pickhardt Payments & Zero Base Fee for Lightning Network](https://www.stephanlivera.com/361)
[The power of valves for better flow control, improved reliability & lower expected payment failure rates on the Lightning Network](https://www.bitmex.com/blog/the-power-of-htlc_maximum_msat-as-a-control-valve-for-better-flow-control-improved-reliability-and-lower-expected-payment-failure-rates-on-the-lightning-network)
