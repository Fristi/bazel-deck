---
layout: cover
theme: purplin
highlighter: shiki
colorSchema: light
---

# Bazel for Scala

<p text="2xl" class="!leading-8">
How does it work (with Scala)?
</p>

<div class="abs-br mx-14 my-12 flex">
  <!-- <logos:vue text="3xl"/> -->
  <div class="ml-3 flex flex-col text-left gap-1">
    <div>Scala.amsterdam</div>
    <div class="text-sm opacity-50">16 February 2022</div>
  </div>
</div>

<BarBottom  title="Bazel for Scala">
  <Item text="Fristi/bazel-deck">
    <carbon:logo-github />
  </Item>
  <Item text="">
    <img
      src="https://vectos.net/assets/img/logo.png"
      class="w-16"
    />
  </Item>
</BarBottom>

---
layout: 'intro'
---

<h1 text="!5xl">Mark de Jong</h1>

<div class="leading-8 opacity-80">
Functional Programming 🥑<br>
Freelance full-stack developer at DHL<br>
Company site: <a href="https://vectos.net" target="_blank">Vectos</a>.<br>
</div>

<img src="https://vectos.net/assets/img/mark.jpg" class="rounded-full w-40 abs-tr mt-30 mr-20"/>

<BarBottom  title="Bazel for Scala">
  <Item text="Fristi/bazel-deck">
    <carbon:logo-github />
  </Item>
  <Item text="">
    <img
      src="https://vectos.net/assets/img/logo.png"
      class="w-16"
    />
  </Item>
</BarBottom>

---
layout: center
---

# Bazel for Scala 

<div grid="~ cols-2">
<img src="/scala.svg" w="30" m="auto"/>
<img src="/bazel.svg" w="30" m="auto"/>
</div>


---

# Why would you want Bazel?

<v-clicks>

- One language for builds
- Mono repo
  - Pull requests in one place
  - Protocol definitions are in sync
  - Decreased processing time
- Decreased build times and resource utilization
- Maybe a replacement as SBT or next to it, up to you 😄 

</v-clicks>

---

# Bazel - Unique selling points

<div grid="~ cols-2" class="gap-10">

  <div>

  <v-clicks>

  ###### Fast and correct builds

  - Reproducible build
  - Caching (compile, tests and artifacts)
  - Remote execution

  </v-clicks>

  </div>

  <div>

  <v-clicks>

  ###### Polyglot build

  - Builds Scala, Java, Rust, TypeScript, Docker, etc
  - Language of choice: Starlak
  - Limited expressivity of the language
  - Works on Linux, MacOS and Windows

  </v-clicks>

  </div>

</div>

---
layout: center
---

# How does it work?

---


# Describe your graph as targets

### Two types of files

<div grid="~ cols-2" class="gap-5">

  - `WORKSPACE` - defines global inputs like the Scala compiler and JVM deps.
  - `BUILD` - Use the global inputs to construct targets and connect them to form a graph

  <img src="/graph-01.png" w="150">

</div>

--- 


# Define a `WORKSPACE`

- For every repo there is _one_ `WORKSPACE` file which defines  global inputs with _rules_
    - Global inputs with pinned versions like  
      - Scala compiler
      - Typescript
      - JVM deps
    - These global inputs are the same for _all_ targets

---

# Define target(s) in `BUILD`

<div grid="~ cols-2" class="gap-5">

  ```python
  scala_library(
    name = "services",
    srcs = glob(["main/**/*.scala"]),
    deps = ["@maven//:dev_zio_zio_2_13"]
  )
  ```

  <div>

  ## Typical target settings

  <v-clicks>

  - Name
  - Dependencies
    - External like `@maven//:dev_zio_zio_2_13`
    - Internal like `//core/:library`
  - Source files
  - Other settings specific to the target
  - You use global inputs like the Scala compiler and JVM deps
  - By defining targets you construct your _build graph_

  </v-clicks>

  </div>

</div>

---

# Connect your graph

<div grid="~ cols-2" class="gap-5">

  <div>

  ###### /services/BUILD

```python {all|2}
scala_library(
  name = "services",
  srcs = glob(["main/**/*.scala"]),
  deps = ["@maven//:dev_zio_zio_2_13"]
)
```

  ###### /api/BUILD

```python {all|5}
scala_image(
  name = "api",
  srcs = glob(["main/**/*.scala"]),
  main_class = "Hello",
  deps = ["//services"]
)
```

  </div>

  <img src="/graph-02.png" w="150" />
  
  
</div>

---

# Achieve caching, by reproducibility

<v-clicks>

- To achieve reproducible builds you need stable inputs like
  - Scala compiler version (2.12, 2.13, 3.x)
  - External JVM dependencies (e.g.: junit 3.1.1)
  - Your own source files
- From these inputs a reproducible hash is calculated for your _target_ and used for caching
- Test results are also cachable if the inputs don't change

</v-clicks>

---

## Running Bazel

<div grid="~ cols-2" class="gap-5">

  <div>

  ###### Setup
  - Server/client model
  - Clients have a local cache

  <v-click>

  ###### What do we get?
  - Targets can be build in parallel
  - Read/write target results from cache (*Remote cache*)
  - Targets can run distributed (*Remote Execution*)

  </v-click>

  <v-click>

  ###### Tooling
  - Buildbarn (bazel server)
  - Buildbuddy (includes tracing for targets)

  </v-click>

  </div>

  <img src="/graph-03.png" w="150" />

</div>

---
layout: center
---

# Current state for Scala

---

# rules_scala

<v-clicks>

### Features

- Support for building
  - libraries
  - binaries
  - tests
  - thrift libraries
  - protobuf libraries
- Coverage support

</v-clicks>

<v-clicks>

### Constraints

- You can setup the Scala toolchain in your `WORKSPACE` file
- Multiple scala versions at the same time is _not_ possible
- Does _not_ work with Scala 3.x (yet)
- Experimental settings are needed to achieve transitive dependency inheritance

</v-clicks>

---

# Transitive dependency inheritance

<v-click>

Multi-module project with

- core (depends on external JVM dependency `typelevel/cats-core`)
- services (depends on internal `core`)

The compiler will crash at `services` with _default_ settings due *classpath issues*

</v-click>

<v-click>

### Fix

Setup the Scala toolchain with 

- `dependency_mode` set to `transitive`
- `dependency_tracking_method` set to `ast`

The docs however state that these options are _experimental_

</v-click>

---

# Rules for external JVM dependencies

| Rule | Format | Multi-version | Scala-steward | Maintainer | Active
| -- | -- | -- | -- | -- | --
| bazel-deps | YML | ❌ | ❌ | 1 ex Stripe | ❌
| rules_jvm_external | Starlak | ❌ | ❌ | Bazel | ✅
| bazel_multiversion | YML or Starlak | ✅ | ❌ | Twitter | ✅



---

# Conclusion

<v-clicks>

- Bazel shows potential in being a build tool for mono repos
- Can offer dramatic decrease in build times and resources
- Organizations like Stripe, Pinterest and such usually have a _build_ team
  - Migrate existing builds in stages
  - Support for tooling
  - Support for new use-cases
- For _Scala_ you need to commit to
  - Specific version of the compiler
  - Dare to work with experimental settings
  - A fixed set of dependencies

</v-clicks>

---
layout: center
class: 'text-center pb-5'
---

# Thank You!

[https://github.com/Fristi/bazel-deck](https://github.com/Fristi/bazel-deck)