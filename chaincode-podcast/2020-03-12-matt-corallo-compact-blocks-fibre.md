---
title: Compact Blocks and Fibre
transcript_by: Michael Folkson
tags:
  - compact-block-relay
date: 2020-03-12
speakers:
  - Matt Corallo
media: https://www.youtube.com/watch?v=0uPDDELGjdo
episode: 6
aliases:
  - /chaincode-labs/chaincode-podcast/2020-03-12-matt-corallo-compact-blocks-fibre/
---
# Intro

Adam Jonas (AJ): Welcome Matt

Matt Corallo (MC): Hi

John Newbery (JN): Hi Matt

AJ: Today we are going to do a little bit of a “This is your life Bitcoin”.

MC: I am not dead yet.

AJ: You have a lot of contributions over the years so there is lots to talk about but we’ll start with three. Let’s start with compact blocks. Tell us a little bit about what compact blocks are and then we can dive a little bit deeper.

# Motivation for compact blocks

MC: Compact blocks was the culmination of, at the time I had worked on a series of projects relating to mining block propagation. It was to some extent motivated, the genesis of that work was motivated a little bit by the selfish mining thinking and the concerns around if a larger pool can get a material advantage in block propagation versus smaller pools, this may give them a super linear reward. The design of Bitcoin mining is such that if you have X amount of hash power you should get X percent of the reward, that should be completely linear. If you have 2x it should be exactly 2x the reward, there should be no increase. But certainly if you have poor block propagation and a very large pool this is not the case. We have seen this with a number of various altcoins that have turned the dial up really high on the amount of work that miners have to do to validate blocks or the size of blocks. We have seen where larger pools will make materially more reward than smaller pools per hash rate.

JN: Can we go into that a little bit more? When you say super linear why is that important? Why do we care that the proportion of reward that a miner gets is equal to the proportion of its hash rate?

MC: We want to make sure absolutely that there is no incentive for miners to join a larger pool. Pools provide an important service, they smooth out the reward for their users but it should not be the case that users want to join a larger pool than necessary because if they do we are just going to end up with one big pool or two pools. Then Bitcoin is super centralized. As long as there is not a strong incentive for that to happen, it might happen due to market forces but that can also still be temporary. Somebody can spin up a new pool if they are a little less concerned with a little bit of variance. The market can take care of that problem a little better if you don’t get paid more if you are on a larger pool.

JN: How does block propagation speed play into this? In Bitcoin we have 10 minute blocks. If we had 15 second blocks or very large blocks how would that affect this proportionality of reward?

MC: If you have blocks coming out very quickly, if a pool can see the blocks faster because they mined them or if the time it takes for them to download the blocks from other pools or validate the blocks, make sure they are valid before they can start mining them, if that takes a while that’s wasted time that their competition might not necessarily be wasting. If you mined the block, you know it is valid, you can immediately start mining on top of it. If you didn’t mine the block and you have to download it from someone, that time it takes is time that your competition is mining and you are not. We saw this for a very long time on for example Ethereum where larger pools were making a good chunk more money. If you were not on one of the largest pools you were wasting money, you were wasting hash rate and not making as much as your competition.

AJ: Can you tell us in human terms what kind of propagation we are talking about? What do you see or what have you seen in the past?

MC: I don’t think we’ve ever really had much of a problem with this in Bitcoin. In large part because the network has grown slowly while we’ve been able to stay ahead of it. If we were to take Bitcoin Core circa 2011 and throw it at the network today you would see massive issues here. It would take 30 seconds to verify a block and that is a good chunk of the percentage of a block every 10 minutes.

JN: Let’s talk about those figures. A block should come out every 600 seconds. If it takes 30 seconds to get from the miner to the other miners that is 5 percent. 5 percent of the time the other miners are doing useless work, they are not doing work on the latest tip. And so they will get 5 percent less revenue and that is material because that doesn’t mean 5 percent less profit. It means no profit. Or loss.

