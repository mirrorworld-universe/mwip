# Wallet UI Configuration Specification

## Description

The user interface of the wallet will be made configurable using a configuration object as a query parameter in the wallet UI URL. The configuration schema should contain the configurable options split into 4 main fields:

1. `defaults`  - Will define some basic top-level defaults that will be inherited by the subsequent 3 options. e.g. Default chain, currencies, network, language, e.t.c.
2. `uiConfig` - Handles all visual and theming configurable options on the Wallet UI (e.g. titles, logo, font family e.t.c. Anything visual should be defined under this object.
3. `tokenConfig` - Handles anything to do with the tokens displayed to the users. (Inherits chain and network parameters from the `defaults` key.)
4. `collectionConfig` - Handles anything to do with the NFT collections displayed to the user. (Inherits chain and network parameters from the `defaults` key.)

## Configuration Schema

```tsx
/**
 * Supported chains on the Mirror World Ecosystem
 */
type Chain = "solana" | "ethereum" | "polygon" | "bnb-chain";
/**
 * Supported chain cluster
 *
 */
type Network = "mainnet" | "testnet";
type FiatCurrency = "EUR" | "USD" | "CNY";
type AuthProvider = "google" | "facebook" | "discord" | "twitter";

/**
 * Default options for the Wallet UI configuration schema
 */
interface WalletUIConfigSchemaDefaults {
  /**
   * The name of the chain to be displayed by default to the user
   */
  chain: Chain;
  /**
   * Chain network for the current chain to be selected
   */
  network: Network;
  /**
   * UI currency displayed to the user
   */
  fiatCurrency: FiatCurrency;
  /**
   * If `true` this will prevent the user
   * from changing the chain or network
   * @optional
   * @default true
   */
  strictMode?: boolean;
}

/**
 * Options for configuring the appearance of the
 * wallet UI when open
 */
interface WalletConfigUIOptions {
  theme: {
    /** Logo of the project (should be an absolute URI) */
    logo: string;
    /* Title of the project */
    title: string;
    /* Header text displayed at the top of the wallet page. */
    header: string;
  };
  /**
   * Authentication configuration options
   */
  auth: {
    /**
     * Allowed Authentication providers
     * to be displayed to the user
     */
    providers: AuthProvider[];
  };
}

/**
 * Options for configuring the appearance of the
 * tokens page UI when open
 */
interface WalletConfigTokenOptions {
  /**
   * Options for tokens
   */
  tokens?: {
    /**
     * List of tokens to be displayed to the user.
     * If array is empty or undefined, we display all the
     * tokens the user has by default
     *
     * Accepts a list of token mint addresses for the
     * corresponding chain.
     */
    include: string[];
  };
}

/**
 * Options for configuring the appearance of the
 * NFT Collections page UI when open
 */
interface WalletConfigCollectionsOptions {
  /**
   * Options for tokens
   */
  collections?: {
    /**
     * List of NFT collections to be displayed to the user.
     * If array is empty or undefined, we display all the
     * collections the user has by default
     *
     * Accepts a list of NFT collection addresses for the
     * corresponding chain.
     */
    include: string[];
  };
}

**interface WalletUISchemaConfig {
  /**
   * API Key of the project.
   * @required
   */
  apiKey: string;
  /**
   * Wallet UI defaults from with `uiConfig`, `tokenConfig`
   * and `collectionConfig` inherit their options.
   * @optional
   */
  defaults?: WalletUIConfigSchemaDefaults;
  /**
   * UI configuration options
   * @optional
   */
  uiConfig?: WalletConfigUIOptions;
  /**
   * Token page configuration options
   * @optional
   */
  tokenConfig?: WalletConfigTokenOptions;
  /**
   * NFT Collection page configuration options
   * @optional
   */
  collectionConfig?: WalletConfigCollectionsOptions;
}**
```

## Methodology

The developer will pass the configuration object encoded as base64 or as JWT into the URL. When decoded, the config object should expose the parameters defined by the developer and the wallet UI should respond to the provided parameters.

The final configuration token result encoded as JWT should look something like this

```
eyJjbGllbnQiOiJCQ1k5YVlzaDhpR3NoUXV6TmpCYk9OWUUtdEtEMEpNMzg5bDg3SWlNT1ZlT1UxVEJtUmFacGhLT3lwaGtVcG80MWZ1U01uTzZRUmxsb3hDVi0zbnQ4ZFUiLCJjdXJyZW50TG9naW5Qcm92aWRlciI6Imdvb2dsZSIsInBvcHVwV2luZG93IjoiZmFsc2UiLCJwaWQiOiIyZWE5YWE0Y2I4Y2E4OTg5MWM0MTEzYTNkNGI3OTNjODc2N2Y0MGRhNWI4Y2E0Njc5ZDIxMmRiYzVlYThjNDY2Iiwid2hpdGVMYWJlbCI6IntcImxvZ29EYXJrXCI6XCJodHRwczovL2ltYWdlcy50b3J1c3dhbGxldC5pby93ZWIzYXV0aC1sb2dvLXdoaXRlLnN2Z1wiLFwibG9nb0xpZ2h0XCI6XCJodHRwczovL2ltYWdlcy50b3J1c3dhbGxldC5pby93ZWIzYXV0aC1sb2dvLXdoaXRlLnN2Z1wifSIsImtleU1vZGUiOiJ2MSIsImlzQ3VzdG9tVmVyaWZpZXIiOiJmYWxzZSIsImluc3RhbmNlSWQiOiIyaHdpcXNtaDh0MyIsInZlcmlmaWVyIjoidG9ydXMiLCJ0eXBlT2ZMb2dpbiI6Imdvb2dsZSIsInJlZGlyZWN0VG9PcGVuZXIiOmZhbHNlfQ
```

### Terminologies

From here on out, the following list defines the terms used in this specification implementation guide:

1. **UI Methods**: Refers to all SDK methods on the public interface that invoke or open the wallet UI, namely (`**openWallet`** and **`login`** )

### Implementation Guide for SDKs

All client-side SDKs that support UI integration should be modified to satisfy the following conditions for all methods that invoke the wallet UI, namely (`**openWallet`** and **`login`** )

- [ ]  The UI Methods should accept 4 parameters as options:
    - [ ]  `path` - type `string` *(optional)*
    - [ ]  `walletUIConfig` - type `string` *(optional)*
- [ ]  The SDK should implement the following methods internally:
    - **`private validateWalletConfig(walletUIConfig: WalletUISchema) => boolean`**
        - **Description:** This method accepts the schema configuration and validates it according to the **`WalletUISchema`** specification.
        - **Usage:** This method should be called every time the user seeks to configure the Wallet UI. opening the wallet UI in the UI methods to validate that the configuration provided by the SDK consumer indeed satisfies the schema.
        - **Returns: `boolean`**
    - **`public openWallet({ path: string, walletUIConfig: WalletUIConfig }) => Promise<void>`**
        - **Description:** This method opens the user wallet with the schema configuration and validates it according to the **`WalletUISchema`** specification.
        - **Usage:** This method opens the wallet UI at the path specified by the params:
            - **`path`** - type `string` *(optional) Defaults to `/`*
            - **`walletUIConfig`** - type `WalletUIConfig` *(optional)*
    - **`public login({ walletUIConfig: walletUIConfig }) => Promise<void>`**
        - **Description:** This method opens the user wallet with the schema configuration at the login page. If the config is provided, it should be validated according to the **`WalletUISchema`** specification and then open the Wallet UI at the login page.
- [ ]  When the UI Methods are called, before opening the wallet UI, the validation of the user’s configuration must pass. If the validation passes, the SDK should then create a URL with the following query parameters:
    
    ```tsx
    interface WalletUIQueryParameters {
    	/*
    	 * [OPTIONAL]
    	 * URL path to the page you want to direct the user
    	 * @default "/"
    	 * @example "/tokens" or "/"
    	 */
    	path?: string;
    	/*
    	 * [OPTIONAL]
    	 * The Wallet UI configuration object serialized as JWT
    	 * @example "eyJjbGllbnQiOiJCQ1k5YVlzaDhpR3NoUXV6TmpCYk9OWUUtdEtEMEpNMzg5bDg3SWlNT1ZlT1UxVEJtUmFacGhLT3lwaGtVcG80MWZ1U01uTzZRUmxsb3hDVi0zbnQ4ZFUiLCJjdXJyZW50TG9naW5Qcm92aWRlciI6Imdvb2dsZSIsInBvcHVwV2luZG93IjoiZmFsc2UiLCJwaWQiOiIyZWE5YWE0Y2I4Y2E4OTg5MWM0MTEzYTNkNGI3OTNjODc2N2Y0MGRhNWI4Y2E0Njc5ZDIxMmRiYzVlYThjNDY2Iiwid2hpdGVMYWJlbCI6IntcImxvZ29EYXJrXCI6XCJodHRwczovL2ltYWdlcy50b3J1c3dhbGxldC5pby93ZWIzYXV0aC1sb2dvLXdoaXRlLnN2Z1wiLFwibG9nb0xpZ2h0XCI6XCJodHRwczovL2ltYWdlcy50b3J1c3dhbGxldC5pby93ZWIzYXV0aC1sb2dvLXdoaXRlLnN2Z1wifSIsImtleU1vZGUiOiJ2MSIsImlzQ3VzdG9tVmVyaWZpZXIiOiJmYWxzZSIsImluc3RhbmNlSWQiOiIyaHdpcXNtaDh0MyIsInZlcmlmaWVyIjoidG9ydXMiLCJ0eXBlT2ZMb2dpbiI6Imdvb2dsZSIsInJlZGlyZWN0VG9PcGVuZXIiOmZhbHNlfQ"
    	 */
    	config: string;
    	/*
    	 * [OPTIONAL]
    	 * The current users' auth token JWT
    	 * @example "eyJjbGllbnQiOiJCQ1k5YVlzaDhpR3NoUXV6TmpCYk9OWUUtdEtEMEpNMzg5bDg3SWlNT1ZlT1UxVEJtUmFacGhLT3lwaGtVcG80MWZ1U01uTzZRUmxsb3hDVi0zbnQ4ZFUiLCJjdXJyZW50TG9naW5Qcm92aWRlciI6Imdvb2dsZSIsInBvcHVwV2luZG93IjoiZmFsc2UiLCJwaWQiOiIyZWE5YWE0Y2I4Y2E4OTg5MWM0MTEzYTNkNGI3OTNjODc2N2Y0MGRhNWI4Y2E0Njc5ZDIxMmRiYzVlYThjNDY2Iiwid2hpdGVMYWJlbCI6IntcImxvZ29EYXJrXCI6XCJodHRwczovL2ltYWdlcy50b3J1c3dhbGxldC5pby93ZWIzYXV0aC1sb2dvLXdoaXRlLnN2Z1wiLFwibG9nb0xpZ2h0XCI6XCJodHRwczovL2ltYWdlcy50b3J1c3dhbGxldC5pby93ZWIzYXV0aC1sb2dvLXdoaXRlLnN2Z1wifSIsImtleU1vZGUiOiJ2MSIsImlzQ3VzdG9tVmVyaWZpZXIiOiJmYWxzZSIsImluc3RhbmNlSWQiOiIyaHdpcXNtaDh0MyIsInZlcmlmaWVyIjoidG9ydXMiLCJ0eXBlT2ZMb2dpbiI6Imdvb2dsZSIsInJlZGlyZWN0VG9PcGVuZXIiOmZhbHNlfQ"
    	 */
    	authToken: string;
    }
    ```
    
    The URL generated should match the following fragment schema:
    
    ![Untitled](Wallet%20UI%20Configuration%20Specification%200a1d4b5dbf144f2cb6625dc130901fb7/Untitled.png)
    

- [ ]  After generating the URL, the SDK can now open the Wallet URL at the following URL:
    
    In TypeScript SDK for example, the resulting URL would look like this:
    
    ```tsx
    window.open("https://auth.mirrorworld.fun/auth/login?**config**=eyJjbGllbnQiOiJCQ1k5YVlzaDhpR3NoUXV6TmpCYk9OWUUtdEtEMEpNMzg5bDg3SWlNT1ZlT1UxVEJtUmFacGhLT3lwaGtVcG80MWZ1U01uTzZRUmxsb3hDVi0zbnQ4ZFUiLCJjdXJyZW50TG9naW5Qcm92aWRlciI6Imdvb2dsZSIsInBvcHVwV2luZG93IjoiZmFsc2UiLCJwaWQiOiIyZWE5YWE0Y2I4Y2E4OTg5MWM0MTEzYTNkNGI3OTNjODc2N2Y0MGRhNWI4Y2E0Njc5ZDIxMmRiYzVlYThjNDY2Iiwid2hpdGVMYWJlbCI6IntcImxvZ29EYXJrXCI6XCJodHRwczovL2ltYWdlcy50b3J1c3dhbGxldC5pby93ZWIzYXV0aC1sb2dvLXdoaXRlLnN2Z1wiLFwibG9nb0xpZ2h0XCI6XCJodHRwczovL2ltYWdlcy50b3J1c3dhbGxldC5pby93ZWIzYXV0aC1sb2dvLXdoaXRlLnN2Z1wifSIsImtleU1vZGUiOiJ2MSIsImlzQ3VzdG9tVmVyaWZpZXIiOiJmYWxzZSIsImluc3RhbmNlSWQiOiIyaHdpcXNtaDh0MyIsInZlcmlmaWVyIjoidG9ydXMiLCJ0eXBlT2ZMb2dpbiI6Imdvb2dsZSIsInJlZGlyZWN0VG9PcGVuZXIiOmZhbHNlfQ"_
    ```
    

## Example Implementation

```tsx
import jwt from "jsonwebtoken";

interface MirrorWorldOptions {
  apiKey: string;
  walletUIConfig?: WalletUISchemaConfig;
}

interface OpenWalletOptions {
  path?: string;
  walletUIConfig?: WalletUISchemaConfig;
}

export class MirrorWorld {
  private _apiKey: string;
  private _walletUIConfig?: WalletUISchemaConfig;
  private _authToken: string;
  constructor(options: MirrorWorldOptions) {
    this._apiKey = options.apiKey;
    if (options.walletUIConfig) {
      this._walletUIConfig = options.walletUIConfig;
    }
  }

  /**
   * This method accepts the schema configuration and
   * validates it according to the WalletUISchema specification.
   * @param config
   * @returns boolean
   */
  private validateWalletConfig(config: WalletUISchemaConfig): boolean {
    // Test config to validate schema
    if (config === null) {
      return false; // invalid
    } else {
      return true; // valid
    }
  }

  /**
   * Opens Wallet UI
   */
  public async openWallet({ path = "/", walletUIConfig }: OpenWalletOptions) {
    const config = walletUIConfig || this._walletUIConfig;

    if (config) {
      // First validate config
      const isValidConfig = this.validateWalletConfig(config);
      if (!isValidConfig) throw new Error("Invalid configuration provided");
      const encodedConfig = jwt.sign(config, this._apiKey);

      // Open wallet with config
      const url = `https://app.mirrorworld.fun${path}?apiKey=${this._apiKey}&config=${encodedConfig}&authToken=${this._authToken}`;
      await window.open(url);
    } else {
      // Open wallet without config
      const url = `https://app.mirrorworld.fun${path}?apiKey=${this._apiKey}&authToken=${this._authToken}`;
      await window.open(url);
    }
  }

  /**
   * Opens login page of wallet UI
   */
  public async login({ walletUIConfig }: OpenWalletOptions) {
    const config = walletUIConfig || this._walletUIConfig;

    if (config) {
      // First validate config
      const isValidConfig = this.validateWalletConfig(config);
      if (!isValidConfig) throw new Error("Invalid configuration provided");
      const encodedConfig = jwt.sign(config, this._apiKey);

      // Open wallet with config
      const url = `https://app.mirrorworld.fun/auth/login?apiKey=${this._apiKey}&config=${encodedConfig}&authToken=${this._authToken}`;
      await window.open(url);
    } else {
      // Open wallet without config
      const url = `https://app.mirrorworld.fun/auth/login?apiKey=${this._apiKey}&authToken=${this._authToken}`;
      await window.open(url);
    }
  }
}
```

## Usage Examples

### Example 1: Predefinition | JS

```tsx
import { MirrorWorld, createWalletUIConfg } from "@mirrorworld/web3.js"

