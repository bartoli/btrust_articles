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
