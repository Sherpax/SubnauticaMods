# SubnauticaMods
My mods for Subnautica and Subnautica: Below Zero

---

## Building / Development Setup

### Software requirements

| Tool | Notes |
|------|-------|
| **Visual Studio 2019 or 2022** | Community edition is free. JetBrains Rider or VS Code with the C# extension also work. |
| **.NET Framework 4.7.2 Developer Pack** | The targeting pack is required because the projects target `net472`. Download from [Microsoft](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net472). |
| **Subnautica** and/or **Subnautica: Below Zero** | Installed via Steam or Epic Games Store. |
| **QModManager** | Must be installed inside the game. Follow instructions on [Nexus Mods](https://www.nexusmods.com/subnautica/mods/201). |
| **SMLHelper** | Must be installed inside the game. Follow instructions on [Nexus Mods](https://www.nexusmods.com/subnautica/mods/113). |
| **AssemblyPublicizer** | Required to make all game-assembly members accessible to mod code. A pre-built binary is available at [CabbageCrow/AssemblyPublicizer](https://github.com/CabbageCrow/AssemblyPublicizer). The `update_game_sources.bat` script expects it at `c:\programming\assembly_publicizer\AssemblyPublicizer.exe`. |

### Repository layout

```
SubnauticaMods/
├── .dependencies/          # Non-gitignored third-party DLLs (Harmony, SMLHelper, Unity, …)
│   ├── gameSN/             # Subnautica stable references
│   ├── gameSNexp/          # Subnautica experimental references
│   ├── gameBZ/             # Below Zero stable references
│   └── gameBZexp/          # Below Zero experimental references
├── Common/                 # Shared source code (Shared Project, included by every mod)
├── Common.Tests/           # NUnit tests for the Common library
├── <ModName>/              # One folder per mod, each with its own .csproj
├── project.props           # Shared MSBuild properties (target framework, output paths, …)
├── post-build.props        # Triggers post-build.bat after every build
├── post-build.bat          # Copies build output to the game's QMods folder
├── update_game_sources.bat # Extracts and publicizes game DLLs (run once per game update)
└── SubnauticaMods.sln      # Visual Studio solution
```

### Build configurations

Every project exposes the following MSBuild configurations (selected in Visual Studio's toolbar or via `--configuration`):

| Configuration | Game | Branch | Notes |
|--------------|------|--------|-------|
| `SN.dev` | Subnautica | stable | Enables `DEBUG`, `LOAD_CONFIG`. **Default** when none is specified. |
| `SN.publish` | Subnautica | stable | Release build; treats warnings as errors. |
| `SN.testbuild` | Subnautica | stable | Debug build without `LOAD_CONFIG`. |
| `SNexp.dev` | Subnautica | experimental | Same as `SN.dev` but against experimental-branch DLLs. |
| `SNexp.testbuild` | Subnautica | experimental | – |
| `BZ.dev` | Below Zero | stable | Enables `DEBUG`, `GAME_BZ`, `LOAD_CONFIG`. |
| `BZ.publish` | Below Zero | stable | Release build; treats warnings as errors. |
| `BZ.testbuild` | Below Zero | stable | Debug build without `LOAD_CONFIG`. |
| `BZexp.dev` | Below Zero | experimental | – |
| `BZexp.testbuild` | Below Zero | experimental | – |

The preprocessor symbols injected by each configuration are:

- `GAME_SN` / `GAME_BZ` – distinguishes between the two games
- `BRANCH_STABLE` / `BRANCH_EXP` – stable vs. experimental Steam branch
- `DEBUG` / `TRACE` – standard debug symbols
- `LOAD_CONFIG` – forces the mod to load its JSON config file even in debug builds

### Step-by-step setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/zorgesho/SubnauticaMods.git
   cd SubnauticaMods
   ```

2. **Install the .NET Framework 4.7.2 Developer Pack**
   Download and run the installer from https://dotnet.microsoft.com/en-us/download/dotnet-framework/net472.

3. **Install QModManager and SMLHelper in your game**
   Follow the installation guides on Nexus Mods. This puts the required DLLs (e.g. `SMLHelper.dll`, `0Harmony.dll`) into the game's `BepInEx`/`QMods` folder; the copies already committed under `.dependencies/` are usually sufficient to compile, but keep them in sync if you update the mods.

4. **Obtain and publicize the game assemblies**
   The game's managed assemblies (`Assembly-CSharp.dll`, `Assembly-CSharp-firstpass.dll`) are **not** included in the repository (see `.gitignore`). You must generate the publicized versions yourself:

   a. Locate the `Managed` folder inside your game installation:
      - Subnautica: `<game_root>/Subnautica_Data/Managed/`
      - Below Zero: `<game_root>/SubnauticaZero_Data/Managed/`

   b. Download **AssemblyPublicizer** and place it as described above, or run it manually:

      ```
      AssemblyPublicizer.exe --input=Assembly-CSharp.dll --output=<repo>\.dependencies\gameSN\Assembly-CSharp.pb.dll
      AssemblyPublicizer.exe --input=Assembly-CSharp-firstpass.dll --output=<repo>\.dependencies\gameSN\Assembly-CSharp-firstpass.pb.dll
      ```

      Repeat for `gameBZ` (and `gameSNexp` / `gameBZexp` if you use experimental branches).

   c. Alternatively, run the helper script (Windows only, adjusts paths in the script first):

      ```bat
      update_game_sources.bat SN
      update_game_sources.bat BZ
      ```

5. **Open the solution and build**
   Open `SubnauticaMods.sln` in Visual Studio, select the desired configuration (e.g. `SN.dev`), and press **Build → Build Solution** (or `Ctrl+Shift+B`).

   The compiled `.dll` will be placed in `<ModFolder>/bin/<Configuration>/` and, if the game's `QMods` folder exists at `c:\games\subnautica\QMods` (or the BZ equivalent), it will be copied there automatically by `post-build.bat`.

6. **Run the tests (optional)**
   The `Common.Tests` project uses NUnit. You can run tests from the **Test Explorer** in Visual Studio or from the command line:

   ```bash
   dotnet test Common.Tests/Common.Tests.csproj -c SN.dev
   ```

### Modifying an existing mod

- Source files for each mod live inside their own folder (e.g. `ConsoleImproved/src/`).
- Shared utilities are in `Common/`; changes there affect every mod.
- Each mod has a `mod.SN.json` and/or `mod.BZ.json` manifest. The post-build script automatically renames the correct one to `mod.json` before copying it to the game.
- The `config.cs` file in each mod defines the user-configurable options (serialized as `config.json` in the QMods folder at runtime).

---

### Published on NexusMods
Mod | Nexus description | Nexus SN | Nexus BZ
-|-|:-:|:-:
| [**ConsoleImproved**](https://github.com/zorgesho/SubnauticaMods/tree/master/ConsoleImproved) | Console autocomplete for commands and techtypes + full history (saved between sessions) | [341](https://www.nexusmods.com/subnautica/mods/341)| [144](https://www.nexusmods.com/subnauticabelowzero/mods/144)
| [**CustomHotkeys**](https://github.com/zorgesho/SubnauticaMods/tree/master/CustomHotkeys) | This mod allows you to bind console commands to hotkeys and also adds a lot of new console commands | [502](https://www.nexusmods.com/subnautica/mods/502)| [147](https://www.nexusmods.com/subnauticabelowzero/mods/147)|
| [**DayNightSpeed**](https://github.com/zorgesho/SubnauticaMods/tree/master/DayNightSpeed) | Mod for changing speed of day/night cycle. It's like daynightspeed console command, only better. | [361](https://www.nexusmods.com/subnautica/mods/361)| [145](https://www.nexusmods.com/subnauticabelowzero/mods/145)|
| [**DebrisRecycling**](https://github.com/zorgesho/SubnauticaMods/tree/master/DebrisRecycling) | This mod allows you to deconstruct small Aurora debris (including cargo crates) with Habitat Builder to metal salvage | [324](https://www.nexusmods.com/subnautica/mods/324)| -|
| [**FloatingCargoCrate**](https://github.com/zorgesho/SubnauticaMods/tree/master/FloatingCargoCrate) | Big floating storage. You can build it with Habitat Builder and move it with Propulsion Cannon. | [303](https://www.nexusmods.com/subnautica/mods/303)| -|
| [**GravTrapImproved**](https://github.com/zorgesho/SubnauticaMods/tree/master/GravTrapImproved) | Now you can switch between types of objects that are attracted by grav trap. Also, now grav trap can pick up chunks left by treaders and you can lure out and catch crashfish with trap. | [299](https://www.nexusmods.com/subnautica/mods/299)| [143](https://www.nexusmods.com/subnauticabelowzero/mods/143)|
| [**HabitatPlatform**](https://github.com/zorgesho/SubnauticaMods/tree/master/HabitatPlatform) | Buildable floating platform for habitat building | [569](https://www.nexusmods.com/subnautica/mods/569)| -|
| [**PrawnSuitGrapplingArmUpgrade**](https://github.com/zorgesho/SubnauticaMods/tree/master/PrawnSuitGrapplingArmUpgrade) | Upgraded grappling arm for prawn suit. Quicker, stronger and with longer rope. | [368](https://www.nexusmods.com/subnautica/mods/368)| [141](https://www.nexusmods.com/subnauticabelowzero/mods/141)|
| [**PrawnSuitJetUpgrade**](https://github.com/zorgesho/SubnauticaMods/tree/master/PrawnSuitJetUpgrade) | Prawn suit jump jet now works above water. Also mod adds new prawn suit upgrade that allows thrusters to work longer before need to recharge. | [286](https://www.nexusmods.com/subnautica/mods/286)| [140](https://www.nexusmods.com/subnauticabelowzero/mods/140)|
| [**PrawnSuitSettings**](https://github.com/zorgesho/SubnauticaMods/tree/master/PrawnSuitSettings) | Mod for tweaking various Prawn Suit parameters | [359](https://www.nexusmods.com/subnautica/mods/359)| [142](https://www.nexusmods.com/subnauticabelowzero/mods/142)|
| [**PrawnSuitSonarUpgrade**](https://github.com/zorgesho/SubnauticaMods/tree/master/PrawnSuitSonarUpgrade) | Adds sonar module to Prawn Suit | [278](https://www.nexusmods.com/subnautica/mods/278)| -|
| [**RemoteTorpedoDetonator**](https://github.com/zorgesho/SubnauticaMods/tree/master/RemoteTorpedoDetonator) | New upgrade module that allows you to detonate launched torpedoes at any moment | [291](https://www.nexusmods.com/subnautica/mods/291)| [251](https://www.nexusmods.com/subnauticabelowzero/mods/251)|
| [**StasisModule**](https://github.com/zorgesho/SubnauticaMods/tree/master/StasisModule) | Stasis technology in a form of a vehicle module | [883](https://www.nexusmods.com/subnautica/mods/883)| [254](https://www.nexusmods.com/subnauticabelowzero/mods/254)|
| [**StasisTorpedo**](https://github.com/zorgesho/SubnauticaMods/tree/master/StasisTorpedo) | Stasis technology in a form of a torpedo | [882](https://www.nexusmods.com/subnautica/mods/882)| [253](https://www.nexusmods.com/subnauticabelowzero/mods/253)|
| [**TrfHabitatBuilder**](https://github.com/zorgesho/SubnauticaMods/tree/master/TrfHabitatBuilder) | This is how Habitat Builder looks now | [377](https://www.nexusmods.com/subnautica/mods/377)| -|
| [**UITweaks**](https://github.com/zorgesho/SubnauticaMods/tree/master/UITweaks) | This mod adds various tweaks to the user interface such as bulk crafting, renaming beacons in the inventory and more | [562](https://www.nexusmods.com/subnautica/mods/562)| [148](https://www.nexusmods.com/subnauticabelowzero/mods/148)|
| [**WarningsDisabler**](https://github.com/zorgesho/SubnauticaMods/tree/master/WarningsDisabler) | Mod for disabling various warnings and messages | [358](https://www.nexusmods.com/subnautica/mods/358)| [146](https://www.nexusmods.com/subnauticabelowzero/mods/146)|

### Hidden on NexusMods
Mod | Reason of hiding | Nexus description | Nexus link
-|-|-|:-:
| [**ModsOptionsAdjusted**](https://github.com/zorgesho/SubnauticaMods/tree/master/ModsOptionsAdjusted) | Merged to [SMLHelper](https://www.nexusmods.com/subnautica/mods/113) (v2.6.0) | Mod that adjusts UI controls with text labels within mods options panel | [316](https://www.nexusmods.com/subnautica/mods/316)|
| [**RenameBeacons**](https://github.com/zorgesho/SubnauticaMods/tree/master/RenameBeacons) | Merged to [UITweaks](https://www.nexusmods.com/subnautica/mods/562) (v1.0.0) | Mod that allows you to rename beacons in the inventory | [300](https://www.nexusmods.com/subnautica/mods/300)|
| [**SeamothStorageSlots**](https://github.com/zorgesho/SubnauticaMods/tree/master/SeamothStorageSlots) | Merged to [SlotExtender](https://www.nexusmods.com/subnautica/mods/142) (v2.6.2) | Mod that allows you to install 4 torpedo systems and 4 storage modules to seamoth at the same time | [281](https://www.nexusmods.com/subnautica/mods/281)|