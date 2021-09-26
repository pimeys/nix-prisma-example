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
