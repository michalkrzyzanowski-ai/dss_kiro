# Implementation Plan: The Last Fare (core mission, REQ-LF-01 to REQ-LF-05)

REQ-LF-06 and REQ-LF-07 (receipt storage and printing) are follow-up scope and have no tasks here.

- [x] 1. Add mission state and cab stop
  - Add `missionState`, `passenger`, and the `CAB_STOP` constant (`sx(3)`, `sy(2)`, r 40)
  - Extend `reset()` to set `collecting` and an empty passenger
  - _Requirements: REQ-LF-01.4, REQ-LF-04.1 to 04.6_

- [x] 2. Reveal the passenger on the third battery
  - Add `onThirdBattery()`: set `waiting`, place the passenger at the stop, show the "drive to the cab stop and press E" hint
  - _Requirements: REQ-LF-01.1, 01.2, 01.5, 01.6_

- [x] 3. Implement boarding
  - Add pure `tryBoard`, `isStopped`, `inPickupZone`, `handleBoardKey`
  - Wire `e`/`E` in the keydown handler
  - Show the "drive to the portal" hint on a successful board
  - _Requirements: REQ-LF-02.1 to 02.8, REQ-LF-05.1_

- [x] 4. Change portal entry and win handling
  - Win only with an occupied seat via `enterWon()`; otherwise show "Pick up your passenger"
  - Keep the dark-portal hint; ignore input once `won`
  - _Requirements: REQ-LF-03.1 to 03.6_

- [x] 5. Rendering
  - `drawCabStop()` ring and passenger sprite; passenger drawn inside the cab while aboard
  - _Requirements: REQ-LF-01.3, REQ-LF-02.2_

- [x] 6. Help text
  - Document the E key and the new goal
  - _Requirements: REQ-LF-05.3_

- [x] 7. Verify
  - Scripted scenario run covering reveal, board gating, empty-cab hint, win, input lock, and restart
