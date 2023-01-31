# MWIP-3 Programmatic User Authentication with SDK (Developer Access Only)

> This proposal is an upgrade/improvement to the [MWIP-1 Server-side Authentication Proposal](mirrorworld-universe/mwip#mwip#1). 

As previously highlighted in [MWIP-1] this proposal seeks to provide developers with a means to execute write operations on the Mirror World API using an authentication schema that does not compromise the end-user's asset security, but provides a means by which they can perform write operations without having the authentication verification tied to the project credentials.

The missing piece of [MWIP-1] was that it did not cover this use case. In this proposal, [MWIP-3] should cover individual developers which chare projects with other developers. This satisfies the above contrait and allows the following:

1. Mutation of chain state without having to go through action approval logic on the wallet UI. (Useful for back end operations or scripting).
2. Can perform these operations on the server-side where they may not be able to open URIs.
3. For projects with multiple developers, will allow developers to authentic

### Who is affected by this change
- The nature of this change does not affect the end user's user experience.
- It brings an improvement to the Developer Experience for performing programmatic operations to the chain using **the Bearer's own wallet**.


## Specification
In order to login with the Mirror World Smart SDK programmatically, the developer will be required to create an **Access Secret Key**. This key will identify them similar to a JWT auth token programmatically. It can be used to authorize transactions and operations on behalf of your the Bearer's wallet instance.

With the Access Secret Key, the developer can skip the Wallet UI approval step for approved actions. This will allow for scripting use cases and performing programmatic transactions against the Mirror World Smart SDK's API cluster.


### Access Secret Key Creation
The developer will be able to create an Access Secret Key on the [Mirror World Developer Dashboard](https://app.mirrorworld.fun) on the Profile Page &rarr; Security &rarr; Credentials page. If they already have one, they can choose to delete this key or regenerate a new key.

The developer may, after generating the Access Secret Key, copy it and place it store it in a safe place or execution runtime. We shall make thi clear on the developer dashboard that the **The Access Secret Key is a sensitive credential that can be used to perform write operations to the blockchain on their behalf**.

## Proposed Usage
The following procedure will at a high level, illustrate how to mint an NFT for to the developer's walleet on Solana Devnet using this authentication scheme.

### TypeScript/JavaScript Usage

```ts
import { MirrorWorld } from "@mirrorworld/web3.js"

// Create the SDK instance
const mirrorworld = new MirrorWorld({
  apiKey: "YOUR_PROJECT_API_KEY",
  authentication: {
    accessSecretKey: "ACCESS_SECRET_KEY" // <-- Get this from the developer dashboard
  }
})

// Mint NFT on Solana
const mintResult = await mirrorworld.mintNFT({
  name: `NFT_NAME`,
  symbol: `NFT_SYMBOL`,
  metadataUri: "https://my-collection-metadata-uri/metadata.json",
  sellerFeeBasisPoints: 300,
  collection: "9pd6wUcfZpPBsrQFxqEkMjfbyaqraQRsiQtD8D4wqa6W",
}, {
  confirmation: "finalized"
})

```
### HTTP Request Usage Example
The same procedure invoked with the HTTP API request will involve 2 steps:
1. Request Approval Token programmatically for the mint nft action.
2. Send mint nft request to complete mint.

The full list of supported actions can be found [here][supported-actions]

```bash
curl --location --request POST 'https://api.mirrorworld.fun/v1/auth/actions/request' \
  --header 'x-api-key: <API_KEY>' \
  --header 'Authorization: Bearer <ACCESS_SECRET_KEY>' \
  --data-raw '{
    "type": "mint_nft"
  }'
```
See the full Approvals Workflow HTTP API reference [here][approvals-api-reference]

If this request is made with an Access Secret Key, the Mirror World API shall respond with a [Verified Action Object][action-object-type] [i.e. with a `status: verified` property] and append the `authorization_token` to verified action object. This `authorization_token` should then be consumed in the `x-authorization-header` in the next mutation request as shown below:

```bash
curl --location --request POST 'https://api.mirrorworld.fun/v1/devnet/solana/mint/nft' \
--header 'x-api-key: <API_KEY>' \
--header 'Authorization: Bearer <ACCESS_SECRET_KEY>' \
--header 'x-authorization-token: <ACTION_AUTHORIZATION_TOKEN>' \
--data-raw '{
  "collection_mint": "9pd6wUcfZpPBsrQFxqEkMjfbyaqraQRsiQtD8D4wqa6W",
  "name": "NFT_NAME",
  "symbol": "NFT_SYMBOL",
  "url": "https://my-collection-metadata-uri/metadata.json",
  "seller_fee_basis_points": 300,
  "confirmation": "finalized"
}'
```

### Managing Secret Key Security
If the developer has reason to believe that their Access Secret Key is compromised, they may generate a new one from the Security section of the Developer Dashboard. This will invalidate the old one and generate a new one for them. They could delete it the compromised one entirely.

# SDK Maintainers' Note
In order to support this authentication schema, all client-side SDKs support the following:

1. Allow `accessSecretKey` property as an instantiation parameter of the SDK.
2. When performing actions that require approval, detect whether the user has provided an `accessSecretKey`. If so, use this token as teh `Authorization: Bearer XXXX-XXXXX-XXXX` header. Instead, check to see if the Action Request API returns the `authorization_token` in the response payload. If it **DOES NOT** return the `authorization_token` property, then proceed to request approval through the wallet UI if this is a browser environment.
3. Otherwise, skip the UI approval step and pass the `authorization_token` in the actual write operation API.

> You may need to add some environment/language specific logic to assist the developer using your SDK to use it the right way.

<p align="center">
  <img src="https://github.com/mirrorworld-universe/mwip/raw/main/assets/MWIP-3%402x.png" width="400" />
</p>

## Implementation Guide

The following section describes how this specification should be implemented in the SDKs:

|Property|Description|
|---|---|
| Semver Type | Minor (Non-breaking) |
| Target Frameworks | **Rust**, **JavaScript**, **_Java_** |
| **Requirements** | <ol><li>The SDK should accept either one of two authentication tokens: `authToken` or the `accessSecretKey`.</li><li>All authenticated requests should accept either one of the `accessSecretKey` or the `authToken`.</li><li>If the user provides both the `authToken` and the `accessSecretKey`, the `authToken` must have a higher priority than the `accessSecretKey`.</li><li>It is recommended to extract the logic that handles the acquisition of the `authorization_token` from action approval in a separate method / function. Please see the `getApprovalToken` method in the [implementation guide](#implementation-guide) for your reference. </li><li>The SDK should be able to detect whether the current runtime is a browser environment or server environment. If it is a browser environment, please log [this warning](#credential-warning) to the user if they use the `accessSecretKey` to the SDK in the browser environment.</li><li>Your implementation should include tests that ensure that your implementation logic is compliant with the above flow chart is compliant</li></ol> |

### Pseudo Code Implementation
This is an pseudo code implementation of the above logic.

```ts
// SDK source code
export enum ClusterEnvironment {
  mainnet = "mainnet",
  testnet = "devnet",
}

export interface MirrorWorldAuthorizationParams {
  accessSecretKey?: string;
  authToken?: string;
}

export interface MirrorWorldOptions {
  apiKey: string;
  env?: ClusterEnvironment;
  authentication?: MirrorWorldAuthorizationParams;
}

export enum ActionType {
  // ...
  MintNFT = "mint_nft",
  ListNFT = "list_nft",
  TransferSOL = "transfer_sol",
  // ...
}

export class MirrorWorld {
  private _apiKey: string;
  private _env: ClusterEnvironment;
  private _accessSecretKey?: string;
  private _authToken?: string;

  constructor(options: MirrorWorldOptions) {
    this._apiKey = options.apiKey;
    this._env = options.env || ClusterEnvironment.mainnet;
    this._accessSecretKey = options.authentication?.accessSecretKey;
    this._authToken = options.authentication?.authToken;
  }

  /**
   * Helper getter to provide the authentication token.
   * The user's authToken acquired from the wallet UI.
   * 💡 The JWT has a the higher priority over the accessSecretKey.
   */
  private get authenticationCredential() {
    return this._authToken || this._accessSecretKey;
  }

  /**
   * Gets an the approval token for an action.
   *
   * Note: This is pseudo code and it only implements the bare minimum.
   * Please adjust as needed for your SDK framework
   */
  private getApprovalToken({ action }: { action: ActionType }) {
    return new Promise<string>(async (resolve, reject) => {
      // Step 1: Request for approval token.
      // Docs: https://developer.mirrorworld.fun/#e62f77cd-f098-4f17-ab4e-e45ca75e0a4a
      const response = await fetch(`https://api.mirrorworld.fun/v1/auth/actions/request`, {
        method: "POST",
        headers: {
          "x-api-key": this._apiKey,
          /**
           * ====================================================================================
           * 💡 Here we provide the authentication credential used to authenticate the user.
           *    This can be either:
           *     1) Their JWT acquired from the Wallet Login UI, or
           *     2) The user's Secret Access Key.
           * ====================================================================================
           */
          Authorization: `Bearer ${this.authenticationCredential}`,
        },
        body: JSON.stringify({
          type: action,
          // ... Add other params for requesting approval
        }),
      })
        .then((res) => res.json())
        .catch(reject);

      // Step 2: Check whether response contains the `authorization_token` property.
      if (response.data.authorization_token) {
        // Skip UI Wallet approval. Just return token
        return response.data.authorization_token;
      } else {
        // No token is returned. User Action is pending approval. User should approve.
        // Open wallet at the wallet API
        const action_uuid = response.data.action_uuid;
        const popup = window.open(`https://auth.mirrorworld.fun/approve/${action_uuid}`);

        popup?.addEventListener("message", (message) => {
          // User has approved action and authorization oken has been returned
          // Close the popup window
          popup.close();

          // Parse message and return the approval token
          const approvedAuthorizationToken = message.data.authorization_token;
          resolve(approvedAuthorizationToken);
        });
      }
    });
  }

  public async mintNFT(payload: { name: string; symbol: string; metadataUri: string; collection: string }) {
    const mintNFTData = validateUserPayload(payload);

    const action = ActionType.MintNFT;

    // Request for authorization_token
    const authorizationToken = await this.getApprovalToken({ action });

    // Request Mint NFT
    const successFulMintNFTResponse = await fetch(`https://api.mirrorworld.fun/v1/${this._env}/solana/mint/nft`, {
      method: "POST",
      headers: {
        "x-api-key": this._apiKey,

        Authorization: `Bearer ${this.authenticationCredential}`,
        "x-authorization-token": authorizationToken,
      },
      body: JSON.stringify(mintNFTData),
    });

    // Success! return mintNFT response to the user
    return successFulMintNFTResponse;
  }
}
```

### Example Usage
#### Minting NFT (Node.js)
```ts
// In Node.js (No browser environment)
const mirrorworld = new MirrorWorld({
  apiKey: "PROJECT_API_KEY",
  authentication: {
    accessSecretKey: "MY_SECURE_ACCRESS_SECRET_KEY",
  },
});

