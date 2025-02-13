<picture style="width: 76px; height: 96px" width="76" height="96">
  <source media="(prefers-color-scheme: dark)" srcset="https://user-images.githubusercontent.com/43314/216831744-e17e8282-669d-4716-b728-1ba31edda3f0.svg">
  <source media="(prefers-color-scheme: light)" srcset="https://user-images.githubusercontent.com/43314/216831743-2afcda16-c2e4-406d-9183-ebfcd2d50537.svg">
  <img style="width: 76px; height: 96px" width="76" height="96" alt="" src="https://user-images.githubusercontent.com/43314/216831743-2afcda16-c2e4-406d-9183-ebfcd2d50537.svg">
</picture>

# PaperAge

Easy and secure paper backups of (smallish) secrets using the Age format ([age-encryption.org/v1](https://age-encryption.org/v1)).

[![Rust build](https://github.com/matiaskorhonen/paper-age/actions/workflows/rust.yml/badge.svg)](https://github.com/matiaskorhonen/paper-age/actions/workflows/rust.yml) [![codecov](https://codecov.io/gh/matiaskorhonen/paper-age/graph/badge.svg?token=KM9VSJ6CCT)](https://codecov.io/gh/matiaskorhonen/paper-age) [![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/matiaskorhonen/paper-age)](https://github.com/matiaskorhonen/paper-age/releases/latest) [![Crates.io](https://img.shields.io/crates/v/paper-age)](https://crates.io/crates/paper-age)

## Features

* Accepts input either from a file or stdin
* Encrypts that input with a passphrase
* Outputs a PDF with a QR code of the encrypted ciphertext
* Support for both A4 and letter paper sizes
* The error correction level of the QR code is optimised (less data → more error correction)
* The passphrase **isn't** rendered on the PDF so that it can be printed on an untrusted printer (for example at work or the library)
* You don't need PaperAge to recover from the backup: use any QR code scanner and [any implementation of Age](https://github.com/FiloSottile/awesome-age#implementations).

## Limitations

* The maximum input size is about 1.9 KiB as QR codes cannot encode arbitrarily large payloads
* Only passphrase-based encryption is supported at the moment

## Threat models and use cases

* The main use case is keeping secrets, such as TFA recovery codes, in a safe place
* Adding the passphrase by hand allows the use of public printers, for example in libraries, offices, copy shops, and so forth
* For extra protection, memorize the passphrase or store it separately from the printout
* Needing to scan and decrypt protects against unsophisticated adversaries even if the passphrase is right there (the average burglar isn't going to care about your Mastodon account)
* If you need protection from nation-states or other advanced threats, look elsewhere

## Example

This is what the output PDF looks like (left A4, right letter). The QR code is easily readable with an iPhone (or any modern smartphone).

![Preview of the A4 and letter PDFs](https://github.com/user-attachments/assets/53c4ef67-298a-4522-a2e3-600693430963)

If you want to try decoding it yourself, the passphrase is `snakeoil`.

## Installation

Release builds are available for macOS (Apple Silicon and Intel), Linux (ARM and x86-64), and Windows (x86-64).

While the Windows build *should* work on both Windows 10 and 11, only Windows 11 is “officially” supported.

### Homebrew

Add the PaperAge Tap to install the latest version with Homebrew:

```sh
brew tap matiaskorhonen/paper-age
brew install paper-age
```

### Binary

Download the latest release from the [Releases](https://github.com/matiaskorhonen/paper-age/releases) page, extract the files, and install the `paper-age` binary somewhere in `PATH` (for example `/usr/local/bin`).

```sh
# Download the latest release (pick your OS)
# macOS (Intel or Apple Silicon):
curl -Lo paper-age.tar.gz https://github.com/matiaskorhonen/paper-age/releases/download/v1.3.4/paper-age-universal-apple-darwin.tar.gz
# Linux (x86-64):
curl -Lo paper-age.tar.gz https://github.com/matiaskorhonen/paper-age/releases/download/v1.3.4/paper-age-x86_64-unknown-linux-gnu.tar.gz
# Linux (ARM):
curl -Lo paper-age.tar.gz https://github.com/matiaskorhonen/paper-age/releases/download/v1.3.4/paper-age-aarch64-unknown-linux-gnu.tar.gz

# Verify the artifact attestation using the GitHub CLI tool (optional)
gh attestation verify paper-age.tar.gz --repo matiaskorhonen/paper-age

# Extract the files
tar -xf paper-age.tar.gz

# Install the binary in /usr/local/bin
sudo install paper-age /usr/local/bin/
# Or: sudo mv paper-age /usr/local/bin/
```

### Cargo

If you already have Rust installed, PaperAge can be installed with Cargo:

```sh
cargo install paper-age
```

### Artifact attestations

Starting with v1.3.1, PaperAge releases have [artifact attestations](https://github.com/matiaskorhonen/paper-age/attestations). Attestations are generated using [GitHub's tooling](https://docs.github.com/en/actions/security-guides/using-artifact-attestations-to-establish-provenance-for-builds).

## Usage

```
paper-age [OPTIONS] [INPUT]
```

### **Arguments**

* `<INPUT>` — The path to the file to read. Defaults to standard input. Max. ~1.9KB.

### **Options**

* `-t`, `--title <TITLE>` — Page title (max. 64 characters)

  Default value: `PaperAge`
* `-n`, `--notes-label <NOTES_LABEL>` — Notes label below the QR code (max. 32 characters)

  Default value: `Passphrase:`
* `--skip-notes-line` — Skip the notes placeholder line (e.g. Passphrase: ________)
* `-o`, `--output <OUTPUT>` — Output file name. Use - for STDOUT.

  Default value: `out.pdf`
* `-s`, `--page-size <PAGE_SIZE>` — Paper size [default: `a4`] [possible values: `a4`, `letter`]
* `-f`, `--force` — Overwrite the output file if it already exists
* `-g`, `--grid` — Draw a grid pattern for debugging layout issues
* `--fonts-license` — Print out the license for the embedded fonts
* `-v`, `--verbose...` — More output per occurrence
* `-q`, `--quiet...` — Less output per occurrence
* `-h`, `--help` — Print help
* `-V`, `--version` — Print version

## Notes/passphrase field

The notes field below the QR code can be customised with the `--notes-label <TEXT>` and `--skip-notes-line` arguments. There's no enforced limit for the label length but eventually the text will overflow the page bounds.

### Examples

* Print a placeholder for a hint instead of the passphrase:

  ```sh
  paper-age --notes-label="Hint:"
  ```

* Print a timestamp instead of the notes field:

  ```sh
  paper-age --notes-label="Created at: $(date -Iseconds)" --skip-notes-line
  ```

## Compression

PaperAge is entirely agnostic about the input file type. If you need to squeeze in more data, you can apply compression to the input file before passing it on to PaperAge, for example:

```sh
gzip --best --stdout in.txt | paper-age --output=compressed.pdf --title="in.txt.gz"
```

Compression ratios vary wildly depending on the input data, so whether or not this is worth it is up to you.

## Scanning the QR code

On iOS, it's best to use the [Code Scanner](https://support.apple.com/en-gb/guide/iphone/iphe8bda8762/ios) from Control Center instead of the Camera app. The Code Scanner lets you copy the QR code contents to the clipboard instead of just searching for it.

On Android, the built-in camera app should let you copy the QR code contents to the clipboard. The [Google Lens](https://play.google.com/store/apps/details?id=com.google.ar.lens&hl=en) app seems to work fine too.

## Development

Run the latest from git locally, assuming you have already [installed Rust](https://www.rust-lang.org/learn/get-started):

1. Pull this repo
2. Run the tests: `cargo test`
3. Get help: `cargo run -- -h`
4. Encrypt from stdin: `echo "Hello World" | cargo run -- --title="secrets from stdin" --out="stdin.pdf"`
5. Run with maximum verbosity:  `echo "Hello World" | cargo run -- -vvvv`

### Releases

Releases are compiled and released on GitHub when new versions are tagged in git.

Use [cargo release](https://github.com/crate-ci/cargo-release) to tag and publish a new version, for example:

```sh
cargo release 1.2.3
```

⚠️ Append `--execute` to the command to actually execute the release.

## License & Credits

PaperAge is released under the MIT License. See [LICENSE.txt](LICENSE.txt) for details.

Includes the SIL Open Font Licensed [IBM Plex Mono](https://www.ibm.com/plex/) font. See [IBMPlexMono-LICENSE.txt](src/assets/fonts/IBMPlexMono-LICENSE.txt).

Uses the Rust implementation of Age from [github.com/str4d/rage](https://github.com/str4d/rage) and the [printpdf](https://github.com/fschutt/printpdf) library.

Thanks to [Ariel Salminen](https://arie.ls) for the PaperAge icon.
