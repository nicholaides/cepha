# Development

## Publishing a new version

To release a new version:

1. Make sure you're on the `main` branch and the working tree is clean
2. Make sure the tests are passing in CI
3. Update the version number in `package.json` and commit with message like: `Bump version to 0.0.4`
4. Run `npm pack --dry-run` to make sure the files we want to publish are included
5. Run `npm publish`
