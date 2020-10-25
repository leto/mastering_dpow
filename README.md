# Mastering Delayed Proof-of-Work

## by Duke Leto

This is a short technical document aimed at developers who are interested to learn
more about DPoW and/or want to add it to their a cryptocoin to protect
against consensus (51%) attacks and double spend attacks. These attacks are becoming very common
against coins from large to small.

This book assumes basic knowledge of the Bitcoin protocol, Bitcoin source code,
C++ and basic compiler knowledge. No prior knowledge of Hush is assumed, this
work contains a basic introduction of Hush features.

# High Level Overview of Hush DPoW

Delayed Proof-of-Work is a general concept and the Hush Developers have implemented
a production system which implements the algorithm for HUSH and for any blockchains
which want to pay HUSH for the service of double spend protection.

DPoW sends blockhash data from your coin, to potentially another coin like HUSH, which is then sent to Bitcoin,
providing a small coin with a small hashrate the security of BTC hashrate. This
data is embedded in OP\_RETURN metadata which Notary Nodes continuosly
send. Notary nodes run full nodes of coins they inject data into.

Delayed-Proof-of-Work is an abstract algorithm that is not tied to any particular
blockchain. Anybody can use it and implement the service.


## Global Notary Node Network

For example, say there exist a network of 64 notary nodes spread around the
globe.  They are not in any one physical location, which makes the system
resilient against a datacenter outage or natural disaster in one geographical
location. The algorithm NNs use require 13 notary nodes to come together and
agree on what the latest Merkle root and other metadata that will be embedded
into the Hush and Bitcoin blockchains inside the OP\_RETURN data of a
multisig transaction. This means that to take down the notarization process, to
stop the data flow, one must attack and knock offline 52 different servers in
diverse locations,  simultaneously. In this case, notarization data would stop
flowing, but no "fake" or "incorrect" notarization data can be created, because
the source code of Hush has the public keys of notary nodes embedded inside.

Only notary nodes have the corresponding private keys to make transactions that
the network will trust as valid notarizations. So DDoS attacks can only stop
notarization data from flowing temporarily, they can not be used to knock
actual notaries off-line and then impersonate or man-in-middle the process.

## Adding or changing DPoW is a Hard Fork

To be clear, DPoW are consensus-level changes to a cryptocoin source code,
they require a coordinated effort of all exchanges/mining pools/users upgrading,
since it is a hard fork. Take this into consideration when planning upgrade
timelines.

# DPOWconfs protect against double spends with no code changes

A recent innovation called "DPoW confs" allows
exchanges to seemlessly protect themselves against double spend attacks, with
no code changes at all! Exchanges can simple continue to look at the
`confirmations` key of whichever RPCs they use. DPoW confs is enabled by
default, and when enabled, the confirmation count of a transaction will never
go above 1, until it is notarized.

To summarize

  * confirmations=0 is unconfirmed (same as before)
  * confirmations=1 is confirmed, but not notarized
  * confirmations>1 means confirmed + notarized

Since exchanges currently require anything from 10 to 100 or more
confirmations, their current code already enforces that transactions are
notarized.

The old/original concept of `confirmations` is now stored in the
`rawconfirmations` key of any RPC that returns `confirmations`.

DPoW confs are defaulted to on in the latest versions of HUSH, exchanges
do not need to change any code, any HUSH daemon setting or CLI flags. They
are protected by default.

# Cost of DPoW

The raw cost of many, say 64 global notary nodes making transactions roughly once every
75 seconds (the Hush block interval) and continually on other coins has a cost.

Various costs related to DPoW can be broken down into these categories:

  * Transaction fees denominated in HUSH (paid by notaries making transactions)
  * Transaction fees in the coin being notarized (paid by notaries)
  * Yearly cost paid to HUSH for DPoW services
  * One-time integration costs to add DPoW to external coins

