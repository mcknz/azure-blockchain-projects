#Project Bletchley - The Cryptlet Fabric
##Cryptlets in Depth 

The promise that blockchain has to revolutionize how business and individuals trade value over the internet is staggering.  Enterprise organizations see the vision for disrupting their industry and themselves.  The atom has been split, so now the race to take advantage of it begins.  

For example, blockchains will need a way to securely receive external data variables as well as access to secure execution of off-chain code.  Security advances in silicon chips will soon introduced a new level of security into CPU architectures called enclaves.  The use of enclaves allows for execution of code to be performed in a secure isolated container in which the results of the computation are attested to be tamper proof.

Bletchley's Cryptlet Fabric introduces a secure, attested third tier into blockchain applications. Given the distributed nature of blockchain this Cryptlet Fabric will function as a service or application fabric in the cloud (Azure/Azure Stack, AWS, Google, Private).  Development of the Cryptlet Fabric will provide opportunities to address some of the most concerning limitations of today’s blockchain platforms like performance, scale and security. 

Cryptlets can be consumed as a service in libraries residing in the cloud or created and hosted in a variety of environments.  Cryptlets will be discoverable and published by both developers and SmartContract modelers to include in application design and development experiences.  Microsoft and other 3rd parties will offer Cryptlet libraries that expose functionality that is both horizontal and vertical in nature.  

Additionally, you may want to develop your own Cryptlets that integrate data or logic from an external system like validating customer's data without disclosing the it to the blockchain itself.  You can create your own Cryptlet, write it in the language of your choice and deploying to either Azure, Azure Stack or localhost* using the Bletchley SDK. 

![<Event UC>](<images/azureRegions.png>)
*As of September 2016 – Updates https://azure.microsoft.com/en-us/regions/*

As both public and consortium blockchains begin to span the globe, Azure’s global scale can deliver a secure execution and data environment in standard open way.  

Although similar to blockchain oracles, Cryptlets provide much more in the way of security and trust in a scalable ecosystem.

Providing enterprise grade software with cutting edge advances in hardware, the Cryptlet Fabric makes a great companion layer for cross cutting capabilities so developers and consortiums can focus on solutions driving distributed computing into new frontiers.

This paper explores some of the features and behaviors for Cryptlets in the Project Bletchley Roadmap.  These features and behaviors are under development and subject to radical change.

**Note - Code examples are prototypes and are for demonstration purposes only**

