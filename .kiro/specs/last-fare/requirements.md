# Requirements Document

## Introduction

"The Last Fare" extends the single-file canvas game Dark Portal (`index.html`, described in `GAME-BRIEF.md`). In the existing game the player drives a ghost cab, collects three batteries to light three lamps, and enters the powered portal to win. This feature adds a ghost passenger. Collecting the third battery powers the portal and also reveals a passenger waiting at a marked cab stop. The player must stop at the cab stop, press E to board the passenger, and deliver the passenger to the powered portal to win.

Requirements REQ-LF-01 to REQ-LF-05 are the core pickup mission. Requirements REQ-LF-06 and REQ-LF-07 (Store ride receipt, Print fare receipt) are planned follow-up scope and are not part of the core pickup mission. Existing behavior (driving, collisions, battery collection, trail marks, hints) is unchanged except where stated.

### Assumptions

- A1. The game has no velocity. "Stopped" means no arrow key (keyboard or on-screen button) is held.
- A2. The pickup zone is a circle centered on a street point with a radius of 40 px on the 1000 x 680 canvas. Streets are 80 px wide, so the whole zone lies on a street. The exact center is a design decision, but it must be on a street, reachable by the cab, and must not overlap a battery, the portal, or the start position.
- A3. The E key is case-insensitive: `e` and `E` behave the same.
- A4. The empty-cab hint text is exactly "Pick up your passenger".
- A5. One receipt is stored per successful delivery, keyed by its ride ID.
- A6. "Local emulated S3" means a local S3-compatible endpoint running on the developer's machine. Bucket and key naming, ride ID format, and how the browser reaches the endpoint are left to design.
- A7. Any AWS resource must use the project's assigned Region. No Region is hardcoded in this spec or in the game. The emulator is local only and no real AWS resource is created for this feature.
- A8. "Win" means the existing win screen is shown and the cab no longer moves.

## Glossary

- **Game**: The Dark Portal canvas game in `index.html`, including the Last Fare mission.
- **Cab**: The purple ghost cab driven by the player.
- **Portal**: The portal at the top-right corner. It is dark until three batteries are collected, then powered.
- **Cab_Stop**: The marked location on a street where the passenger waits.
- **Pickup_Zone**: The circular area around the Cab_Stop within which boarding is allowed (see A2).
- **Passenger**: The single ghost passenger revealed by the third battery.
- **Seat**: The single passenger seat in the Cab. It is either empty or occupied by the Passenger.
- **Empty_Cab**: A Cab whose Seat is empty.
- **Stopped**: No arrow key is held (see A1).
- **Mission_State**: One of: `collecting` (fewer than three batteries), `waiting` (three batteries, Passenger at Cab_Stop), `aboard` (Passenger in the Seat), `won` (win screen shown).
- **Restart**: Pressing R (either case) or activating the Restore button.
- **Receipt**: A stored record of one successful passenger delivery, containing at least the Ride_ID.
- **Ride_ID**: The unique identifier of one delivery and its Receipt.
- **Receipt_Store**: The local S3-compatible storage that holds Receipts.
- **Ride_ID_Display**: The UI element that shows the Ride_ID.
- **Error_Message**: A visible text message shown to the player when a Receipt write fails.
- **Print_View**: The printable version of a saved Receipt.

## Requirements

### Requirement 1: Passenger reveal (REQ-LF-01)

**User Story:** As a player, I want the third battery to reveal a waiting passenger, so that I have a new goal after powering the portal.

#### Acceptance Criteria

1. REQ-LF-01.1: WHILE fewer than three batteries are collected, THE Game SHALL show no Passenger and no Cab_Stop passenger marker on the canvas.
2. REQ-LF-01.2: WHEN the third battery is collected, THE Game SHALL power the Portal and show exactly one Passenger at the Cab_Stop on the same frame.
3. THE Game SHALL draw the Cab_Stop with a visible marker showing the Pickup_Zone extent once the Passenger is revealed.
4. THE Game SHALL place the Cab_Stop on a street such that the Cab can drive from the start position to the Pickup_Zone without leaving the streets.
5. WHEN the third battery is collected, THE Game SHALL show a hint that tells the player to drive to the cab stop and press E.
6. WHEN the first or second battery is collected, THE Game SHALL keep the Portal dark and show no Passenger.

### Requirement 2: Boarding (REQ-LF-02)

**User Story:** As a player, I want to board the passenger only when I have stopped at the cab stop, so that pickup feels deliberate.

#### Acceptance Criteria

1. WHEN E is pressed, AND the Mission_State is `waiting`, AND the Cab is Stopped, AND the Cab center is inside the Pickup_Zone, THE Game SHALL move the Passenger from the Cab_Stop into the Seat.
2. WHILE the Passenger is in the Seat, THE Game SHALL draw the Passenger inside the Cab and remove the Passenger from the Cab_Stop.
3. WHEN E is pressed while any arrow key is held, THE Game SHALL leave the Mission_State, the Passenger and the Seat unchanged.
4. WHEN E is pressed while the Cab center is outside the Pickup_Zone, THE Game SHALL leave the Mission_State, the Passenger and the Seat unchanged.
5. WHEN E is pressed while the Mission_State is `collecting`, THE Game SHALL leave the Mission_State and the Seat unchanged.
6. WHEN E is pressed while the Seat is occupied, THE Game SHALL keep exactly one Passenger in the Seat and show no Passenger at the Cab_Stop.
7. WHEN E is pressed repeatedly or held (key repeat) in any Mission_State, THE Game SHALL board at most one Passenger per Restart cycle.
8. WHEN the Passenger boards, THE Game SHALL show a hint that tells the player to drive to the portal.

