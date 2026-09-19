# IC2CRE API (Minecraft 1.21.1 / NeoForge / Java 21)

Build the API and runtime Maven artifacts:

```powershell
.\gradlew.bat build publishAllPublicationsToLocalDistributionRepository
```

The repository is generated under `build/maven/1.21.1`. The public release repository uses one `maven` branch for all Minecraft versions, with a separate directory per game version. Maven versions also include the Minecraft version to prevent coordinate/cache collisions. In a NeoForge 1.21.1 add-on project:

```groovy
repositories {
    maven {
        url = uri('https://raw.githubusercontent.com/BigFish520/IC2-CRE/maven/1.21.1/')
        content { includeGroup('ic2cre') }
    }
}
dependencies {
    compileOnly 'ic2cre:ic2cre-api:1.21.1-0.3_5h-dev'
    runtimeOnly 'ic2cre:ic2cre:1.21.1-0.3_5h-dev'
}
```

Replace the repository path and version for your environment. The API sources artifact is published alongside the binary. Alternatively, use `compileOnly files('libs/ic2cre-api-<version>.jar')`. Do not shade the API into an add-on or install the API JAR as a mod: the runtime mod already contains these classes. Declare your runtime dependency on `ic2cre` in the add-on's NeoForge metadata.

## Export readable Java source files

From the development checkout:

```powershell
.\gradlew.bat exportApiSources
```

This exports `.java` files and the license to `build/api-sources`. From a release JAR, with Java 21 but without Minecraft/NeoForge on the classpath:

```powershell
java -jar IC2CRE_0.3_5h-dev_NeoForge_1.21.1.jar --extract-api ic2cre-api-sources
```

The destination must not exist and its parent directory must exist. The exporter refuses to overwrite an existing directory. The embedded source archive is built from the same `src/api/java` source set as the API binary. It excludes runtime implementation, add-on, client implementation and test sources. `--help` prints usage.

## Extension points

| Purpose | Public entry point |
| --- | --- |
| Energy network and energy tiles | `ic2cre.api.energy`, including `EnergyNet` and `prefab` |
| Environmental flow query | `EnvironmentPhysicsEngine.get(level)` returns only `EnvironmentPhysicsApi` |
| Ore scanning | `ic2cre.api.scanner.OreScanner` delegates to the runtime-owned scanner |
| Electric item charging/use | `ic2cre.api.item.ElectricItem.getManager()` |
| Item lookup by registered ID | `ic2cre.api.item.Ic2CreItems.getItem(...)` |
| Read-only functional keys | `ic2cre.api.input.Keys.getKeyboard()` |
| Reactor items and core access | `ic2cre.api.reactor.IReactorComponent`, `IReactor`, `IReactorChamber` |
| Crop cards and crop state | `ic2cre.api.crops.CropCard`, `ICropTile` |
| Crop registration/query | `ic2cre.api.crops.Crops.getRegistry()` |
| Base-seed registration | `ICropRegistry.registerBaseSeed`, `BaseSeed` |
| Upgrade parameters and server tick callbacks | `ic2cre.api.upgrade.IUpgradeItem` |
| Explicit custom wrench drops | `IWrenchable.getWrenchDrops`; null preserves normal block drops |
| HU, kinetic energy, gas and redstone | Respective capability contracts in `ic2cre.api` |
| Fuel/property recipes | `ic2cre.api.recipe`, existing recipe types and fluid tags |
| Tools, armor, rotors and upgrades | Existing contracts in `ic2cre.api.item`, `tool`, `upgrade` |

Call runtime-bound services after IC2CRE initialization; register crop cards in enqueued common setup work. Mutating world, item or reactor state belongs on the server. Energy operations use **mEU**, not old IC2 EU (`1000 mEU = 1 EU`). The consumer compilation fixture in `src/apiTest` is compiled against the API JAR without implementation classes.

## Public / internal boundary

Public contracts, immutable data used by those contracts, and reusable API helpers (such as gas stacks/tanks and energy prefabs) are exported. Environment scheduling, caches, persistence, terrain/flow algorithms, packet dispatch, rotor animation, KU balance formulas and Mekanism fuel implementation data remain in `common` and are not exported as API source. Scanner algorithms are likewise internal; the API contains only the service contract and scan results. Descriptors and hooks without runtime consumers are not supported extension points, including multi-tile energy delegation. Add-ons should use the implemented energy tile/capability contracts instead.

`build` runs `verifyApiBoundary`: it rejects implementation/client dependencies and known lifecycle owners in the API, checks the approved environment-contract surface, and verifies that API binaries and exported source archives exactly match the public source inventory. Introducing a new environment API type requires explicitly reviewing this boundary. This is an automated regression guard, not a substitute for design review.

## Legacy migration boundary

The compatibility baseline is IC2 2.8.222 behavior, rewritten for 1.21.1, not binary or unchanged-source compatibility. Legacy `ic2.api.*` imports and EU amounts must be adapted. `IElectricItemManager.use` reports success as a boolean; capacity and tier queries are available. Reactor explosion contributions use `influenceExplosion(stack, reactor)`; inert installable components may implement `IBaseReactorComponent`. Crop cards may use a namespaced `ResourceLocation`, including textures in the add-on's namespace. Base-seed registrations match item and DataComponents, ignoring count.

Old reflection-based network field updates, metadata item variants and global mutable recipe managers are not copied: use validated NeoForge payloads, registered item IDs/DataComponents and the existing recipe/data-pack system. Old Forge fluid/item transport uses NeoForge capabilities instead of duplicate legacy interfaces. Unsupported remote-upgrade flags and the no-op reactor explosion accumulator have been removed. Upgrade output transformation currently applies to the batch crafter; other processors do not claim this callback. HUD, legacy custom electric-manager routing, biome-bonus registration and remaining legacy event hooks are not yet one-to-one ports and must not be assumed supported.
