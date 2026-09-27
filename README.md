# pkg-fixture

A test fixture for wand's package manager. `tools/check_packages.wand` in
[wand-lang/wand](https://github.com/wand-lang/wand) fetches it from GitHub
and runs the `wand p` commands against it. The check runs once a day, after
each wand release, and by hand from the Packages workflow.

Do not use this package in real code. It exists only to be imported by that
check.

## Files

| File | Case it covers |
|---|---|
| `pkg-fixture.wand` | The root module. Its name has a hyphen, so `import github.com/wand-lang/pkg-fixture` binds `pkg_fixture`. |
| `words.wand` | A module below the root: `import github.com/wand-lang/pkg-fixture/words` binds `words`. |
| `_private/secret.wand` | A private module. Another package that imports it gets an error. |
| `wand.pkg` | The record, then the interface section that `wand p release` writes. |

## Releases

Each tag was made by `wand p release`, so each tag also tests how the types
decide the version.

| Tag | Change | Case it covers |
|---|---|---|
| `v0.1.0` | `version`, `greet name`, `words.shout` | The first release. `wand p add …@0.1.0` pins it. |
| `v0.1.1` | Adds `bye` | An addition, so a patch before 1.0. `wand p upgrade` moves 0.1.0 to 0.1.1, within one major. |
| `v0.2.0` | `greet` takes a greeting as well as a name | A breaking change: `wand p release` refused it as a patch. Before 1.0 each minor is a major, so 0.2.0 is used beside 0.1.x with `wand p add …@0.2.0 --name fixture2`. |

The `wand` field in each tag is `0.85.0`, the oldest wand the package works
with. Every later wand before 1.0 accepts it.

## What the check does

Each step starts from a new, empty cache, so every version comes from GitHub.

1. `wand p init` a package that uses the fixture.
2. `wand p add github.com/wand-lang/pkg-fixture@0.1.0`.
3. Import the root module and `words`, and run them.
4. `wand p add github.com/wand-lang/pkg-fixture/words` is refused: the
   package is already required. This URL is not a repository, so GitHub
   answers "not found" first. The step checks that wand goes on to the
   repository and does not stop for a password.
5. `wand p upgrade` moves 0.1.0 to 0.1.1.
6. Run again, and get the 0.1.1 code.
7. `wand p add …@0.2.0 --name fixture2`.
8. Import both majors in one file, and run it.
9. Import `_private/secret`, and get the error.
10. `wand p tidy` removes the 0.2.0 entry after nothing imports it.

## Changing the fixture

A change here can make the check fail. After any change:

- Release it with `wand p release`, so that the tag and the interface
  section agree.
- Update `tools/check_packages.wand` and this README in the same change.
