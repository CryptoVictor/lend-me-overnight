# Lend Me Overnight, Settle It Now: replication package

Code, raw data and analysis behind the paper *Lend Me Overnight, Settle It Now: A Multi-Criteria
Experimental Evaluation of Three Blockchains for Atomic Interbank Credit Settlement*.

**Research question.** Which blockchain is best suited to run an overnight interbank credit protocol
with delivery-versus-payment (DvP) settlement?

**Method.** Multi-criteria decision analysis. Three public EVM networks (Ethereum, Avalanche C-Chain
and Monad), measured on their testnets (Sepolia, Fuji and Monad Testnet), are scored on five
criteria weighted with AHP: resilience 29.78%, contract security 29.78%, latency 15.78%, cost 15.78%
and EVM compatibility 8.88%. Some inputs come from each network's official documentation; the
rest are measured by deploying the same smart contract, unchanged, on each testnet.

> The project documentation is written in Portuguese. Every number reported in the paper comes from
> [`pesquisa/3-resultados/resultados.md`](pesquisa/3-resultados/resultados.md), which the analysis
> script generates.

## Repository layout

```
.
├── contracts/   the evaluated smart contract (Solidity + Foundry), deployed unchanged on all three testnets
├── benchmark/   data collection (Node.js) and analysis (Python), plus how the experiments were run
├── dados/       raw data from the 2026-10-16 collection: one CSV row per transaction, metadata, prices, logs
└── pesquisa/
    ├── 2-metodo/                criteria matrix (weights, scales, threats to validity) and measurement protocol
    ├── 3-resultados/            results, charts and resumo.json (generated, do not edit by hand)
    └── 4-custo-infraestrutura/  monthly infrastructure cost per network (descriptive only, not scored)
```

| Path | Contents |
|---|---|
| [`benchmark/README.md`](benchmark/README.md) | **How the experiments were run and how to repeat them**: measured object, tests T1 and T2, metric definitions, parameters and step-by-step instructions |
| [`pesquisa/2-metodo/matriz-de-criterios.md`](pesquisa/2-metodo/matriz-de-criterios.md) | Single source of the study's decisions: question, networks, criteria, AHP weights, scales, threats to validity and references |
| [`pesquisa/2-metodo/roteiro-medicao.md`](pesquisa/2-metodo/roteiro-medicao.md) | Measurement protocol: tests, failure classification, data format, normalization and sensitivity analysis |
| [`pesquisa/3-resultados/resultados.md`](pesquisa/3-resultados/resultados.md) | Ranking, sensitivity analysis, cost and one section per network |
| [`dados/README.md`](dados/README.md) | Description of every raw data file |
| [`contracts/README.md`](contracts/README.md) | The contract: offer state machine, atomic DvP settlement, tests and deployment |

## Results at a glance

The three testnets were measured in parallel on 2026-10-16, between 22:40 and 23:22 UTC:
1,095 transactions, with no failures.

| Rank | Network | Median latency (T1) | T2 success rate | Cost per credit cycle | Score |
|---|---|---|---|---|---|
| 1 | Monad | 0.99 s | 100% | US$ 0.0014 | 90.1 |
| 2 | Avalanche | 2.64 s | 100% | US$ 0.029 | 86.0 |
| 3 | Ethereum | 11.04 s | 100% | US$ 0.296 | 58.5 |

The ranking holds in all 17 weight scenarios of the sensitivity analysis. With three public EVM
networks running the same code, resilience, security and compatibility give every network the same
score, so the ranking is decided by latency and cost alone.

Costs use mainnet gas prices and token prices sampled during the collection window. Sepolia changed
its gas rules during the study (Glamsterdam upgrade, 2026-10-06), so a credit cycle used 3.50 times
the gas measured on Fuji, which follows the rules still in force on Ethereum mainnet.

## Reproducing the analysis

Requirements: Node 20+, Python 3.10+ with `pandas`, `matplotlib` and `pytest`, and Foundry.

```bash
git clone --recursive <this repository>
cd benchmark
npm install
npm run analisar          # regenerates pesquisa/3-resultados/ from dados/
npm test                  # Node tests (an Anvil integration test needs Foundry)
python3 -m pytest analise/tests
```

Re-running `npm run analisar` on the committed data gives the same ranking, figures and numbers;
only the generation timestamp changes.

To run the contract's own test suite:

```bash
cd contracts
forge build
forge test -vv
```

Repeating the on-chain collection (new deployments, funded test wallets and faucets on each
testnet) is described step by step in [`benchmark/README.md`](benchmark/README.md), section 5.

## License

The contract in `contracts/` is released under the MIT license ([`contracts/LICENSE`](contracts/LICENSE)).
