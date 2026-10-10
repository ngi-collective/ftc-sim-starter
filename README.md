# ftc-sim starter

The FTC SDK (FtcRobotController v12.0, BIOBUZZ 2026-2027), set up to test your OpModes against
[ftc-sim](https://github.com/ngi-collective/ftc-sim), a simulated robot on a simulated field.

- **Tests** drive your OpModes with no robot attached and check where the robot ends up.
- **A Driver Hub in your browser**: a 3D field, gamepad input, the robot's camera view and the
  live score.
- **The competition APK is unchanged.** The simulator lives in a separate `simulated` build, and
  the `robot` build that goes to a Control Hub contains none of it.

Not affiliated with or endorsed by FIRST. FIRST Tech Challenge and FTC are trademarks of FIRST.

## Start a new team repository

1. Click **Use this template** → **Create a new repository**.
2. Clone it, and **keep SDK updates mergeable** (once, right after creating it):

   ```bash
   git remote add upstream https://github.com/FIRST-Tech-Challenge/FtcRobotController.git
   git fetch upstream --tags
   git merge -s ours --allow-unrelated-histories v12.0 -m "Record FTC SDK v12.0 as merged"
   git push
   ```

   A repository made from a template starts with a single commit and no link to FIRST's history.
   This merge changes no files. It records that your code already contains SDK v12.0, so when
   FIRST releases the next SDK, `git merge upstream/master` brings in only what changed. Skip it
   and that merge conflicts on every file the SDK touches.

3. Open the folder in **Android Studio** (Narwhal 3 Feature Drop or later, the floor SDK 12.0
   sets). Its bundled JDK and SDK manager are all the build needs.

Already have a team fork of FtcRobotController? You don't need this repository. Add the plugin to
your own `TeamCode/build.gradle` as described in the
[ftc-sim README](https://github.com/ngi-collective/ftc-sim#use-it-in-your-teams-code).

## Run the tests

In Android Studio's Gradle tool window: **TeamCode** → **Tasks** → **verification** →
**testSimulatedDebugUnitTest**. Or in a terminal:

```bash
./gradlew :TeamCode:testSimulatedDebugUnitTest
```

Outside Android Studio, `./gradlew` needs `JAVA_HOME` set to a JDK between 17 and 21. Android
Studio's bundled JDK works. With Studio in its default location:

```bash
# macOS
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
# Windows, in Git Bash
export JAVA_HOME="/c/Program Files/Android/Android Studio/jbr"
```

Without it, `./gradlew` runs whatever `java` is first on your PATH, which may be missing, the
wrong version, or a tool manager's shim that has no version set for this folder.

Two tests come with it:

- `MecanumDrivePoseTest` runs `BasicMecanumTeleOp` with simulated gamepad input and checks the
  robot's pose on the field: forward stick drives forward, strafe goes sideways, twist turns in
  place, no power stays put, and a wall stops the robot while its encoders keep counting.
- `SyntheticCameraAcceptanceTest` runs FIRST's own `ConceptAprilTagEasy` sample, unmodified,
  against the simulated camera, and reads the sample's own telemetry: it must report the red
  HIVE's AprilTags, by their season names, at a plausible range.

GitHub Actions runs them on every push, and builds the competition APK
(`.github/workflows/test.yml`).

## Drive it in a browser

```bash
./gradlew :TeamCode:dashboard
```

Then open http://localhost:8765. Pick an OpMode, press INIT and START, and drive with a gamepad or
the keyboard. Your OpModes appear there as they do on a Driver Station.

## Make it your robot

- **`TeamCode/robot-config/example.json`** is the robot's physics: chassis size and mass, wheel
  radius, gear ratio, motor speed, encoder resolution, camera mount. Measure yours and edit the
  numbers, or copy the file under a new name. The simulator reads it again at every INIT.
- **`TeamCode/src/simulated/java/.../simulated/ExampleRobot.java`** names the devices, the same
  names your Robot Controller configuration uses. Names are code so that a rename breaks the
  build. If you rename the class, update
  `TeamCode/src/simulated/resources/META-INF/services/org.ngicollective.ftcsim.hardware.SimulatedRobot`,
  which registers it.
- **`TeamCode/build.gradle`** sets the simulator and season versions in its `ftcSim { }` block.
  New releases are listed in the
  [ftc-sim repository](https://github.com/ngi-collective/ftc-sim).

The plugin compiles TeamCode against Android SDK 34, because the simulator's vision tests need it.
Upstream uses 30. The minimum and target SDK, and so the Control Hubs your APK runs on, stay the
same.

## What is here, beyond FIRST's SDK

| Path | What it is |
| --- | --- |
| `TeamCode/build.gradle` | the `org.ngi-collective.ftc-sim` plugin and its `ftcSim { }` block |
| `TeamCode/robot-config/` | the example robot's physics, and the field |
| `TeamCode/src/main/java/.../BasicMecanumTeleOp.java` | a robot-centric mecanum TeleOp, with `DriveHardware`, `ChassisMotors`, `JoystickInput` and `MecanumMovement` |
| `TeamCode/src/simulated/` | `ExampleRobot` and its registration |
| `TeamCode/src/testSimulated/` | the two tests |
| `.github/workflows/test.yml` | CI |
| `gradle.properties` | Gradle's heap raised to 2 GB; 1 GB can run out building both APKs and the tests |

## Licence

The FTC SDK is FIRST's; see `LICENSE`. The example robot, OpModes and tests come from ftc-sim
and are BSD-3-Clause; see `LICENSE-ftc-sim`. Change or delete them as you like.