// Minting an NFT
const mintedNFT = await mirrorworld.mintNFT({
  name: `NFT_NAME`,
  symbol: `NFT_SYMBOL`,
  metadataUri: "https://my-nft-metadata-uri/metadata.json",
  collection: "FihfE3fXMWNTZ1piVxxdc1YnzaAw4cao6WGXz3C7PMu9",
});
```

#### Batch Minting NFTs (Node.js Script)
```ts
import { MirrorWorld } from "@mirrorworld/web3.js";

// Create the SDK instance
const mirrorworld = new MirrorWorld({
  apiKey: "YOUR_PROJECT_API_KEY",
  authentication: {
    accessSecretKey: "ACCESS_SECRET_KEY",
  },
});

async function batchMintNVerifiedCollectionNFTsForMyGame() {
  // 1. Prepare your collection data
  const newCollectionData = {
    name: "MWIP-3 Collection",
    symbol: "MWI",
    metadataUri: "https://my-collection-metadata-uri/metadata.json",
  };

  // 2. Create Verified Collection
  const verifiedCollection = await mirrorworld.createVerifiedCollection(newCollectionData);

  const nftsToMintInCollections = [
    {
      name: `MWIP-3 #1`,
      symbol: `MWIP`,
      metadataUri: "https://my-nft-metadata-uri/1.json",
      collection: verifiedCollection.mintAddress,
    },
    {
      name: `MWIP-3 #2`,
      symbol: `MWIP`,
      metadataUri: "https://my-nft-metadata-uri/2.json",
      collection: verifiedCollection.mintAddress,
    },
    {
      name: `MWIP-3 #3`,
      symbol: `MWIP`,
      metadataUri: "https://my-nft-metadata-uri/3.json",
      collection: verifiedCollection.mintAddress,
    },
    // ... As many as you want to mint
  ];

  const mintedNFTs = await Promise.all(
    await nftsToMintInCollections.map((token) => mirrorworld.mintNFT(token)),
  );

  // ✅ Done - Batch minting completed.
  console.log("New Verified NFTs:", mintedNFTs);
}


