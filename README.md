
# Monorepo Testing

Just trying out some things for running a monorepo. This uses [npm workspaces](https://docs.npmjs.com/cli/v8/using-npm/workspaces) and [Release Please](https://github.com/googleapis/release-please).

## What I did

I started out running everything manually (no CI or anything) in order to learn how it all works in a safe(ish) environment. I'm copying a lot from two Financial Times repos:

  * https://github.com/Financial-Times/origami
  * https://github.com/Financial-Times/dotcom-tool-kit

### Basics

I created three packages (`packages/*`) with some basic content. I set up a root level `package.json` file which sets up all folders under `packages` as workspaces:

```json
{
  "workspaces": [
    "packages/*"
  ]
}
```

Now any npm script you run with the `--workspaces` option will run that command on each of the folders in `packages`, e.g.

```
npm run test --workspaces
```

You can also run a command for an individual workspace:

```
npm run test --workspace packages/example-1
```

### Release Please

I opted to manually bootstrap Release Please based on what I found in the existing FT repos. This involved creating [`release-please-config.json`](release-please-config.json) (configurations) and [`.release-please-manifest.json`](.release-please-manifest.json) (a workspace-to-version map).

Release Please requires commits to follow the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) spec. **TODO: work out how we can enforce conventional commits easily in a repo or in PRs. I assume Tool Kit or Origami already do this**

The Release Please workflow is to create a "Release PR" which updates all the `package.json` version numbers and auto-generates a changelog. I did this with the following (token = a GitHub token with the `repo` scope):

```
release-please --token=XXXXX --repo-url=rowanmanning/monorepo-testing release-pr
```

This generates a pull request which [looks like this](https://github.com/rowanmanning/monorepo-testing/pull/2).

Once you merge this PR, nothing magic happens with tagging and GitHub releases – that's done with another command (note `--monorepo-tags` - this is required to prefix the created Git tags with the package name):

```
release-please --token=XXXXX --repo-url=rowanmanning/monorepo-testing github-release --monorepo-tags
```

This creates GitHub releases and tags :tada:

### Publishing to npm

The last manual step is to publish to npm. I did this locally by pulling down changes and publishing all workspaces as public packages:

```
npm publish --workspaces --access public
```

## Automating

The next step is to automate the process above. We'd want the following to run every time a change is made to the `main` branch:

  * Run `release-please release-pr`
  * Run `release-please github-release`

Both commands are safe to run every time, because if there is nothing to release then they'll do nothing.

We'd also want to do the following for every new tag:

  * Run `npm publish` scoped to the workspace that the tag is a release for

I suspect all of the above could be largely copied from dotcom-tool-kit.
