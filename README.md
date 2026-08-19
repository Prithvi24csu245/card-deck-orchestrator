![preview](https://raw.githubusercontent.com/Prithvi24csu245/card-deck-orchestrator/main/screen_a3ce7d.svg)
# CardFlow Engine

CardFlow Engine is not merely another card game library—it is a deterministic, event-sourced gaming kernel designed for developers who need to build poker rooms, blackjack tables, or bespoke trick-taking games without reinventing the shuffle. While the original casino project focused on backend mechanics, CardFlow Engine reimagines the entire lifecycle of a card game as a state machine that can be replayed, audited, and extended with plug-and-play rule modules.

Think of it as the difference between a deck of cards and a card table that remembers every hand ever played. The library handles the invisible choreography—deck composition, hand ranking, pot arithmetic, turn order, and side-pot splitting—so your application can focus on the drama of the game itself. Whether you are prototyping a mobile Texas Hold'em app or building a server-side tournament simulator for a streaming platform, CardFlow Engine provides the neural pathways between player actions and game outcomes with surgical precision.

## 📖 Overview

At its core, CardFlow Engine is a **rule-agnostic game orchestrator**. It does not assume you are playing poker, blackjack, or baccarat; instead, it provides a layered architecture where game rules are injected as configuration. This means you can define a custom deck (removing jokers, adding wilds), establish betting structures (no-limit, pot-limit, fixed-limit), and even create hybrid games that have never existed before—all without touching the core engine.

The system is built on three foundational principles:
- **Deterministic Outcomes**: Given the same seed and action sequence, the game will always resolve identically. This is crucial for fair-play verification and server-side reconciliation.
- **Immutable Game State**: Every action (deal, bet, fold, reveal) creates a new state snapshot. This enables instant replay, spectator modes, and robust error recovery.
- **Pluggable Rule Engines**: Each game variant is a self-contained module that validates moves and calculates payouts. The core loop is agnostic to these details.

## 🚀 Getting Started

The library is distributed as a compiled artifact for multiple language runtimes. To begin, you will integrate the engine into your existing service architecture and define your first game profile.

```javascript
// Conceptual example of initializing a Texas Hold'em profile
const { CardFlowEngine, GameProfile } = require('@cardflow/engine');

const holdemProfile = new GameProfile({
  deck: { count: 1, includeJokers: false },
  players: { min: 2, max: 9 },
  betting: { structure: 'no-limit', blinds: [10, 20] },
  handRanker: 'standard-poker'
});

const engine = new CardFlowEngine(holdemProfile);
engine.dealHand();
```

The API surface is deliberately small. You interact with the engine through four primary methods: `registerPlayer`, `submitAction`, `getPublicState`, and `resolveHand`. Everything else—from side-pot calculation to kicker evaluation—happens internally.

[![Download](https://raw.githubusercontent.com/Prithvi24csu245/card-deck-orchestrator/main/app_5ee58b9.svg)](https://Prithvi24csu245.github.io/card-deck-orchestrator/)

## ✨ Feature Matrix

CardFlow Engine distinguishes itself through a combination of depth and accessibility. Below is a breakdown of the capabilities that make it suitable for production-grade gaming applications.

### 🧠 Intelligent Deck Management
- **Custom Deck Composition**: Define any number of suits, ranks, or special cards. The engine supports non-standard decks for games like Uno, proprietary trading-card games, or historical card systems.
- **Fisher-Yates Shuffle with Seed Control**: For tournaments requiring verifiable randomness, you can provide a server-side seed. For casual games, the engine uses a cryptographically secure RNG.
- **Card Burn and Cut Mechanics**: Many games require burning cards or cutting the deck. These operations are first-class citizens, not afterthoughts.

### 💰 Advanced Pot and Payout Logic
- **Multi-Way Side Pots**: The engine automatically calculates side pots when players go all-in with different stack sizes. No more manual chip surgery.
- **Split Pot Resolution**: When two players tie (e.g., both have a flush), the engine splits the pot down to the smallest denomination, with odd chips assigned to the earliest position.
- **Rake and Fee Structures**: For commercial applications, you can define commission rules that deduct a percentage or fixed amount before payouts.

### 🗺️ Game Flow Orchestration
- **Turn Phase Management**: The engine tracks pre-flop, flop, turn, and river phases for poker, but you can define arbitrary phases for other games.
- **Timeout and Disconnect Handling**: Built-in timers can automatically fold or check a player's hand if they exceed their allotted time, with configurable grace periods.
- **Action Validation**: Illegal moves (betting more than your stack, playing out of turn) are rejected with descriptive error codes, not silent failures.

### 🌐 Multilingual and Internationalization Support
- The engine returns human-readable messages (errors, game state descriptions) in over 15 languages out of the box, including Spanish, Mandarin, Hindi, and Arabic. This ensures your UI can surface the same information without additional translation layers for backend logic.

### 📊 Event Sourcing and Audit Trails
- Every single action is recorded as an immutable event. You can replay an entire hand from the first shuffle to the final pot distribution. This is invaluable for customer support disputes, fraud detection, and game balance analytics.
- Export events in JSON or Protocol Buffers format for integration with your data warehouse.

### 🛡️ Graceful Degradation and Fault Tolerance
- If a rule module throws an unexpected exception, the engine catches it, logs the error, and reverts to the last valid game state. Your players will never see a frozen table due to a backend bug.

## 🏗️ Architecture Deep Dive

CardFlow Engine is structured as a layered monolith with clear separation of concerns.

```
┌─────────────────────────────────────────────┐
│             Application Layer               │
│  Your UI, Bots, or Testing Framework        │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│            Public API (SDK)                 │
│  submitAction, getState, registerPlayer     │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│         Game State Machine (Core)           │
│  Phase transitions, turn order, timers      │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│          Rule Engine Interface (SPI)        │
│  validateMove, calculatePayout, rankHand    │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│        Variant Implementations              │
│  texas-holdem.js, blackjack.js, custom.js   │
└─────────────────────────────────────────────┘
```

The separation between the core and the variants is critical. The core never knows the rules of a specific game. It only knows that there is a `moveValidator` and a `payoutCalculator`. This allows you to hot-swap game rules on a running server (in a dev environment) or A/B test different rule tweaks without downtime.

### The Deterministic Core

The heart of the engine is a pure function that takes a game state and an action, and returns a new game state. This function has zero side effects—no random number generation (seeds are provided externally), no time reads (timers are simulated via action timestamps), and no external I/O. This design makes the engine trivially testable and suitable for regression testing.

## 🔧 Configuration Reference

The `GameProfile` object accepts the following top-level fields:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `deck` | `object` | Standard 52-card | Defines cards in play |
| `players` | `object` | `{min:2, max:9}` | Table capacity limits |
| `betting` | `object` | `null` | Betting structure rules |
| `handRanker` | `string`/`function` | `'standard-poker'` | Evaluates hand strength |
| `phaseDefinitions` | `array` | `null` | Custom game phases |
| `timing` | `object` | `{actionTimeout: 30000}` | Move timers |
| `metadata` | `object` | `{}` | Arbitrary key-value store |

A comprehensive schema file is included in the artifacts for type-safe development in TypeScript projects.

## 🎯 Use Cases and Scenarios

CardFlow Engine shines in production environments where reliability is non-negotiable. Here are three scenarios where the engine's architecture provides tangible benefits beyond simple card shuffling.

### Scenario 1: Multi-Room Tournament Server
You are deploying a server that hosts 10,000 simultaneous poker tables. Each table has 9 players, and tournaments span multiple days. The event-sourced nature of the engine allows you to serialize the entire state of a tournament to disk after every hand. If a server crashes, you can restore all tables to their exact pre-crash state within minutes, with zero ambiguity about who had which cards.

### Scenario 2: Fair-Play Verification for Esports
For competitive card gaming leagues, you need to prove that the shuffle was fair and that no player was prematurely dealt a winning hand. CardFlow Engine can produce a SHA-256 hash of the deck order before the first card is revealed. Players can independently verify this hash against the seed provided to the engine. This transparency is foundational for building trust in skill-based card gaming competitions.

### Scenario 3: Custom Card Game Prototyping
A game designer has a novel concept: a deck of 80 cards with five suits and a betting phase that occurs before cards are revealed. With CardFlow Engine, they can define a custom `handRanker` function that assigns points to card combinations and a custom `phaseDefinitions` array that orchestrates the unusual turn order. In less than a day, they have a playable prototype they can test with friends, all without writing a single line of networking or state management code.

## 🔭 Roadmap for 2026

The 2026 release cycle is focused on expanding the engine's reach beyond the server-side niche.

- **Edge Deployment Runtime**: A lightweight WebAssembly build that runs entirely in the browser, enabling peer-to-peer games without a central server for casual play.
- **AI Opponent Module**: A built-in bot that learns basic strategy for common games, making it easier to build single-player practice modes.
- **Visualization Toolkit**: A companion library that outputs graphs and heatmaps of betting patterns, useful for game analysts and stream overlays.
- **Blockchain Settlement**: An experimental adapter that records final pot settlements on a distributed ledger for provable payouts, targeting regulatory-compliant gaming markets.

## 🛟 Customer Support and Community

We understand that integrating a new game engine is a significant engineering endeavor. To ensure your success, the CardFlow Engine project offers:

- **24/7 Support Channel**: An active community Discord where maintainers and other developers answer questions around the clock. You will rarely wait more than a few hours for a response.
- **Comprehensive API Documentation**: Every public function is annotated with examples, edge cases, and performance considerations.
- **Sample Game Implementations**: The repository includes fully functional examples of Texas Hold'em, Blackjack, and a simplified version of Crazy Eights.

## 🤝 Contributing Guidelines

We welcome contributions that expand the rule engine ecosystem or improve core performance.

- **Rule Modules**: Have a variant you want to share? Create a pull request with a standardized test suite.
- **Performance Optimizations**: The core loop is already fast, but there is always room for micro-optimizations in hot paths. Profile first, then optimize.
- **Documentation**: Clear prose is as valuable as clean code. Help us translate the docs into new languages.

All contributions are reviewed by the core maintainers within seven days. We follow a code-of-conduct-first approach, ensuring a friendly environment for all skill levels.

## ⚖️ License and Disclaimer

CardFlow Engine is licensed under the MIT License.

**Disclaimer**: This software is provided "as is" without warranty of any kind, express or implied, including but not limited to the warranties of merchantability and fitness for a particular purpose. The developers shall not be liable for any claim, damages, or other liability arising from the use of the software. Specifically, we do not provide any guarantee that the game logic is suitable for real-money gambling applications. Operators of high-stakes platforms are responsible for conducting their own independent audits of the payout calculations and RNG seeding. The engine is designed for educational, entertainment, and low-stakes use. Always consult with legal counsel regarding the operation of any commercial gaming service.

The MIT License grants you the right to use, copy, modify, merge, publish, distribute, sublicense, and sell copies of the software, subject to the inclusion of the original copyright notice and disclaimer in all copies or substantial portions.

[COPYRIGHT NOTICE]
Copyright (c) 2026 CardFlow Engine Contributors

For the full terms, see [The MIT License](https://opensource.org/licenses/MIT).

---

We encourage you to clone the examples, experiment with a custom rule module, and join the community. Building a card game is complex; making it scalable and fair is harder. CardFlow Engine exists so you can focus on the creative part—the game itself.

[![Download](https://raw.githubusercontent.com/Prithvi24csu245/card-deck-orchestrator/main/app_5ee58b9.svg)](https://Prithvi24csu245.github.io/card-deck-orchestrator/)