# Create a Listing

The `strangemood listing init` is used to create listings from the CLI. You can see a full list possible parameters with `strangemood listing init --help` (`--help` works on all commands).&#x20;

#### Strangemood charter

One of the most important parameters is `--charter`, it determines a set of parameters on the program and where fees go. You must use the same charter to work with other Strangemood clients and listings. The current Strangemood charter is deployed at the following addresses:

* Testnet: `5ugGJvE1RiYhDWnTgNFQL6RVq6SUo3wzJiPAZ3XZuckF`
* Mainnet-beta: `D8hxZ192cEtaWBYTQfpLKDtHPUMjd8AVkAAVTdLyLujx`&#x20;

You can also [create your own charter](create-a-charter.md) if you want.&#x20;

#### Create listing example

Here is an example of using the create listing command to make a listing with a price of 60 Sol:

```bash
strangemood listing init --charter=5ugGJvE1RiYhDWnTgNFQL6RVq6SUo3wzJiPAZ3XZuckF --decimals=0 --price=60 --uri=ifps://2MqnSpSRMsGZmT78qn8nFjQk5bTLWUFhnW9faWq6vnQm
```

If your command is successful it will return the new listings address.

#### Manage Listings

You can see the listings you've created with `strangemood listing ls` and use the following commands to manage them.

Set a listing as available/unavailable for sale:

```
strangemood listing set isAvailable EnXZJoYDhr7jTM7B41jRMbb2wXAQLPo93aSCUA7LdcK3 true
```

Update a listings metadata uri:

```
strangemood listing set uri EnXZJoYDhr7jTM7B41jRMbb2wXAQLPo93aSCUA7LdcK3 ifps://2MqnSpSRMsGZmT78qn8nFjQk5bTLWUFhnW9faWq6vnQm
```
