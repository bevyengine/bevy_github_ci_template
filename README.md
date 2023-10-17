# Bevy GitHub CI Template

Template for setting up continuous integration (CI) and continuous deployment (CD) on a GitHub project for Bevy. This template enables you to test your code, build for the Web, Linux, Windows, MacOS, and publish to GitHub Releases, Itch.io, and GitHub Pages.

It creates two workflows:

* [CI](#ci)
* [Release](#release)

## CI

Definition: [.github/workflows/ci.yaml](./.github/workflows/ci.yaml)

![ci workflow](https://user-images.githubusercontent.com/104745335/268799840-06b772e8-7901-4b86-9a88-e4afee8a0167.png)

This workflow runs on every commit to the `main` branch, and for every PR targeting the `main` branch.

It will use Rust stable on Linux with caching between different executions. The following commands are executed:

* `cargo test`
* `cargo clippy -- -D warnings`
* `cargo fmt --all -- --check`

If you are using anything OS-specific or Rust nightly, you should update the [ci.yaml](./.github/workflows/ci.yaml) file  accordingly.

## Release

Definition: [.github/workflows/release.yaml](./.github/workflows/release.yaml)

### Run Workflow

This workflow runs every time you push a tag to your repo.

![Release workflow](https://github-production-user-asset-6210df.s3.amazonaws.com/104745335/275751118-be1bb6da-dbf1-413d-8d8b-8d3ba92f4251.png)

Example using git:

```sh
git tag -a "v1.0.0" -m "First official release"
git push --tags
```

 By default, the workflow will build for the Web, Linux, Windows, and MacOS. The builds will be uploaded to a GitHub Release. If the name of your repository does not match the name of your binary, you should update the environment variable [`binary`](.github/workflows/release.yaml#L7) in the `release.yaml` file to your binary name.

You can configure the [build](.github/workflows/release.yaml#L15) and [publish](.github/workflows/release.yaml#L21) targets by changing the [environment variables](.github/workflows/release.yaml#L4)  in the `release.yaml` file.

#### Manual Workflow run in GitHub

You can configure and trigger the workflow directly in your GitHub repository. Navigate to the `Actions` section, click `Release` on the sidebar, then press the `Run workflow` button and select the configuration for the build and publish targets you want.

![Run manual workflow](https://user-images.githubusercontent.com/104745335/270079051-b5fb52c8-ed89-4f91-965e-670f38f87ddb.png)

The configuration in GitHub takes priority over the environment variables in the [release.yaml](.github/workflows/release.yaml#L4) file. A manual workflow run also enables you to override the tag version eliminating the need to create a tag to trigger the workflow.

If you want to publish to itch.io, you will either have to configure the environment variable `itchio_target`, or define the `itchio_target` input for every run.

### Build

You can build for the following platforms:

* For Linux and Windows, create a .zip archive containing the executable and the `assets` folder.
* For MacOS, generate two versions: one for Intel x86_64 and one for Apple Silicon ARM64, each packaged in a .dmg image containing a .app file with the assets.
* For the Web, produce a .zip archive with the WebAssembly (wasm) binary, JavaScript bindings, an HTML file for loading it, and the assets.

By default, it builds for all platforms, but you can disable specific targets by removing their name in the env variable [`build_for`](.github/workflows/release.yaml#L15) in the release.yaml file.

For example, `build_for: web, linux, macos` will build for the Web, Linux and MacOS but will not build for windows.

The build files will be uploaded to the artifacts section of their respective workflow runs. To access them, go to your GitHub repository, navigate to the `Actions` section, click on `Release` in the sidebar, select the `workflow run` triggered by your tag, and scroll down to find your artifacts.

![Artifacts](https://github-production-user-asset-6210df.s3.amazonaws.com/104745335/275743001-ce09f21c-3f3b-476d-a0a1-3f56ba2a46a9.png)

### Publish

You can publish to the following platforms:

* [Github Releases](#publish-to-github-releases)
* [Itch.io](#publish-to-itchio)
* [Github Pages](#publish-to-github-pages)

By default, it only publishes to GitHub Releases, but you can enable or disable any target by writing or removing their name in the env variable [`publish_to`](.github/workflows/release.yaml#L21) in the release.yaml file.

For example, `publish_github_pages: pages` will only publish the web version of your game to GitHub Pages.

Every time you trigger this workflow, it will upload the files for every platform you enable in the build process.

The naming convention for the uploaded files will follow this structure:

`<Name of your binary>_<tag>_<platform>.<format>`

The format will be .zip for web, Windows, and Linux, and .dmg for MacOS.

For example: `bevy-game_v3.6_linux.zip`

### Publish to Github Releases

This action will occur automatically every time you push a tag if you have written `github_releases` to the env variable [`publish_to`](./.github/workflows/release.yaml#L28) in the release.yaml file.

However, if you prefer more configuration options to manage your releases, you can do so through the GitHub CLI or the web browser:

1. [Github CLI](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository?tool=cli): You can use `gh release create` to create a GitHub release iteratively from your terminal.
2. [Web Browser](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository?tool=webui)

Once you complete the process, a new release will be available on GitHub, with the archives for each platform accessible as downloadable assets.

If you want to publish by manually triggering a workflow in GitHub, instead of writing your env variables in the release.yaml file,  write all your configuration in the GitHub Workflow form and then run the workflow. Refer to the ['Manual Workflow Run in GitHub'](#manual-workflow-run-in-github)

![github release](https://user-images.githubusercontent.com/104745335/268805270-ff824032-4191-4528-9d45-a5511fef4f94.png)

### Publish to Itch.io

To publish to itch.io, follow this release flow:

1. Create an API key at <https://itch.io/user/settings/api-keys>
2. In your GitHub repository, go to the `Settings` tab, click on `Secrets` under the `Actions` section in the sidebar, and add a repository secret named `BUTLER_CREDENTIALS`, with the API key as its value.
3. Create your game page on itch.io if you haven't already.
4. In the release.yaml file, write  `itchio` in the env variable [`publish_to`](./.github/workflows/release.yaml#L21).
5. Go to the env variable [`itchio_target`](./.github/workflows/release.yaml#L19) in the release.yaml and set it to your itch.io username and the name of the game on itch.io, separated by a slash (`/`). For example: `cart/build-a-better-buddy`. Double-check the URL of your game to ensure the name is correct.

Once these steps are completed, any tag pushed to GitHub will trigger an itch.io release, and it will use the tag as the [user version](https://itch.io/docs/butler/pushing.html#specifying-your-own-version-number).

If you want to make your game playable directly on your Itch.io page, follow the steps above, and make sure to build it for the web by writing `web` or `wasm` in the env variable [`publish_for`](./.github/workflows/release.yaml#L21).
Next, to make the game visible on your Itch.io page, go to your game's configuration on Itch.io, and change the `Kind of project` to `HTML`. Additionally, locate your uploaded web files and check the box that says, `This file will be played in the browser`.

![Kind of project](https://user-images.githubusercontent.com/104745335/268805329-fb70e23e-44ee-4f2f-9d20-11d58ddeec9a.png)

![Play in browser](https://github-production-user-asset-6210df.s3.amazonaws.com/104745335/268780679-fa14874c-040b-41ff-8a04-71cf141970dc.png)

If you want to publish by manually triggering a workflow in GitHub, instead of doing the 4 and 5 steps, write all your configuration to in the GitHub Workflow form and then run the workflow. Refer to the ['Manual Workflow Run in GitHub'](#manual-workflow-run-in-github).

### Publish to Github pages

If you want to publish your game to be playable in the browser on a [GitHub page](https://pages.github.com/) follow these steps:

1. In the release.yaml file, verify that you are writing `github_pages` to the env variable [`publish_to`](./.github/workflows/release.yaml#L21) and `web` in the env variable [`build_for`](./.github/workflows/release.yaml#L15).
2. Trigger the [release.yaml](./.github/workflows/release.yaml) workflow by pushing a tag.
3. In your GitHub repository, go to the `Settings` tab, then click on `Pages` in the sidebar. Navigate to the `Build and Deployment` section and select `Github Actions` as the `Source`. ![Github Pages](https://user-images.githubusercontent.com/104745335/270078892-a6da763e-5a4a-4f55-8d3c-41a14e8423e2.png)
4. Wait a few minutes and your page will be available at a URL following this structure: `https://<Your GitHub username>.github.io/<Name of your repository>/`

If you want to publish by manually triggering a workflow in GitHub, instead of doing the 1 and 2 steps, write all your configuration in the GitHub Workflow form and then run the workflow. Refer to the ['Manual Workflow Run in GitHub'](#manual-workflow-run-in-github).

## Adapting the Workflows for Your Project

If you'd like to use the GitHub workflows provided here for your own project, you may need to make a few adjustments:

1. For web builds, ensure that your project includes an `index.html` file under the `/wasm` directory.
2. Make sure that the name of your repository matches the name of your binary. If the name of your repository is different, simply change the env variable [`binary`](.github/workflows/release.yaml#L7) in the release.yaml file to match the name of your binary.
3. If your project doesn't have an `assets` folder:
    1. You can create one and add a `.gitkeep` file to it to enable you to push it to your repository.
    2. Alternatively, if your project does not use assets, you can remove the `cp -r assets` statements from the build jobs in the workflow.
4. If you are using a nightly toolchain or a specific Rust version, make sure to adapt the toolchain version as needed.
5. If you encounter the error `Error: Resource not accessible by integration,`. Go to your GitHub repository's settings, and under `Actions -> General,` ensure that `Read and Write permissions` are selected under `Workflow permissions` near the bottom.

## License

Licensed under either of

* Apache License, Version 2.0
   ([LICENSE-APACHE-2.0](LICENSE-Apache-2.0) or <http://www.apache.org/licenses/LICENSE-2.0>)
* MIT License
   ([LICENSE-MIT](LICENSE-MIT) or <http://opensource.org/licenses/MIT>)
* CC0-1.0 License
   ([LICENSE-CC0-1.0](LICENSE-CC0-1.0) or <https://creativecommons.org/publicdomain/zero/1.0/legalcode>)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
triple licensed as above, without any additional terms or conditions.
