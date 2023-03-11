# Bevy GitHub CI Template

This repo show how to set up CI on a GitHub project for Bevy.

It creates two workflows:

* [CI](#CI)
* [Release](#Release)

## CI

Definition: [.github/workflows/ci.yaml](./.github/workflows/ci.yaml)

This workflow runs on every commit to `main` branch, and on every PR targeting the `main` branch.

It will use rust stable on linux, with cache between different executions, those commands:

* `cargo test`
* `cargo clippy -- -D warnings`
* `cargo fmt --all -- --check`

If you are using anything OS specific or rust nightly, you should update the file [ci.yaml](./.github/workflows/ci.yaml) to use those.

## Release

Definition: [.github/workflows/release.yaml](./.github/workflows/release.yaml)

This workflow runs on every tag.

It will build:
* For Linux and Windows, a .zip archive containing the executable and the `assets`.
* For macOS, a dmg image with a .app containing the assets.
* For wasm, a .zip archive with the wasm binary, the js bindings, an html file loading it, and the assets.

If you don't want to target some of those platforms, you can remove the corresponding job from the file [release.yaml](./.github/workflows/release.yaml).

If you don't want to attach the builds to the GitHub release, set `env.add_binaries_to_github_release` to `false`.

### Git Tag from GitHub UI

You can follow [Managing releases in a repository](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository)

### Git Tag from the CLI

Execute the following commands: 

```sh
git tag -a "my-game-1.0" -m "First official release"
git push --tags
```

### Result

A new release will be available in GitHub, with the archives per platform available as downloadable assets.

The `git` commands above produced this release: [my-game-1.0](
https://github.com/bevyengine/bevy_github_ci_template/releases/tag/my-game-1.0).

## Using the workflows in your own project

If you would like to use the GitHub workflows included here for your own project, there are a few things you might have to adapt:

1. The release workflow relies on the `index.html` file under `/wasm` for web builds
2. Make sure that the env variable `binary` ([release.yaml](.github/workflows/release.yaml#L10)) matches the name of your binary
3. In case your project doesn't have an `assets` folder
   1. Either create one and put a `.gitkeep` file in it to be able to push it
   2. Or remove the `cp -r assets` statements in the build jobs
4. Adapt the used toolchain if you are using nightly
5. In your GitHub repo's settings, under `Actions -> General` make sure "Read and Write permissions" is selected under "Workflow permissions" near the bottom. This fixes the error `Error: Resource not accessible by integration`.


### Publish on itch.io

The release flow can be configured to push the releases to itch.io:

1. Create an API key in https://itch.io/user/settings/api-keys
2. Go to the repository's Settings tab in GitHub, click on Secrets->Actions in the sidebar,and add a repository secret named `BUTLER_CREDENTIALS` set to the API key.
3. Uncomment `env.itch_target` in `release.yaml` and set it to the itch.io username and the name of the game on itch.io, separated by a slash (`/`)

Once that is done, any tag pushed to GitHub will trigger an itch.io release and use the tag as the [user version](https://itch.io/docs/butler/pushing.html#specifying-your-own-version-number).
