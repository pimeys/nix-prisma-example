## Prisma project with Nix

First what you want is [nix-direnv](https://github.com/nix-community/nix-direnv/) with flakes support. You can go without flakes, but I can't provide you help how to get the right channels, and flakes are awesome so read the docs in this repo how to enable them for your setup.

When running `direnv allow` in the project directory, it will install prisma cli and the engines to your dev environment. After the whole thing is finished, check the installed prisma version:

```bash
❯ prisma --version
Environment variables loaded from .env
Warning Precompiled binaries are not available for nixos.
prisma                  : 3.1.1
@prisma/client          : 3.1.1
Current platform        : linux-nixos
Query Engine (Node-API) : libquery-engine  (at ../../../../nix/store/12i1q096cjjnqay4ayl9m27l4qsw3c2h-prisma-engines-3.1.1/lib/libquery_engine.node, resolved by PRISMA_QUERY_ENGINE_LIBRARY)
Migration Engine        : migration-engine-cli  (at ../../../../nix/store/12i1q096cjjnqay4ayl9m27l4qsw3c2h-prisma-engines-3.1.1/bin/migration-engine, resolved by PRISMA_MIGRATION_ENGINE_BINARY)
Introspection Engine    : introspection-core  (at ../../../../nix/store/12i1q096cjjnqay4ayl9m27l4qsw3c2h-prisma-engines-3.1.1/bin/introspection-engine, resolved by PRISMA_INTROSPECTION_ENGINE_BINARY)
Format Binary           : prisma-fmt  (at ../../../../nix/store/12i1q096cjjnqay4ayl9m27l4qsw3c2h-prisma-engines-3.1.1/bin/prisma-fmt, resolved by PRISMA_FMT_BINARY)
Default Engines Hash    : c22652b7e418506fab23052d569b85d3aec4883f
Studio                  : 0.423.0
```

As written in the output, the cli uses the binaries and node-api library described in the `flake.nix` env vars. Always check the prisma version installed through the flake is exactly the same as the `@prisma/client` version in the `package.json`.

When upgrading, running `nix flake update` should be enough to get the latest engines/cli. Remember to commit the `flake.lock` file to the version control. After that, update the `@prisma/client` in the package.json.

If working with others who do not use nix, you can modify the `.envrc` file so it will not try running nix commands if not using nix:

```bash
if command -v nix-shell &> /dev/null
then
    use flake
fi
```

If I'm having a vacation or get hit by a bus, and you need to update prisma in the nix monorepo, there's two things to update: the [cli npm package](https://github.com/NixOS/nixpkgs/blob/master/pkgs/development/node-packages/default.nix#L281-L298) and the [rust engines](https://github.com/NixOS/nixpkgs/blob/master/pkgs/development/tools/database/prisma-engines/default.nix). See an [example pull request](https://github.com/NixOS/nixpkgs/pull/139430).

## Updating

If the repository the `flake.nix` has a new version of Prisma, to get the same version to your project requires a few steps.

First make your flake to point to the latest commit in the repository:

``` bash
> nix flake update
```

After that, direnv should catch the changed files, and install the latest Prisma to your environment. Check the version:

``` bash
❯ prisma --version
Environment variables loaded from .env
Warning Precompiled binaries are not available for nixos.
prisma                  : 3.2.0
@prisma/client          : 3.1.1
Current platform        : linux-nixos
Query Engine (Node-API) : libquery-engine  (at ../../../../../nix/store/vckf62m16nxk9wyh5js2psmdsypl4lq1-prisma-engines-3.2.0/lib/libquery_engine.node, resolved by PRISMA_QUERY_ENGINE_LIBRARY)
Migration Engine        : migration-engine-cli  (at ../../../../../nix/store/vckf62m16nxk9wyh5js2psmdsypl4lq1-prisma-engines-3.2.0/bin/migration-engine, resolved by PRISMA_MIGRATION_ENGINE_BINARY)
Introspection Engine    : introspection-core  (at ../../../../../nix/store/vckf62m16nxk9wyh5js2psmdsypl4lq1-prisma-engines-3.2.0/bin/introspection-engine, resolved by PRISMA_INTROSPECTION_ENGINE_BINARY)
Format Binary           : prisma-fmt  (at ../../../../../nix/store/vckf62m16nxk9wyh5js2psmdsypl4lq1-prisma-engines-3.2.0/bin/prisma-fmt, resolved by PRISMA_FMT_BINARY)
Default Engines Hash    : afdab2f10860244038c4e32458134112852d4dad
Studio                  : 0.435.0
```

Now we see the `@prisma/client` is using an older version, so change the versioning accordingly in the `package.json` file. It might be required to delete the `node_modules` directory before running `prisma generate`. After a successful update the version output should list the same version for `prisma` and `@prisma/client`:

``` bash
❯ prisma --version
Environment variables loaded from .env
Warning Precompiled binaries are not available for nixos.
prisma                  : 3.2.0
@prisma/client          : 3.2.0
```

If the nixpkgs master doesn't yet have the latest version, point to the repository that has the update by changing the nixpkgs url in the flake:

``` nix
inputs.nixpkgs.url = "github:NixOS/nixpkgs/master";
```

changes to

``` nix
inputs.nixpkgs.url = "github:pimeys/nixpkgs/prisma-3_2_0";
```

By just entering the directory, direnv should catch the changes and install the new derivation. After this is done, handle the update process as written above.