### Requirement 3: Delivery and empty-cab portal entry (REQ-LF-03)

**User Story:** As a player, I want the portal to require my passenger, so that the mission is to deliver the last fare.

#### Acceptance Criteria

1. WHEN the Cab enters the powered Portal AND the Seat is occupied, THE Game SHALL set the Mission_State to `won` and show the win screen.
2. WHEN the Cab enters the powered Portal AND the Cab is an Empty_Cab, THE Game SHALL show the hint "Pick up your passenger".
3. WHEN the Cab enters the powered Portal AND the Cab is an Empty_Cab, THE Game SHALL keep the Mission_State unchanged, keep the win screen hidden, and keep the Cab controllable.
4. WHEN the Cab enters the dark Portal, THE Game SHALL show the existing hint "The portal is dark. Find all three batteries." and keep the Mission_State unchanged.
5. WHILE the Mission_State is `won`, THE Game SHALL ignore arrow keys and E presses.
6. THE Game SHALL treat the Portal entry test as the existing distance test (Cab center within the Portal radius of the Portal center).

### Requirement 4: Restart from every mission state (REQ-LF-04)

**User Story:** As a player, I want R to restore the whole game, so that I can replay from any point.

#### Acceptance Criteria

1. WHEN Restart occurs, THE Game SHALL place the Cab at the start position with its initial heading.
2. WHEN Restart occurs, THE Game SHALL show all three batteries uncollected, all three lamps dark, and the Portal dark.
3. WHEN Restart occurs, THE Game SHALL show no Passenger at the Cab_Stop and an empty Seat.
4. WHEN Restart occurs while the Mission_State is `collecting`, `waiting`, `aboard`, or `won`, THE Game SHALL set the Mission_State to `collecting`.
5. WHEN Restart occurs while the Mission_State is `won`, THE Game SHALL hide the win screen.
6. WHEN Restart occurs, THE Game SHALL clear any hint text.
7. WHEN Restart occurs, THE Game SHALL accept R in upper case, R in lower case, and a click on the Restore button with identical results.
8. WHEN Restart occurs, THE Game SHALL require the player to collect the three batteries again before the Passenger reappears.

### Requirement 5: Input handling (REQ-LF-05)

**User Story:** As a player, I want the new E key to behave consistently with the existing controls, so that the game feels predictable.

#### Acceptance Criteria

1. WHEN `e` or `E` is pressed, THE Game SHALL treat both as the board action.
2. WHEN the browser window loses focus, THE Game SHALL release all held arrow keys, so that the Cab counts as Stopped.
3. THE Game SHALL list the E key and its purpose in the on-screen help text.
4. THE Game SHALL keep the arrow key, on-screen button, and R behavior of the existing game unchanged outside the stated changes.

### Requirement 6: Store ride receipt [planned follow-up] (REQ-LF-06)

**User Story:** As a player, I want each completed ride saved as a receipt, so that my delivery is recorded.

#### Acceptance Criteria

1. WHEN the Game enters the `won` Mission_State by delivering the Passenger, THE Game SHALL write exactly one Receipt containing a new Ride_ID to the Receipt_Store.
2. WHEN the Receipt write succeeds, THE Game SHALL show the Ride_ID in the Ride_ID_Display.
3. WHILE a Receipt write is pending or has failed, THE Game SHALL leave the Ride_ID_Display without a Ride_ID.
4. WHEN a Receipt write succeeds, THE Receipt_Store SHALL contain one object for that Ride_ID, and the Ride_ID shown on screen SHALL equal the Ride_ID inside that object.
5. WHEN the Cab enters the powered Portal AND the Cab is an Empty_Cab, THE Game SHALL write no Receipt.
6. IF a Receipt write fails, THEN THE Game SHALL show an Error_Message, show no Ride_ID, and leave no partial or empty Receipt object in the Receipt_Store.
7. IF a Receipt write fails, THEN THE Game SHALL still show the win screen.
8. WHEN Restart occurs, THE Game SHALL write no Receipt and delete no existing Receipt.
9. WHEN Restart occurs, THE Game SHALL clear the Ride_ID_Display and any Error_Message.
10. WHILE the Mission_State remains `won`, THE Game SHALL write no second Receipt for the same delivery.
11. WHEN two deliveries are completed in separate Restart cycles, THE Receipt_Store SHALL contain two Receipts with different Ride_IDs.
12. THE Game SHALL use only a local S3-compatible endpoint for the Receipt_Store and SHALL NOT hardcode an AWS Region.

### Requirement 7: Print fare receipt [planned follow-up] (REQ-LF-07)

**User Story:** As a player, I want to print my fare receipt, so that I have a paper copy of the ride.

#### Acceptance Criteria

1. WHEN a Receipt has been saved for the current delivery, THE Game SHALL show a Print control on the win screen.
2. WHILE no Receipt has been saved for the current delivery, THE Game SHALL show no Print control.
3. WHEN the Print control is activated, THE Game SHALL open a Print_View carrying the same Ride_ID as the stored Receipt.
4. THE Print_View SHALL show the Ride_ID as read from the stored Receipt, not from a new value.
5. IF a Receipt write fails, THEN THE Game SHALL show no Print control.
6. WHEN Restart occurs, THE Game SHALL hide the win screen, the Print control, the Print_View, and the Ride_ID_Display.
7. WHEN the Print control is activated, THE Game SHALL write no new Receipt.
