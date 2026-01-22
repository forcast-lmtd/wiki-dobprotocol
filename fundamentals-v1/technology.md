---
description: by Fernando Castillo, CTO.
icon: screwdriver-wrench
---

# Technology

Dobprotocol is built on a blockchain-based architecture  that utilizes smart contracts to enable asset transactions and profit distribution between shareholders. The platform is developed using **Solidity** with support for **Ethereum** blockchain and its Layer-2 derived blockchains such as **Base**.

The core of our platform are the **participation pools**, defined by a **participation token** minted and assigned to the shareholders in each pool. These pools are capable of **deposits, withdrawals and distributions** in any **ERC20** **token** supported by the blockchain or in the **native currency** of the blockchain itself. Dober simplifies the process of asset tokenization and profit distribution through participation pools, smart contracts, and transparent marketplace, providing a secure and efficient platform for shareholders.

Many concepts are involved within the workflow of Dober, such as Factory-pattern, Proxy-pattern or Eternal Storage, among many others. The key concepts and their pipelines are detailed in the following sections.

### 1.1- Participation Pool

Participation Pools are smart contracts that enable asset trading and reward distribution among shareholders, these pools act as collective funds where users acquire participation by providing collateral or acquiring portions of the tokenized asset. The pools are designed to facilitate the sharing of profits among participants according to their ownership percentage supporting deposit, withdrawal and distribution operations within a secure state variable system managed through role-permissions and ownership structures.

#### 1.1.1 Participation Token

The Participation Tokens are used to administer the participation in each Pool. These tokens live within each pool and are minted only once producing a fixed amount and minimal share participation. On an asset tokenization, the creator can decide the division amount of the asset. On Investment pools, the owner can define the minimal share percent allowed per user.

These tokens are defined under the ERC20 Token structure and support all its functions and methods. The total amount of tokens is secured under a locked mint function that blocks the creation of new tokens after the initial mint, producing a fixed quantity of tokens that is used as participation percentage on the pool. These tokens support allowance and native interaction with other ERC20 tokens to enable trade markets through secure entities.

#### 1.1.2 State variables

Each participation pool has its own set of rules and parameters, such as distribution intervals and commission rates. The sensitive data on each pool is protected by its address in an external Storage (explained later in section x.2 Storage and Logic) in a way that each pool can see only its own data. They key state variables in a Participation Pool are:

* **owner**: The address of the owner of the Participation Pool. The owner can transfer prepays and funds to each other, do manual distributions and add more tokens for distributions.
* **operational**: The address of the Dober account used to process the automatic distributions. In the first instance this address pays the network fees, which are then refunded by the Pool after distribution.
* **treasury**: The address of the Treasury Pool which will receive the commissions. On each distribution, a 3% flat commission is applied and assigned to the Treasury Pool.
* **(coef, intercept)**: Regression parameters used to estimate the network fees on distributions made by the Operational account. This estimated gas is then refunded to the Operational gas after distribution.
* **commission**: The flat commission applied to each Distribution. For all assets and general purpose Pools it is a 3% commission.
* **distributionConfig**: The distribution configuration for each currency/token. This configuration involves coin address, distribution intervals and maximum number of distributions.
* **funds**: The amount of currency/token available for distributions.
* **prepay**: The amount of currency available to cover network fees.
* **userFunds**: The amount of currency/tokens assigned to an user after a distribution.

#### 1.1.3. Distributions

Distributions are configured to be automatically processed within a fixed amount of times using an external service. These automatic distributions are conditioned by the network fees, which in first instance are paid by the operational address. Once a distribution is executed, an estimation of the expected network fee is made and used to refund costs to the  Operational address. This estimation comes from a linear regression that produces a fit on the distribution cost for different numbers of participants with minimal error.

<figure><img src="../.gitbook/assets/unnamed (3).png" alt="Dobprotocol technology distribution"><figcaption></figcaption></figure>

Experiment results from evaluating distributions on different numbers of participants on the Mumbai test network. In addition to regression fit parameters (coefficient and intercept), the median gas price is also estimated, which is used in Pool Master Configuration to estimate the complete gas cost for all distributions within a pool.

Distributions can also be executed manually by any participant or the owner of the Pool, in which case the executor must cover the network fees. However the distributions are conditioned to a minimum interval of time between each execution, which is configured on the pool by the Owner. This interval cannot be lower than 12 hours, a safety measure imposed to avoid non-profitable distribution and network saturation.

Once a distribution configuration is completed, e.g., the final distribution date has been reached. The pool can continue to make distributions but in a fixed scheme of 12-hours, and automatic distributions are disabled. It is important for the Owner to properly configure the distribution configuration in a way that the pool can continue to operate during the expected amount of time.

To make a distribution, the contract needs to receive the full list of the participant addresses in order to validate participation shares (requires 100% participation to proceed) and assign the corresponding amounts to each address. The list of addresses is not stored in the pool as such, however, it can be generated from the event logs on the Participation token ERC20 contract. It is strongly recommended to use a central system to query and manage these events. For our platform this service is fully integrated and optimized to guarantee easy user manipulation with the contracts and manages automatic distributions following the rules specified through the state variables on each pool.

#### 1.1.4. Functionalities and interactions

Within a Participation Pool, a shareholder can:

* Deposit funds for distribution or pay gas fees.
* Withdraw rewards for each specific token or currency supported by the Participation Pool.
* Sell or buy participation tokens in the transparent marketplace.
* Force a manual distribution of rewards by paying the corresponding network fees.

Alongside with this, the Owner of each pool can provide additional benefits to the shareholders such as governance decisions in the Pool-related project, third-party promotions or discounts, access to external platforms, etc

#### 1.1.5 Distribution Example