// Create wallet UI config before initializing SDK.
const walletUIConfig = createWalletUIConfig({ /* ... */ })

const mirrorworld = new MirrorWorld({
	apiKey: "xxxxxx-xxx-x-x-x-xx",
	walletUIConfig
})

async function onClickLoginButton() {
	// The login method should parse `walletUIConfig` and encode it into JWT.
	// The code should be appended to the URL
	// https://auth.mirrorworld.fun/auth/login?walletUIConfig=eyJjbGllbnQiOiJCQ1k5YVlzaDhpR3NoUXV6TmpCYk9OWUUtdEtEMEpNMzg5bDg3SWlNT1ZlT1UxVEJtUmFacGhLT3lwaGtVcG80MWZ1U01uTzZRUmxsb3hDVi0zbnQ4ZFUiLCJjdXJyZW50TG9naW5Qcm92aWRlciI6Imdvb2dsZSIsInBvcHVwV2luZG93IjoiZmFsc2UiLCJwaWQiOiIyZWE5YWE0Y2I4Y2E4OTg5MWM0MTEzYTNkNGI3OTNjODc2N2Y0MGRhNWI4Y2E0Njc5ZDIxMmRiYzVlYThjNDY2Iiwid2hpdGVMYWJlbCI6IntcImxvZ29EYXJrXCI6XCJodHRwczovL2ltYWdlcy50b3J1c3dhbGxldC5pby93ZWIzYXV0aC1sb2dvLXdoaXRlLnN2Z1wiLFwibG9nb0xpZ2h0XCI6XCJodHRwczovL2ltYWdlcy50b3J1c3dhbGxldC5pby93ZWIzYXV0aC1sb2dvLXdoaXRlLnN2Z1wifSIsImtleU1vZGUiOiJ2MSIsImlzQ3VzdG9tVmVyaWZpZXIiOiJmYWxzZSIsImluc3RhbmNlSWQiOiIyaHdpcXNtaDh0MyIsInZlcmlmaWVyIjoidG9ydXMiLCJ0eXBlT2ZMb2dpbiI6Imdvb2dsZSIsInJlZGlyZWN0VG9PcGVuZXIiOmZhbHNlfQ
	await mirrorworld.login()
}
```

### Example 2:  Inline definition | JS

```tsx
import { MirrorWorld, createWalletUIConfg } from "@mirrorworld/web3.js"

