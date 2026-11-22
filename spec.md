Concrete Pour Planner

Purpose

A planning tool for a concrete pour. The user outlines a slab on satellite imagery or types its dimensions, and the app computes how much concrete to order, how many truckloads it takes, how many trucks the batch plant needs to keep the pour going without a break, and when each truck should arrive.

Users

A construction management student, superintendent, or project engineer planning a slab pour and calling in the order to a batch plant.

Platform

A single `index.html` file with plain HTML, CSS, and JavaScript.

The map uses Leaflet 1.9.4 from unpkg and Esri World Imagery tiles:

`https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}`

The required Esri map attribution must be displayed. No API keys or server-side services are required.

Inputs

Slab outline:

- Provide separate “Draw slab” and typed-dimension controls. The drawing mode and typed-dimension mode are alternate ways to define the same slab.
- Drawn mode: a “Draw slab” button starts drawing. The user clicks to add each corner and finishes by double-clicking or clicking the first point. A polygon requires at least three vertices and may not be self-intersecting.
- If the user is drawing and selects “Draw slab” again, cancels drawing, or starts a new drawing, discard the unfinished polygon and begin again. A completed slab remains visible if drawing is canceled.
- Corners of a completed drawn outline can be dragged afterward. Dragging immediately updates the slab area, label, and all calculations.
- Typed mode: provide separate Length and Width fields in feet. A typed rectangle is drawn on the map with north-south and east-west edges centered at the map center.
- Switching to typed mode replaces the current drawn outline with a rectangle using the typed values. Switching to draw mode clears the typed rectangle and starts a new drawn slab. Each mode retains no separate slab state.
- Length and Width must be finite decimal values greater than 0. Do not impose a maximum dimension. Negative, zero, or nonnumeric values are invalid.
- On an invalid or empty input, show an inline validation message near that field and do not produce a misleading result. The summary and timeline retain no calculated values until all required values are valid.

Slab thickness:

- Entered in inches. Default: `6`.
- It must be a finite decimal value greater than 0.

Waste allowance:

- Entered as a percentage. Default: `5`.
- It must be a finite decimal value greater than or equal to `0`. No upper limit is imposed.

Truck capacity:

- Entered in cubic yards per full truckload. Default: `10`.
- It must be a finite decimal value greater than `0`.

One-way travel time from the batch plant:

- Entered in minutes. Default: `20`.
- It must be a finite decimal value greater than `0`.

Placement rate:

- Entered in cubic yards per hour and represents the sustained crew placement rate. Default: `30`.
- It must be a finite decimal value greater than `0`.

Washout and reload time at the plant:

- Entered in minutes per truck. Default: `10`.
- It must be a finite decimal value greater than or equal to `0`.

Time the first truck should arrive:

- Entered as a local time. Default: `7:00 AM`.
- Display and accept both a 12-hour time with AM/PM and the browser-supported time control value. A schedule must remain valid if it crosses midnight.
- It must be a valid time. Empty or invalid values do not update the schedule.

All numeric inputs permit decimal values unless stated otherwise. Labels include their units, and default values are visible or prefilled. Every input change updates the valid calculations immediately.

Calculations

Slab area:

- For a drawn outline, calculate in square feet using:

  `A = R² × |sum over edges of (lon2 - lon1) × (2 + sin lat1 + sin lat2)| / 2`

  with angles in radians, `R = 6,378,137 m`, and `1 m² = 10.7639 ft²`.

- For a typed rectangular slab, calculate `length × width` in square feet.
- Display area to a practical precision but do not use a rounded display value as the basis for subsequent calculations.

Volume to order:

- `area × thickness / 12 / 27 × (1 + waste / 100)`
- The result is rounded up to the next `0.5 cubic yards`.
- Do not round the unrounded volume before truck assignment.

Truckloads:

- `truckloads = ceil(volume / truck capacity)`.
- Every load is full truck capacity except the final load. For example, a 47-cubic-yard order with 10-cubic-yard trucks produces 10, 10, 10, 10, and 7 cubic yards.
- The first truckload uses the first truck, the next uses the next truck, and loads continue cyclically through the trucks in rotation.

Unload times:

- Full-truck unload time in hours: `truck capacity / placement rate`.
- Express this value in minutes when constructing arrivals and schedules.
- A partial final load uses `partial load volume / placement rate` hours, so it does not wait for a full unload period.

Truck arrivals:

- Truck 1 arrives at the first-truck arrival time.
- Truck `k` arrives at `start + (k - 1) × full-truck unload time`.
- The unload time of the final partial truck does not change the spacing of prior arrivals.
- Arrival times wrap across midnight and are displayed in local clock time.

Round trip:

- `2 × one-way travel + full-truck unload time + washout`.
- Each component is in minutes.
- Use the full-truck unload time, not the partial final load's unload time.

Trucks needed in rotation:

- `round trip / full-truck unload time`, rounded up.
- Limit the result to the number of truckloads.
- The result is at least 1 and represents the cyclic pool of trucks needed to keep unloading continuous.
- When the number of loads is 1, trucks in rotation is 1.
- The first load is assigned to truck 1, the second to truck 2, and so on. After truck N is assigned a load, the next load returns to truck 1.

Pour duration and finish time:

- Pour duration begins at the first-truck arrival and ends when the final load is empty.
- `pour duration = (truckloads - 1) × full-truck unload time + partial-or-full unload time for the final load`.
- If the final load is full, use the full-truck unload time for the last load.
- Finish time is the first-truck arrival plus the pour duration, wrapping across midnight if necessary.
- Do not include subsequent return trips or washout after the pour is finished.

