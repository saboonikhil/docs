---
sidebar_position: 10
description: >-
  Set the source Archive and the block range
---

# Initialization

:::info
The method documentation is also available inline and can be accessed via suggestions in most IDEs.
:::

:::info
If contract address(-es) supplied to `EvmBatchProcessor` are stored in any wide-scope variables, it is recommended to convert them to flat lower case. This precaution is necessary because same variable(s) are often reused in the [batch handler](/evm-indexing/context-interfaces) for data filtration, and all contract addresses in batch context data are **always** in flat lower case.
:::

The following setters configure the global settings of `EvmBatchProcessor`. They return the modified instance and can be chained.

**`setDataSource({archive?: string | undefined, chain?: string | undefined})` (required)**: Sets the blockchain data source. An archive, a RPC endpoint or both can be specified.

+ `archive`: An archive endpoint providing the data for the selected network. See [supported networks](/evm-indexing/supported-networks) for a list of endpoints for public EVM Archives and a usage example. The endpoints are also published to the [Archive registry](/archives/overview/#archive-registry) and exposed with the `lookupArchive` function of `@subsquid/archive-regitry` package.

  If omitted, the processor will do all the data ingestion over the RPC endpoint.

+ `chain?`: A JSON-RPC endpoint for the network of interest. May be specified as an URL string or as an object (see below). HTTPS and WSS endpoints are supported. Required in most cases, or more precisely when
  * the squid has to follow the chain with latency below about half a day, or
  * the processor has to make [contract state queries](/evm-indexing/query-state).

  Specifying `chain` as an object allows for fine-tuning the connection. The object is typed as follows:
  ```ts
  {
    url: string

    // max number of concurrent connections, default 10
    capacity?: number

    // default 100
    maxBatchCallSize?: number

    // requests per second, default is no limit
    rateLimit?: number

    // in milliseconds, default 30_000
    requestTimeout?: number
  }
  ```

  If `chain` is not set, the processor will stop syncing once it reaches the highest block available at the Archive. If you need this behavior but also need an RPC for use in [state queries](/evm-indexing/query-state), use the `useArchiveOnly()` setter described below.

[//]: # (???? update the latency figure once the dust settles)

**`setFinalityConfirmation(nBlocks: number)`**: Sets the number of blocks after which the processor will consider the consensus data final. Use a value appropriate for your network. For example, for Ethereum mainnet a widely cited value is 15 minutes/75 blocks. **Required for RPC ingestion** (i.e. whenever a `chain` was supplied to `setDataSource()`, but `useArchiveOnly()` was not called or was unset).

**`setBlockRange({from: number, to?: number | undefined})`**: Limits the range of blocks to be processed. When the upper bound is specified, processor will terminate with exit code 0 once it reaches it.

**`useArchiveOnly(yes?: boolean | undefined)`**: Explicitly disables data ingestion from an RPC endpoint. Use this if you are making an Archive-only squid that also needs to [query contract state](/evm-indexing/query-state).

**`setChainPollInterval(ms: number)`**: Sets the RPC poll interval in milliseconds. Default: 1000.

## RPC ingestion of traces and state diffs

Archives use the [debug API](https://geth.ethereum.org/docs/interacting-with-geth/rpc/ns-debug) to get [traces](../traces) and the [trace API](https://openethereum.github.io/JSONRPC-trace-module) to extract [state diffs](../state-diffs). The default behavior of the processor during RPC ingestion is the same, but it can be overridden.

**`useDebugApiForStateDiffs(yes?: boolean | undefined)`**: Use the debug API to get state diffs. **WARNING:** this will significantly increase the amount of data retrieved from the RPC endpoint. Expect download rates in the megabytes per second range.

[//]: # (???? Check the validity of the traffic claim on release)

**`preferTraceApi(yes?: boolean | undefined)`**: Use the trace API to retrieve traces. Useful when requesting traces without state diffs: under these condition the processor will use the `trace_block()` method when ingesting trace data, and it is a very cheap call on many node providers.

## Less common settings

**`setPrometheusPort(port: string | number)`**: Sets the port for a built-in prometheus metrics server. By default, the value of PROMETHEUS_PORT environment variable is used. When it is not set, processor will pick an ephemeral port.

**`includeAllBlocks(range?: Range | undefined)`**: By default, processor will fetch only blocks which contain requested items. This method modifies such behavior to fetch all chain blocks. Optionally a `Range` (`{from: number, to?: number | undefined}`) of blocks can be specified for which the setting should be effective.
