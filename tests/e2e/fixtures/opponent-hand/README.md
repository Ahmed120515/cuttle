# Opponent-hand reproduction and acceptance checklist

Prepared for T1 / Ahmad with Codex assistance, 2026-11-24.
Baseline: cdb0dcd8. Scope: issue #1290. #1096 was closed as its duplicate.
These are development-only game-state fixtures, not a completed animation fix.

## Reproduce

Start the local app, create a two-player game, and note its game ID. Load one of
these JSON files through the existing development fixture endpoint:

```sh
curl -f -H 'Content-Type: application/json' \
  --data-binary @tests/e2e/fixtures/opponent-hand/hidden.json \
  http://localhost:1338/api/game/GAME_ID/game-state
```

Replace GAME_ID and use the backend port for your stack (1338 with offset 1).
This replaces that test game's state; use a disposable local game only.
The loader defaults to player zero's turn. View the game as player zero.

For Cypress, use the existing setup and fixture commands:

```js
cy.setupGameAsP0();
cy.fixture('opponent-hand/hidden.json').then((state) => {
  cy.loadGameFixture(0, state);
});
```

| Fixture | Action | Expected result |
| --- | --- | --- |
| hidden | Play 8 spades using Glasses | Three opponent backs become 6 hearts, queen hearts, 8 clubs; no card count change. |
| mixed | Inspect, then play 8 spades using Glasses | 6 hearts starts revealed; only the other two need to reveal. |
| last-card | Draw the final deck card | Deck becomes empty and opponent hand is visible. |
| empty-deck | Inspect | All three opponent cards are visible without glasses. |
| remembered | Inspect without glasses | All three previously seen cards remain visible. This is a staged state, not proof of the removal sequence. |

Each file supplies an explicit deck. The fixture loader places all otherwise
unused cards into scrap, making draws reproducible. Revealed-card references
intentionally refer to cards already located in a hand.

## Acceptance criteria for the team's implementation

- Keep one opponent-card rendering path per responsive layout: normal and xs carousel.
- Visibility changes flip affected cards individually in place, with no whole-hand
  slide, fade, or temporary disappearance. Card order/count stay stable.
- Do not expose unknown card faces before a game rule reveals them.
- Preserve remembered reveals: removing glasses must not hide already seen cards.
  See the existing removal sequence in `specs/in-game/revealedCards.spec.js`.
- Verify the final-card reveal and mixed known/unknown hand, not only glasses play.
- Verify at desktop 1280x800 and mobile 390x844. Mobile cards remain reachable,
  and the carousel and surrounding controls are usable without page overflow.
- Repeat reveal checks after reload; persistent game visibility must survive.
- Belard should run existing revealed-card tests and add regression coverage for
  the refactor. Visual animation quality requires a recording/manual observation;
  checking the final DOM alone is insufficient.

## Evidence and remaining review

The earlier local demonstration recorded hidden/revealed states and played a
Glasses eight successfully. It did not establish frame-by-frame animation timing.
The baseline template uses separate visibility branches and SLIDE_UP/SLIDE_DOWN.
The new fixture files have structural validation; each new scenario still needs
an in-app/Cypress execution before being reported as passed.

The maintainer has not replied to T1's #1290 comment as of this check. Preserve
current revealed-card semantics unless maintainers explicitly request otherwise.
Review and explain these AI-assisted artifacts before submitting upstream.
