# action-bump-cli

[bump](https://github.com/mattn/bump) CLI を使って、任意のファイルのセマンティックバージョンをバンプする GitHub Action です。

正規表現パターンでバージョン文字列の位置を指定するため、あらゆるファイル形式に対応できます。

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `file` | Yes | バージョン文字列を含むファイルパス |
| `pattern` | Yes | バージョンを捕捉するキャプチャグループ付き正規表現 |
| `release_type` | No* | `major`, `minor`, `patch` のいずれか |
| `version` | No* | 明示的に設定するバージョン文字列（例: `2.0.0`） |

\* `release_type` と `version` は排他的です。どちらか一方を指定してください。

## Outputs

| Output | Description |
|--------|-------------|
| `version` | バンプ後の新しいバージョン文字列 |

## Quick Start

```yaml
- uses: supermomonga/action-bump-cli@v1
  id: bump
  with:
    file: package.json
    pattern: '"version":\s*"(\d+\.\d+\.\d+)"'
    release_type: minor

- run: echo "New version: ${{ steps.bump.outputs.version }}"
```

## Pattern Examples

| File Type | Pattern |
|-----------|---------|
| package.json | `"version":\s*"(\d+\.\d+\.\d+)"` |
| Cargo.toml | `version\s*=\s*"(\d+\.\d+\.\d+)"` |
| .gemspec / version.rb | `VERSION\s*=\s*['"](\d+\.\d+\.\d+)['"]` |
| .csproj | `<Version>(\d+\.\d+\.\d+)</Version>` |
| pyproject.toml | `version\s*=\s*"(\d+\.\d+\.\d+)"` |
| VERSION (plain text) | `(\d+\.\d+\.\d+)` |

パターンには正確に **1つのキャプチャグループ** が必要です。キャプチャグループがバージョン文字列にマッチする必要があります。

## Chaining: 複数フィールドの連鎖バンプ

出力された新バージョンを後続のステップで使うことで、同じファイル内の複数のバージョンフィールドを一括更新できます。

```yaml
# <Version> をバンプして新バージョンを取得
- uses: supermomonga/action-bump-cli@v1
  id: bump
  with:
    file: MyApp.csproj
    pattern: '<Version>(\d+\.\d+\.\d+)</Version>'
    release_type: minor

# <FileVersion> にも同じバージョンを設定
- uses: supermomonga/action-bump-cli@v1
  with:
    file: MyApp.csproj
    pattern: '<FileVersion>(\d+\.\d+\.\d+)</FileVersion>'
    version: ${{ steps.bump.outputs.version }}
```

## Example Workflows

完全なワークフロー例は [`examples/`](examples/) ディレクトリにあります。

- [RubyGem](examples/rubygem.yml) - `version.rb` の `VERSION` 定数をバンプ
- [npm](examples/npm.yml) - `package.json` の `version` フィールドをバンプ
- [C# (.csproj)](examples/csproj.yml) - `Version`, `FileVersion`, `AssemblyVersion` を連鎖バンプ

## How It Works

1. 入力パラメータをバリデーション
2. `go install github.com/mattn/bump@latest` で bump CLI をインストール
3. `bump` コマンドを `-w` フラグ付きで実行（ファイルを直接書き換え、新バージョンを stdout に出力）
4. 新バージョンをアクションの出力として設定

## Requirements

- GitHub-hosted runner には Go がプリインストールされているため、追加セットアップは不要です
- Self-hosted runner を使用する場合は、Go がインストールされている必要があります

## License

MIT