Contact Hush [Telegram](https://t.me/hushcoin) for more info about pricing.

Also note, that if you decide to migrate your existing coin to a [Hush Smart
Chain](https://github.com/MyHush/hush-smart-chains), there are no integrations needed or integration development costs. Just
the cost for notarization transactions. This can be accomplished by doing an
airdrop from your current chain to a new Hush Smart Cain, where people use their
private keys to unlock the funds they owned on the original chain. This can be
done by the project itself or Hush Developers can help with this process.

# Integrating DPoW Into Your Coin

Let us assume Alice is the lead developer of a cryptocoin based on the Bitcoin
source code. For example, since Litecoin and Zcash are Bitcoin forks, any LTC
or ZEC forks count as Bitcoin forks, too. The version of Bitcoin internals that
the coin forked is important, this determines which header file that will be
immediately compatible or at least a close starting point for development work.

For example, if your coin is based on Bitcoin 0.15.x, there already exists a
header file that matches that version of Bitcoin internals exactly. The GAME
coin was the first external coin to start using dPoW and [this commit](https://github.com/gamecredits-project/GameCredits/commit/e65fe302111408c02d2bf7e286205d4273fa0fed)
shows how they integrated. This was first created by James Lee and then the author of this document ported that header file to Bitcoin
0.11.x internals, which Zcash and all Zcash forks have as their internals. So
if your coin is a Zcash fork, you should use the
[komodo\_validation011.h](https://github.com/MyHush/hush/blob/e529ad8d35d4dabf66fa10982e056149414f179b/src/komodo_validation011.h)
header file. If your coin has SegWit support, or is 0.14-0.16 internals, the
[komodo\_validation015.h](https://github.com/jl777/chips3/blob/dev/src/komodo_validation015.h)
file is most likely the best starting point to integrate.

For coins with older internals, such as BTC 0.10.x and earlier, use the BTC
0.11.x as a starting point, so less changes are required.

If your coin has made various internals changes and selectively added BIPs
or other internals changes, you will most likely need to make various changes
specific to your coin.

## Calculating Notarization Lag

We will use Hush, a Zcash fork, as an example to estimate notarization lag,
i.e. the time it takes for notarization data to make it from one chain to
another. To estimate the "worst case" time it takes notarization to get to
Bitcoin, we can simple add up all the blocktimes involved, so we will be adding
blocktimes of

    HUSH + BTC = 75s + 600s

which is 675s or about 11 minutes for blockhash data from Hush to be notarized all
the way to Bitcoin. This is the time it would take such that an exchange could
ask the Bitcoin blockchain if a given txid has been notarized.

Note that the above block-times are "worst case" under normal network
conditions, i.e. a transaction is made a millisecond after a block, so it has
to wait the entire block interval on HUSH. And then is unlucky enough to have
to wait an entire Bitcoin block interval.

It's possible that a bug or attack or large difficulty change makes one block
much longer than the average block interval time, which would increase the
time it takes for notarization data to make it to BTC. This is why a specific
time cannot be used to decide it's "safe", very often 11 minutes would be enough
time for Hush block hash data to get to the Bitcoin chain, but there could
be times when it is not.

On average, users will only wait half of each coins block time, which
means 11/2 = 5.5 minutes for Hush blockhash data to be written to Bitcoin.

## Is My PoW algorithm compatible with DPoW?

There are many PoW algorithms to choose from these days and thankfully
the DPoW algorithm is blissfully ignorant of all these differences. It
does not care if something is SHA256 versus Equihash, all it sees is a
256 bit block hash, i.e. a 256 bit integer. It's just a number. Any
coin using a 256 bit blockhash is compatible, which includes most
known coins.



## Integrating a Zcash fork

For the specific case of adding DPoW to a Zcash fork, the 0.11.x header file,
first implemented for Hush, should be used. Zcash forks are characterized by
being pre-Segwit and having some differences in how UTXOs are stored in LevelDB
compared to post-Segwit coins.

## Integrating a Litecoin fork

For Litecoin forks, the version of BTC internals inside is what makes the biggest
difference. Being pre-Segwit means using the 0.11.x header file as a starting point,
and post-Segwit can use 0.15 header file.

## Integrating a CryptoNote/Monero fork

There is recent work in this area by BLUR, see this repo for details: https://github.com/blur-network/dpow-blur

## Integrating a non-Bitcoin-derived coin

It will be most likely be challenging to integrate a coin not derived from
Bitcoin source code but we are eager to see if it's possible. The currently
available 0.11 and 0.15 header files can be used if the project is in C++ but
if the coin is written in another language then those header files must be
ported to that language first.

If your coin reimplements Bitcoin protocol from scratch, then it may still be
compatible.

### Integration Procedure

This is a high-level procedure of how to integrate an arbitrary coin. It will
not describe every single step but it will be applicable to most situations.

* Create a new branch for dpow work
* Copy the correct header file (komodo\_validation0XX.h) to the `src/` directory of your coin
* Copy the [komodo\_rpcblockchain.h](https://github.com/MyHush/hush3/blob/dev/src/komodo_rpcblockchain.h) to your `src/` directory. This file will not need
  many, if any, changes

* If you are already using BTC internals 0.11/0.15, skip the next step
* Make any API changes required to get header files compiling on your
    specific version of BTC internals
* Make sure your code dynamically generates addresses (0.11 does, 0.15 does
    not yet)
* Do a fresh sync of 2 new nodes on the branch with dpow code
    * One node with have txindex=1
    * One node will not have txindex
* If both nodes can sync from scratch with no bad errors, then you are most
    likely done with integration!
* Contact Hush for the next round of instructions, which is Notary Nodes
    testing your code

* List of integrated coins and their header files
  * [Hush](https://github.com/MyHush/hush/blob/dev/src/komodo_validation011.h) BTC 0.11.2
  * [EMC2](https://github.com/emc2foundation/einsteinium/blob/master/src/komodo_validation013.h) BTC 0.13
  * [KREDS](https://github.com/leto/kreds-core/blob/dpow/src/komodo_validation015.h) BTC 0.15
  * [GAME](https://github.com/gamecredits-project/GameCredits/blob/master/src/komodo_validation015.h) BTC 0.15
  * [CHIPS](https://github.com//jl777/chips3/blob/master/src/komodo_validation015.h) BTC 0.15
  * [SUQA](https://github.com/leto/SUQA-0.17.1/blob/dpow/src/komodo_validation017.h) BTC 0.17.1
  * [GIN](https://github.com/GIN-coin/gincoin-core/blob/master/src/komodo_validation012.h) BTC 0.12

### When do notarizations actually start?

Here are the steps for notarizations to begin:

  * Consensus changes to full node
  * RPC changes to full node
  * Regtests for dpow (if applicable)
  * Payment of Hush DPoW yearly service fee
  * Notary nodes asked to install/configure new daemon
  * Notaries work on any changes to iguana that are needed
  * Notaries begin to install full node + sync
  * Notaries update iguana changes for new full node
  * When 13 notaries have synced+configured the full node, notarization rounds begin

When all notary nodes have installed, configured and synced the new full node,
notarizations will happen with greater frequency.

### Iguana Changes

Iguana is the low-level software which actually does the complex work
of notarizing data across chains. Some glue is needed when new
coins are added to notarization process:

 * Update dpowassets, example: https://github.com/jl777/komodo/pull/887
 * 7776 file, example: https://github.com/jl777/SuperNET/pull/890
 * Other miscellaneous changes

For instance, Hush was the first coin with 2 bytes of base58 prefix to
be notarized. This caused issues because the internals of the
coin assumes a 1 byte prefix in various places. Thankfully one of
the amazing notary node operators stepped in and wrote a good chunk
of C++ code to enable 2 byte prefixes, which makes many Zcash forks
compatible!

### Notary Node duties when adding a coin

This is a non-exhaustive list of things that notary node operator will do
to enable the new coin on production:

* Verify notary addresses match those generated/hardcoded in code
* Install full node software
* Compile on correct branch
* Fully sync full node with txindex=1
* Import their address to the new coin via a WIF generated from pubkey
* Run full node with -pubkey option
* Receive coins funds to be used in notarization UTXOs
* Do a round of testing with enough NNs to do a dpow round
  to verify code more
* Set up automated procedures specific to each coin, such as UTXO splitting/etc