Let's illustrate the use of a Participation Pool. Suppose we have a pool with 5 shareholders, where each one has the following share percentage:

* Address 1 -> 15%
* Address 2 -> 5%
* Address 3 -> 0.5%
* Address 4 -> 70%
* Address 5 -> 9.5%

This pool has 10000 MATIC to distribute, where the 3% (300 MATIC) goes to the Treasury Pool and the remaining 97% (9700 MATIC) goes to participants, assigned according to its respective share percentage:

* Address 1 -> 15% -> 1455 MATIC
* Address 2 -> 5% -> 485 MATIC
* Address 3 -> 0.5% -> 48,5 MATIC
* Address 4 -> 70% -> 6790 MATIC
* Address 5 -> 9.5% -> 921.5 MATIC
* TOTAL -> 9700 MATIC
* COMMISSION -> 300 MATIC

### 1.2. Storage and Logic

Initially, the whole logic and state variables of each Participation Pool were stored in the same smart contract, providing simplified coding and reduced gas costs. However, the logic behind these pools was very complex and vast, producing extremely heavy smart contracts and limiting our capacity for robustness, readability and upgradeability. To solve these problems, we implement a storage/logic pattern which separates state variables from logic, allowing to produce cleaner contracts with support for upgradeability (explained later on x.3. Upgradeable Logic).

The pattern used here is deeply inspired by the Eternal Storage pattern first proposed in 2016, with some modifications for security and transparency. Eternal Storage is a type of contract that defines the base structure to store different data types, defining functions to set or update, get and delete data from the storage. To cover all possible scenarios without the need to change the contract, Eternal Storage is defined to store all the common and basic data types, including uint256, string, boolean, bytes, bytes32 and address. However, the basic structure of Eternal Storage does not guarantee a secure system to make data private for different users.

Private Eternal Storage is our proposed modification. Here we define three different roles: Guardian, Admin and User. Guardian is the main controller of the storage, also known as the owner of the storage. A Guardian can assign and remove the roles Admin and User to any address, but it cannot access storage data. An Admin can add Users but cannot remove them, and just like Guardian, cannot access storage data. Finally, an User cannot assign any role but it can access storage. The following diagram illustrate this interactions:

<figure><img src="../.gitbook/assets/dobprotocol technology storage.png" alt="dobprotocol technology storage"><figcaption><p>Diagram of Role interactions in Private Eternal Storage</p></figcaption></figure>

This role-based mechanism ensures administration of Eternal Storage while also guarantee private data storage for each User. When using this structure within the Dober environment, Users are the Participation Pools and Guardian is a secure Address administered by Dober and used to deploy the main contracts. Admin is the Factory Contract that deploys new Pools.

### 1.3 Upgradeable logic

The separation of Storage and Logic provides the possibility to introduce Upgradeable logic. In simple words, Upgradeable Logic utilizes a Proxy Pattern where a Proxy Contract delegate calls to an Implementation Contract address that can be changed anytime. However, even when the Implementation Contract address changes, the state variables remain the same since they live within the Proxy contract.

When combining this Proxy/Implementation pattern with Eternal Storage and Storage/Logic patter we can produce a robust environment for our contracts development and deployment where the data is stored in a permanent storage location and the logic for every contract can be changed freely for each contract with low deploy fees and without altering the state variables!. All of this is administered by a Main contract called PoolMaster that follows the Factory Pattern for deployment of new Participation Pools through the Proxy pattern. The following diagram illustrates this whole relationship:

<figure><img src="../.gitbook/assets/dobprotocol upgradeable logic.png" alt="dobprotocol upgradeable logic"><figcaption></figcaption></figure>

Example Diagram of Participation Pools deployed through Pool Master using Proxy/Implementation and Storage/Logic patterns. Illustrate cases where each Proxy can point to different Implementation addresses.

This means that each Pool Owner can decide whether to update or not its Logic to a new version by paying the fee required to change the Implementation address. The PoolMaster manages all the versions but the final decision to upgrade a contract will always be of each Pool Owner. The PoolMaster itself can (and will) be deployed using the same proxy-pattern and logic/storage-pattern used for Participation Pool, allowing upgradeability on the creation of upgradeable pools.

We implement the Upgradeability Logic Following the Proxy pattern described in [EIP1967](https://eips.ethereum.org/EIPS/eip-1967) and the UUPS proxiable pattern described in [EIP1822](https://eips.ethereum.org/EIPS/eip-1822).

### 1.4 Participation Market

Participation Market is a helper contract designed to aid the transaction of Participation Token using a transparent and secure structure. This contract lives within the ERC20 patterns and works with its functions. In particular, Participation Market asks for each seller to authorize an allowance for him and administrate sales producing records and calculating commissions.

There are two main use cases for the Participation Market. One is for initial public sales of Dober tokens (explained in Tokenomics) and the other is for marketplace transactions. Just like distributions of Participation Pools, marketplace transactions have a commission of 3% which is sent to Dober funds for a variety of usages explained in Tokenomics. A user can sell through our marketplace any number of Participation Tokens with the following steps:

1. Set an allowance for the desired number of Participation Tokens to sale, assigned to the Participation Market contract.
2. Configure sale properties such as token price and minimum sale unit.
3. At any time the allowance or the sale properties can be updated.

However, only 1 sale can be configured per user per Participation Token, which prohibits a user from defining different prices for different minimum sale units. Putting a token for sale has no cost more than network gas fees and commissions are applied on each effective sale.

When a user wants to buy Participation Tokens it must provide the exact amount of tokens to buy, which should be a multiple of the minimum sale unit, and the paid amount, which should match exactly the total price of bought tokens. In any other case the transaction will revert.