// Run your script. Sit back and watch your NFTs mint!
batchMintNVerifiedCollectionNFTsForMyGame()
  .catch(console.error)
```


#### Minting NFT (Browser)
```ts
// In browser environment
const mirrorworld = new MirrorWorld({
  apiKey: "PROJECT_API_KEY",
  authentication: {
    authToken: "MY_AUTH_TOKEN_JWT",
  },
});

// Minting an NFT
const mintedNFT = await mirrorworld.mintNFT({
  name: `NFT_NAME`,
  symbol: `NFT_SYMBOL`,
  metadataUri: "https://my-nft-metadata-uri/metadata.json",
  collection: "FihfE3fXMWNTZ1piVxxdc1YnzaAw4cao6WGXz3C7PMu9",
});
```

### Warnings
|Condition|Messages|
|---|---|
|<a id="credential-warning">User provides `accessSecretKey` in browser environment</a>|We noticed you provided an `accessSecurityKey` in a browser environment. Please make sure not to commit this key to GitHub in a public repository or use it in production in the browser as it can be used to impersonate you if it is compromised. Read more about managing your Access Secret Key's security in the documentation at `https://docs.mirrorworld.fun/further-reading/security/access-secret-key` |

### Proposal Documentation
- This feature will ship the following documentation:
  - Product docs on developer dashbord security credentials management
  - SDK API reference refactor to support `accessSecretKey` param.

