# Wheel Align

A phone-based wheel alignment tool that runs as a Progressive Web App on iPhone. It uses the phone's motion sensors to measure **toe, camber, caster, and thrust angle**, compares the readings with the vehicle's specs while you work, and prints a customer report. Workflow and layout are inspired by the Gyraline G1.

No build step, no server, no accounts. Everything runs in the browser, and your data stays on the phone.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app (UI, sensor math, wizards, report) |
| `manifest.json` | Makes it installable ("Add to Home Screen") |
| `sw.js` | Service worker: offline support and update handling |
| `specs.json` | Shared vehicle spec list (see [Vehicle specs](#vehicle-specs)) |
| `icon-180.png`, `icon-512.png` | App icons |

## Install

1. Put all the files in the root of a GitHub repo.
2. In the repo, go to Settings → Pages and deploy from the main branch.
3. Open the Pages HTTPS address in **Safari** on the iPhone. (iOS only allows motion sensors over HTTPS.)
4. Tap Share → **Add to Home Screen** and launch from the icon.
5. Turn off **Portrait Orientation Lock** in Control Center. The app is landscape only and shows a rotate prompt in portrait.
6. On first launch tap **Grant Sensor Access** and allow Motion & Orientation.

## Calibrate first

The calibration wizard runs on first launch. Run it again any time from **Recalibrate** on the dashboard, and whenever the adapter or phone case changes. There are two flip tests, and each one cancels the tilt of the surface you test on.

1. **Level flip, position 1:** phone in its adapter, flat on a level surface, screen up. Tap Record and keep it still.
2. **Level flip, position 2:** spin the phone and adapter 180° on the same spot. Tap Record. The average removes the adapter's and surface's tilt.
3. **Yaw drift:** leave it still for 10 seconds. The app measures gyro drift and subtracts it from toe readings.
4. **Camber flip, position 1:** hold the phone in its adapter flat against a vertical wall or flat plate, landscape, screen facing out. Tap Record.
5. **Camber flip, position 2:** turn it upside down (180°) against the same spot. Tap Record. The app removes the adapter's camber offset from all camber and caster readings.

Each step shows a picture of the phone position. Every step has a **Skip step** button.

Tips: use the flattest surface you can find for the level flip, and make the adapter touch the wall the way it touches the rim.

## Measuring

The dashboard has a tile for each job. Every job follows the same steps: place the phone, tap **Ready**, level it, tap **Arm capture**, then let go.

**Auto-capture:** once armed, the reading is taken automatically after the phone has stayed level for 0.9 seconds, and a short beep plays (if the iPhone isn't silenced). If the phone drifts out of level, the countdown restarts. Tap the button again to cancel.

**Leveler:** the diamond turns green when the phone is within tolerance (0.15° for toe, 1° for camber and caster).

| Job | What you do |
|---|---|
| **Quick Align** | Total toe. Measure the left wheel, then the right. |
| **Camber: front** | Measure the front left wheel, then the front right. |
| **Camber: rear** | Measure the rear left wheel, then the rear right. |
| **4-Wheel / Thrust** | Rear left, rear right, front left, front right. Gives individual toe for each wheel and a thrust angle. |
| **Caster: left / right** | Set the wheels straight ahead, turn 15° out, then 15° in (past center). The phone's live angle tells you when you're at the target. |

A small car diagram shows the current wheel in blue and finished wheels in green.

### How to hold the phone

- **Toe:** phone lying screen **up**, in landscape, with its long bottom edge against the rim through the adapter. Keep the phone on the **outboard** side of the rim on every wheel. This is what lets the adapter's own angle cancel out of the toe numbers.
- **Camber and caster:** back of the phone flat against the wheel, landscape, screen facing **away** from the car.

Keep the app open between wheels. Toe depends on a continuous yaw reference, so don't close or reload the app partway through a job.

### Sign conventions

- Toe: **+ is toe-in**, − is toe-out
- Camber: **+ is top of the wheel tilted out**
- Caster: **+ is top of the steering axis tilted rearward**
- Thrust angle: **+ is to the right**

## Vehicle specs

Choose a vehicle on the dashboard's **Vehicle & specs** tile. The Year, Make, and Model dropdowns list only vehicles that are in `specs.json` or that you added on the phone. Once a vehicle is selected, the live reading turns green or red against spec while you measure (front and rear camber, the second front-toe wheel, and the 4-wheel toe steps), and the results screen and report show each reading against its spec. Caster is compared on the results screen.

### Adding a vehicle

Tap **+ Add vehicle** and fill in:

- **Year:** a single year (`2023`), a range (`2020-2025`), or blank for all years
- **Make** and **Model**
- **Specs:** a target and a ± tolerance for any of front camber, rear camber, front caster, front total toe, rear total toe, and thrust angle. Use **decimal degrees** (0.17 is 0° 10'), and negative numbers for negative camber.

Toe specs are whole-axle totals. Each wheel's individual toe is judged against half of the total. Use **Edit specs** to change a vehicle and **Delete** to remove one added on the phone.

### Sharing the list across devices

Vehicles added in the app live only on that phone. To keep them permanently, tap **Copy list**, then replace `specs.json` in your repo with what was copied. Every device then gets the same vehicles.

`specs.json` format:

```json
{
  "2020-2025|TESLA|MODEL Y": { "camF": [-0.5, 0.5], "casF": [0, 1], "toeF": [0, 0.1], "toeR": [0, 0.1], "thr": [0, 0.1] },
  "HONDA|CIVIC": { "toeF": [0, 0.1] }
}
```

(The numbers above only show the structure. They are not real specs. Take actual values from the manufacturer's service manual or your alignment data source.)

- Keys are `YEAR|MAKE|MODEL` or `YEAR1-YEAR2|MAKE|MODEL`, in uppercase. `MAKE|MODEL` covers all years.
- Each spec is `[target, tolerance]`.
- Spec keys: `camF` front camber, `camR` rear camber, `casF` front caster, `toeF` front total toe, `toeR` rear total toe, `thr` thrust angle.
- Keys starting with `_` are ignored, so they can hold notes.

## Before / After and reports

The **Stage** button on the dashboard switches between **Before** and **After**. Readings are saved under the current stage.

Open **Report** to enter shop name, customer, technician, plate or VIN, odometer, and notes. **Print or save PDF** opens the iOS print sheet. Use the share sheet there to save a PDF. The report includes:

- vehicle and customer details
- a table of each measurement: before, after, spec, and in or out of spec
- a car diagram with per-wheel toe
- the calibration offsets used and the unit setting

**New job** clears all measurements and customer details.

## Units

The **Units** button in the top bar of the dashboard switches between degrees and minutes (`-1° 30'`) and decimal degrees (`-1.50°`).

## Limits worth knowing

- **Toe accuracy depends on the yaw reference.** iPhones don't give a compass-stable heading in this mode, so yaw comes from the gyroscope and can drift. Calibrate the drift, work in one continuous session, and move steadily between wheels.
- **Thrust angle** is the rear thrust direction compared with the direction of the front axle, because yaw alone can't locate the car's true centerline.
- Verify the numbers against a known reference before trusting them on customer vehicles.

## Updating

Redeploy the changed files to GitHub. The app checks the network first, so closing and reopening it while online brings in the new version. The small **Version** label in the corner of the home screen shows which version the phone is running.

When you release a change, bump the version in two places and keep them in step:

1. `VER` near the top of the script in `index.html`
2. the cache name (`align-vN`) at the top of `sw.js`

If a phone is stuck on an old version, close the app, open the site once in Safari while online, then relaunch. As a last resort, delete the home-screen icon and clear the site's data in Settings → Apps → Safari → Advanced → Website Data. **That also erases the saved calibration, added vehicles, and job data**, so use Copy list first.

## Privacy

Everything is stored in the browser on your phone. The app makes no calls to outside services. It loads only its own files from your hosting.

## The math

For anyone who wants to check the work. The same formulas are in a comment at the top of the script in `index.html`.

- The phone's orientation is a rotation matrix built from the sensor's alpha, beta, and gamma angles. Calibration corrects it for the phone's and adapter's tilt.
- **Toe:** heading of the phone's long axis. Total toe is the difference between the left and right headings. Individual toe is measured against the rear thrust line.
- **Camber:** the tilt of the screen's outward direction from horizontal, read from gravity.
- **Caster:** `(C1 − C2) / (sin T1 + sin T2)`, where C1 is the camber at T1 degrees turned out and C2 is the camber at T2 degrees turned in.
