---
sidebar_position: 2
---

# Action Templates

Game's `Room` holds a set of all action templates.

With Action Templates you're helping to decipher players intention during play. They may interact with UI in many different ways at any time they wish. It's our job to filter out "illegal" moves.

Action template is simply a definition of:

- `interaction` - _what did client touch/say?_
- `conditions` - _can client perform this action?_
- `command` - _what exactly should be done?_

Here's how a basic action may look like:

```ts title="./actions/takeOneCard.ts"
import { commands, ActionTemplate, Deck, Hand } from "@cardsgame/server";

import { MyGameState } from "./state";

export const TakeOneCard: ActionTemplate<MyGameState> = {
  name: "TakeOneCard",
  description: `Player grabs new card from the deck`,

  interaction: () => [
    {
      type: "deck",
    },
  ],

  conditions: (con) => {
    con().itsPlayersTurn();
  },

  command: (state, event) => {
    const topDeck = state.query<Deck>({ type: "deck" }).getTop();
    const playersHand = state.query<Hand>({ owner: event.player });

    return new commands.Sequence([
      new commands.ChangeParent(topDeck, playersHand),
      new commands.NextPlayer(),
    ]);
  },
};
```

## 1. `interaction`

Interactions definition may relate to player interacting with an in-game object, or sending a custom command.

### A) _Player interacts with game entity_

Define `interaction` as a function which returns an array of `QuerableProps`.

:::danger TODO: link that somewhere?

Read the full description of `QuerableProps`.

:::

```ts
interaction: (player: Player) => QuerableProps[]
```

Describe what kind of elements relate to this action by their props. This is the first place to quickly filter out unrelated interactions. Reference to `player` is provided if you need to query by ownership.

#### Examples

```ts
// Any deck which belongs to interacting player
interaction: (player: Player) => [
  {
    type: "deck",
    owner: player,
  },
];
```

```ts
// Try executing this action when player
// taps any deck named "mainDeck"
interaction: () => [{ type: "deck", name: "mainDeck" }];
```

```ts
// Any classic card,
// of suit either Hearth or Spades,
// sitting inside a container named "playersHand"
interaction: () => [
  {
    type: "classicCard",
    suit: ["H", "S"],
    parent: {
      name: "playersHand",
    },
  },
];
```

### B) _Player is sending a custom messge_

Custom message may relate to user clicking some UI button, but it's completely up to you.

```ts
interaction: string;
```

#### Example

In `interaction` field provide the name of event related to this action:

```ts
interactions: "passTurn";
```

Client may send more details available in `data` field of the message (typed `ClientPlayerMessage`), which will be available to be asserted in the next step: "conditions".

## 2. `conditions`

_Is players intention legal?_

Use [`conditions` framework](./conditions.md), first and only argument of `conditions` function, to define a set of rules for this action. If one of these rules fail, the action will be ignored.

[`conditions`](./conditions.md) have references to the player, their whole event object and current game's state. You can use its API to construct easily readable assertions.

```ts
// Example for card interaction
// You can name it `con` for short.
checkConditions: (con) => {
  con().itsPlayersTurn();

  // Grab current player's `hand` and remember it
  // under alias "chosenCards"
  con({
    type: "hand",
    parent: {
      owner: con.getPlayer(),
      type: "container",
    },
  }).as("chosenCards");

  // Change subject to previously remembered "chosenCards"
  // and ensure its got nothing inside.
  con("chosenCards").children.not.empty();
};

// Example for custom command with expected additional data
checkConditions: (con) => {
  con().itsPlayersTurn();
  con("data").its("suit").equals("S");
};
```

If any of these conditions fail, the whole action is disregarded, and internal CommandsManager will simply try checking the next available Action Template.

[Read more about Conditions](./conditions.md).

## 3. `command`

Construct and return an actual `Command` to execute.

```ts
command: (state, event) => {
  const source = state.query<Hand>({
    type: "hand",
    owner: event.player,
  })
  const cards = source.getSelectedChildren<ClassicCard>()
  const pile = state.query<Pile>({ type: "pile" })

  return new commands.Sequence("PlayCards", [
    new commands.ChangeParent(cards, pile),
    new commands.FaceUp(cards),
    new commands.NextPlayer(),
  ])
},
```
