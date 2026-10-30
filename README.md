# Multi-Cash-Flow Derivatives Hedging – C++ Pricer + C# Hedging Engine over gRPC

Two-service application that **hedges a derivative paying several cash flows** Pᵢ at dates tᵢ
(P = Σ Pᵢ), using real calendar dates.

> Co-authored with **Tariq Sereir** ([@tsereir](https://github.com/tsereir)) – course project, Grenoble INP – Ensimag (2023–2024).

## Architecture

```
 market data + test parameters
            │
            ▼
 ┌─────────────────────────┐   gRPC: PriceAndDeltas   ┌──────────────────────────┐
 │  HedgingEngine (C#/.NET) │ ───────────────────────▶ │  PricerEngine (C++)       │
 │  - rebalancing oracle    │ ◀─────────────────────── │  - Black–Scholes model    │
 │  - portfolio update      │    price, deltas         │  - multi-flow payoff      │
 │  - cash-flow handling    │                          │  - Monte-Carlo pricer     │
 └─────────────────────────┘                           └──────────────────────────┘
            │
            ▼
   output/portfolioN.json  (hedging portfolio over time)
```

- **PricerEngine (C++)**: gRPC server that prices the multi-flow product and its deltas by Monte Carlo,
  conditionally on the observed past and on the monitoring dates already reached.
- **HedgingEngine (C#)**: gRPC client that walks through the market data, decides when to rebalance
  (fixed-period oracle), updates the replicating portfolio and accounts for the cash flows paid.

## Code structure

```
src/PricerEngine/    C++: pricing_server.cpp, pricer.*, OptionMultiFlux.*, BlackScholesModel.*, pricing.proto
src/HedgingEngine/   C#: Hedger, Portfolio, RebalancingOracle, GrpcClient
output/              hedging portfolios produced on the course test cases
```

## Build and run

**Pricer (C++)**: CMake ≥ 3.15, gRPC + Protobuf (`brew install grpc`), [PNL](https://github.com/pnlnum/pnl), nlohmann/json.

```bash
cd src/PricerEngine && mkdir build && cd build
cmake .. -DCMAKE_PREFIX_PATH="/path/to/pnl/build" -DCMAKE_BUILD_TYPE=Release
make && ./pricing_server
```

**Hedging engine (C#)**: .NET SDK.
