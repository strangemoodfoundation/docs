# Using Solana & Anchor

Strangemood's node package uses [Solana](using-solana-and-anchor.md#install), a fast, environmentally efficient blockchain. To hack on Solana, **you should install a browser-extension wallet like** [**Phantom**](https://phantom.app)**.**

****

Additionally, Strangemood uses [Anchor](https://github.com/project-serum/anchor), a popular framework for Solana that provides some security benefits and UX improvements. **If you're using React,** **you may want to create a few helpers to interact with Solana through Anchor.**&#x20;

****

{% tabs %}
{% tab title="TypeScript" %}
```typescript
// helpers.ts
import { Transaction, ConfirmOptions } from '@solana/web3.js';
import { Provider } from '@project-serum/anchor';

// Sets up an anchor wallet
export function useAnchorWallet(): any {
  const { publicKey, signTransaction, signAllTransactions } = useWallet();

  return {
    signTransaction: async (tx: Transaction): Promise<Transaction> => {
      if (!signTransaction) throw new Error('Wallet is not connected');
      return signTransaction(tx);
    },
    signAllTransactions: (txs: Transaction[]): Promise<Transaction[]> => {
      if (!signAllTransactions) throw new Error('Wallet is not connected');
      return signAllTransactions(txs);
    },
    publicKey,
  };
}

export function useAnchorProvider(opts: ConfirmOptions = {}) {
  const { connection } = useConnection();
  const wallet = useAnchorWallet();
  const provider = new Provider(connection, wallet, opts);
  return provider;
}
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
// helpers.js
import { Transaction, ConfirmOptions } from '@solana/web3.js';
import { Provider } from '@project-serum/anchor';

// Sets up an anchor wallet
export function useAnchorWallet() {
  const { publicKey, signTransaction, signAllTransactions } = useWallet();

  return {
    signTransaction: async (tx) => {
      if (!signTransaction) throw new Error('Wallet is not connected');
      return signTransaction(tx);
    },
    signAllTransactions: (txs) => {
      if (!signAllTransactions) throw new Error('Wallet is not connected');
      return signAllTransactions(txs);
    },
    publicKey,
  };
}

export function useAnchorProvider(opts = {}) {
  const { connection } = useConnection();
  const wallet = useAnchorWallet();
  const provider = new Provider(connection, wallet, opts);
  return provider;
}
```
{% endtab %}
{% endtabs %}

****
