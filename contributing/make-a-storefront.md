---
description: Build a storefront to sell Strangemood games
---

# Make a Storefront

## About storefronts

Strangemood has no single storefront, anyone can build one if they know how to use the Strangemood protocol. Checkout is an example implementation of a simple checkout page that can be used to purchase any Strangemood listing.&#x20;

**Note:** Though it is not covered in this guide, storefronts have the opportunity to earn revenue via a listing's bounty. A future guide on this topic will be made.&#x20;

## Getting started

This guide will follow the implementation in the [Strangemood Checkout Github repo](https://github.com/strangemoodfoundation/checkout). Before you begin you will need Node js and a package manager:

* [Downloading and installing Node js and npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
* **Recommended:** [Yarn | Getting Started](https://classic.yarnpkg.com/en/docs/getting-started)

You will also need a Solana browser wallet like [Phantom](https://phantom.app) to be able to purchase listings.&#x20;

### Running checkout

Start by cloning the repo:

```
git clone https://github.com/strangemoodfoundation/checkout.git
cd checkout
```

Install its dependencies and start the server:

{% tabs %}
{% tab title="yarn" %}
```
yarn install
yarn dev
```
{% endtab %}

{% tab title="npm" %}
```
npm install
npm run dev
```
{% endtab %}
{% endtabs %}

In your browser, go to `http://localhost:3000` and you should see a hello message. To purchase a listing you can navigate to `/r/<listing-id>`. For example, you can go to `http://localhost:3000/r/32Xcde9un5KAh3F15XL43TrVke9h8vjKTgHQEyDxE1mg`.

## How it works

Checkout uses [Next js](https://nextjs.org) as well as Typescript. Source code for the checkout page is located in `pages/r/[publicKey]/index.tsx`. This section will review source code for important parts of it's functionality.&#x20;

### Retrieve a listing

```typescript
import { fetchStrangemoodProgram } from "@strangemood/strangemood";
import * as anchor from "@project-serum/anchor";

export async function getProvider(net: solana.Cluster) {
  let conn = new solana.Connection(solana.clusterApiUrl(net));
  const keypair = solana.Keypair.generate();
  const wallet = new anchor.Wallet(keypair);
  
  const provider = new anchor.Provider(conn, wallet, {});
  anchor.setProvider(provider);

  return provider;
}

export async function getProgram(options?: {
  net?: solana.Cluster;
}): Promise<Program<Strangemood>> {
  const program = await fetchStrangemoodProgram(
    await getProvider(options?.net as any)
  );
  return program as any;
}

export async function loadListing(
  publicKey: string,
  net: solana.Cluster
): Promise<Listing> {
  const program = await getProgram({net,});

  let account = await program.account.listing.fetch(
    new solana.PublicKey(publicKey)
  );
  // Continued 
}
```

This function takes two arguments, the listing's `publicKey` (after the `/r/` in url) and the Solana `net` the listing lives on. [More information about Solana networks](https://docs.solana.com/clusters).&#x20;

Fetching the listing on line 30 returns `account` which is how the listing infromation is stored on the Solana blockchain. For a Strangemood account, this includes information like the price, if it can be purchased, and its metadata.&#x20;

### Retrieve a listing's metadata

Once we have a listing's account, we can get find a link to its metadata at `account.uri`:

```typescript
import { getListingMetadata } from "./graphql";

export async function loadListing(
  publicKey: string,
  net: solana.Cluster
): Promise<Listing> {
  // Get listing account

  const uri = account.uri;
  const data = await getListingMetadata(uri);
  
  // Continued 
}
```

The `uri` is a link to an [OpenMetaGraph](https://www.openmetagraph.com) document for the listing. OpenMetaGraph is a format for creating and querying structured metadata stored on IPFS. They are formatted as follows: `ipfs://<cid>`.

{% hint style="info" %}
You can view the schema for a Strangemood listing using the OpenMetaGraph schema explorer: [https://www.openmetagraph.com/alias/bafkreiczgupdf5ha7jt5oqn77koptvclt7edfzriu34ozgeqnasmhyio6a](https://www.openmetagraph.com/alias/bafkreiczgupdf5ha7jt5oqn77koptvclt7edfzriu34ozgeqnasmhyio6a)
{% endhint %}

The `lib/graphql.ts` file has code for retrieving and parsing the listing's metadata. The graphql query for retrieving the metadata can be seen in this section:&#x20;

```typescript
const query = gql`
    query ($key: String) {
      get(key: $key) {
        name
        description
        primaryImage {
          height
          width
          alt
          src {
            contentType
            uri
          }
        }
        // ...
      }
    }
  `;

  const data = await request(
    `https://www.openmetagraph.com/api/graphql?schema=${LISTING_METADATA_SCHEMA}`,
    query,
    {
      key: uri.replace("ipfs://", ""),
    }
  );
```

There are also typescript interfaces defined that make it easier to work with:

```typescript
export interface ListingMetadata {
  name: string;
  description: string;
  primaryImage: ImageNodeMetadata;
  // ...
}
```

Once you have loaded the listing's account and metadata you have everything you need to populate the checkout page.&#x20;

### Purchase a listing

```typescript
import { WalletContextState } from "@solana/wallet-adapter-react";
import { Transaction, Connection } from "@solana/web3.js";
import { Strangemood, MAINNET, purchase } from "@strangemood/strangemood";
import { PublicKey, Transaction } from "@solana/web3.js";
import { Provider } from "@project-serum/anchor";
import * as anchor from "@project-serum/anchor";
import BN from "bn.js";

export async function grabStrangemood(
  connection: Connection,
  wallet: WalletContextState
) {
  const { publicKey, signTransaction, signAllTransactions, connected } = wallet;

  const anchorWallet = {
    signTransaction: async (tx: Transaction): Promise<Transaction> => {
      if (!signTransaction) throw new Error("Wallet is not connected");
      return signTransaction(tx);
    },
    signAllTransactions: (txs: Transaction[]): Promise<Transaction[]> => {
      if (!signAllTransactions) throw new Error("Wallet is not connected");
      return signAllTransactions(txs);
    },
    publicKey,
  };

  const provider = new Provider(connection, anchorWallet as any, {});
  const idl = await anchor.Program.fetchIdl<Strangemood>(
    MAINNET.strangemood_program_id,
    provider
  );
  return new anchor.Program(idl, MAINNET.strangemood_program_id, provider);
}

async function onPurchase(publicKey: PublicKey) {
    const program = await grabStrangemood(connection, wallet);

    const { instructions } = await purchase({
      program: program as any,
      signer: program.provider.wallet.publicKey as any,
      listing: publicKey,
      quantity: new BN(1),
    });
    
    const tx = new Transaction();
    tx.add(...instructions);
    await program.provider.send(tx);

    router.push("/library");
}

```

The `grabStrangemood` function fetches the Strangemood anchor program. This is passed to the `purchase` function along with the listing's public key, a wallet signature, and the quantity to purchase.&#x20;

We create a transaction with the returned instructions and submit them using `program.provider.send`. This completes the purchase on the Solana blockchain!