Outputs

The map:

- Keep the slab outline visible after it is drawn or typed.
- Show a slab label with the area and ordered volume. Label text is formatted with readable units, for example “2,400 ft² • 47.0 cy”.
- For an invalid slab or input, remove misleading calculated text from the label and show the nearest applicable validation message.

Summary cards:

- Display concrete to order in cubic yards.
- Display truckloads as an integer.
- Display trucks in rotation as an integer.
- Display pour duration in hours and minutes, or in minutes when under 1 hour.
- Display finish time in local clock time.
- Summary values use the rounded order volume and practical time rounding. Unit labels remain visible.

Truck schedule:

- Show one row per truckload. Number rows by load and also show the cyclic truck number assigned to the row.
- Each row shows travel out, unloading, trip back, and washout in distinct colors on a clock-time axis.
- Model the row as the assigned truck's timing sequence; it must account for that truck receiving later loads at its next cycle.
- The unloading segment for a full load has the full-truck unload duration. The final partial-load segment uses its partial-load unload duration.
- Use a clock-time axis beginning at or before the first truck arrival and extending through all segments needed to display the scheduled rows.
- Hovering shows exact start and end times. The timeline must also provide keyboard focus or a screen-reader text equivalent for the same times.
- When the schedule crosses midnight, label or clearly distinguish the relevant clock times.

Order summary:

- Show a plain-text batch-plant order containing ordered volume, number of loads, first-truck arrival time, spacing between truck arrivals, and the cyclic truck requirement.
- Use each full load's size and identify the final partial load in the load details.
- Include a “Copy” button that copies the plain-text order to the clipboard.
- After a successful copy, give brief visible feedback such as “Copied.” If clipboard access fails, keep the text visible and report that automatic copying was unsuccessful.

Updates:

- Every valid input, corner drag, and slab mode change updates the map, summary cards, timeline, and order summary immediately.
- Invalid input states display validation messages and do not update or overwrite valid results with a calculated result derived from invalid values.

Look and feel

- Open with an example slab already drawn near the center of the map so that the app is useful immediately.
- The example must not alter the default input values. It may be deleted, replaced, or redrawn using the draw controls.
- Use a clean, professional construction-app style: white cards with soft shadows, one accent color, and a modern system font with no required external font.
- The accent color must have sufficient contrast for text and controls.
- Place the map and inputs on the left and the summary, timeline, and order summary on the right on wide screens.
- Stack map, inputs, summary, timeline, and order summary vertically on phone layouts.
- Make inputs, the Draw slab button, corners, timeline rows, and Copy button keyboard accessible where practical. Provide clear labels and visible focus states.
- Provide a short next-step hint that reflects the current state, such as “Choose Draw slab and click at least three corners,” “Drag a corner or enter dimensions,” or “Correct the highlighted input.”
- On narrow screens, preserve readable timeline labels. Allow horizontal scrolling for the clock-time axis rather than compressing times until unreadable.

Validation

Typed slab 40 ft × 60 ft × 6 in, 5% waste, 10 cy trucks, 20 min travel, 30 cy/hr, 10 min washout, first truck 7:00 AM:

- Area: `2,400 ft²`.
- Unwasted volume: `44.44 cy`.
- Volume including waste: `46.67 cy`.
- Ordered volume: `47.0 cy`, rounded up to the next 0.5 cy.
- Truckloads: `5` (10, 10, 10, 10, and 7 cy).
- Full-truck unload time: `20 min`, so loads arrive at 7:00, 7:20, 7:40, 8:00, and 8:20 AM.
- The 7-cy final load unloads in `14 min`.
- Round trip: `2 × 20 + 20 + 10 = 70 min`.
- Trucks in rotation: `4`.
- Pour duration: `1 h 34 min`.
- Finish time: `8:34 AM`.
- Truck assignments repeat cyclically, for example: truck 1, truck 2, truck 3, truck 4, then truck 1.

Map-area example:

- An outline with corners `(40.2500, -111.6500)`, `(40.2510, -111.6500)`, `(40.2510, -111.6490)`, and `(40.2500, -111.6490)` has an area of approximately `101,800 ft²`, within 1%.

Typed slab 20 ft × 20 ft × 4 in, 0% waste, 10 cy trucks:

- With the remaining default values unchanged, area is `400 ft²`.
- Unwasted volume is `4.94 cy`.
- Ordered volume is `5.0 cy`.
- Truckloads: `1`.
- Trucks in rotation: `1`.
- Full-truck unload time: `20 min`.
- Pour duration: `10 min`.
- Finish time is `7:10 AM` when the first truck arrives at the default 7:00 AM.

Additional acceptance behavior:

- Volume display rounds to one decimal place. Truckloads and trucks in rotation are integers.
- A final partial load may not be treated as a full load for unload time or pour duration.
- Drawn outline calculations must match the stated 1% geographic-area tolerance.
- Typed 40 × 60 dimensions must take precedence over geographic shape and produce the exact rectangular result above.
- Changing an input or dragged corner must update all dependent outputs without a page reload.
- The schedule must show cyclic truck assignments rather than only reporting the count.
- Invalid or empty values must show validation and must not update with calculated values derived from those invalid values.
- Arrival schedules must wrap correctly past 11:59 PM and across midnight.
- Completing or canceling a slab drawing must leave no impossible, self-intersecting, or incomplete result.

Out of scope

- Reinforcement
- Finishing crews
- Pump trucks
- Weather
- Costs
- Saving plans
- Printing or PDF export
- Unit conversion
- Geocoding
- User accounts