+ [**Utility Cryptlets**](CryptletsDeepDive.md#utility-cryptlets)
+ [**Contract Cryptlets**](CryptletsDeepDive.md#contract-cryptlets)
+ [**Expected Use Cases**](CryptletsDeepDive.md#expected-use-cases)
+ [**Common Characteristics**](CryptletsDeepDive.md#common-characteristics)
+ [**Cryptlet Development**](CryptletsDeepDive.md#cryptlet-development)
+ [**Cryptlet Registration**](CryptletsDeepDive.md#cryptlet-registration)
+ [**CryptletContainers**](CryptletsDeepDive.md#cryptlet-containers)
+ [**Enclaving**](CryptletsDeepDive.md#enclaving)
+ [**Cryptlet Registration**](CryptletsDeepDive.md#cryptlet-registration)
+ [**Conclusion**](CryptletsDeepDive.md#conclusion)


##Overview
Cryptlets are off-chain code modules that are written in any language that can execute within a secure, isolated, trusted container and is communicated with over secure channels.  Cryptlets can be used in SmartContracts and UTXO systems when additional functionality or information is needed, upgrading the blockchain “oracle” approach with a Cryptlet and CryptoDelegate or adapter.
Utility Cryptlets are organized into libraries or services that perform common functionality like encryption, time based event publication and access to external data.  Utility Cryptlets are used by SmartContract developers who need access to external data or functionality in a trusted way and are referenced at design time.  Utility Cryptlets libraries are provided by the attester and others from 3rd parties like ISVs.  Utility Cryptlets will represent a good portion of “out of the box” Cryptlet Fabric capabilities. (i.e. encryption, authentication, volatile data lookup, notification cryptlets)

Utility Cryptlets can provide data into Message Queue contract for SmartContracts to reference at their leisure.  For example, you could write a Message Queue SmartContract that subscribes to a Market Data Cryptlet that publishes prices from trusted source every 2 hours when the markets are open.  The Message Queue SmartContract can then be used by other SmartContracts to look up prices when they need them.  The Message Queue SmartContract will maintain the entire history of recorded prices with the signature and hashes for audit and replay.

Contract Cryptlets are created and bound to a specific SmartContract instance and is signed by the parent SmartContract and usually runs within an enclave.  The SmartContract using a Contract Cryptlet will inherit property accessors (stored procedures) that implement address checks ensuring that only its Contract Cryptlet can perform certain functions.  The SmartContract and its Contract Cryptlet are bound together with their digital signatures similar to a blockchain wallet.

There are three basic use cases for Cryptlets: Event, Control and Static. Cryptlets are not restricted to only these use cases, these are just the obvious ones.

###Event

Event Driven Cryptlets expose events that SmartContracts can subscribe to.  These Cryptlets provide notification services based on some event criteria and can also optionally pass in secure data upon notification.

In this example, the SmartContract subscribes to the MarketEvent published by the MarketWatcher Cryptlet.  In the subscription it tells the MarketEvent when to wake it up, only if the markets were open that day and what prices it wants when the event is fired.  The MarketWatcher Cryptlet will then wake the SmartContract up with a message at exactly 4:00 PM EST, if the markets were open that day and provide the LIBOR rate and the price of gold in the message.  The message is routed directly to the call back function CalculatePrice() which will then perform the logic with the data provided. 

###Control

Control is the pattern that Contract Cryptlets, described in another section of this paper, follow.  SmartContracts that use Control will delegate all of the business logic that needs to be performed to the Cryptlet.  The SmartContract inherits from the ContractCryptlet class that wires up the signature and meta data required for functionality and implements the data or state with a variable + property accessor to store state.  

![<Control UC>](<images/ControlUC.png>)

In this example, the SmartContract is basically defining a state table, accessor properties to update the data with messages sent by the Control Cryptlet.  This pattern allows for the logic within a SmartContract to be run on a defined number of compute resources (at least 1) and located, scaled and secured independently from the blockchain itself.  

## Common Characteristics
Cryptlets must go through a registration process in order to setup their environment for execution and dependent secure infrastructure.  There are several ways this can be done depending on the Cryptlet type.
	//Utility Cryptlet
	...
	}
```
Cryptlet's configuration uses standard JSON formatting as extensions to the Cryptlet JSON schema:
{
```
Cryptlets get registered with the CryptletHostContainerService, the Cryptlet is tokenized and assigned its public/private key pair, the JSON configuration file and header are recorded to the CryptletRegistrationBlockchain.  Cryptlet Registrar is a blockchain database maintained by the CryptletHostContainerService.  Cryptlet identities and meta data are stored in the blockchain and linked to distributed data stores for meta data and policy.  Utility Cryptlets are registered independently from SmartContracts and are referenced by them, where Contract Cryptlets are registered at runtime when the SmartContract is deployed to the blockchain or the Control Cryptlet Deployment tool in the Bletchley SDK.


![<Contract Cryptlet>](<images/contractCryptlet.png>)


At design time, developers make references to Cryptlets they intend to use recording the references and public keys of these Cryptlets into their code.  For example, the Cryptlet interface in the SmartContract being imported:

Using Ethereum contract’s init() or constructor functions, Cryptlets are assigned an attested host and their reference is stored in the SmartContract and event subscriptions between Cryptlets and the SmartContract can be established.  SmartContracts using cryptlets can derive from the CryptletProxy base class that will provide most of the default configuration.   SmartContracts that will delegate their runtime logic to a Contract Cryptlet will inherit from ContractCryptlet base class.  The CryptoDelegate will work against the base classes to intercept delegation requests and perform security checks.  
//using a Utility Cryptlet
Wiring up event callback functions with attributes and creating proxies for calling Cryptlet static methods, optionally adding access control entries for addresses that are able to respond or call back is performed by the CryptoDelegate.
var stockClient;
	stockClient = StockClient();
Cryptlet behaviors like process isolation using enclaving to run the CryptletContainer and Cryptlet in an isolated enclave can be set in code.  Process isolation can also be set with policy in the CryptletHostContainerService that should always override code settings.

`PropertyPrivacy.processIsolation = true;`
[encryptField='ContractSignersOnly']
A Contract Cryptlet will be declared in the SmartContract itself and can be installed during deployment to the blockchain via the SmartContract constructor.  The SmartContract using full delegation of processing to the Cryptlet will simply store the authentic values and signatures instead of running the logic.  This will allow Contract Cryptlets to be more dynamic and can support scale up models where more performance is required and parallel execution is desired.    
```csharp
Contract CreditDefaultSwap is ContractCryptlet{
SmartContracts that use Contract Cryptlets will need to provide property accessors for values as functions allowing the Cryptlet to update its state on the blockchain.  Inheriting from ContractCryptletBase will provide the default accessors following naming conventions as well as snapshot state functions.  These can be overridden for each variable or as a whole with a ContractStateSnapshot.
Contract CreditDefaultSwapStateSnapshot{
	...
When the SmartContract below is installed into the blockchain the constructor is called.  Because it inherits from ContractCryptlet, its constructor will be passed to the CryptoDelegate for the Contract Cryptlet to be created.  The CryptoDelegate passes the configuration and parameters for the Contract Cryptlet to the CryptletHostContainerService for creation.  
//SmartContract using Contract Cryptlet constructor
```
The CryptletHostContainerService will create the Cryptlet based on the information and code passed in to setup and deploy the Cryptlet into the cloud.  For example, the language/platform: Java, C#, etc. code for the Cryptlet are passed and validated for the Contract Cryptlet to be created, installed and run in the cloud.
geth --cryptoDelegatePath 'https://azure.com/myconsortium' --cryptoDelegateSignature '0x231dw...'

![<Secure Communications>](<images/secureCommunications.png>)

Additionally, Cryptlets and their CryptletContainer could be signed by or include digital signatures from identities that would allow a Cryptlet to do work “on-behalf of” one or more identities.  For example, a user could create a Cryptlet and sign it with their digital signature and when invoked would perform actions as an agent for the user in a business process.  Interesting scenarios can come out of Cryptlets that are signed at design time and those that could potentially be enlisted and signed at runtime to work on behalf of scenarios.

## Cryptlet Development
Cryptlets can be created using the Bletchley SDK.  A developer will need to select one of the languages supported and type desired, which will include all of the references and settings for the CryptletContainer.  For example, a developer can choose to create a Cryptlet in C# without enclaving, which would set the base CryptletContainer to the .NET CORE CryptletContainer with enclaving disabled.

The developer then inherits from either the UtilityCryptlet or ContractCryptlet base class, builds the project and deploys to a test or production environment.

```csharp
public class CustomerValidatorCryptlet as UtilityCryptlet
{
...
}

```

## Cryptlet Registration
Cryptlets are registered with the CryptletHostContainerService which maintains the CryptletRegistrationBlockchain and linked distributed storage.  Registration of a Cryptlet can be done either with the CryptletRegistrationCLI, Deployment Wizard or using the CryptoDelegate for Contract Cryptlets.
CryptletContainers will host a runtime VM environment to support different platforms like Java, C# and perhaps functional languages and is responsible for bootstrapping Cryptlets, managing Cryptlet lifecycle and I/O from the enclave.  CryptletContainers are wrapped around Cryptlets as packages for deployment by either the CryptoDelegate creating a new Contract Cryptlet using the CryptletHostContainerService or by the Cryptlet deployment tool. Initial target runtimes include .NET Core and the JVM which are the most popular in the enterprise.

CryptletContainer provides access to keys for the hosted Cryptlet from the Azure Key Vault, or if using an enclave, a sealed state. Also it provides the I/O interfaces for the Cryptlet to communicate with other enclaves and unsecure processes.  CryptletContainers instantiate Cryptlets providing them their identification and access to private keys for encryption and signing.
class CryptletContainer {
## Enclaving
Use of secure computation via enclaves (isolation of code within a secure boundary) implemented via hardware using hardware or Hypervisor-based solutions provides Cryptlets the environment to perform isolated execution and encryption services.  Building off of MSR’s Secure Computation Interfaces the CryptletHostContainerService, CryptoDelegate and CryptletContainer can implement these interfaces to setup a secure operating environment while abstracting away the actual enclaving technology being used.
```cpp

...
...
![<Cryptlet Bootstrap>](<images/cryptletBootstrap.png>)

Enclaved Cryptlets can fetch and store keys in the Azure Key Vault for various purposes.  The CryptletContainer provides the interfaces for communicating to the KeyVault and other Cryptlets that are in separate enclaves.
The addition of a blockchain Cryptlet Fabric tier can be described as Blockchain 3.0: Data and logic on a blockchain with Cryptlets called via CryptoDelegates from a Smart Contract for off-chain functionality forming a three-tiered architecture.

![<EVOLVE>](<images/evolution.png>)

Creating a formal security model for interacting with the real world and providing interoperability with existing systems in a trusted way will help adoption within enterprise consortiums and provide an additional level of re-use for developers writing cryptlets.  It also opens up an entire Cryptlet Fabric tier for blockchains of any sort and provides a flexible execution environment allowing for partial delegation of logic and full delegation for greater scale.


Cryptlets will provide the infrastructure required to deliver a suite of blockchain functionality in the middle tier as libraries or services initially identified as: Blockchain Gateway Services, Identity & Key Management, Privacy & Encryption as well as Data Services (machine learning, analytics and BI Dashboards) are among them.

##Appendix
