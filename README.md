# BlockAds Filter List Compiler

A CLI tool for downloading ad-blocking filter lists, extracting domains from various formats, building Trie and Bloom Filter data structures, and serializing them into highly-optimized binary files designed specifically for the BlockAds Android Go engine (mmap-friendly, custom binary format).

## Features

- **Downloads Filter Lists:** Retrieves ad-blocking lists defined via a configuration file (`config.json`) or plain URLs.
- **Parses Domains and CSS Rules:** Efficiently parses domains from formats like `hosts`, `AdBlock Plus`, and extracts cosmetic CSS hiding rules (`##selector`). 
- **Builds Optimized Data Structures:**
  - Fast **Trie** domain matching tree mechanism (subdomain matching support).
  - Compact **Bloom Filter** setup with double-hashing (FNV-1a and FNV-1) for quick lookups maintaining a 0.1% False Positive Rate.
- **Fast and Concurrent:** Support highly-concurrent multi-threaded downloading and processing.
- **Custom Binary Formats:** Outputs binary files (`.trie` and `.bloom`) adhering to BlockAds Go engine's custom memory-mapped formats for blazing-fast mobile ad blocking.

## Usage

You can run the compiler manually via Go:

```bash
# General execution relying on config.json in the current directory
go run main.go

# Uses a custom JSON configuration file
go run main.go -config lists.json

# Read pure URLs line-by-line from a text file (ignores config)
go run main.go -urls filter.txt

# Designates a custom output directory (default is "output")
go run main.go -output builds/

# Change concurrency level (default is 4 concurrent jobs)
go run main.go -concurrency 8
```

## Configuration File (`config.json`)

The configuration file is formatted in JSON consisting of an array of objects describing each filter list:

```json
[
  {
    "name": "My Custom Filter",
    "id": "my_filter",
    "url": "https://example.com/filter.txt",
    "description": "A very nice description here",
    "isEnabled": true,
    "isBuiltIn": true,
    "category": "ads"
  }
]
```

## Generated Artifacts

Inside the directory specified by the `-output` flag, the application produces the following artifacts for each filter list block:
1. `[id].trie`: Binary mmap-friendly Trie file.
2. `[id].bloom`: Binary Bloom Filter representation.
3. `[id].css`: Extracted cosmetic CSS filters logic.
4. `filter_lists.json`: A generated summary and manifest index enumerating all compiled lists along with final `.bloom`/`.trie` remote GitHub artifact paths and sizes. This manifest is used periodically by applications to consume the built filters.

## Automated Updates via GitHub Actions

This repository is equipped with an automated GitHub Actions pipeline located in `.github/workflows/update_filters.yml`. This pipeline ensures that the latest filter criteria are synced into optimized binaries regularly.

- **Schedule**: Executes automatically every 6 hours (`0 */6 * * *`).
- **Manual Trigger**: Can be executed manually via the `workflow_dispatch` trigger in the GitHub Actions tab.
- **Process**: 
  1. Checks out the repository.
  2. Sets up the Go Environment (1.22+).
  3. Builds the CLI compiler statically natively (`CGO_ENABLED=0`) into `blockads-filtering`.
  4. Runs the compiled executable to process all data described in the active `config.json` with 4 concurrent routines, dumping data into the `output/` directory.
  5. Scans for modifications. If changes are detected, a new commit bearing the message *"Auto-update filter lists (Trie & Bloom) [skip ci]"* is made dynamically and pushed back. 

This mechanism allows developers to define ad-block filter remote targets inside `config.json`, without worrying about building and storing daily updates locally. The workflow provides an autonomous CDN-like update edge for applications interacting with the BlockAds engine format.
