# capacitor
A non-custodial offline micropayments mailbox using atomic swaps between Bitcoin Lightning Network and 2-of-3 multisignature Liquid BTC addresses.

## Purpose

While Lightning Network is fantastic for online payments where both parties (nodes, wallets) are online and available at the same time, for some use cases this may not be ideal. For example, if a wallet-to-wallet transaction is desired but the parties are in different timezones, then the coordination to get online at the same time is cumbersome. Another case is making a donation to a wallet offline where the recipient is not online. Solutions such as static invoices, keysend, offers, and hodl invoices don't work in this case because they all require the Lightning node to be responsive.

To solve this time issue, *capacitor* acts as a Bitcoin mailbox, storing your Bitcoin in a secure, non-custodial way such that the capacitor server cannot steal any funds. Note, the service could still go offline or be shutdown, but no funds are at risk. You can send money to a user's capacitor mailbox by address at any time, even when the recipient's node or wallet is offline. The money can then be "picked up" from the mailbox at any time later, completing the transaction.

## How it works

To do this, capacitor performs an atomic swap of a Lightning Network payment into a 2-of-3 multisig L-BTC contract. The multisig parties are the sender, recipient, and capacitor itself. When a deposit is made, the sender simply pays a Lightning Network invoice. The keys used for L-BTC redemption are xpub addresses of the sender, recipient, and capacitor with new ones generated for each separate payment. 

The deposit performs a swap with an L-BTC multisig contract of the same amount from sender to recipient, who both provide an L-BTC destination addresses in advance. Funds get paid into the capacitor's lightning wallet, while the funds are simultaneously locked in the L-BTC contract for a possible refund back to the sender (with two keys or expiry). The L-BTC transaction is then constructed and delivered to both users for self-validation, so that they know it can be redeemed properly if anything goes wrong. However, the L-BTC transaction is not broadcast to the network. It is only broadcast (by the sender) if there is an issue with the transaction, e.g. the capacitor service disappears. 

Once deposited and funds locked in the contract, the recipient can fetch their funds at any time. By providing their L-BTC signature, they can authenticate/verify that they are the true recipient. In this withdrawal, an atomic swap between the L-BTC to their Lightning Network wallet occurs. Funds get paid out to the recipient's lightning wallet, while the funds locked in the L-BTC get simultaneously paid out to the capacitor. The two keys that are used belong to the recipient and the capacitor. The sender's key is not needed (as they could be offline anyways).

## Why L-BTC and not just BTC?

In order to keep settlement costs as low as possible, L-BTC was chosen. If the L-BTC chain becomes prohibitively expensive, another BTC sidechain may be used (or created) in the future for this purpose. In most cases with successful transactions, L-BTC transactions are actually not broadcast so it only acts as an insurance layer.

## Payment notification

As an additional feature, capacitor provides an API and HTTP callback to let someone know when a payment has arrived into their mailbox. This can be used to notify a wallet, send an email, ping them on slack/telegram, etc. when there is a new payment for them to come fetch. Also, a balance check polling API address is also avaialble for wallets that are not online all the time to receive a callback, and wish to ocassionally just check if there are funds and then pull them automatically using the keys in their wallet. In this way, wallets can actually use capacitor without the users knowing of it's existence. The users can simply make LN payments to another user's capacitor address seamlessly, all within the wallet itself.

## Easy payments and withdrawals

To facilitate quick and easy payments/withdrawals the LNURL specification will be used together with wallets to avoid the need to copy-and-paste Lightning Network invoices in the user interface. There will also be APIs for programmatic payments/withdrawals for more advanced wallet services. Also, for more seamless operation within wallets, a wallet might be able to first query if the recipient is currently online, and simply do a direct LN payment first. If they are not online, then it can fallback to a capacitor payment.

## Fees and Capacitor Markets

Since anyone can run a capactor node, the best ones will be well-mantained, have good uptime, good support services, reputable, and have reasonable fees. It is important that sender/receiver use the same capacitor node since the capacitor is the escrow/signatory of the 2-of-3 multisig. However, there may be a way in the future to construct a capacitor network, where there is no single point of failure, and capacitor transactions are held by multiple capacitor nodes in a path chain or route. This may increase privacy and compatibility, but this is a much more advanced system, and outside the scope of this initial concept.

## Issues

Privacy is a main concern here, since the capactor knows the two parties and it unwires the onion-routing work done in Lightning Network to increase anonymity. So it is yet to be determined if privacy can be preserved somehow. This could be part of the multiple-capacitor-routing system or perhaps wrapped in messaging the way the "experimental offers feature" in c-lightning currently works.
