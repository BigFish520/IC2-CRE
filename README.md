# IC2CRE Maven Repository

Public Maven artifacts for IC2CRE. This branch contains release artifacts, not the private implementation source repository.

## Minecraft 1.21.1 / NeoForge / Java 21

```groovy
repositories {
    maven {
        url = uri('https://raw.githubusercontent.com/BigFish520/IC2-CRE/maven/1.21.1/')
        content { includeGroup('ic2cre') }
    }
}

dependencies {
    compileOnly 'ic2cre:ic2cre-api:1.21.1-0.4-dev'
    runtimeOnly 'ic2cre:ic2cre:1.21.1-0.4-dev'
}
```

The API sources JAR contains only public API Java sources. The runtime JAR includes the API classes; do not shade the API JAR into an add-on or install it as a separate mod. Declare the `ic2cre` runtime dependency in your NeoForge mod metadata.

## Multi-version layout

Energy Control releases are suspended pending further development. Energy Control is not included in this Maven publication.

All Minecraft versions share this `maven` branch. Each game version has a separate Maven root (`1.21.1/`, and future versions such as `26.1.2/`). Maven artifact versions use `<Minecraft version>-<mod version>` to avoid coordinate and cache collisions. Do not combine game-version roots in a consumer project or overwrite an existing fixed release version.

Only reviewed binaries, public API sources, Maven metadata/checksums, licensing and usage documentation belong here. Never merge private source history or publish implementation sources, test artifacts or credentials.

The Raw URL works without GitHub Packages authentication. A GitHub Pages URL is optional and must not be assumed enabled.