// First initialize SDK
const mirrorworld = new MirrorWorld({
	apiKey: "xxxxxx-xxx-x-x-x-xx",
	walletUIConfig
})

// User clicks dApp login button
async function onClickLoginButton() {

	// Create UI config just before opening wallet UI
	// according to `WalletUIConfig` schema
	const walletUIConfig = createWalletUIConfig({ /* ... */ })

	// Login method
	await mirrorworld.login({
		walletUIConfig // <- Provide wallet UI config to login method.
	})
}
```

### Example 3: Login with Google and Twitter Only | JS

```tsx
import { MirrorWorld, createWalletUIConfg } from "@mirrorworld/web3.js"

// First initialize SDK
const mirrorworld = new MirrorWorld({
	apiKey: "xxxxxx-xxxxx-xxxxx",
})

async function onClickLoginWithGoogleAndTwitterOnly() {

	// Create UI config just before opening wallet UI
	// according to `WalletUIConfig` schema
	const walletUIConfig = createWalletUIConfig({
		uiConfig: {
			auth: {
				providers: ["google", "twitter"]
			}
		}
	})

	await mirrorworld.login({
		overrides: {
			walletUIConfig
		}
	})
}

onClickLoginWithGoogleAndTwitterOnly()
```

### Example 4: Open wallet with Ethereum chain selected by default | JS

```tsx
import { MirrorWorld, createWalletUIConfg } from "@mirrorworld/web3.js"

// First initialize SDK
const mirrorworld = new MirrorWorld({
	apiKey: "xxxxxx-xxxxx-xxxxx",
})

// Opens wallet with Ethereum token displayed as default
async function onClickOpenEthereumWallet() {

	// Create UI config just before opening wallet UI
	// according to `WalletUIConfig` schema
	const walletUIConfig = createWalletUIConfig({
		defaults: {
			chain: "ethereum",
			network: "mainnet"
		}
	})

	// Opens wallet UI with Ethereum as the selected
	await mirrorworld.openWallet({
		overrides: {
			walletUIConfig
		}
	})
}

onClickOpenEthereumWallet()
```

## Wallet UI

The wallet UI will decode the `config` query param from the URL upon loading the Wallet UI page. When the config has been parsed correctly it should render the UI according to the options specified by the developer.