MC: Absolutely. Because mining is a super competitive market, you talk about some of these pools that are charging a few percent fee, if they are out 0.5 percent on revenue or 0.1 percent on revenue that might actually be a really significant part of their profit. A little bit less so for miners but also there you are looking at a super competitive market where your profit margin is not going to be that fat. It is going to be pretty thin. Not a large percentage in terms of wasted hash rate is going to turn into a fairly large percentage of lost profit.

AJ: As block propagation starts to get larger you also have not just people working on blocks that is wasted work but then you are confusing the network by propagating blocks if you found a different nonce or whatever the case might be. Is that the right way to think about it?

MC: You would see more orphans. Hopefully the network would not have too many problems. A lot of people do accept one confirmation transactions right now. That is maybe not an ideal thing to do. People who were doing that might see more issues but for the most part I think people are aware that if you are accepting a one confirmation transaction and it is for substantial value you are taking on some risk. Hopefully it wouldn’t confuse the network too much but it would be wasted hash rate which is also wasted security for the network. Because hash rate is providing security and anti double spend protection and anti re-org protection. If we have 5 percent orphans then that is 5 percent less security that an attacker doesn’t have to fight against.

JN: I am just going to make a small note here that Matt used the word [orphan block](https://bitcoin.stackexchange.com/questions/5859/what-are-orphaned-and-stale-blocks). I prefer stale block. I think orphan block is confused with when you have a block that doesn’t connect to the main chain. But moving on.

AJ: Captain nit.

JN: Just wanted to nit that one.

MC: That was a lot of the motivation for this work. There were two directions that I ended up going with. First is you are never going to beat a carefully laid out network topology in terms of relay. If you are talking about a selfish mining model where any advantage, even if it is one millisecond, is a clear advantage. This is a little different than the propagation model we were just talking about. Any advantage, if you are in that model then you need a very carefully laid out topology. I spent a lot of time building relay networks that are centralized but carefully laid out to be optimal for block propagation and then open them up so that anyone can use them. Hopefully smaller pools can use them. Larger pools do have these networks, they have built them themselves. They use them to propagate their own blocks. This is an alternative to say “If you are a smaller pool and you don’t want to set these kinds of things up, you don’t know how, you don’t have the time to invest or the money to invest, you can just use this and it is mostly good. You should just use this.

# Selfish mining

Bitcoin Mining is Vulnerable paper: https://www.cs.cornell.edu/~ie53/publications/btcProcFC.pdf

Optimal Selfish Mining Strategies in Bitcoin: https://diyhpl.us/~bryan/papers2/bitcoin/Optimal%20selfish%20mining%20strategies%20in%20bitcoin.pdf

AJ: Before we go too far I just want to define selfish mining for folks that don’t quite understand what that is.

MC: Selfish mining is an attack model, it spawned a number of further research. Selfish mining was an early [paper](https://diyhpl.us/~bryan/papers2/bitcoin/Optimal%20selfish%20mining%20strategies%20in%20bitcoin.pdf) in 2012, 2013 about Bitcoin mining strategies. If you are a miner, when you find a block what do you do with it? Do you immediately broadcast it and tell everyone? Do you sit it on it and try to mine something on top of it and then tell people? What happens when some other miner finds a block? Do you immediately mine on it? There’s the normal strategy of you find a block, you tell everyone. There’s the normal strategy of someone tells you about a block, you immediately start mining on it. But there is a lot of academic research around do you actually do that? Is that actually the most profitable thing you can do? It turns out that if you are a smaller miner and other miners are using this normal strategy and there is a block reward, you almost certainly want to just do that. If there is no block reward and we are relying on the fee market things look a little hairier. Or if you are a very substantial miner or if you have a very substantial network advantage in terms of you can propagate blocks than everybody else and you can see when others find blocks before they have told everyone about those blocks, then there are other strategies that might be more optimal.

JN: And notably other strategies that don’t require 51 percent. You could use selfish mining if you have for example 40 percent of the hash power. It might end up being more profitable than being a honest miner extending the longest chain.

MC: Yeah. It is a weird attack in the sense that it gives you super linear profit. It doesn’t allow you to arbitrarily double spend. It doesn’t allow you to arbitrarily censor people’s transactions on the chain. But it does give you super linear profit and potentially allows you to force some of your competition out of business which of course would eventually allow you to become a 51 percent attacker.

# Compact blocks

Compact blocks FAQ: https://bitcoincore.org/en/2016/06/07/compact-blocks-faq/

Compact Block Relay (BIP 152): https://github.com/bitcoin/bips/blob/master/bip-0152.mediawiki

Implementation of Compact Blocks in Bitcoin Core: https://github.com/bitcoin/bitcoin/pull/8068

AJ: Back to the original discussion, propagation.

MC: One direction in solving network propagation is coming to the understanding that no peer-to-peer network is going to beat a carefully laid out topology in trying to provide competition for larger pools who have their own carefully laid out topology. Then the other direction which is compact blocks, working on making sure that the peer-to-peer network is as fast as we can make it. It has constraints in that the connections are random and you are never going to be able to find the absolute lowest latency path between any two peers in a network that is designed to be as robust as possible. The peer-to-peer network is not designed to be absolutely blazing fast. It is designed to make sure that if there is a block everybody can see it and everybody comes to consensus on what the current chain is. With that in mind we should make it as fast as we can but there are limits. Compact blocks is one part of it. Compact blocks is how do we make sure a block going over the network is smaller so that we can propagate it faster. Pure network bandwidth, there are only so many bytes you can shove down a small pipe at once. If a block is one megabyte that is non-trivial if your goal is to send it over the wire in under a second or a hundred milliseconds or ten milliseconds, one megabyte is pretty non-trivial. Especially if you talking about any kind of packet loss crossing the Great Firewall, you are looking at a lot of packet loss in and out of China. Or if you are on a residential connection or something like that it starts to be a non-trivial amount of data. Compact blocks is saying “Well look, here’s this block. It has got a bunch of transactions in it. The chances are we have already seen all of those transactions. They have been in our mempool for a while. That is why the miner included them. Why are we sending these things again?” What you are doing essentially is you are saying “I am going to take all these transactions. I’m going to look at what was already in my mempool. I am going to send you a list of the transactions in the block.” You are going to look at what is in your mempool, you are going to say “I have got most of these. This one I’m missing.” You’re going to ask for that one and you are going to receive it. Now you’ve got your block. This also makes a really big difference, I assume most of the listeners were around in the early 2000s or late 1990s…

AJ: Like alive?

MC: Alive and on the internet and tried to use torrents, would not irregularly have someone screaming from down the hall “Turn off your damn torrents because the internet is not working. You are eating all the upload bandwidth. There are buffer bloat issues.” Bitcoin actually had that problem as well. When a new block came out, if you were running a full node that was publicly accessible, the chances are your internet fell over because all of a sudden when a new block comes out you are uploading it to 20 people. In the absence of good queueing strategies uploading something to 20 people will make your internet fall over. Because compact blocks removes these peaks in network bandwidth, it smooths that out, it largely solves that issue for a lot of people.

JN: Just recapping, prior to compact blocks, this was back in 2015, 2016 that this was implemented, prior to that I am running a node on the Bitcoin network, I have peers that I connect to, I receive unconfirmed transactions outside of blocks which go into my mempool. When I receive a block most if not all of the transactions in that block are transactions I’ve already seen. I am downloading the same data at least twice. After compact blocks you have pre-cached all of those transactions you have seen already. You don’t need to download them again. You just download the header and some short identifiers for the transactions. What kind of difference does that make? In 2016 a full block would have been 1 megabyte, now it is maybe 2 megabytes. For a compact block what size do we get?

MC: It is maybe 2000 transactions times 6 bytes per transaction. So maybe 10 kilobytes, maybe 15. Any transactions that you are missing cause a roundtrip. More often than not you are not missing any transactions but any transactions that you are missing, you are going to have to request those. You have to send those again. Usually it is just the initial sketch. It is just the block header and then 6 bytes per transaction. 6 bytes is essentially just the transaction hash but with some extra data shoved in just to make things unique. It is essentially just always that and that’s all you need.

JN: We’ve reduced a 2 megabyte block to around 18 kilobytes which is a pretty nice win.

MC: Yeah. It makes a real difference for either people who have buffer bloat issues on their upload link or people who are worried about downloading blocks from users on residential connections. It makes a pretty big difference.

AJ: This is a technical podcast so we don’t want to go too deep into the philosophy but in terms of thinking about consumer grade internet connections and the importance of enabling normal users in their homes to be able to use Bitcoin, where do you draw the line of performance for the network and performance for the individual user?

MC: It kind of gets back to what is the goal of the peer-to-peer network. In my mind the goal of the peer-to-peer network pretty clearly is a censorship resistant consensus system to enable this network of nodes which users decide to run wherever they are, whether it is a LTE modem or a 10 gigabyte fibre line, to come to consensus. These nodes, whatever they are, need to come to consensus. It needs to be robust. It needs to not fall over if half of the nodes are malicious or 90 percent of the nodes are malicious. That’s the goal. The goal is not let’s make this as screaming fast as possible, let’s optimize who you connect to for the lowest latency. Once you start doing that it becomes very hard to keep that robustness property. If you only connect to nodes with low latency you are only connected to nodes near you and an attacker just has to spin up a bunch of nodes near you. All of a sudden that is the only thing you are connected to. They are conflicting goals.

JN: And also in that model you get a lot of very closely connected subgraphs and it would be easier for an attacker to partition the network potentially.

MC: Right. These are conflicting goals. Bitcoin should strive to be robust in my mind. That is kind of where this divergence of “Here’s a peer-to-peer network that is super robust. We make it as fast as possible and easy to use.” It is not going to cause buffer bloat problems on your home network but also here are some other more centralized things that are as fast as we can make them so that there is better competition across larger and smaller pools.

# FIBRE

FIBRE: https://bitcoinfibre.org/

AJ: Something like FIBRE for instance? Built in transition.

JN: What is FIBRE and if you wanted to write that down in the English language what letters would you use?

MC: In the British language or the English language?

JN: What is FIBRE?

MC: The backronym is Fast Internet Bitcoin Relay Engine. Compact blocks was built as a part of FIBRE and then spun out and included in the Bitcoin peer-to-peer network, not the way other around. FIBRE is the second generation of these kinds of centralized but open relay networks that I have built. The first generation was something like compact blocks but still built on top of standard TCP. It still had [head-of-line blocking](https://en.wikipedia.org/wiki/Head-of-line_blocking) problems and FIBRE largely was intended to solve head-of-line blocking. If you have a TCP socket it is modeled after a UNIX socket. If I write bytes into a TCP socket there are going to be all these packets that go over the wire. It is going to come out to your application on the other end in the same order that it was written. There is hopefully going to be no corruption, it is just going to be all in order. All in order is great unless you received packets 2 through 15 but you missed Packet 1. Now the application can’t receive anything because it has to be in order so we have to go back and potentially cross an ocean again, ask the sender “I missed Packet 1. Can you resend Packet 1?” Then we download that and all of a sudden we have crossed the ocean a few times. Now we have the data and that’s great but we are talking about a few megabytes of data, if it is compressed with something like compact blocks, a few kilobytes, sending that through a 1 gigabyte line like you have on both servers is a few milliseconds but a round trip across the ocean is 80 to 150 if halfway across the globe. That act of crossing the ocean again to fetch something you’re missing is way, way worse than just sending the data 5 times. The initial version of the Bitcoin relay network that I deployed still used TCP, it had even better compression than compact blocks but required a lot more memory per connection. It wasn’t suitable for the Bitcoin peer-to-peer network but in a centralized network it worked reasonably well. It got a lot of blocks down to 1 or 2 packets so hopefully packet loss was less of an issue. But you still had a very long tail of “If you happen to lose a packet during sending your block propagation is going to be pretty slow.” FIBRE was a redesign saying “The biggest problem is we don’t ever want to have to ask for lost data.” The obvious solution here is technologies called forward error correction.

# Forward error correction

https://en.wikipedia.org/wiki/Error_correction_code#Forward_error_correction

MC: Forward error correction, if you are familiar with Shamir’s Secret Sharing, the simple version of forward error correction is exactly the same math as Shamir’s Secret Sharing. You can picture Shamir’s Secret Sharing. You have this key that you’ve split across ten shards but you only need five to reconstruct it. You can imagine the exact same thing. I’ve got this block, I’ve got maybe 1 packet from within a block. I’ve split it into three shards but you only need one to reconstruct it. Now I can send all three across the wire and if I lose one or two that’s fine. I still receive the data on the other end without having to go and ask for it again.

JN: In that example you are just sending the data three times? Error correction is used in many places. If you have a CD with data on it is not just one copy of that data, there are duplicates. You just recombine them and if part of that CD gets scratched you can still read the data from it. It is used in a lot of places.

MC: Right. The math can get much more fancy. You can send 1.1 times the data and miss 10 percent of packets and that’s fine. It doesn’t matter, you are not just resending 10 percent of packets and hoping that the packet that you lost was one of the ones you resend. You can imagine a simple version if you just want one extra packet, you just XOR all the data together, if you lost one packet that lost packet is going to be all the other ones XORed together. This can scale up obviously. There is math that gets very complicated very quickly. But you can do simple extra data sending. Now FIBRE did not implement any of this math. I am not competent enough in my linear algebra skills to sit down and figure out high performance forward error correction libraries. I used some external ones that are actually very high quality. Then I built a custom protocol based on essentially compact blocks but with some additional data to make it very efficient to actually send Bitcoin blocks over the wire using your local mempool as an extra piece of data. FIBRE essentially, you can imagine it is sending first the compact block header, with some additional forward error correction. Then the receiving end, if they aren’t missing any transactions from the compact block, they are done. They don’t even have to receive any transaction data. They have downloaded this compact block with forward error correction so that even with some packet loss they have still got it without having to ask for it again. Then they’ve got the block. That said, because our real goal here is to not have to go ask for data that you missed. If you are missing a transaction you don’t want to have to ask for it. FIBRE not only sends the compact block with forward error correction, it also sends all the transaction data with forward error correction. What you do is you’ve received this compact block header, you find any transactions that you do have and you start initializing your forward error correction data with those transactions as if you’d received packets. But of course you have received nothing, you just already have them in your mempool. Then as you receive additional packets with forward error correction you can fill in the transactions that you were missing. The sender doesn’t need to know which transactions you are going to be missing, it is all forward error correction across the transactions in the block. If you have most of them then you only need a few packets but it doesn’t matter which ones you are missing. You just need a few packets and then suddenly the forward error correction will spit out the full block data. Now you’ve got the block. It works pretty well. If you just need the compact block header usually you will end up receiving the block within 1 to 3 milliseconds plus the speed of light. It does take 20, 30, 40, 50 milliseconds depending on which ocean you are crossing as to how long the data takes to get there. 1 to 3 milliseconds plus speed of light, if you have additional transaction data forward error correction does take a little bit of CPU time but you are still talking 7-9 milliseconds in addition to the speed of light.

# Comparison between TCP and UDP

https://en.wikipedia.org/wiki/User_Datagram_Protocol#Comparison_of_UDP_and_TCP

JN: Wow, that is really cool. That data is sent using UDP?

MC: Yes. That is all UDP because it doesn’t have this head-of-line blocking issue and so we can process those packets as they come in as individual packets.

JN: Very quickly for the listeners who aren’t network nerds, what is the difference between TCP and UDP in ten seconds or twenty seconds?

MC: UDP is just “I have a packet, I send it across the network. Hopefully it makes it and I receive packets.” TCP is built on the same basic packet principle but it does all the fancy stuff for you to give you a nice stream abstraction. It will keep track of which packets it is missing. It will rerequest those, it will resend packets that didn’t make it to the other end. At the end you get a nice stream of bytes that are in order and you don’t have to worry about this whole packet thing and lost packets and all of that nonsense.

JN: From the application standpoint it is reliable. Whereas UDP, anything goes. You might drop packets but because you have forward error correction in this protocol it is ok to drop packets occasionally.

AJ: That is really cool. I think we are going to wrap for today because we’ve covered two things that go very well together, compact blocks and FIBRE. You have already promised that you are going to come on and talk more. Can’t wait to explore more of your projects further, thanks for coming on.

