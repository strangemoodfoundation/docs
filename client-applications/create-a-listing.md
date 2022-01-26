# Create a Listing

A "Listing" is a software license (a game or an app) that's available for sale on the Strangemood protocol.&#x20;

### Using IPFS

Listings typically store metadata, such as titles, descriptions, and game files, on [IPFS](https://ipfs.io). IPFS returns a hash of the file called a "Content ID". Listings then store that content ID.&#x20;

{% hint style="warning" %}
Files on IPFS do not live forever [unless they are persisted by other nodes in the network](https://docs.ipfs.io/concepts/persistence/). The Strangemood Foundation attempts to automatically "pin" files from listings under its domain using contribution funds from the co-op. However, this process is guaranteed, so **you should pin CIDs that are uploaded or commonly accessed by your application.** It can improve load times for your users, and protect your application from centralization by a co-op.

\
Due to decreasing hardware costs and a glut of storage supply, many services, such as [web3.storage](https://web3.storage) and [estuary.tech](https://estuary.tech), offer significant amounts of data for free or extremely low-cost. You may also want to consider popular options like [pinata.cloud](https://www.pinata.cloud).&#x20;
{% endhint %}



### **Using OpenMetaGraph**

While you can technically link to anything in a listing, by convention, Strangemood uses [OpenMetaGraph](https://openmetagraph.com) format: a typed, composable format that can be read by disconnected applications. **If you do not use OpenMetaGraph format, the listings you create may appear broken to users in other applications.**

****

OpenMetaGraph's type system lets you compose multiple schemas together that let clients like wallets or game-libraries do a "best-effort" parsing of your listing's data. For example, a wallet may not know how to understand a "platform", but it does know how to understand "title".&#x20;



To ensure listings are usable in many different contexts, your application should consider implementing the following schemas:

| Name                              | Schema                                                |
| --------------------------------- | ----------------------------------------------------- |
| Title, Description, Primary Image | ipfs://QmUmLdYHHAqDYNnRGeKbHg4pxocFV1VAZuuHuRvdNiY1Bb |
| Creator                           | ipfs://QmciSnNRLu6RLBPv7BNBW9aAJxfdgVjuJDJxdB7Z3TNh8A |
| Platforms                         | ipfs://Qma1ujMNX6am8ziTr2qJuWCeK3pvvxsTey9wtaZWikVd4L |
| Versions                          | ipfs://QmX6GHm2hQNm6FujqaGDZaaWjJmgGUrHR5CxCfBkNuj8XX |



Here's a few examples of uploading an OpenMetaGraph document, and persisting it somewhere on IPFS.&#x20;

{% tabs %}
{% tab title="Using web3.storage" %}
```javascript
const metadata = {
  "object": "omg",
  "version": "0.1.0",
  "schemas": ["ipfs://QmUmLdYHHAqDYNnRGeKbHg4pxocFV1VAZuuHuRvdNiY1Bb"],
  "elements": [
    {
      "object": "string",
      "key": "title",
      "value": "My Cool Title"
    },
    {
      "object": "file",
      "key": "photos",
      "contentType": "image/png",
      "uri": "ipfs://some-cid"
    },
    {
      "object": "file",
      "key": "photos",
      "contentType": "image/png",
      "uri": "ipfs://another-cid"
    }
  ]
}
```
{% endtab %}

{% tab title="Using a pinning service" %}
```javascript
const metadata = {
  "object": "omg",
  "version": "0.1.0",
  "schemas": ["ipfs://QmUmLdYHHAqDYNnRGeKbHg4pxocFV1VAZuuHuRvdNiY1Bb"],
  "elements": [
    {
      "object": "string",
      "key": "title",
      "value": "My Cool Title"
    },
    {
      "object": "file",
      "key": "photos",
      "contentType": "image/png",
      "uri": "ipfs://some-cid"
    },
    {
      "object": "file",
      "key": "photos",
      "contentType": "image/png",
      "uri": "ipfs://another-cid"
    }
  ]
}

const client = create('https://ipfs.rebasefoundation.org/api/v0');
const { cid } = await client.add(JSON.stringify(metadata));
```



To keep your application fast and decentralized, you should pin the CID on your backend.&#x20;

```javascript
// server.js
import fetch from 'node-fetch';

const result = await fetch('https://api.pinata.cloud/psa/pins', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: ('Bearer ' +
      process.env.PINATA_API_TOKEN) as string,
  },
  body: JSON.stringify({
    cid,
  }),
});

```
{% endtab %}
{% endtabs %}

### Publishing to Solana

You can allow your users to publish new listings for sale with the `InitListing` instruction in the Strangemood protocol.&#x20;



When you create a Listing, you're also creating a [Solana Program Library (SPL) Mint](https://spl.solana.com/token). In other words, you're making a new token on Solana, that will be distributed to users when they purchase. Off-chain applications can check a wallet's public key to see if they have that token associated with the listing, and if they do, allow them access. Listing tokens are frozen upon purchase, and are only transferable by the lister.&#x20;



A user can purchase an arbitrary number of tokens from the Listing. A lister's application may use this to implement tiered pricing. For example, 1 Listing Token may purchase a basic version, and 2 Listing Tokens may purchase a special-edition.&#x20;



Like traditional tokens, Listing Tokens do not need to be integers. Use the `decimals` argument to make a listing token sub-dividable. For example, a Listing created with `decimals=2` would allow a user to purchase `1.25` Listing Tokens.&#x20;



Listing prices are set in [lamports](https://docs.solana.com/terminology#lamport) (the "cents" of Solana) per _amount_ in the Listing Mint. For example, if the price is 10 lamports, and the user buys 100 Listing tokens, then they've paid 1,000 lamports.&#x20;



Listings may be marked as "consumable", which allows the Lister to burn the tokens of any user, at any point. Listers may choose to use this in order to implement "pay-to-upgrade", in-app purchases, subscriptions, or usage based pricing.&#x20;



Listings may also be marked as "refundable", which allows a user to refund their purchase, until the Lister calls `SetReceiptCashable` which marks the listing as no longer refundable.&#x20;



{% tabs %}
{% tab title="React" %}
```typescript
import { initListing, fetchStrangemoodProgram } from '@strangemood/strangemood'
import { useAnchorProvider } from './helpers'
import { BN } from '@project-serum/anchor';

const provider = useAnchorProvider();

// ...

const program = await fetchStrangemoodProgram(provider);
const {
  tx,
  signers,
  publicKey, // the listing's public key
} = await initListing({
  program,
  signer: provider.wallet.publicKey,
  price: new BN(1000), // Price in Lamports!
  decimals: 0, // not subdividable
  uri: "ipfs://some-cid", 
  is_consumable: false,
  is_refundable: true,
  is_available: true
});

await provider.send(tx, signers);

```
{% endtab %}
{% endtabs %}

