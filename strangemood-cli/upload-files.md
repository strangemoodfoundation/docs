# Upload Files

You may need a place to store files when creating your listing. For example, your listing needs a primary image, images, and videos. We reccomend storing that content on [IPFS](https://ipfs.io) and provide an easy way to do so. However, you are welcome to store files anywhere you want as long as you have a link!

#### File uploads

You can use the `listing file upload` command to store files on IPFS.

```
strangemood listing file upload <listing_address> ~/image.png
```

This will return a [CID](https://docs.ipfs.io/concepts/content-addressing/) that you can reference in your metadata with `ipfs://your_cid`.&#x20;

#### Encrypted uploads

Because [everything stored on IPFS is public](https://docs.ipfs.io/concepts/privacy-and-encryption/#what-s-public-on-ipfs), most sellers will want to encrypt their game before storing it. That is why we build [Precrypt](https://precrypt.org), process for storing encrypted files and only letting them be decrypted by holders of a specific token. You can encrypt your files and delegate access to the Precrypt network using the `-e` command.

```
strangemood listing file upload -e <listing_address> ~/game_files.zip
```

This will return two CIDS:

* File CID: The cid of the encrypted file stored on IPFS
* Key CID: The encrypted key file that only the precrypt node can use to generate decryption keys for users who have purchased the listing.&#x20;

This means that you can store all of your files, including game binaries, on IPFS and only users who have purchased your games access them.&#x20;
