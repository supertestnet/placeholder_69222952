# Aggeus Interface
An attempt to create a polymarket-like interface using my Aggeus protocol as a backend

=================
Recent progress
=================

I made it so a user can click on a market and make a deposit. I divide up their deposit into offers of 10k sats each, and anything remaining that is less than 10k sats just stays in their wallet as change. 

I didn't want to write the bitcoin scripts yet, so for now I made all Yes psbts deposit funds to tb1pgkykn7dqs4umtr3ggej3dekz2fa7ecjm8tzcns3fq6y508uf7dvsvhnswg and all No psbts deposit funds to tb1ppywlyhjxh35jxt3u47jdhj4gm55ewq654qr5qj8wtjfzjx9cjd0qurfgum (that way I can have users check whether their funds have changed state by checking if the psbt got broadcast). Note: later I changed this; I made it so that the Yes and No destinations are all different for two reasons: now, users can easily check if they have money in them to see if their offers got taken, and the code I wrote to *make* them different lays the groundwork for replacing these placeholders later.

Note that the user should not have a way to doublespend this input without cooperation from the coordinator or til a timelock expires, otherwise he can force the coordinator into a race condition by taking the other side of the trade, then, when the coordinator tries to broadcast the user's "original" psbt, the user doublespends the input, that way the coordinator is on the hook if he loses on the "other side" of the trade without the ability to get reimbursed by winning on "this side" of the trade -- and users should be warned that if they deposit money into the Yes or No addresses, their money may be gone forever if the oracle decides against them, or locked up til resolution day otherwise.

Once the user makes their psbts, the coordinator lists their offers based on the current outcome percentages, and the user also keeps a log of their offers (amounts and outcome percentages when they made the offer). For now, I hard-coded a single coordinator for all markets, and I have to pass a #run_coordinator=true parameter in the url to run him. If no one is running him, the page throws an error when loading a market.

If someone deposits into either side, the site looks for existing offers that "match" what the new user wants, within 2% "topside." For example, if the current outcome percentages are 60% Republican and 40% Democrat, someone betting on Democrat looks for any offers that cost 4200 sats or less, rather than the 4000 sats you might expect without this buffer.

If any matches are found, the coordinator acts as a proxy for those two users. For now, the smart contract between the proxy and the new user is represented by this address: tb1pr60dhp5hwpyeas00zfsczpgnmsddn49fj3wnlnzug37sk2w0r0kq5w7847. Note: later I changed this; I made it so that the smart contracts representing this are all different.

If the new user deposits more money than is available from the other side, then after matchmaking as much as possible, the remaining money from the new user becomes a set of new offers based on the current outcome percentages.

For all users, the Yes and No boxes have a "Your Info" button which, if clicked, displays info about their offers; namely, how much they deposited and at what outcome percentages, how much they are expected to earn on resolution day if their side wins, and how much they might earn if they try to sell their position right now (though with a reminder that it's not guaranteed that anyone will take their offers right now). I also have fields for showing the difference between the amount of money they've deposited that is now sitting in "offers that are waiting for a counterparty" and the amount in "offers that already have a counterparty," though I do not yet update those fields when their contracts become active.

=================
Next steps
=================

Implement support for selling a position

In service of that, consider this: John paid 3800 sats for his position because Kelly listed it for sale as if the odds were at 62%/38% instead of 60%/40%, to make her offer more attractive. If John relists the contract when the odds are at 20%/80%, he should actually list it for sale as if the odds were at 22%/78%, to make his offer more attractive, and thus expect to gross 7800 sats, for a net gain of 7800 - 3800 = 4000 sats. So he needs to make a psbt where the coordinator can take the money in the smart contract as long as he pays John 7800 sats.

Implement support for settling contracts on resolution day

Swap the placeholder functions with equivalent versions from my aggeus_market repo

Ensure users deposit money into an address with a timelocked unilateral branch and a 2 of 2 branch, or make the money go into a "single branch" 2 of 2 after they receive it into a regular 1 of 1 address, but then ensure that they *get* a timelocked unilateral withdrawal before moving it *into* the 2 of 2 (maybe one where they have to "activate" the timelocked path by creating a connector)

In the coordinator's accept_offers command, instead of taking a psbt from the user, have him tell you his inputs and the offers he is accepting, and then *recreate* the psbt

In the coordinator's create_offers command, validate the user submitted data before entering it into your db

Ensure all parties verify that any presigned transactions they trust only spend confirmed taproot utxos

=================
Done
=================

Ensure that when someone creates new offers, the coordinator learns about them via the api -- right now he only updates them if the user is *running* the coordinator -- I will probably need a new "create_offers" method for this [done]

Update the fields telling users how much money they can withdraw [done]

Ensure a user's stored offers change from pending to active once they become active [done]

Ensure the psbt which the coordinator tries to broadcast has valid signatures for all of its inputs [done]
