"I think one missing code I'd like to see is something to tell the requestor that the server recognizes the RPC in question but has specifically chosen not to support it for some reason.
And we would be happy to use something similar for Infura to let end-users know that a particular method is not supported on our platform. It would be ideal if service providers could return an standard JSONRPC error response w/ a specific code like -32xxx method unsupported on this platform so that client libraries can respond accordingly. Basically this would be an error that can tells the client ""yes the node recognized your request as a valid Ethereum RPC listed in this EIP, but we are rejecting it for the reason stated in the error message""."	ryanschenider of Infura
I would like to add "Unsupported" error code which would be a good way for Metamask style sub-providers to tell the client to use a different provider. Not sure if NOT FOUND can be used in this instance...	**tymat**

***

"Joined to express my frustration along with my colleagues: ethereum lacking an RPC spec that follows a W3C / EIP-style standards process almost renders it useless for real applications. At its core ethereum is RPC. Without it...applications just couldn't exist. Major companies are totally relying on a freely-editable wiki for API info to build high-value applications, or trying to, without huge success. User adoption is part of ethereum, but business adoption is also part of it, and business adoption requires stable, standards-based specs like what's presented here.

It's getting and harder to take ethereum seriously as a solution for big business at major companies, so thank you @bitpshr for this.

+1000 to land as draft so geth, partity, and others can help fine-tune over time."	**evalcode**

***

I thought of one more error case that would be useful to have an explicit code for: the concept of exceeding quotas. For example, Geth has a configurable RPC time-out, but currently just unceremoniously closes the TCP stream, would be preferable for it to return a JSONRPC error when the processing deadline is exceeded. Likewise currently there’s nothing preventing a requestor from asking for every event ever published via a poorly crafted eth_getLogs request, would be nice if the clients could prevent that and return an appropriate error. Same for max batch size and maximum payload size for requests. I think all of these cases could be covered by a single quota-related error code with proper message strings. Thoughts?	**ryanschneider**

***

"Seems reasonable; shooting for maximum agnosticism in terms of the messaging itself, what about something like this (which can always be further-clarified by specific clients):

Code	Message	Meaning	Category
-32005	Limit exceeded	Request exceeds defined limit	non-standard"	
"The must in this sentence my need to be changed or wording added to cover RPC implementations which don't implement all methods.

infura doesn't support stateful methods like eth_sign or filters.
trinity doesn't plan to implement any account support so eth_sendTransaction and eth_sign won't be supported.
There is discussion to add an ""Unsupported"" status code which I think is a good way for RPC servers to signal that they do not support an RPC method, however this sentence suggests that not implementing a method would make a JSON-RPC server implementation non-compliant which seems wrong.

This could be updated to just use should instead of must
We could define a subset of the endpoints which are required and the remaining ones are recommended (aka optional)
Open to other suggestions."	pipermeeriam
"I think this needs to be expanded to cover recommended behavior and error codes for the common error cases. This is often where the ambiguity and different behavior across different implementations shows up.

eth_getTransactionByHash: what should the response be if no transaction is found for that hash (if error, what error code is recommended).
eth_getTransactionReceipt:
what should the behavior be if no receipt is found for the provided transaction hash (if error, what error code is recommended)
if the txn hash is known, but unmined should this behave differently than when the txn is just not yet mined.
Same type of questions for all of the other lookups for things like eth_getBlockByHash, etc."	**pipermeeriam**

***

I like the idea of just removing eth_send* and eth_sign* (except send raw) from the specification altogether. I’d rather users not be able to keep their funds in clients either.	**shanejonas**

***

"A test suite isn't a specification, it just tests conformity to some specification. The schema described here is not sufficient to specify certain primitives which all methods rely upon. This is exactly how you get into the situation of strange corner cases that each client implements differently while still being ""to-spec"".

The places where I see the current documentation lacking the most:

definition of pending state
definition of error messages
lack of consensus between client devs over changes
I'm glad that this EIP is working on point 3. But we need strong definitions of the primitives used to build the RPC methods rather than specific test cases which will just lead to overfitting.

It's misleading to label one specific implementation of ambiguous methods as ""correct""."	rpheimer
"What I see as a major hole in these tests is that they don't appear to have a mechanism for setting up starting state. What these tests really should be testing is along the lines of:

Given state A; call RPC method B then C; expect result D

This means tests necessarily need to be more complex than a simple ""given request, expect result"". One way to achieve this (pseudocode only):

initializeGenesisBlock()
transactionHash = submitTransaction(myTransaction)
expect(transactionHash).to.not.be.null
mineAllTransactions()
newBlock = rpc_eth_getBlockByNumber(""latest"")
expect(newBlock.transactions).to.contain(transactionHash)
Alternatively, the test could explicitly setup expected state. For example:

startingState: [
	accounts: [
		{ address: ""0x1234abcd"", eth: ""5"", signingStrategy: ""SIGN_EVERYTHING"" },
	],
	blocks: [
		{ transactions: [], uncles: [], hash: ""0xabcd1234"", ... },
		{ transactions: [...], uncles: [...], hash: ""..."", ... },
	],
	contractState: [
		...
	],
	...
]
Then the test would call a one or more RPC methods and assert that it gets back a very well defined response.

The key here is that all tests should setup the current full chain state, then call a particular RPC method and assert on the results. Building a test suite like this is expensive, but is also valuable for multi-implementation compatibility. Note that this test doesn't care how an implementation achieves the result (implementation details), but it is well defined that given a particular state and an RPC call, an expected outcome is achieved.

As to how to codify ""setup"", I'll leave that up for discussion but I do think it is critical that COMPLETE setup is included in the test. The test should assume nothing about the current state of the world when it executes.

Personally, I'm a fan of the first style because it is generally easier to write tests when the test defines the path to the expected state, rather than requiring that the thing under test has the ability to set the current state. The first mechanism also allows true black box testing, where the test can be run against an endpoint without knowing its implementation. The second mechanism requires that the test have a back-door mechanism to setup current state."	**micah zoltu**

***

"I just now stumled upon this when searching for something like swagger to define and generate documentation for our json-rpc apis. Unfortunately, swagger isn't for json-rpc.

@cdetrio Have you found any good editors to write/generate the specifications above? Or did you do it by hand?
I also found jrgen, which is more geared towards our usecase, but doesn't appear quite as handy as the swagger editor.

I'd really like to have not only a specification, but also an editor which makes writing/validating the specification intuitive and simple. If it involves writing files by hand, then running some npm script to validate and generate docs based on our custom specification file, then I'm afraid it will just bitrot after a while."	**holiman**

***