### Error Handling
- Escalate all API errors to the client-side. All errors will be escalated to the user's application code with the error messages.
- Only catch errors that necessary to be caught within your logic. Otherwise, please allow the errors to bubble up to the user to allow them to debug the error.

### Internal error scenarios
N/A

### SDKs
|SDK|Link | 
|---|---|
| TypeScript/JavaScript | https://github.com/mirrorworld-universe/mirrorworld-sdk-js |
| Android | https://github.com/mirrorworld-universe/mirrorworld-sdk-android |
| Rust | https://github.com/mirrorworld-universe/mirrorworld-sdk-rust |
| Unity | https://github.com/mirrorworld-universe/mirrorworld-sdk-unity |
| iOS | https://github.com/mirrorworld-universe/mirrorworld-sdk-ios |


### Collaboration
All communication on this development will happen on the Mirror World Slack([DM me](https://twitter.com/codebender828)) and [Discord][mw-discord]

[MWIP-1]: mirrorworld-universe/mwip#mwip#1
[MWIP-3]: mirrorworld-universe/mwip#mwip#3
[developer-dashboard]: https://app.mirrorworld.fun
[mw-discord]: https://mirrorworld.fun/discord
[approvals-api-reference]: https://developer.mirrorworld.fun/#800dcde2-66c3-446b-b157-f7e79c9d255d
[supported-actions]: https://github.com/mirrorworld-universe/mirrorworld-sdk-js/blob/9f863d6a64e7b0e30dfb77b027e8a2c75123312e/packages/web/src/types/actions.ts#L29-L43
[action-object-type]: https://github.com/mirrorworld-universe/mirrorworld-sdk-js/blob/9f863d6a64e7b0e30dfb77b027e8a2c75123312e/packages/web/src/types/actions.ts#L3-L20
