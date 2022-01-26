# Purchase a Listing

## Overview

When a user buys something on Strangemood, their SOL moves into an escrow account, called a "Receipt". If the Listing is marked as non-refundable, the Receipt may be immediately "Cashed", which transfers the money from the escrow to the Lister. Otherwise, the Receipt may be refunded if the purchaser runs `Cancel` which closes the Receipt. If the Lister runs `SetReceiptCashable` then the Listing can no be refunded.&#x20;

![The Strangemood Purchasing Flow](<../.gitbook/assets/Screen Shot 2022-01-25 at 3.49.22 PM.png>)

### Cashiers

When you initiate a purchase, you must also specify the "cashier" that is allowed to run the `cash` instruction. When cash is run, the Rent (SOL) in the receipt is given to the cashier as a minimal incentive to run a marketplace and offsets the fees of running the instruction. It's currently about 0.0019 SOL per transaction, but may decrease over time as hardware costs decrease.  **Your application should make itself the cashier.**&#x20;



To do this, consider creating a keypair with Solana's development tooling. Install [Solana's Developer tools](https://docs.solana.com/cli/install-solana-cli-tools) if you have not already. Then, generate a new keypair for your cashier.

```bash
solana-keygen new --outfile cashier.json
```



Then, get the public address of your cashier, you'll need it later.

```
solana address -k cashier.json
```

## The Purchase Instruction

When you run `purchase` , users are not paid out immediately. Instead, SOL moves into an escrow, called a "Receipt", which you will need to "cash" later to finalize the transaction. Your application should store the public key for this receipt, so that it can cash the receipt later.&#x20;

```typescript
import { fetchStrangemoodProgram } from "@strangemood/strangemood";
import * as anchor from "@project-serum/anchor";
import { useAnchorProvider } from "./helpers";

const provider = useAnchorProvider();

// ...

const program = await fetchStrangemoodProgram(provider);
const { ix, receipt } = await purchase({
  program,
  cashier: new anchor.web3.PublicKey("your-apps-pubkey"),
  signer: provider.wallet.publicKey,
  listing: new anchor.web3.PublicKey("listing-pubkey"),
  quantity: new anchor.BN(1),
})
await provider.send(ix); 
```

{% hint style="info" %}
Don't forget users can purchase _multiple_ listing tokens. By convention, 1 token should be a minimum purchase.&#x20;
{% endhint %}



## The Cash Instruction

![](<../.gitbook/assets/Screen Shot 2022-01-26 at 11.47.40 AM.png>)

You can cash a Receipt to pay out all parties if the receipt is marked as "cashable". If the Listing is non-refundable, you can cash the receipt immediately. Otherwise, you'll need to wait until the lister marks the Receipt as "cashable" (effectively making it no longer refundable).&#x20;



Cashing requires your application's keypair, and so should be done on a server. Put the contents of the keypair (generated from `solana-keygen` above) into an environment variable like `CASHIER_KEYPAIR` .&#x20;



{% tabs %}
{% tab title="TypeScript" %}
```typescript
import { Provider, web3 } from '@project-serum/anchor';
import {
  Transaction,
  clusterApiUrl,
  PublicKey
} from '@solana/web3.js';
import { cash, fetchStrangemoodProgram } from '@strangemood/strangemood';

// An anchor-friendly wallet generator
function cashierWallet() {
  let private_key = process.env.CASHIER_KEYPAIR;
  if (!private_key) {
    throw new Error('Unexpectedly did not find process.env.CASHIER_KEYPAIR');
  }
  const keypair = Keypair.fromSecretKey(
    Uint8Array.from(JSON.parse(private_key))
  );

  return {
    signTransaction: async (tx: Transaction): Promise<Transaction> => {
      tx.sign(keypair);
      return tx;
    },
    signAllTransactions: async (txs: Transaction[]): Promise<Transaction[]> => {
      txs.forEach((tx) => tx.sign(keypair));
      return txs;
    },
    publicKey: keypair.publicKey,
  };
}

const conn = new web3.Connection(clusterApiUrl("mainnet-beta"));
const provider = new Provider(conn, cashierWallet(), {});
const program = await fetchStrangemoodProgram(provider);
  
const { tx } = await cash({
  program,
  signer: provider.wallet.publicKey,
  receipt: new PublicKey("some-receipt-pubkey"),
});

const signature = await program.provider.send(tx);
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
import { Provider, web3 } from '@project-serum/anchor';
import {
  clusterApiUrl,
  PublicKey
} from '@solana/web3.js';
import { cash, fetchStrangemoodProgram } from '@strangemood/strangemood';

// An anchor-friendly wallet generator
function cashierWallet() {
  let private_key = process.env.CASHIER_KEYPAIR;
  if (!private_key) {
    throw new Error('Unexpectedly did not find process.env.CASHIER_KEYPAIR');
  }
  const keypair = Keypair.fromSecretKey(
    Uint8Array.from(JSON.parse(private_key))
  );

  return {
    signTransaction: async (tx) => {
      tx.sign(keypair);
      return tx;
    },
    signAllTransactions: async (txs) => {
      txs.forEach((tx) => tx.sign(keypair));
      return txs;
    },
    publicKey: keypair.publicKey,
  };
}

const conn = new web3.Connection(clusterApiUrl("mainnet-beta"));
const provider = new Provider(conn, cashierWallet(), {});
const program = await fetchStrangemoodProgram(provider);
  
const { tx } = await cash({
  program,
  signer: provider.wallet.publicKey,
  receipt: new PublicKey("some-receipt-pubkey"),
});

const signature = await program.provider.send(tx);
```
{% endtab %}
{% endtabs %}



